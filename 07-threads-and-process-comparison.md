# Threads And Process Comparison

Previous: [Scheduling, Priority, And Interrupts](06-scheduling-priority-and-interrupts.md) | [Index](index.md) | Next: [Races, Locks, Semaphores, And Atomics](08-races-locks-semaphores-and-atomics.md)

**Section purpose:** Define threads, TCBs, process-vs-thread context, and why UNIX/RTOS models differ.

## Section Bridge

**Arriving from:** [Scheduling, Priority, And Interrupts](06-scheduling-priority-and-interrupts.md). The previous section covered: Cover scheduling policy, priority, system-call scheduling points, interrupts, and context switches.

**This section answers:** Define threads, TCBs, process-vs-thread context, and why UNIX/RTOS models differ.

**Watch for the next question:** once this section lands, the next natural question is why we need **Races, Locks, Semaphores, And Atomics** next.

> **Reading note:** Read this as one continuous block. The slide-level `Flow` notes explain local transitions; the section-level transition at the end connects this topic to the next one.

---

## 56. What Is A Thread

> **Flow:** From **Summary So Far**, move into **What Is A Thread**. This page should answer the natural follow-up and prepare for **Why Do We Need A Thread**.


A thread is a schedulable execution stream within a process.

Threads in the same process generally share:

- Virtual address space.
- Heap.
- Global variables.
- File descriptors.
- Process credentials.
- Current working directory.
- Signal dispositions.

Each thread has its own:

- Stack.
- Program counter.
- Registers.
- Thread-local storage.
- Scheduling state.
- Signal mask in many systems.

```mermaid
flowchart TB
  P["Process"]
  P --> VM["Shared virtual memory"]
  P --> FD["Shared file descriptors"]
  P --> T1["Thread 1: registers + stack"]
  P --> T2["Thread 2: registers + stack"]
  P --> T3["Thread 3: registers + stack"]
```

> **Side note:** Threads are cheaper than processes mainly because they share the expensive process resources. That same sharing is why they are dangerous.

---

## 57. Why Do We Need A Thread

> **Flow:** From **What Is A Thread**, move into **Why Do We Need A Thread**. This page should answer the natural follow-up and prepare for **What Is A Thread Control Block**.


The short answer:

```text
We need threads when one process needs more than one execution stream
while still sharing the same process resources.
```

A process gives us a protected container:

- address space
- heap
- loaded code
- file descriptors
- credentials
- current directory
- signal dispositions
- resource limits

A thread gives us another path of execution inside that same container:

- its own program counter
- its own registers
- its own stack
- its own scheduler state
- its own blocking/running lifecycle

```mermaid
flowchart TB
  subgraph P["One process: resource container"]
    VM["virtual address space"]
    HEAP["heap and globals"]
    FD["file descriptor table"]
    CODE["loaded code and libraries"]
    T1["thread A<br/>pc + registers + stack"]
    T2["thread B<br/>pc + registers + stack"]
    T3["thread C<br/>pc + registers + stack"]
  end

  T1 --> VM
  T2 --> VM
  T3 --> VM
  T1 --> FD
  T2 --> FD
  T3 --> FD
  T1 --> HEAP
  T2 --> HEAP
  T3 --> HEAP
```

That one idea is the center of the whole course:

```text
Process = protection and resource ownership
Thread  = execution inside that protection boundary
```

### Grandma Explanation

Imagine a restaurant.

The restaurant building is the **process**:

- one kitchen
- one pantry
- one cash counter
- one set of tables
- one address and license

The workers are **threads**:

- one cook prepares food
- one waiter serves customers
- one cashier bills
- one cleaner resets tables

Why not open a new restaurant for every job?

Because that would be expensive. Every restaurant would need a new kitchen, pantry, license, counter, tables, and rent.

Instead, one restaurant has many workers. They share the same kitchen and pantry, but each worker has their own current task.

The danger is also obvious:

- two workers may grab the same ingredient
- one worker may move a tray while another is using it
- one worker may block the kitchen door
- everyone may wait for everyone else

