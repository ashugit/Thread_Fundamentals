# Process, Memory, And Executable Image

Previous: [Concurrency Intuition](01-concurrency-intuition.md) | [Index](index.md) | Next: [REX, UNIX, And Virtual Memory](03-rex-unix-and-virtual-memory.md)

**Section purpose:** Explain process anatomy: PCB, stack, heap, executable bytes, ELF, and loading.

## Section Bridge

**Arriving from:** [Concurrency Intuition](01-concurrency-intuition.md). The previous section covered: Build the vocabulary before introducing OS objects.

**This section answers:** Explain process anatomy: PCB, stack, heap, executable bytes, ELF, and loading.

**Listen for the next question:** once this section lands, the audience should naturally ask why we need **REX, UNIX, And Virtual Memory** next.

> **Teaching note:** Read this as one continuous block. The slide-level `Flow` notes explain local transitions; the section-level handoff at the end tells you how to move the room into the next topic.

---

## 5. What Is A Process

> **Flow:** From **What All Are Common Ways Of Concurrency**, move into **What Is A Process**. This page should answer the natural follow-up and prepare the room for **What Is Process Control Block, Taking UNIX As Case**.


A process is an executing program plus the operating-system state required to manage it.

A process normally includes:

- Virtual address space.
- Code/text segment.
- Data segment.
- Heap.
- One or more stacks.
- File descriptor table.
- Signal handling state.
- Credentials and permissions.
- Environment variables.
- Current working directory.
- Process ID and parent relationship.
- Scheduling state.
- Accounting/resource usage.

Important distinction:

- **Program:** passive bytes stored on disk.
- **Process:** active execution instance of that program.

You can run the same program ten times and get ten processes.

> **Speaker side-note:** A process is the OS boundary where isolation becomes real. If process A scribbles over its own heap, process B should not be corrupted because they do not share the same virtual address space by default.

---

## 6. What Is Process Control Block, Taking UNIX As Case

> **Flow:** From **What Is A Process**, move into **What Is Process Control Block, Taking UNIX As Case**. This page should answer the natural follow-up and prepare the room for **What Is The Stack**.


The Process Control Block, or PCB, is the kernel's record for a process.

UNIX-like systems do not always call one single structure "PCB" in user-visible terms. Internally, the information is spread across structures such as task/process descriptors, memory descriptors, file tables, signal structures, credential structures, and scheduler entities.

Conceptually, a UNIX process PCB contains:

- Process ID, parent PID, process group, session.
- Process state: running, runnable, sleeping, stopped, zombie.
- CPU register save area.
- Kernel stack pointer.
- Scheduling policy, priority, CPU affinity, runtime accounting.
- Address-space metadata, page tables, memory mappings.
- Open file descriptor table.
- Signal masks, pending signals, handlers.
- User/group credentials and capabilities.
- Resource limits.
- Exit status and wait information.

```mermaid
flowchart TB
  P["Process descriptor / PCB concept"]
  P --> S["Scheduler state"]
  P --> R["Saved registers"]
  P --> M["Memory map / VM"]
  P --> F["File descriptor table"]
  P --> G["Credentials"]
  P --> SIG["Signals"]
  P --> K["Kernel stack"]
```

> **Speaker side-note:** The PCB is not "inside the process". It is kernel-owned metadata about the process. User code cannot directly edit it; user code asks the kernel through system calls.

---

## 6A. UNIX Process Model In The Bach Mental Frame

Maurice J. Bach's treatment of UNIX is useful because it does not present the OS as magic. It presents UNIX as a set of kernel data structures and algorithms that cooperate:

- process table
- user area / per-process state
- file table
- inode table
- buffer cache
- scheduler queues
- sleep and wakeup paths
- system-call entry points

The important mental frame:

> A UNIX process is not just code running. It is a kernel-managed object connected to files, memory, credentials, signals, and scheduler state.

For concurrency, this matters because every "running program" has two faces:

```text
User face:
  code, stack, heap, libraries, variables

Kernel face:
  process slot, credentials, open files, signal state,
  memory mappings, scheduling state, wait channel
```

When a process blocks:

- user code stops executing
- kernel state records why it stopped
- scheduler chooses another runnable entity
- later, an event wakes the blocked process

This is the bridge from Bach-style UNIX to modern concurrency:

- The core idea is not old.
- The names and implementation details evolved.
- The model of kernel-owned process state, wait queues, files, memory, and scheduling still pays rent.

> **Speaker side-note:** For engineers who read Bach years ago, bring back the data-structure mindset. Do not say "Linux does it exactly this way"; say "the conceptual split remains: user execution plus kernel-owned state."

---

## 7. What Is The Stack

> **Flow:** From **What Is Process Control Block, Taking UNIX As Case**, move into **What Is The Stack**. This page should answer the natural follow-up and prepare the room for **What Is Heap**.


The stack is memory used for function calls and automatic local state.

It commonly stores:

