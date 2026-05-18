# Operating Systems: Introduction & Processes

## Table of Contents
1. [What is an Operating System?](#what-is-an-operating-system)
2. [Types of Operating Systems](#types-of-operating-systems)
3. [Process Concept](#process-concept)
4. [Process States](#process-states)
5. [Process Control Block (PCB)](#process-control-block)
6. [Context Switching](#context-switching)
7. [System Calls](#system-calls)
8. [Inter-Process Communication (IPC)](#inter-process-communication-ipc)
9. [User Mode vs Kernel Mode](#user-mode-vs-kernel-mode)
10. [OS Services & Structure](#os-services--structure)
11. [Key Interview Questions](#key-interview-questions)
12. [Practice Exercises](#practice-exercises)

## What is an Operating System?

An **Operating System** is system software that manages computer hardware and software resources, providing common services for application programs.

### OS as Resource Manager
- **Processor**: Schedules which process runs when
- **Memory**: Allocates/deallocates RAM to processes
- **I/O Devices**: Manages disks, keyboards, displays, network
- **File System**: Organizes data storage and retrieval
- **Security**: Protects resources from unauthorized access

### OS as Extended Machine
- Provides abstraction over bare hardware
- Hides complexity (disk blocks → files, raw memory → virtual address spaces)
- Standard API for all applications

### OS Goals
1. **Convenience**: Make computer easy to use
2. **Efficiency**: Manage resources optimally
3. **Ability to evolve**: Accommodate new hardware and software

## Types of Operating Systems

| Type | Characteristics | Examples |
|------|----------------|----------|
| **Batch OS** | Jobs submitted in batches, no user interaction, turn-around time high | IBM OS/360 (historical) |
| **Multiprogramming** | Multiple jobs in memory, CPU switched when job waits for I/O | Early mainframes |
| **Time-Sharing (Multitasking)** | CPU rapidly switches between tasks, interactive | Unix, Linux, Windows |
| **Real-Time OS (RTOS)** | Guaranteed response within strict time constraints | VxWorks, QNX, FreeRTOS |
| **Distributed OS** | Multiple computers appear as one system, resource sharing | Amoeba, Plan 9 |
| **Embedded OS** | Lightweight, fixed function, limited resources | Embedded Linux, RTOS |
| **Mobile OS** | Touch-optimized, power management, app sandboxing | Android, iOS |
| **Network OS** | File/printer sharing over network | Windows Server, NetWare |

### Batch vs Time-Sharing vs Real-Time

| Feature | Batch | Time-Sharing | Real-Time |
|---------|-------|-------------|-----------|
| **User interaction** | None | High (interactive) | Varies |
| **Response time** | Hours | Seconds | Micro/milliseconds |
| **CPU utilization** | Low | High | Must be predictable |
| **Scheduling** | FCFS | Round Robin / Priority | Priority-based preemptive |
| **Use case** | Payroll, billing | Desktop, servers | Medical, avionics, automotive |

## Process Concept

### Process vs Program
| Aspect | Program | Process |
|--------|---------|---------|
| **Nature** | Passive entity (executable file on disk) | Active entity (program in execution) |
| **Lifetime** | Permanent (stored on disk) | Temporary (created → runs → terminated) |
| **Resources** | None | CPU, memory, I/O, files |
| **Uniqueness** | One program can have many processes | Each process has unique PID |

### Process in Memory Layout
```
+-----------------------+ High address
|       Stack            |
|   (function calls,     |
|    local variables)    |
|         ↓              |
|                        |
|         ↑              |
|        Heap            |
|   (dynamic memory:     |
|    malloc/new/objects) |
|                        |
+------------------------+
|   Uninitialized Data   |  (BSS segment)
+------------------------+
|   Initialized Data     |  (Data segment — global/static vars)
+------------------------+
|   Text (Code)          |  (machine instructions, read-only)
+-----------------------+ Low address
```

### Process Attributes
- **PID**: Unique Process ID
- **PPID**: Parent Process ID
- **Program Counter**: Address of next instruction
- **CPU Registers**: Current values while executing
- **Memory Management Info**: Page table, base/limit registers
- **I/O Status**: List of I/O devices, open files
- **Accounting Info**: CPU time used, limits

## Process States

### Five-State Model
```
                    admitted
         NEW ─────────────────→ READY
                                  │
                    dispatched    │    interrupt
         RUNNING ←───────────────┼──────────────┐
           │                     │              │
   exit    │   I/O or event wait │              │
           ↓                     ↓              │
     TERMINATED              WAITING ──────────┘
                               I/O or event complete
```

| State | Description |
|-------|-------------|
| **NEW** | Process being created |
| **READY** | Process ready to run, waiting for CPU |
| **RUNNING** | Instructions being executed |
| **WAITING (Blocked)** | Waiting for I/O or event to complete |
| **TERMINATED** | Process finished execution |

### Seven-State Model (adds Suspend states)
Adds **SUSPENDED READY** and **SUSPENDED WAITING** when memory is swapped to disk.

### State Transitions (summary)
- NEW → READY: admitted by long-term scheduler
- READY → RUNNING: dispatched by short-term scheduler
- RUNNING → READY: interrupted (time slice expired)
- RUNNING → WAITING: I/O request or wait for event
- WAITING → READY: I/O or event completed
- RUNNING → TERMINATED: exit, killed, or error

## Process Control Block (PCB)

The **PCB** is a data structure maintained by the OS for each process.
Contains everything the OS needs to manage a process:

```
+-----------------------------------+
|        Process ID (PID)            |
+-----------------------------------+
|        Program Counter             |
+-----------------------------------+
|        CPU Registers               |
+-----------------------------------+
|        CPU Scheduling Info         |
|  (priority, queue pointers)        |
+-----------------------------------+
|        Memory Management Info      |
|  (base/limit regs, page tables)    |
+-----------------------------------+
|        Accounting Info             |
|  (CPU time, time limits)           |
+-----------------------------------+
|        I/O Status Info             |
|  (open files, devices allocated)   |
+-----------------------------------+
```

## Context Switching

When the CPU switches from one process to another:

1. Save current process state (PCB of running process)
2. Load saved state of next process (PCB of next process)
3. Resume execution

### Context Switch Cost
- **Direct cost**: CPU cycles to save/load registers, update structures
- **Indirect cost**: Cache misses (L1/L2/L3/TLB) — cache is cold for new process
- **Typical**: often microseconds on modern systems, but the exact cost depends on CPU architecture, OS, workload, cache/TLB state, and whether the switch crosses address spaces.

### Frequency
- Occurs on: time slice expiry, I/O request, interrupt, system call, preemption
- Too frequent: overhead dominates; too infrequent: poor responsiveness

## System Calls

A **system call** is the programmatic way for a user program to request a service from the OS kernel.

### Categories of System Calls
| Category | Examples |
|----------|----------|
| **Process Control** | fork(), exec(), exit(), wait(), kill() |
| **File Management** | open(), read(), write(), close(), lseek() |
| **Device Management** | ioctl(), read(), write() |
| **Information** | getpid(), gettimeofday(), sysinfo() |
| **Communication** | pipe(), shmget(), socket(), connect(), send() |
| **Protection** | chmod(), setuid(), chown() |

## Inter-Process Communication (IPC)

Processes have separate address spaces, so they need OS-supported mechanisms to communicate.

| IPC Mechanism | How it works | Good for |
|---------------|--------------|----------|
| **Pipe** | Byte stream between related processes | Shell pipelines, parent-child communication |
| **Named Pipe (FIFO)** | Pipe with a filesystem name | Unrelated local processes |
| **Message Queue** | Kernel-managed queue of messages | Structured asynchronous messages |
| **Shared Memory** | Multiple processes map the same memory region | Fast large-data exchange |
| **Semaphore** | Counter used for synchronization | Coordinating access to shared resources |
| **Signal** | Lightweight notification to a process | Interrupting, terminating, reloading config |
| **Socket** | Endpoint for local or network communication | Client-server systems |

Shared memory is fast because data is not copied through the kernel after setup, but it needs synchronization such as semaphores or mutexes to avoid races.

### System Call Flow
```
User Program → system call (e.g., read())
  ↓
C library wrapper → software interrupt / syscall instruction
  ↓ (trap to kernel mode)
Kernel sys_read() handler → verify parameters
  ↓
Perform operation (disk read, etc.)
  ↓ (return to user mode)
Result returned to user program
```

## User Mode vs Kernel Mode

Modern CPUs have at least two privilege levels:

| Feature | User Mode | Kernel Mode |
|---------|-----------|-------------|
| **Access** | Limited (own address space only) | Full (all memory, hardware) |
| **Instructions** | Non-privileged only | All instructions (including privileged) |
| **Crash impact** | Only crashes current process | Crashes entire system (kernel panic) |
| **Entry** | Boot time (login) | Trap, interrupt, system call |
| **Purpose** | Run applications safely | Run OS code |

### Dual-Mode Operation
- **Mode/privilege state** in CPU control registers: exact representation is architecture-specific. Conceptually, one state allows privileged kernel execution and another restricts user programs.
- Switched to kernel mode on: interrupt, trap, system call
- Switched to user mode on: return from kernel

## OS Services & Structure

### Key OS Services
- **User Interface**: CLI (shell), GUI (desktop), batch
- **Program Execution**: Load program into memory, run, end
- **I/O Operations**: Abstract hardware I/O
- **File System Manipulation**: Create, delete, read, write, permission
- **Communications**: IPC, networking
- **Error Detection**: Handle CPU, memory, I/O, user program errors
- **Resource Allocation**: CPU scheduling, memory, file locks
- **Protection & Security**: Access control, authentication

### OS Structure Approaches

| Structure | Description | Examples |
|-----------|-------------|----------|
| **Monolithic** | All services in one kernel space (fast, but big & fragile) | Linux, Unix, Windows (hybrid) |
| **Layered** | OS split into layers, each uses services of layer below (modular, slower) | THE, MULTICS |
| **Microkernel** | Minimal kernel, services as user-space processes (stable, slower IPC) | MINIX, QNX, Mach (macOS) |
| **Hybrid** | Monolithic kernel + loadable modules | Linux, macOS, Windows |

```
Monolithic:              Microkernel:
+--------------------+   +------------------------+
| Applications        |   | Applications | Servers |
+--------------------+   +-------------+---------+
|      Kernel         |   |   Microkernel (IPC)  |
| (FS, Network,       |   +----------------------+
|  Drivers, Sched,    |   |      Hardware        |
|  Memory Mgmt)       |   +----------------------+
+--------------------+
```

## Key Interview Questions

1. **Q: What's the difference between a process and a thread?**
   A: Process is a program in execution with its own memory space, resources, and PCB. Thread is a lightweight unit of execution within a process — shares memory and resources with sibling threads but has its own stack, PC, and registers.

2. **Q: What happens during a context switch?**
   A: The OS saves the current process's state (PCB), loads the next process's state, and resumes execution. This involves saving/restoring registers, program counter, memory management info, and causes cache/TLB flushes.

3. **Q: Why do we need user mode and kernel mode?**
   A: To protect the OS from user programs. A buggy or malicious user program can't crash the system or access other processes' memory because it runs in restricted user mode. Only trusted kernel code runs in privileged mode.

4. **Q: How does a system call work?**
   A: User program calls a library function → library sets up arguments in registers → executes trap/syscall instruction → CPU switches to kernel mode → kernel handler executes → result returned → CPU switches back to user mode.

5. **Q: Monolithic vs Microkernel — tradeoffs?**
   A: Monolithic is faster (function calls vs IPC overhead) but a bug in any module can crash the kernel. Microkernel is more stable (services isolated in user space) but IPC overhead makes it slower. Linux is monolithic (with modules), macOS/iOS use hybrid (Mach microkernel + BSD services).

## Practice Exercises

### Beginner
1. List three differences between a program and a process.
2. Draw the five process states and label every transition.
3. Explain why a context switch is not free.
4. Match each system call category with one example call.
5. Choose an IPC mechanism for shell pipelines, shared cache data, and network communication.

### Hands-on
1. Write a small program that calls `fork()` and prints parent and child PIDs.
2. Run `ps` or `top`, pick a process, and identify its PID, state, and CPU usage.
