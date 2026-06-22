# Appendices

Previous: [Production Glue And Closing Mental Model](12-production-glue-and-closing.md) | [Index](index.md) | Next: none

**Section purpose:** Keep optional snippets, decision tables, and delivery rhythm separate from the main flow.

## Section Bridge

**Arriving from:** [Production Glue And Closing Mental Model](12-production-glue-and-closing.md). The previous section covered: Add memory model, false sharing, backpressure, structured concurrency, debugging, and final advice.

**This section answers:** Keep optional snippets, decision tables, and delivery rhythm separate from the main flow.

**Watch for the next question:** after this section, you should be ready for exercises and discussion.

> **Reading note:** Read this as one continuous block. The slide-level `Flow` notes explain local transitions; the section-level transition at the end connects this topic to the next one.

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

## Lead Into Next Section

**Core takeaway to close with:** Keep optional snippets, decision tables, and delivery rhythm separate from the main flow.

**Transition to next section:** Use these pages after the main path for exercises, discussion, or a second pass through the material.

**Continue reading:** Use this section as exercises and reference material when you want a second pass through the course.

**Pause check before moving on:** pause and summarize the section in one sentence and name the resource or boundary that became clearer.

Previous: [Production Glue And Closing Mental Model](12-production-glue-and-closing.md) | [Index](index.md) | Next: none