- Return addresses.
- Function arguments that are not passed in registers.
- Local variables with automatic storage duration.
- Saved registers.
- Stack frames for nested function calls.

Typical call flow:

```c
int add(int a, int b) {
    int result = a + b;
    return result;
}

int main(void) {
    return add(2, 3);
}
```

Each active call has a frame. When a function returns, its frame is popped.

Important properties:

- Fast allocation and deallocation.
- Usually per-thread, not globally shared.
- Limited size.
- Stack overflow can crash the process or corrupt memory in low-protection systems.
- In C/C++, returning a pointer to a local stack variable is invalid.

> **Speaker side-note:** Stack is a major reason threads are not free. A process with thousands of kernel threads can reserve significant virtual memory for stacks even before doing useful work.

---

## 8. What Is Heap

> **Flow:** From **What Is The Stack**, move into **What Is Heap**. This page should answer the natural follow-up and prepare the room for **What Is Executable Code In Execution**.


The heap is memory used for dynamic allocation.

Examples:

```c
int *p = malloc(sizeof(int));
*p = 42;
free(p);
```

In C++:

```cpp
auto p = std::make_unique<int>(42);
```

Heap properties:

- Allocation lifetime is not tied to a single function call.
- Objects can outlive the function that created them.
- Shared across threads in a process unless isolated by allocator design.
- Managed manually in C, usually RAII in modern C++, garbage-collected in languages such as Java and Go.
- Can fragment.
- Requires synchronization inside general-purpose allocators.

Heap bugs:

- Memory leak.
- Use after free.
- Double free.
- Buffer overflow.
- Data race on heap object.

> **Speaker side-note:** Stack bugs are often lifecycle bugs. Heap bugs are lifecycle plus ownership plus sharing bugs. Once threads enter the picture, heap ownership must be explained explicitly.

---

## 9. What Is Executable Code In Execution

> **Flow:** From **What Is Heap**, move into **What Is Executable Code In Execution**. This page should answer the natural follow-up and prepare the room for **What Is The Executable Format, Say ELF**.


Executable code in execution means CPU instruction bytes loaded into memory and being fetched, decoded, and executed by a processor.

At runtime:

- Program file is read by the kernel loader.
- Code sections are mapped into process virtual memory.
- CPU program counter points to the next instruction.
- Instructions operate on registers and memory.
- Branches, calls, returns, interrupts, and traps change control flow.

Important nuance:

- The code bytes may be read-only and shared among processes.
- The same executable file can map the same text pages into many processes.
- Each process gets separate data/heap/stack, even if code pages are shared.

```mermaid
flowchart LR
  ELF["Executable file on disk"] --> MAP["Mapped into virtual memory"]
  MAP --> PC["Program counter points into text"]
  PC --> CPU["CPU fetch/decode/execute"]
```

> **Speaker side-note:** A program does not "run from disk". Disk supplies bytes. Execution happens from memory and CPU registers after the OS and loader prepare the process.

---

## 10. What Is The Executable Format, Say ELF

> **Flow:** From **What Is Executable Code In Execution**, move into **What Is The Executable Format, Say ELF**. This page should answer the natural follow-up and prepare the room for **ELF In Deeper Details**.


ELF means Executable and Linkable Format.

ELF is common on UNIX-like systems including Linux. It can represent:

- Executable files.
- Shared libraries.
- Relocatable object files.
- Core dumps.

An ELF file includes:

- ELF header.
- Program headers.
- Section headers.
- Code section, commonly `.text`.
- Read-only data, commonly `.rodata`.
- Initialized data, commonly `.data`.
- Uninitialized data metadata, commonly `.bss`.
- Symbol tables.
- Relocation information.
- Dynamic linking information.

Two views matter:

- **Link-time view:** sections help linkers and debuggers.
- **Run-time view:** segments tell the loader what to map.

> **Speaker side-note:** Engineers often memorize `.text`, `.data`, `.bss`, but miss the key distinction: sections are mostly for tooling; program headers are what the kernel loader cares about when creating a process image.

---

## 11. ELF In Deeper Details

> **Flow:** From **What Is The Executable Format, Say ELF**, move into **ELF In Deeper Details**. This page should answer the natural follow-up and prepare the room for **What The Binary Loading At Run Time**.


ELF has three major structural ideas:

1. **ELF header**
   - Magic bytes.
   - Architecture.
   - Endianness.
   - 32-bit or 64-bit.
   - Entry point.
   - Offsets to program header and section header tables.

2. **Program header table**
   - Runtime loader instructions.
   - `PT_LOAD`: loadable segment.
   - `PT_DYNAMIC`: dynamic linking metadata.
   - `PT_INTERP`: interpreter path, usually dynamic loader.
   - Permissions: read, write, execute.

3. **Section header table**
   - Linker/debugger organization.
   - `.text`, `.data`, `.bss`, `.symtab`, `.strtab`, `.rela.*`, `.debug_*`.

Simplified mapping:

```mermaid
flowchart TB
  ELF["ELF File"]
  ELF --> EH["ELF Header"]
  ELF --> PH["Program Headers: runtime map"]
  ELF --> SH["Section Headers: linker/debugger"]
  PH --> TXT["RX segment: .text + rodata portions"]
  PH --> DATA["RW segment: .data + .bss"]
  PH --> DYN["Dynamic linking segment"]
```

> **Speaker side-note:** Explain permissions here. A modern OS tries to map code as read-execute, data as read-write, and avoid writable-executable pages. This matters for security and for concurrency because memory sharing depends on page permissions and mapping type.

---

## 12. What The Binary Loading At Run Time

> **Flow:** From **ELF In Deeper Details**, move into **What The Binary Loading At Run Time**. This page should answer the natural follow-up and prepare the room for **Binary Loading In Run Time At Deeper Details**.


When a UNIX-like system executes a binary, roughly:

1. User calls `execve(path, argv, envp)`.
2. Kernel opens the executable.
3. Kernel validates executable format.
4. Kernel creates a new address-space image for the current process.
5. Loadable ELF segments are mapped.
6. Stack is prepared with arguments, environment, and auxiliary vector.
7. Dynamic loader is mapped if needed.
8. CPU registers are initialized.
9. Instruction pointer is set to the entry point.
10. Control returns to user mode at the new program image.

`execve` does not create a new process by itself. It replaces the current process image.

Classic UNIX program launch:

```c
pid_t pid = fork();
if (pid == 0) {
    execl("/bin/ls", "ls", "-l", NULL);
    _exit(127);
}
waitpid(pid, NULL, 0);
```

> **Speaker side-note:** `fork` creates a process; `exec` replaces what that process runs. Shells depend on this split.

---

## 13. Binary Loading In Run Time At Deeper Details

> **Flow:** From **What The Binary Loading At Run Time**, move into **Binary Loading In Run Time At Deeper Details**. This page should answer the natural follow-up and prepare the room for **Summary So Far**.


A dynamically linked ELF load involves cooperation between kernel and user-space dynamic loader.

Detailed path:

- Kernel reads ELF header.
- Kernel reads program headers.
- Kernel maps `PT_LOAD` segments into virtual address space.
- If `PT_INTERP` exists, kernel maps the interpreter, for example dynamic loader.
- Kernel builds initial user stack:
  - `argc`
  - `argv[]`
  - `envp[]`
  - auxiliary vector entries such as page size and program headers.
- Kernel jumps to dynamic loader entry point.
- Dynamic loader maps required shared libraries.
- Dynamic loader performs relocations.
- Dynamic loader resolves symbols eagerly or lazily.
- Dynamic loader calls constructors.
- Dynamic loader transfers control to program entry/startup code.
- C runtime startup eventually calls `main`.

```mermaid
sequenceDiagram
  participant U as User process
  participant K as Kernel
  participant L as Dynamic loader
  participant M as main()
  U->>K: execve()
  K->>K: validate ELF + map segments
  K->>L: enter loader in user mode
  L->>L: map libraries + relocate
  L->>M: call runtime start, then main()
```

> **Speaker side-note:** This slide is where young engineers realize `main` is not the first instruction. Runtime startup, loader, relocations, TLS setup, constructors, and libc initialization happen first.

---

## 14. Summary So Far

> **Flow:** From **Binary Loading In Run Time At Deeper Details**, move into **Summary So Far**. This page should answer the natural follow-up and prepare the room for **What Was QComm REX Operating System, Say On ARM7**.


We have built the base mental model:

- A program is a file; a process is an executing instance.
- The kernel tracks process state using PCB-like structures.
- The stack holds active function-call state.
- The heap holds dynamic objects.
- Executable code is mapped into memory and executed by CPU fetch/decode/execute.
- ELF is a structured file format with runtime segment metadata.
- Loading a binary means mapping segments, building the initial stack, and transferring control to runtime startup or the dynamic loader.

Concurrency connection:

- Processes are schedulable units.
- Memory layout controls isolation and sharing.
- The loader creates the execution environment.
- The OS must save/restore enough process state to pause and resume execution.

> **Speaker side-note:** Do this summary slowly. If they do not own process, stack, heap, executable mapping, and loader, later discussions about threads and coroutines become API trivia.

---

## Lead Into Next Section

**Core takeaway to close with:** Explain process anatomy: PCB, stack, heap, executable bytes, ELF, and loading.

**Verbal handoff:** Once the audience understands that a process is an executing program plus kernel state, move from the general UNIX process model into the contrast with embedded RTOS thinking.

**Opening line for next file:** "Now open [REX, UNIX, And Virtual Memory](03-rex-unix-and-virtual-memory.md); it answers the next pressure point in the model."

**Pause check before moving on:** ask the room to summarize the section in one sentence and name the resource or boundary that became clearer.

Previous: [Concurrency Intuition](01-concurrency-intuition.md) | [Index](index.md) | Next: [REX, UNIX, And Virtual Memory](03-rex-unix-and-virtual-memory.md)