So threads are useful because they let many workers make progress in one shared place. They are dangerous because the shared place must be managed carefully.

### Young Engineer Explanation

A process is expensive because it owns heavyweight resources:

- virtual memory mappings
- page tables
- file descriptor table
- loaded executable image
- shared libraries
- heap
- credentials
- kernel bookkeeping

If every concurrent activity becomes a separate process, you get strong isolation, but sharing state becomes more expensive:

- pipes
- sockets
- shared memory
- serialization
- IPC protocols
- more kernel crossings
- more process lifecycle management

A thread is the compromise:

- keep one process container
- add multiple schedulable execution streams
- share memory directly
- share file descriptors directly
- avoid duplicating the whole process resource set
- allow blocking work without stopping every execution path
- allow CPU work to run on multiple cores

Common reasons engineers use threads:

- **Responsiveness:** a UI thread stays responsive while a worker thread loads data.
- **I/O overlap:** one thread waits on disk/network while another continues.
- **CPU parallelism:** multiple threads compute on different cores.
- **Service concurrency:** a server handles many requests using a worker pool.
- **Runtime support:** JVM/Go/.NET/Python runtimes use helper threads for GC, JIT, finalization, timers, signal handling, profiling, or I/O.
- **Blocking API integration:** if a library blocks, a thread can absorb that blocking without freezing the entire process.
- **Shared in-memory state:** caches, queues, pools, and indexes can be shared without IPC.

```mermaid
flowchart LR
  A["Need concurrent activity"] --> B{"Need strong isolation?"}
  B -->|yes| P["Use processes<br/>higher isolation, higher sharing cost"]
  B -->|no| C{"Need shared memory or lower overhead?"}
  C -->|yes| T["Use threads<br/>shared state, lower resource cost"]
  C -->|no| E["Use event loop/coroutines<br/>good for many waits"]
  T --> R["Must design ownership, locks,<br/>ordering, cancellation, backpressure"]
```

The young engineer mistake is thinking:

```text
thread = make it faster
```

The better model is:

```text
thread = add another independently schedulable execution path inside this process
```

It may make the program faster. It may make the program more responsive. It may simply allow a clean structure. It may also make the program slower if it adds contention, cache misses, context switches, lock waits, and debugging complexity.

### Seasoned Engineer Explanation

Threads exist because the process abstraction solves isolation, but many real systems need controlled sharing inside the isolation boundary.

The process boundary is excellent for:

- fault containment
- permissions
- independent lifetimes
- address-space isolation
- security boundaries
- deployment and supervision

But process isolation becomes costly when the workload needs:

- a large shared cache
- a shared heap of domain objects
- shared connection pools
- shared memory indexes
- low-latency handoff
- tight coordination between execution streams
- parallel CPU work over the same in-memory dataset

Threads give you a second axis:

```text
process boundary: what resources are shared and protected?
thread boundary: what execution stream is currently running or blocked?
```

Veteran-level design questions are not "should I use threads?"

They are:

- What invariant becomes shared?
- Which thread owns which state?
- What is the handoff protocol?
- Can blocking in one thread starve another?
- Is the contention cost lower than IPC cost?
- Will cache-line bouncing dominate?
- What is the cancellation model?
- What happens during shutdown?
- What happens after `fork()` if the process is multithreaded?
- Are locks protecting memory visibility as well as mutual exclusion?
- Is the runtime also running GC/JIT/timer/profiler/helper threads?
- What metrics will reveal lock wait, run queue pressure, pool starvation, and queue age?

```mermaid
flowchart TB
  W["Workload pressure"] --> I{"Isolation more important<br/>than sharing?"}
  I -->|yes| PROC["Multiple processes<br/>fault isolation, IPC, supervision"]
  I -->|no| S{"Shared mutable state needed?"}
  S -->|yes| TH["Threads<br/>shared heap, locks, atomics, condition variables"]
  S -->|no| ACT["Actors, queues, event loops,<br/>coroutines, worker pools"]

  TH --> COST["Costs to manage<br/>races, deadlocks, cache contention,<br/>memory ordering, shutdown"]
  PROC --> PCOST["Costs to manage<br/>IPC, serialization, fd passing,<br/>process lifecycle"]
  ACT --> ACOST["Costs to manage<br/>backpressure, fairness, cancellation,<br/>blocking hazards"]
```

