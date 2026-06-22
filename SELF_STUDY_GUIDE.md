# Self Study Guide

Use this guide to turn the markdown modules into an actual learning path.

The material is dense by design. Do not try to memorize it. Your goal is to build a model you can use while debugging, designing, and reviewing systems.

## How To Study Each Module

For every module:

1. Read the section bridge.
2. Skim the headings.
3. Read slowly from top to bottom.
4. Pause at every diagram and redraw it from memory.
5. For every code snippet, answer:
   - What state is shared?
   - Who owns the state?
   - What can interleave?
   - What protects the invariant?
   - What can block?
6. Read the `Lead Into Next Section` to connect the current topic to the next one.
7. Write a three-sentence summary in your own words.

Template:

```text
Module:
Core idea:
The bug this helps me understand:
The abstraction boundary:
One thing I would now check in production:
One question I still have:
```

## Study Plan: 5 Sessions

### Session 1: Vocabulary And Process Foundations

Read:

- [00 Orientation](00-orientation.md)
- [01 Concurrency Intuition](01-concurrency-intuition.md)
- [02 Process, Memory, And Executable Image](02-process-memory-and-executable-image.md)

You should be able to answer:

- What is concurrency?
- Why is concurrency not the same as parallelism?
- What is a process?
- What state does the kernel track for a process?
- How does the Bach-style UNIX model split user execution from kernel-owned process state?
- What are stack, heap, code, data, and executable image?
- Why does `main()` not mean "first instruction executed"?

Practice:

- Draw a process memory layout from memory.
- Explain stack vs heap to a junior engineer.
- Explain what the loader does before `main()`.

### Session 2: VM, Fork, Exec, And Kernel Boundary

Read:

- [03 REX, UNIX, And Virtual Memory](03-rex-unix-and-virtual-memory.md)
- [04 Fork, Exec, Copy-On-Write, And File Descriptors](04-fork-exec-copy-on-write-and-fds.md)
- [05 Kernel Space And User Space](05-kernel-space-user-space.md)

You should be able to answer:

- Why does UNIX use virtual memory?
- Why does an RTOS-style model often avoid full UNIX-like process isolation?
- How does learning the REX-style shared-image model make UNIX process and VM machinery easier to understand?
- What does `fork()` copy immediately?
- What remains shared after `fork()`?
- Why does copy-on-write exist?
- What does `execve()` replace?
- Why do file descriptors survive exec?
- What is the difference between a file descriptor, an open file table entry, and an inode-like file object?
- How does a leaked web socket, DB connection, or log fd remain a UNIX resource leak?
- Why does user code need system calls?

Practice:

- Draw `grep error app.log > errors.txt` as `fork`, fd redirection, and `exec`.
- Explain why COW makes `fork+exec` practical.
- Explain what survives and what dies across `exec`.
- Explain the path from high-level web resource to socket/fd/kernel state.

### Session 3: Scheduling, Interrupts, Context Switching, Threads

Read:

- [06 Scheduling, Priority, And Interrupts](06-scheduling-priority-and-interrupts.md)
- [07 Threads And Process Comparison](07-threads-and-process-comparison.md)

You should be able to answer:

- What makes a task runnable?
- When can scheduling happen?
- How can an interrupt cause a context switch?
- What is saved during a context switch?
- How does a REX-style context switch differ from a UNIX process switch?
- What does watchdog/kickdog design prove?
- Why does a successful `write`, `send`, or `publish` often mean "accepted by next layer" rather than "processed by final consumer"?
- What is a thread?
- What is shared between threads?
- What is separate per thread?

Practice:

- Explain why a blocked high-priority task does not run.
- Explain priority inversion.
- Explain how a high-priority spin loop can starve an RTOS system.
- Explain process context vs thread context.

### Session 4: Shared State And Synchronization

Read:

- [08 Races, Locks, Semaphores, And Atomics](08-races-locks-semaphores-and-atomics.md)
- Selected expansions from [14 Deep Expansion Pack](14-deep-expansion-pack.md):
  - Futex
  - Condition variables
  - Deadlock/livelock/starvation
  - Data race vs race condition

