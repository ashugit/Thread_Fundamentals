# Concurrency Deep Dive

A self-study course for engineers who want to understand concurrency from the operating-system level up to language runtimes and backend architecture.

## Opening Note

This material is trying to trigger the question **why**.

In an AI-assisted and agentic programming world, many of us operate at a higher layer than before. Code is generated quickly, and sometimes it is not read with the same care as handwritten code. That makes systems understanding more important, not less. To design better systems and review generated code intelligently, engineers still need to recognize what the code is asking the OS, runtime, scheduler, memory model, and backend architecture to do.

This is not the best reference for **how** to implement every topic. Great books, official documentation, source code, papers, talks, and excellent YouTube lectures already exist for that. This is meant to be a quick, dense read on the **whys** that tend to stay with you: why processes exist, why threads are dangerous, why virtual memory matters, why runtimes differ, why event loops stall, why queues save and hurt systems, and why production failures often sit below the API surface.

## Point Of View And Authorship

This is practitioner-written material, not an official specification. The point of view comes from early work with Qualcomm REX-style real-time software on non-VM, single-core embedded systems; later Linux work in embedded environments; UNIX grounding through Maurice J. Bach's *The Design of the UNIX Operating System*; and many years after that building and architecting full-stack web systems.

That background is the reason for the shape of the course. It starts with the simpler discipline of shared-image embedded tasking, then uses UNIX/Linux to explain why richer systems add processes, virtual memory, file descriptors, user/kernel boundaries, runtime schedulers, and backend process architecture.

There may be gaps, oversimplifications, or implementation-specific details that need correction. If you find one, please file an issue. If you want to co-edit or contribute larger changes, reach out to the author first so the flow and intent stay coherent.

The course follows this stack:

```text
CPU and interrupts
  -> operating-system scheduler
  -> processes and virtual memory
  -> threads and synchronization
  -> language runtimes
  -> backend architecture and production debugging
```

The fuller framing is in [What This Material Is About](00-orientation.md).

## Start Here

1. Read [What This Material Is About](00-orientation.md).
2. Follow the [Modular Reading Index](index.md).
3. Use [Self Study Guide](SELF_STUDY_GUIDE.md) to pace the material.
4. Use [Exercises And Checkpoints](EXERCISES_AND_CHECKPOINTS.md) to test understanding.
5. Use [Deep Expansion Pack](15-deep-expansion-pack.md) when a topic needs more depth.
6. Use [References](REFERENCES.md) to verify implementation-specific claims and go deeper.

## What You Will Learn

By the end, you should be able to explain:

- Concurrency vs parallelism.
- Process vs thread vs coroutine vs goroutine.
- Stack, heap, executable image, ELF, and binary loading.
- How `main()` is invoked in a C program through ELF entry, dynamic loader, and C runtime startup.
- Virtual memory, page faults, copy-on-write, `fork`, and `exec`.
- File descriptors and why they matter during process launch.
- Kernel space vs user space.
- Scheduling, interrupts, priority, and context switches.
- REX-style RTOS scheduling, preemption, and watchdog/kickdog mechanics.
- Why REX-style real-time non-VM design and UNIX/Linux VM-backed design solve different problems.
- How learning a simpler REX-style shared-image task model makes UNIX process, VM, file descriptor, and kernel-management machinery easier to reason about.
- Classic UNIX ideas that still matter: process state, fd/file/inode separation, syscall trap, sleep/wakeup, scheduler queues, and buffering.
- How embedded systems instincts translate into web backend systems.
- Race conditions, mutexes, semaphores, critical sections, atomics, and deadlocks.
- C, C++, Java, Python, Ruby, JavaScript, and Go runtime concurrency models.
- Coroutines and event loops.
- How embedded/RTOS instincts translate into backend concepts such as probes, deadlines, event loops, shared caches, and restart policy.
- Backend concurrency architecture tradeoffs.
- Production debugging signals: queue age, lock wait, event-loop lag, thread-pool saturation, GC pauses, and more.

