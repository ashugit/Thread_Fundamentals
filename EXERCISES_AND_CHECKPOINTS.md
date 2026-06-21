# Exercises And Checkpoints

Use these questions after each module. The goal is not exam-style memorization. The goal is to prove you can reason about concurrency when the API layer stops being enough.

## How To Use

For each checkpoint:

1. Answer without looking.
2. Draw the relevant diagram.
3. Explain one production failure this concept could cause.
4. Explain what metric or log would expose it.

If you cannot answer a checkpoint clearly, reread the module and write your own example.

---

## 00. Orientation

Checkpoint questions:

- Why is this material not just a thread tutorial?
- What production symptoms does concurrency explain?
- What is intentionally out of scope?
- What does "mechanism -> tradeoff -> failure mode -> architecture choice" mean?

Exercise:

Write down one bug you have seen that could be explained by scheduler behavior, runtime behavior, locking, queueing, or event-loop blocking.

---

## 01. Concurrency Intuition

Checkpoint questions:

- What is concurrency?
- What is parallelism?
- Can a single-core system be concurrent?
- Can a concurrent system fail even if nothing runs in parallel?

Exercise:

Explain concurrency using a kitchen analogy, then explain it using a backend request fan-out example.

---

## 02. Process, Memory, And Executable Image

Checkpoint questions:

- What is the difference between a program and a process?
- What does the kernel track in a process descriptor/PCB-like structure?
- What lives on the stack?
- What lives on the heap?
- What does the ELF program-header view give the loader?
- Why is `main()` not the first instruction?

Exercise:

Draw a process memory layout and annotate:

- `.text`
- `.rodata`
- `.data`
- `.bss`
- heap
- mmap region
- stack

---

## 03. REX, UNIX, And Virtual Memory

Checkpoint questions:

- Why can an RTOS-style task model be simpler than UNIX processes?
- What does UNIX gain from virtual memory?
- What does UNIX pay for virtual memory?
- Why is shared memory dangerous in a non-VM system?

Exercise:

Compare a REX-style task and a UNIX process across:

- isolation
- scheduling
- memory corruption blast radius
- debugging style
- interrupt interaction

---

## 04. Fork, Exec, Copy-On-Write, And File Descriptors

Checkpoint questions:

- What does `fork()` create?
- What does `execve()` replace?
- What is copied immediately during `fork()`?
- What is shared after `fork()`?
- Why does copy-on-write exist?
- What survives across `exec()`?
- Why do file descriptors matter during shell redirection?

Exercise:

Draw this command:

```sh
grep error app.log > errors.txt
```

Show:

- shell before fork
- child after fork
- fd redirection
- child after exec
- what happens to stdout

---

## 05. Kernel Space And User Space

Checkpoint questions:

- Why does user mode exist?
- Why does kernel mode exist?
- Why is a syscall not just a function call?
- Why must the kernel distrust user pointers?
- How can a syscall become a scheduling point?

Exercise:

Explain the path of:

```c
read(fd, buf, len);
```

Include fd validation, user-buffer validation, possible blocking, wakeup, and return.

---

## 06. Scheduling, Priority, And Interrupts

Checkpoint questions:

- What does runnable mean?
- When can scheduling happen?
- How does a timer interrupt lead to preemption?
- Why does a blocked high-priority task not run?
- What is priority inversion?
- What is the difference between an interrupt and a page fault?
- What does a watchdog/kickdog prove in an RTOS?

Exercise:

Create a timeline where:

- low-priority task holds a lock
- high-priority task waits for it
- medium-priority task runs
- priority inversion occurs

Then explain priority inheritance.

Watchdog exercise:

Design a dog task for three tasks:

- RX task
- protocol task
- logging task

For each, define what "real progress" means.

---

## 07. Threads And Process Comparison

Checkpoint questions:

- What is shared by threads in the same process?
- What is private to each thread?
- What is a TCB?
- Why does UNIX need threads if it already has processes?
- Why might REX-style systems not need a separate "thread inside process" concept?