Threads are also a historical bridge between the REX-style and UNIX-style mental models.

In a REX-style RTOS, a task may already be the primary schedulable unit in one shared system image. You do not first think "process container" and then "thread inside it." You think "task that runs, blocks, wakes, and shares memory carefully."

In UNIX, the process comes first as the protected container. Threads are introduced when one protected container needs multiple execution streams. That is why learning REX-style tasks helps: it makes the execution-stream idea obvious before UNIX adds VM, file tables, credentials, process hierarchy, and syscall boundaries.

### What Threads Buy

Threads buy:

- **Lower sharing cost than processes:** memory is already shared.
- **Lower creation/switch cost than full process designs in many systems:** no new address-space container is required.
- **Better responsiveness:** one blocked activity need not block all activity.
- **CPU parallelism:** multiple cores can execute threads from the same process.
- **Natural modeling of independent activities:** request workers, background flushers, compaction threads, GC helpers.
- **Compatibility with blocking code:** legacy or blocking APIs can be isolated in worker threads.

### What Threads Charge You

Threads charge you with:

- race conditions
- deadlocks
- priority inversion
- lock convoying
- false sharing
- memory-ordering bugs
- non-deterministic tests
- shutdown complexity
- cancellation complexity
- thread-pool starvation
- observability complexity

This is the senior trade:

```text
Threads reduce some boundary costs by sharing a process.
Threads increase correctness cost because shared state now needs discipline.
```

> **Side note:** Threads solve both performance and structure problems. But if the main reason is "I do not want to think about state ownership", threads will punish you.

---

## 58. What Is A Thread Control Block

> **Flow:** From **Why Do We Need A Thread**, move into **What Is A Thread Control Block**. This page should answer the natural follow-up and prepare for **Context Of Thread Vs Context Of Process**.


A Thread Control Block, or TCB, is kernel/runtime metadata for a thread.

It typically contains:

- Thread ID.
- Saved registers.
- Stack pointer.
- Program counter.
- Thread state: running, runnable, blocked.
- Priority and scheduling policy.
- CPU affinity.
- Thread-local storage pointer.
- Signal mask.
- Kernel stack pointer.
- Accounting information.
- Linkage into scheduler queues.

In user-space threading libraries, there may also be runtime TCBs:

- Coroutine/goroutine metadata.
- Stack bounds.
- Cancellation state.
- Runtime scheduler links.
- Await/future state.

TCB ownership model:

```mermaid
flowchart TB
  K["Kernel scheduler"]
  TCB["Thread Control Block<br/>state, priority, saved registers"]
  KS["Kernel stack"]
  US["User stack"]
  TLS["Thread-local storage"]
  P["Process resources<br/>VM, fd table, credentials"]

  K --> TCB
  TCB --> KS
  TCB --> US
  TCB --> TLS
  TCB --> P
```

> **Side note:** PCB and TCB are conceptual tools. Real kernels may merge or split these structures. What matters is what state exists and who owns it.

---

## 59. Context Of Thread Vs Context Of Process

> **Flow:** From **What Is A Thread Control Block**, move into **Context Of Thread Vs Context Of Process**. This page should answer the natural follow-up and prepare for **Does QComm REX Need A Thread, Why?**.


Thread context:

- Registers.
- Stack.
- Program counter.
- Thread-local storage.
- Scheduling state.

Process context:

- Address space.
- File descriptor table.
- Credentials.
- Signal dispositions.
- Resource limits.
- Process ID hierarchy.
- One or more thread contexts.

Switch between threads in same process:

- Save/restore registers and stack.
- Same address space.
- Same heap and globals.

Switch between processes:

- Save/restore registers and stack.
- Change address space.
- Different VM mappings.
- Different process resources.

Resource boundary comparison:

```mermaid
flowchart TB
  subgraph ProcA["Process A"]
    AVM["VM map A"]
    AFD["fd table A"]
    AT1["thread A1<br/>registers + stack"]
    AT2["thread A2<br/>registers + stack"]
  end

  subgraph ProcB["Process B"]
    BVM["VM map B"]
    BFD["fd table B"]
    BT1["thread B1<br/>registers + stack"]
  end

  AT1 -.same-process thread switch.-> AT2
  AT2 -.process switch.-> BT1
  AT1 --> AVM
  AT2 --> AVM
  BT1 --> BVM
  AT1 --> AFD
  BT1 --> BFD
```

> **Side note:** A process is a resource container plus execution. A thread is primarily execution context inside that container.

---

## 60. Does QComm REX Need A Thread, Why?

> **Flow:** From **Context Of Thread Vs Context Of Process**, move into **Does QComm REX Need A Thread, Why?**. This page should answer the natural follow-up and prepare for **Does UNIX Need A Thread, Why?**.


In a REX-style RTOS, the word "thread" may not be necessary if the system already has "tasks".

This is an important learning shortcut. In a non-VM task-oriented system, the schedulable unit is already visible and concrete. A task has its own stack and saved CPU context, but it may share the broader memory image with other tasks. That lets the learner understand "thread-like execution" before introducing the UNIX split between a process as a resource container and a thread as an execution stream inside that container.

REX task model can provide:

- Independent execution stacks.
- Scheduler-visible units.
- Priorities.
- Blocking/wakeup on signals/events.
- Context switching.

That is thread-like in many practical ways.

Why no UNIX-style thread distinction?

- No heavy process abstraction to subdivide.
- Tasks already share the system image/address space.
- Isolation boundary is not process-centric.
- RTOS design starts with schedulable tasks, not forked processes.

UNIX needs the distinction because a process owns a protected address space, file descriptor table, credentials, and lifecycle. Threads were added so multiple execution streams could share that protected container. REX-style tasks help the learner see the execution stream first; UNIX then adds the container around it.

> **Side note:** In UNIX, threads are "inside a process". In an RTOS without UNIX processes, a task may already be the fundamental thread-like thing.

---

## 61. Does UNIX Need A Thread, Why?

> **Flow:** From **Does QComm REX Need A Thread, Why?**, move into **Does UNIX Need A Thread, Why?**. This page should answer the natural follow-up and prepare for **When Thread In UNIX Makes Sense**.


UNIX can run concurrent work with processes alone, but threads solve different problems.

Without threads:

- Use multiple processes.
- Communicate through pipes, sockets, shared memory, files.
- Better isolation.
- More overhead for shared state.

With threads:

- Share memory naturally.
- Lower context/memory overhead than processes.
- Easier to share caches, pools, in-memory indexes.
- Can exploit multicore CPU parallelism inside one service.
- Can keep blocking calls from freezing entire application.

UNIX does not strictly need threads, but modern workloads benefit from them.

> **Side note:** Processes are safer; threads are more intimate. Use threads when shared memory is truly a benefit, not just a convenience.

---

## 62. When Thread In UNIX Makes Sense

> **Flow:** From **Does UNIX Need A Thread, Why?**, move into **When Thread In UNIX Makes Sense**. This page should answer the natural follow-up and prepare for **How Context Switch Between UNIX Thread And Process Differ**.


Threads make sense when:

- Shared in-memory state is central and high-volume.
- Work is CPU-bound and parallelizable.
- Blocking calls need isolation but process overhead is too high.
- A thread pool can bound concurrency.
- Runtime or framework expects threaded execution.
- Low-latency communication through shared memory matters.

Threads are less attractive when:

- Failure isolation matters more.
- Work units are independent.
- Shared state would require complex locking.
- Scaling across machines is the real goal.
- The language runtime prevents CPU parallelism, as classic CPython GIL does for Python bytecode.

> **Side note:** Threading is an architecture decision. "Can use threads" is not the same as "should use threads."

---

## 63. How Context Switch Between UNIX Thread And Process Differ

