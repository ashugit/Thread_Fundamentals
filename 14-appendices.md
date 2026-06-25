# Appendices

Previous: [Production Glue And Closing Mental Model](13-production-glue-and-closing.md) | [Index](index.md) | Next: [Deep Expansion Pack](15-deep-expansion-pack.md)

**Focus:** Keep optional snippets, decision tables, and delivery rhythm separate from the main flow.

## Bridge

**Coming from:** [Production Glue And Closing Mental Model](13-production-glue-and-closing.md).

**Read this for:** Keep optional snippets, decision tables, and delivery rhythm separate from the main flow.

**Then:** use these notes for exercises, discussion, or a second pass.

---

## Appendix A. Minimal Code Comparison

Process:

```sh
python worker.py &
python worker.py &
wait
```

Thread:

```python
from threading import Thread

threads = [Thread(target=work) for _ in range(4)]
for t in threads: t.start()
for t in threads: t.join()
```

Coroutine:

```python
import asyncio

async def main():
    await asyncio.gather(fetch("/a"), fetch("/b"), fetch("/c"))

asyncio.run(main())
```

Goroutine:

```go
go work()
go work()
```

> **Side note:** These snippets look equally small. Their runtime implications are completely different.

---

## Appendix B. Practical Architecture Decision Table

| Workload | Usually strong fit | Watch out |
|---|---|---|
| Many slow network calls | Node, Go, Java virtual threads, Python async | Backpressure, timeouts |
| CPU-heavy compute | C++, Java, Go, native Python extensions | Cache, allocation, pools |
| CRUD business app | Python/Django, Java/Spring, Node/TypeScript | DB pool, request fan-out |
| Low-latency infra | C++, Rust, tuned Java/Go | Tail latency, memory layout |
| Embedded real-time | RTOS task model, C/C++ | Interrupt latency, shared memory |
| High isolation workers | Processes | IPC overhead |
| Millions of waiting tasks | Coroutines/goroutines/virtual threads | Blocking calls |

> **Side note:** Pick the model whose operational failure you can explain at 3 AM.

---

## Appendix C. Suggested Delivery Rhythm For Multiple Hours

Recommended pacing:

- 0:00-0:25: concurrency intuition and process basics.
- 0:25-1:05: ELF, loading, VM, kernel/user boundary.
- 1:05-1:45: scheduling, interrupts, context switching, single vs multicore.
- 1:45-2:30: threads, races, locks, semaphores, atomics.
- 2:30-3:20: language runtime models.
- 3:20-4:00: coroutines, Go, backend architecture tradeoffs.
- 4:00+: Q&A and war-story debugging exercises.

Good exercises:

- Draw `fork+exec+fd redirection`.
- Debug `counter++` race.
- Explain why Node server stalls under CPU-heavy JSON.
- Explain why Python threads help I/O but not pure CPU in classic CPython.
- Compare one-process-many-threads vs many-process workers.

> **Side note:** Use pauses. The goal is not to recite slides. The goal is to upgrade the mental model.

---

## Appendix D. UNIX File Model: Descriptor, Open File Description, Inode

One of the most useful UNIX lessons is that "a file" is not one thing inside the kernel.

There are layers:

```text
process fd table
  fd 3
    |
    v
system-wide open file table entry / open file description
  file offset
  open mode/status flags
  reference count
    |
    v
inode / vnode-like object
  file identity
  permissions
  size
  block mapping / filesystem metadata
```

Why this matters:

- Two file descriptors can refer to the same open file table entry.
- Parent and child after `fork()` can share file offset.
- `dup()` creates another descriptor pointing to the same open file description.
- Opening the same pathname twice creates separate open file descriptions.
- The inode identifies the file object; the open file entry tracks an active open instance.

Example:

```c
int fd1 = open("data.txt", O_RDONLY);
int fd2 = dup(fd1);
int fd3 = open("data.txt", O_RDONLY);
```

Conceptually:

```text
fd1 -> open file entry A -> inode for data.txt
fd2 -> open file entry A -> inode for data.txt
fd3 -> open file entry B -> inode for data.txt
```

Consequence:

- Reads through `fd1` advance the offset seen by `fd2`.
- Reads through `fd3` use a separate offset.

Concurrency implication:

- File descriptors are process-local handles.
- Open file descriptions may be shared kernel objects.
- Inodes/filesystem objects are deeper shared objects.
- Correctness depends on which layer is shared.

> **Side note:** This is a classic UNIX distinction worth teaching slowly. Many engineers say "the fd points to a file." Better: fd points to an open file description, which points to the underlying file object.

---

## Lead Into Next Section

**Core takeaway to close with:** Keep optional snippets, decision tables, and delivery rhythm separate from the main flow.

**Transition to next section:** Use these pages after the main path for exercises, discussion, or a second pass through the material.

**Continue reading:** Use this section as exercises and reference material when you want a second pass through the course.

**Pause check before moving on:** pause and summarize the section in one sentence and name the resource or boundary that became clearer.

Previous: [Production Glue And Closing Mental Model](13-production-glue-and-closing.md) | [Index](index.md) | Next: [Deep Expansion Pack](15-deep-expansion-pack.md)