You should be able to answer:

- What is a race condition?
- What is a data race?
- Why does `counter++` fail?
- What invariant does a mutex protect?
- How is a semaphore different from a mutex?
- What belongs inside a critical section?
- Why can deadlock happen?
- Why do condition variables require a `while` loop?

Practice:

- Fix a broken counter.
- Fix a broken queue.
- Find deadlock in a two-lock example.
- Design a bounded producer-consumer queue.
- Review code and write down "this lock protects X invariant."

### Session 5: Runtimes, Coroutines, Backend Architecture, Production Debugging

Read:

- [09 Language Runtimes](09-language-runtimes-c-cpp-java-python-ruby-js.md)
- [10 Coroutines And Golang](10-coroutines-and-golang.md)
- [10A Embedded-To-Cloud Concurrency Dictionary](10A-embedded-to-cloud-dictionary.md)
- [11 Backend Concurrency Architecture](11-backend-concurrency-architecture.md)
- [12 Production Glue And Closing Mental Model](12-production-glue-and-closing.md)

You should be able to answer:

- Why do C/C++ concurrency bugs differ from Java bugs?
- What does GC change about concurrency?
- Why does classic CPython's GIL matter?
- Why does CRuby's GVL matter?
- Why does JavaScript use an event loop?
- Why are coroutines good for waiting but not automatically CPU parallel?
- How are Go goroutines different from Python coroutines?
- Which backend workloads fit Node, Python, Java, C++, or Go?
- What metrics expose concurrency bottlenecks?

Practice:

- Diagnose a Node event-loop stall.
- Diagnose Python CPU-bound threads not scaling.
- Compare Java virtual threads with classic thread pools.
- Design backpressure for an API fan-out service.
- Create a production dashboard checklist.
- Translate one embedded concept you know, such as watchdog, ISR latency, driver buffers, or shared memory, into its closest web-backend equivalent.

## Study Plan: 10 Short Sessions

Use this if you prefer shorter study blocks.

| Session | Modules | Focus |
|---|---|---|
| 1 | 00-01 | Why this matters, concurrency vocabulary |
| 2 | 02 | Process, stack, heap, ELF, loader |
| 3 | 03 | REX vs UNIX, virtual memory |
| 4 | 04 | Fork, exec, COW, file descriptors |
| 5 | 05-06 | Kernel boundary, scheduling, interrupts |
| 6 | 07 | Threads vs processes |
| 7 | 08 | Races, mutexes, semaphores, deadlocks |
| 8 | 09 | Language runtime models |
| 9 | 10 | Coroutines and Go |
| 10 | 11-14 | Backend architecture, production glue, expansions |

## What To Build While Studying

Small exercises help the material stick.

Build or sketch:

- A counter with and without a mutex.
- A bounded queue using semaphore + mutex.
- A deadlock example with two locks.
- A tiny shell-like `fork+exec` diagram.
- A Node route that blocks the event loop and a fixed version.
- A Python example showing I/O threads vs CPU-bound threads.
- A Go program with goroutines and channels.
- A dashboard mock showing queue age, event-loop lag, lock wait, and DB pool wait.

## Self-Assessment Rubric

You understand the material when you can do these without notes:

| Level | Capability |
|---|---|
| Basic | Define process, thread, coroutine, mutex, semaphore, race |
| Intermediate | Explain fork/exec/COW and context switching |
| Strong | Diagnose why a concurrent service is slow or stuck |
| Senior | Choose a concurrency model based on failure modes and team ability |
| Mentor | Teach the material using examples from your own systems |

## How To Use With A Team

For each session:

1. One engineer summarizes the module.
2. One engineer challenges assumptions.
3. One engineer maps it to a production system.
4. One engineer writes down open questions.
5. End with "what would we monitor differently now?"

Suggested team rule:

> No language wars. Every runtime has a concurrency model, tradeoffs, and failure modes.