Exercise:

Draw one process with three threads. Mark:

- shared heap
- shared fd table
- per-thread stack
- per-thread registers
- per-thread program counter

---

## 08. Races, Locks, Semaphores, And Atomics

Checkpoint questions:

- What is a race condition?
- What is a data race?
- Why does `counter++` race?
- What invariant does a mutex protect?
- How is a semaphore different from a mutex?
- What is a critical section?
- What is a deadlock?
- Why can a critical section be too large?
- What is an atomic operation?

Exercise 1: Counter race

Write the failing interleaving for two threads doing:

```c
counter = counter + 1;
```

Exercise 2: Deadlock

Given two locks `A` and `B`, show how opposite lock order deadlocks. Then fix it with global lock ordering.

Exercise 3: Queue

Design a bounded queue using:

- one mutex
- one `slots` semaphore
- one `items` semaphore

Explain what each primitive protects or counts.

---

## 09. Language Runtimes

Checkpoint questions:

- Why is C concurrency close to the OS/hardware model?
- What does C++ RAII improve?
- What does GC change about memory lifetime?
- Why does Java have a strong concurrency story?
- What does the CPython GIL limit?
- What does CRuby GVL limit?
- Why did JavaScript choose an event-loop model?

Exercise:

For C++, Java, Python, Ruby, JavaScript, and Go, write:

- how work is scheduled
- how memory is managed
- what the common concurrency failure mode is

---

## 10. Coroutines And Golang

Checkpoint questions:

- What is saved during a coroutine suspension?
- Why are coroutines good for I/O-heavy systems?
- Why are coroutines dangerous with blocking calls?
- How are Go goroutines different from Python `asyncio` coroutines?

Exercise:

Explain what happens when an async function awaits network I/O. Include:

- continuation
- event loop/runtime scheduler
- readiness/completion
- resume point

---

## 11. Backend Concurrency Architecture

Checkpoint questions:

- When is Node.js a good backend fit?
- When is Python a good backend fit?
- When is Java a good backend fit?
- When is C++ a good backend fit?
- What does "threading model as architecture" mean?

Exercise:

Pick a service you know. Decide whether Node, Python, Java, Go, or C++ fits best. Defend the answer using:

- I/O wait
- CPU work
- shared state
- latency target
- team skill
- observability

---

## 12. Production Glue

Checkpoint questions:

- What is happens-before?
- Why does false sharing hurt?
- Why is backpressure part of concurrency?
- What is structured concurrency?
- What metrics expose waiting?

Exercise:

Design a dashboard for a concurrent backend. Include:

- queue depth
- queue age
- worker utilization
- event-loop lag
- lock wait
- DB pool wait
- p95/p99 latency
- timeout count
- cancellation count

---

## 13. Appendices

Checkpoint questions:

- Which code snippet best represents process concurrency?
- Which represents thread concurrency?
- Which represents coroutine concurrency?
- Which workload fits which model?

Exercise:

Use the architecture table to classify three systems you have worked on.

---

## 14. Deep Expansion Pack

Checkpoint questions:

- What is a futex slow path?
- Why do condition variables use a `while` loop?
- What is the difference between readiness and completion I/O?
- What is a GC safepoint?
- What is NUMA?
- How can a database race happen without a memory data race?
- Why does timeout not prove the operation did not happen?

Exercise:

Pick one real incident and map it to one or more layers:

```text
application design
runtime
synchronization
OS scheduler
VM/memory
CPU/cache
database
distributed workflow
```

Then write what metric would have shortened the incident.

---

## Final Capstone

Explain this system:

```text
HTTP request enters service
service does auth
service fans out to 3 dependencies
service writes DB row
service publishes event
service returns response
background worker consumes event
```

Answer:

- Where can concurrency happen?
- What state is shared?
- What needs isolation?
- What needs a transaction?
- What needs idempotency?
- What needs backpressure?
- What metrics would you add?
- Which runtime model would you choose and why?