> **Flow:** From **When Thread In UNIX Makes Sense**, move into **How Context Switch Between UNIX Thread And Process Differ**. This page should answer the natural follow-up and prepare for **UNIX Thread Vs Process Context Switch: Deeper Details**.


Thread switch in same process:

- Switch registers.
- Switch stack.
- Switch thread-local state.
- Keep same address space.
- Keep same file descriptor table.

Process switch:

- Switch registers.
- Switch stack.
- Switch memory context/page tables.
- Switch process-level accounting/resource view.
- Possibly incur more TLB/cache disruption.

Commonality:

- Both are scheduled by kernel in native threading systems.
- Both require saving/restoring CPU context.
- Both can be preemptive.

> **Side note:** The difference is not "threads do not context switch." They do. The difference is what else must change beyond CPU execution state.

---

## 64. UNIX Thread Vs Process Context Switch: Deeper Details

> **Flow:** From **How Context Switch Between UNIX Thread And Process Differ**, move into **UNIX Thread Vs Process Context Switch: Deeper Details**. This page should answer the natural follow-up and prepare for **What Are Wins With Threads**.


Extra costs in process switch may include:

- Loading new page-table base register.
- Changing address-space identifier.
- TLB invalidation or reduced TLB reuse.
- Different memory working set.
- Different kernel resource pointers.
- More cache misses after switch.

Thread switch may still be expensive:

- Different stack means different cache lines.
- Lock handoff may bounce cache lines between cores.
- Floating-point/vector state may be large.
- Scheduler overhead still exists.
- Kernel/user transition still exists if preemptive.

Optimization details:

- Lazy FPU save/restore historically reduced cost.
- Address Space Identifiers can reduce TLB flushes.
- Per-core run queues reduce scheduler lock contention.
- CPU affinity improves cache locality but can hurt load balance.

> **Side note:** Avoid teaching "thread switch is cheap" as absolute truth. It is cheaper in some dimensions, but contention and cache behavior can dominate.

---

## 65. What Are Wins With Threads

> **Flow:** From **UNIX Thread Vs Process Context Switch: Deeper Details**, move into **What Are Wins With Threads**. This page should answer the natural follow-up and prepare for **What Is A Race Condition In Thread**.


Thread wins:

- Shared memory with no serialization by default.
- Lower overhead than processes for many workloads.
- Parallel CPU execution on multicore.
- Natural model for blocking APIs.
- Thread pools can amortize creation cost.
- Good fit for servers, databases, runtimes, media pipelines.
- Can isolate latency-sensitive work from background work within one process.

But every win has a corresponding risk:

- Shared memory creates races.
- Low overhead encourages too many threads.
- Parallelism creates lock contention.
- Blocking APIs can exhaust pools.
- Debugging interleavings is hard.

> **Side note:** Threads are a power tool. They make easy things easy and hard things extremely hard unless ownership and synchronization are designed.

---

## References For This Section

- [Linux man-pages: `pthreads(7)`](https://man7.org/linux/man-pages/man7/pthreads.7.html)
- [Oracle Java Tutorial: Concurrency](https://docs.oracle.com/javase/tutorial/essential/concurrency/)
- [Java SE 21 API: `ExecutorService`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ExecutorService.html)

Use these when checking thread vocabulary, POSIX thread behavior, and the Java thread/executor bridge.

---

## Lead Into Next Section

**Core takeaway to close with:** Define threads, TCBs, process-vs-thread context, and why UNIX/RTOS models differ.

**Transition to next section:** Threads win by sharing memory, but shared memory is the source of most concurrency bugs. Move next into races and synchronization.

**Continue reading:** Continue with [Races, Locks, Semaphores, And Atomics](08-races-locks-semaphores-and-atomics.md) to follow the next layer of the model.

**Pause check before moving on:** pause and summarize the section in one sentence and name the resource or boundary that became clearer.

Previous: [Scheduling, Priority, And Interrupts](06-scheduling-priority-and-interrupts.md) | [Index](index.md) | Next: [Races, Locks, Semaphores, And Atomics](08-races-locks-semaphores-and-atomics.md)