## Course Structure

| Module | Topic | Purpose |
|---|---|---|
| 00 | [Orientation](00-orientation.md) | Why this course exists and what it does not cover |
| 01 | [Concurrency Intuition](01-concurrency-intuition.md) | Build the vocabulary |
| 02 | [Process, Memory, And Executable Image](02-process-memory-and-executable-image.md) | Understand process anatomy and loading |
| 03 | [REX, UNIX, And Virtual Memory](03-rex-unix-and-virtual-memory.md) | Use a simpler non-VM RTOS model to understand why UNIX adds VM and process management |
| 04 | [Fork, Exec, COW, And FDs](04-fork-exec-copy-on-write-and-fds.md) | Understand UNIX process launch deeply |
| 05 | [Kernel Space And User Space](05-kernel-space-user-space.md) | Understand protection boundaries |
| 06 | [Scheduling, Priority, And Interrupts](06-scheduling-priority-and-interrupts.md) | Understand who runs and why |
| 07 | [Threads And Process Comparison](07-threads-and-process-comparison.md) | Understand threads as execution inside a process |
| 08 | [Races, Locks, Semaphores, And Atomics](08-races-locks-semaphores-and-atomics.md) | Understand shared-state correctness |
| 09 | [Language Runtimes](09-language-runtimes-c-cpp-java-python-ruby-js.md) | Compare runtime choices |
| 10 | [Coroutines And Go](10-coroutines-and-golang.md) | Understand lightweight runtime scheduling |
| 11 | [Embedded-To-Backend Dictionary](11-embedded-to-backend-dictionary.md) | Translate RTOS instincts into backend concurrency terms |
| 12 | [Backend Architecture](12-backend-concurrency-architecture.md) | Map concurrency models to backend systems |
| 13 | [Production Glue](13-production-glue-and-closing.md) | Memory models, backpressure, debugging, final model |
| 14 | [Appendices](14-appendices.md) | Snippets, tables, and pacing |
| 15 | [Deep Expansion Pack](15-deep-expansion-pack.md) | Extra depth for hard follow-up questions |

The course also includes:

- [Self Study Guide](SELF_STUDY_GUIDE.md)
- [Exercises And Checkpoints](EXERCISES_AND_CHECKPOINTS.md)
- [References](REFERENCES.md)

## Recommended Study Modes

### Fast Pass

Use this if you want a conceptual overview.

- Read modules 00-12.
- Skip most speaker notes.
- Do the checkpoint questions in [Exercises And Checkpoints](EXERCISES_AND_CHECKPOINTS.md).
- Estimated time: 4-6 hours.

### Deep Pass

Use this if you want to teach or mentor from the material.

- Read modules 00-14.
- Pause at every diagram and redraw it from memory.
- Run through all exercises.
- Add your own production stories after each section.
- Estimated time: multiple sessions over 1-2 weeks.

### Team Reading Group

Use this for a weekly engineering learning series.

- One or two modules per session.
- One person explains the module.
- Another person challenges the failure modes.
- End each session with one real production example from your own systems.

## Rendering Diagrams

Diagrams are written in Mermaid.

Good ways to view them:

- GitHub markdown rendering.
- VS Code with Mermaid preview support.
- Cursor with Mermaid preview support.

If your editor shows Mermaid as code, install a Markdown Mermaid extension or open the files on GitHub.

## What This Is Not

This is not a complete OS textbook, Linux kernel source tour, real-time certification guide, C++ atomics specification, JVM tuning manual, or distributed consensus course.

It is a practical systems-thinking course about concurrency: mechanisms, tradeoffs, failure modes, and architecture judgment.

## License

This material is open for anyone to use, copy, share, adapt, teach from, or build on in any way. Attribution is appreciated but not required.

Use it freely.
