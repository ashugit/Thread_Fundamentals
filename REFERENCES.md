# References

This course is written as practitioner teaching material, not as a formal specification. These references anchor the implementation-specific parts so readers can verify the claims and go deeper without interrupting the main flow.

Prefer primary or near-primary sources when checking details:

- standards and ABI documents
- official language/runtime documentation
- Linux man-pages for UNIX/Linux user-space interfaces
- project documentation from the runtime maintainers

## UNIX, Linux, Processes, And File Descriptors

- [Linux man-pages: `fork(2)`](https://man7.org/linux/man-pages/man2/fork.2.html)
  - Use for `fork()` semantics, parent/child differences, inherited file descriptors, multithreaded `fork()` warnings, and Linux copy-on-write notes.
- [Linux man-pages: `execve(2)`](https://man7.org/linux/man-pages/man2/execve.2.html)
  - Use for `execve()` semantics: executing a new program image inside the current process.
- [Linux man-pages: `open(2)`](https://man7.org/linux/man-pages/man2/open.2.html)
  - Use for the distinction between file descriptors and open file descriptions.
- [Linux man-pages: `dup(2)`](https://man7.org/linux/man-pages/man2/dup.2.html)
  - Use for descriptor duplication and shared open file descriptions.
- [Linux man-pages: `mmap(2)`](https://man7.org/linux/man-pages/man2/mmap.2.html)
  - Use for memory mappings, file-backed mappings, private mappings, and shared mappings.
- [Linux man-pages: `proc(5)`](https://man7.org/linux/man-pages/man5/proc.5.html)
  - Use when discussing Linux process visibility through `/proc`.

## ELF, Loading, And Program Startup

- [System V ABI: ELF contents](https://refspecs.linuxfoundation.org/elf/gabi4+/contents.html)
  - Use as the ABI-level reference for ELF object files, program headers, program loading, and dynamic linking.
- [Linux man-pages: `elf(5)`](https://man7.org/linux/man-pages/man5/elf.5.html)
  - Use for Linux-oriented ELF structure details, including ELF header, entry point, program headers, loadable segments, and interpreter segment.
- [glibc manual: Program Basics](https://www.gnu.org/software/libc/manual/html_node/Program-Basics.html)
  - Use for C program entry concepts such as `main`, arguments, environment, and process startup from the C library point of view.

## Scheduling, Threads, Locks, And Synchronization

- [Linux man-pages: `sched(7)`](https://man7.org/linux/man-pages/man7/sched.7.html)
  - Use for Linux scheduling policies, priorities, and scheduling API context.
- [Linux man-pages: `pthreads(7)`](https://man7.org/linux/man-pages/man7/pthreads.7.html)
  - Use for POSIX threads overview and Linux threading behavior.
- [POSIX Base Specifications: Threads](https://pubs.opengroup.org/onlinepubs/9799919799/basedefs/V1_chap04.html)
  - Use for POSIX-level concepts, especially memory synchronization and general execution environment definitions.
- [Linux man-pages: `futex(2)`](https://man7.org/linux/man-pages/man2/futex.2.html)
  - Use for Linux futex behavior when explaining how many user-space locks avoid kernel entry in the uncontended path.

## Java And The JVM

- [Oracle Java Tutorial: Concurrency](https://docs.oracle.com/javase/tutorial/essential/concurrency/)
  - Use for the classic Java learning path: processes, threads, synchronization, liveness, executors, and high-level concurrency objects.
- [Java SE 21 API: `ExecutorService`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ExecutorService.html)
  - Use for executors as task-submission and lifecycle-management abstractions over threads.
- [Java SE 21 API: `CompletableFuture`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CompletableFuture.html)
  - Use for future/completion-stage style asynchronous composition.
- [Oracle Java 21 docs: Virtual Threads](https://docs.oracle.com/en/java/javase/21/core/virtual-threads.html)
  - Use for Java virtual threads and the distinction between platform threads and virtual threads.

## Python

- [Python docs: `threading`](https://docs.python.org/3/library/threading.html)
  - Use for Python thread behavior, GIL caveats, and thread synchronization primitives.
- [Python docs: Global Interpreter Lock glossary entry](https://docs.python.org/3/glossary.html#term-global-interpreter-lock)
  - Use for the official CPython GIL definition.
- [Python docs: `asyncio`](https://docs.python.org/3/library/asyncio.html)
  - Use for async/await, event-loop APIs, coroutine execution, and I/O-bound async use cases.
- [PEP 703: Making the Global Interpreter Lock Optional in CPython](https://peps.python.org/pep-0703/)
  - Use when discussing newer free-threaded CPython work and why the GIL story is changing carefully.

## JavaScript, Node.js, And Event Loops

- [Node.js guide: The Event Loop](https://nodejs.org/learn/asynchronous-work/event-loop-timers-and-nexttick)
  - Use for Node's event-loop phases and callback execution model.
- [Node.js guide: Don't Block the Event Loop or Worker Pool](https://nodejs.org/learn/asynchronous-work/dont-block-the-event-loop)
  - Use for the operational warning that Node has both an event loop and worker pool, and that blocking either harms throughput.
- [libuv design overview](https://docs.libuv.org/en/v1.x/design.html)
  - Use for libuv's event-loop and I/O abstraction model under Node.js.
- [MDN: Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)
  - Use for JavaScript promise semantics as a completion model.
- [MDN: `async function`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
  - Use for JavaScript `async`/`await` as syntax over promise-based asynchronous control flow.

## Kubernetes And Cloud Liveness

- [Kubernetes docs: Liveness, Readiness, and Startup Probes](https://kubernetes.io/docs/concepts/workloads/pods/probes/)
  - Use for Kubernetes probe vocabulary and the distinction between liveness, readiness, and startup checks.

## Go

- [Effective Go: Goroutines](https://go.dev/doc/effective_go#goroutines)
  - Use for Go's public explanation of goroutines as lightweight concurrent execution.
- [Go `context` package](https://pkg.go.dev/context)
  - Use for deadline, cancellation, and request-scoped values across API boundaries.
- [Go blog: Go Concurrency Patterns: Pipelines and cancellation](https://go.dev/blog/pipelines)
  - Use for pipeline, cancellation, and goroutine lifecycle discussion.
- [Go memory model](https://go.dev/ref/mem)
  - Use for happens-before and synchronization semantics in Go.

## C And C++

- [C++ reference: threads library](https://en.cppreference.com/w/cpp/thread)
  - Use for `std::thread`, mutexes, condition variables, atomics, futures, and related standard library facilities.
- [C++ reference: coroutine support](https://en.cppreference.com/w/cpp/language/coroutines)
  - Use for C++20 coroutine language mechanics.
- [C++ reference: memory model](https://en.cppreference.com/w/cpp/language/memory_model)
  - Use for data races and memory model language.

## Ruby

- [Ruby docs: Thread](https://docs.ruby-lang.org/en/master/Thread.html)
  - Use for Ruby thread API behavior.
- [Ruby docs: Fiber](https://docs.ruby-lang.org/en/master/Fiber.html)
  - Use for fibers and cooperative execution.
- [Ruby docs: Ractor](https://docs.ruby-lang.org/en/master/ractor_md.html)
  - Use for Ruby's actor-like parallelism model and shareability restrictions.

## Historical And Book References

- Maurice J. Bach, *The Design of the UNIX Operating System*
  - Use for the classic UNIX mental model: process states, scheduler queues, file table/inode distinction, buffer cache, syscall path, and sleep/wakeup reasoning.
- Michael Kerrisk, *The Linux Programming Interface*
  - Use as a modern Linux/UNIX systems-programming companion to the man-pages.

## Citation Style For This Repository

Keep citations lightweight in the main chapters:

```markdown
References for this section:

- [Linux man-pages: `fork(2)`](https://man7.org/linux/man-pages/man2/fork.2.html)
- [Linux man-pages: `execve(2)`](https://man7.org/linux/man-pages/man2/execve.2.html)
```

Do not cite every sentence. Cite where the material makes a platform-specific or implementation-specific claim.
