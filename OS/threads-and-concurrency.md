# Threads & Concurrency

## Table of Contents
1. [Threads vs Processes](#threads-vs-processes)
2. [Types of Threads](#types-of-threads)
3. [Multithreading Models](#multithreading-models)
4. [Concurrency & Race Conditions](#concurrency--race-conditions)
5. [Critical Section Problem](#critical-section-problem)
6. [Synchronization Primitives](#synchronization-primitives)
7. [Classical Synchronization Problems](#classical-synchronization-problems)
8. [Key Interview Questions](#key-interview-questions)
9. [Minimal Pthreads Example](#minimal-pthreads-example)
10. [Practice Exercises](#practice-exercises)

## Threads vs Processes

| Aspect | Process | Thread |
|--------|---------|--------|
| **Definition** | Program in execution | Lightweight unit of execution within a process |
| **Memory** | Separate address space | Shared address space (heap, data, code) |
| **Stack** | One per process | One per thread |
| **Communication** | IPC (pipes, message queues, shared memory) | Direct access to shared memory |
| **Creation** | Heavy (fork + copy) | Lightweight (no copy needed) |
| **Context Switch** | Expensive (TLB flush, cache cold) | Cheap (same address space) |
| **Failure** | Doesn't affect other processes | Can crash entire process |
| **Resources** | Own set of resources | Shares process resources |
| **Scheduling** | OS schedules | OS or user-level schedules |

### Thread Components
Each thread has its own:
- Thread ID
- Program Counter
- Register set
- Stack (local variables, return addresses)

Threads share:
- Code segment
- Data segment (global variables, heap)
- Files and I/O
- Signals

### Why Multithreading?
1. **Responsiveness**: UI thread + computation thread → app stays responsive
2. **Resource sharing**: Threads share memory by default (no IPC overhead)
3. **Economy**: Creating a thread is ~30x cheaper than forking a process
4. **Scalability**: Utilize multiple CPU cores (parallelism)

## Types of Threads

### User-Level Threads (ULT)
- Thread library in user space (no kernel involvement)
- **Examples**: POSIX Pthreads (user mode), Java threads, Go goroutines, Python greenlets (not native threads)

| Pros | Cons |
|------|------|
| Fast creation/switching (no kernel trap) | One blocking call blocks ALL threads in process |
| Scheduling algorithm can be app-specific | Cannot utilize multiple CPUs (only one kernel thread) |
| Portable across OS | No true parallelism |

### Kernel-Level Threads (KLT)
- Managed by the OS kernel directly
- **Examples**: Windows threads, Linux threads (clone()), POSIX threads (kernel mode)

| Pros | Cons |
|------|------|
| True parallelism on multicore | Slower creation/switching (kernel mode switch) |
| One thread blocking doesn't block others | Higher overhead per thread |
| Kernel can schedule threads independently | Less portable |

## Multithreading Models

### Many-to-One
```
User threads:  ──●──●──●──●
                   |
Kernel thread:     ●
```
- Many user threads → one kernel thread
- **Green threads** (Java before 1.2), GNU Portable Threads
- Now mostly obsolete: no parallelism, blocking blocks all

### One-to-One
```
User threads:  ──●──●──●──●
                  |  |  |  |
Kernel threads:   ●  ●  ●  ●
```
- Each user thread → one kernel thread
- **Linux (clone), Windows**
- True parallelism, but many threads = heavy kernel overhead

### Many-to-Many
```
User threads:  ──●──●──●──●
                  | \  |
Kernel threads:   ●   ●  ●
```
- Multiplex many user threads over fewer kernel threads
- **Go goroutines (M:N)**, Java green threads with LWP, old Solaris
- Best of both: parallelism + lightweight user-level creation

## Concurrency & Race Conditions

### Key Terms
- **Concurrency**: Multiple tasks making progress (may share a core)
- **Parallelism**: Multiple tasks executing simultaneously (different cores)
- **Race condition**: Outcome depends on the relative timing of events (non-deterministic)
- **Data race**: Two threads access same memory, at least one writes, no synchronization

### Race Condition Example
```
Thread A: counter++          Thread B: counter++
  LOAD counter → regA          LOAD counter → regB
  regA = regA + 1              regB = regB + 1
  STORE regA → counter          STORE regB → counter

Expected: counter += 2
Actual (if interleaved): counter += 1  (last write wins!)
```

## Critical Section Problem

A **critical section** is code segment that accesses shared data and must be executed atomically.

### Requirements for a Solution
1. **Mutual exclusion**: At most one process in critical section at a time
2. **Progress**: If no process is in critical section, selection of next entrant must not be delayed indefinitely
3. **Bounded waiting**: There must be a limit on how many times others enter before a requesting process is allowed

### Peterson's Solution (for 2 processes)
Classic software solution — works on uniprocessors:
```
Shared: int turn; boolean flag[2] = {false, false};

// Process i wants to enter:
flag[i] = true;
turn = j;                    // give turn to other
while (flag[j] && turn == j);  // busy wait
   // CRITICAL SECTION
flag[i] = false;
// REMAINDER SECTION
```

Requires sequential consistency — may fail on modern CPUs without memory barriers.

## Synchronization Primitives

### 1. Mutex (Mutual Exclusion Lock)
```
lock(mutex)       // acquire — if locked, block (or spin)
  critical section
unlock(mutex)     // release
```
- Binary lock: locked or unlocked
- **Spinlock**: Busy-waits (spins) — good for short critical sections on multicore
- **Blocking mutex**: Deschedules thread — good for longer critical sections

### 2. Semaphore
Integer variable with two atomic operations:
```
wait(S):  while (S <= 0); S--;    // P (proberen) — decrement, block if 0
signal(S): S++;                    // V (verhogen) — increment, wake up waiters
```
- **Binary semaphore (0/1)**: Acts like a mutex
- **Counting semaphore (0..N)**: Controls access to N instances of a resource

### Mutex vs Binary Semaphore
| Feature | Mutex | Binary Semaphore |
|---------|-------|-----------------|
| **Ownership** | Owned by locking thread (only lock owner can unlock) | No ownership (any thread can signal) |
| **Purpose** | Mutual exclusion | Signaling + mutual exclusion |
| **Recursive** | Often supports recursive locking | No recursive concept |
| **Priority inheritance** | Often supported | Rarely supported |

### 3. Condition Variables
Used for waiting for a condition to become true:
```
wait(cond, mutex): atomically release mutex and block; reacquire on wake
signal(cond): wake one waiting thread
broadcast(cond): wake all waiting threads
```
Always used with a mutex — the mutex protects the condition.

### 4. Monitor
High-level synchronization construct combining:
- Shared data → declared as monitor variables
- Operations on data → monitor procedures (mutually exclusive)
- Condition variables → for blocking within monitor

Java's `synchronized` keyword + `wait()/notify()` is a monitor implementation.

### 5. Read-Write Lock
```
read_lock(): multiple readers allowed simultaneously
write_lock(): exclusive access, no readers or other writers
```
- **Reader-preference**: Writers may starve
- **Writer-preference**: Readers may starve
- Used when reads >> writes

### 6. Compare-and-Swap (CAS) — Hardware Support
Atomic instruction (no busy-wait needed):
```
CAS(&addr, expected, new):
  temp = *addr
  if temp == expected: *addr = new
  return temp
```
Foundation for lock-free data structures.

## Classical Synchronization Problems

### Producer-Consumer (Bounded Buffer)
```
Buffer of N slots
Producer: produces items, blocks if full
Consumer: consumes items, blocks if empty
```

```c
Semaphore mutex = 1;
Semaphore empty = N;    // empty slots
Semaphore full = 0;     // filled slots

Producer:                   Consumer:
  produce item                wait(full)
  wait(empty)                 wait(mutex)
  wait(mutex)                 item = buffer[out]
  buffer[in] = item           out = (out+1) % N
  in = (in+1) % N             signal(mutex)
  signal(mutex)               signal(empty)
  signal(full)                consume item
```

### Readers-Writers Problem
```
Multiple readers can read simultaneously
Only one writer can write (no readers, no other writers)
```

**Reader-priority solution**: Readers may starve writers.
```c
Semaphore rw_mutex = 1;  // protects write access
Semaphore mutex = 1;     // protects read_count
int read_count = 0;

Reader:                     Writer:
  wait(mutex)                 wait(rw_mutex)
  read_count++                ...writing...
  if read_count == 1:       signal(rw_mutex)
    wait(rw_mutex)
  signal(mutex)
  ...reading...
  wait(mutex)
  read_count--
  if read_count == 0:
    signal(rw_mutex)
  signal(mutex)
```

### Dining Philosophers
```
5 philosophers, 5 chopsticks (shared between adjacent philosophers)
Each needs 2 chopsticks to eat
```
**Problem**: Deadlock if all grab left chopstick simultaneously.

**Solutions**:
1. At most 4 philosophers at table (counting semaphore)
2. Odd philosopher picks left first; even picks right first
3. Grab both chopsticks atomically (or only if both available)

## Key Interview Questions

1. **Q: What's the difference between a mutex and a semaphore?**
   A: Mutex has ownership (only lock holder can unlock), used for mutual exclusion. Semaphore has no ownership (any thread can signal), used for signaling and controlling access to multiple resources. Mutex is binary; semaphore can be counting.

2. **Q: What is a deadlock? Name the four conditions.**
   A: A deadlock is when processes are each waiting for a resource held by another, forming a circular wait. Four necessary conditions: mutual exclusion, hold & wait, no preemption, circular wait.

3. **Q: How does a spinlock differ from a mutex?**
   A: Spinlock busy-waits (spins) on CPU until the lock is available — good for very short critical sections on multicore. Mutex puts the thread to sleep — good for longer critical sections. Spinlock wastes CPU but avoids context switch overhead.

4. **Q: Why is multithreading used?**
   A: Responsiveness (UI remains interactive), resource sharing (no IPC overhead), economy (cheaper than process creation), and scalability (true parallelism on multi-core CPUs).

5. **Q: What happens if a thread in a process crashes?**
   A: In most implementations, if a thread causes a segmentation fault or unhandled exception, the entire process terminates because threads share the same address space and the OS delivers fatal signals to the process, not individual threads.

## Minimal Pthreads Example

```c
#include <pthread.h>
#include <stdio.h>

int counter = 0;
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

void* work(void* arg) {
    for (int i = 0; i < 100000; i++) {
        pthread_mutex_lock(&lock);
        counter++;
        pthread_mutex_unlock(&lock);
    }
    return NULL;
}

int main() {
    pthread_t t1, t2;
    pthread_create(&t1, NULL, work, NULL);
    pthread_create(&t2, NULL, work, NULL);
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    printf("%d\n", counter);
    return 0;
}
```

Remove the mutex calls and run multiple times to observe the race condition.

## Practice Exercises

### Beginner
1. Explain process vs thread using memory, stack, and failure impact.
2. Identify the shared and private parts of a thread.
3. Explain the difference between race condition and data race.
4. Explain why `counter++` is not atomic.
5. Compare mutex and binary semaphore.

### Interview
1. Explain bounded waiting in the critical section problem.
2. Explain why Peterson's solution is mostly historical on modern CPUs.
3. Compare spinlock and blocking mutex.
4. Explain how condition variables avoid busy waiting.
5. Explain reader starvation and writer starvation in read-write locks.

### Hands-on
1. Implement the pthread counter example with and without a mutex.
2. Implement producer-consumer using semaphores.
