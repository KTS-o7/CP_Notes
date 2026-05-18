# Deadlocks

## Table of Contents
1. [What is a Deadlock?](#what-is-a-deadlock)
2. [Necessary Conditions](#necessary-conditions)
3. [Resource Allocation Graphs](#resource-allocation-graphs)
4. [Deadlock Prevention](#deadlock-prevention)
5. [Deadlock Avoidance](#deadlock-avoidance)
6. [Deadlock Detection & Recovery](#deadlock-detection--recovery)
7. [Key Interview Questions](#key-interview-questions)
8. [Practice Exercises](#practice-exercises)

## What is a Deadlock?

A **deadlock** is a situation where a set of processes are each waiting for a resource held by another process in the set, resulting in none of them making progress.

### Deadlock vs Starvation vs Livelock
| Situation | Definition | Processes blocked? |
|-----------|-----------|--------------------|
| **Deadlock** | Circular wait for resources, none can proceed | Yes (all blocked) |
| **Starvation** | Process waits indefinitely (others get resources first) | Yes (blocked, but others progress) |
| **Livelock** | Processes keep changing state but no actual progress | No (active, but useless) |

### Real-world Analogy
```
Car A wants to go North on 1-lane bridge
Car B wants to go South on same bridge
Both enter from opposite ends → both stop → neither can move
→ Deadlock!
```

## Necessary Conditions (Coffman Conditions)

All four must hold simultaneously for deadlock to occur:

| Condition | Definition | Example |
|-----------|-----------|---------|
| **1. Mutual Exclusion** | Resources can't be shared; only one process at a time | Printer, file write lock |
| **2. Hold and Wait** | Process holds resources while waiting for others | Holds mutex A, waiting for mutex B |
| **3. No Preemption** | OS can't forcibly take resources back; must be released voluntarily | Can't yank a file lock |
| **4. Circular Wait** | Cycle of processes each waiting for resource held by next | P1→R1→P2→R2→P1 |

### Deadlock Necessity
- All 4 conditions must hold → deadlock exists
- Break ANY ONE condition → deadlock cannot happen
- This is the basis for prevention strategies

## Resource Allocation Graphs

Directed graph representation:
- **Process node** (circle): P1, P2, ...
- **Resource node** (rectangle/square): R1, R2, ... with dots for instances
- **Request edge**: Process → Resource (P is waiting for R)
- **Assignment edge**: Resource → Process (R is held by P)

### Deadlock Detection via Graph

```
No Deadlock:                    Deadlock:
   P1 ──→ R1                    P1 ──→ R1
    │      ↓                     │      ↓
    ↓      ↓                     ↓      ↓
   P2 ←── R2                    P2 ←── R2
                                  ↑      ↑
                                  P2 ──→ R1
                                  P1 ──→ R2
```

**Detection**: If graph contains a **cycle**:
- Single instance per resource type → cycle ⇔ deadlock
- Multiple instances per resource type → cycle possible but not guaranteed; need more analysis

### System Resource Allocation Table
```
Process | Allocation | Max Need | Available
P0      | 0 1 0      | 7 5 3   | 3 3 2
P1      | 2 0 0      | 3 2 2   |
P2      | 3 0 2      | 9 0 2   |
P3      | 2 1 1      | 2 2 2   |
P4      | 0 0 2      | 4 3 3   |
```

## Deadlock Prevention

Break one of the four Coffman conditions:

### 1. Break Mutual Exclusion
- Make resources sharable where possible
- **Examples**: Read-only files (share), read-write locks (share for reads)
- **Limitation**: Not all resources can be shared (printers, write locks)

### 2. Break Hold and Wait
- **Strategy A**: Request all resources at once before execution
  - If any unavailable → release all and retry
  - Low resource utilization (holds resources before actually needing them)
- **Strategy B**: Must release all held resources before requesting new ones
  - As if you only hold one resource at a time

### 3. Break No Preemption
- OS can preempt resources from waiting processes
- **Protocol**:
  1. If process requests unavailable resource → OS preempts ALL its currently held resources
  2. Add preempted resources to available list
  3. Process must re-acquire all (old + new) resources later
- Works for resources where state can be saved/restored (CPU, memory)
- Difficult for resources like mutexes (would leave inconsistent state)

### 4. Break Circular Wait
- Impose a **total ordering** on all resource types
- Processes must request resources in increasing order
- **Example**: R1 < R2 < R3 < R4
  - Process must request R2 before R3
  - If it needs R1 but already holds R3 → must release R3 first
- Eliminates cycle: can't have both P1 waiting for higher-numbered resource from P2 and P2 waiting for higher-numbered from P1

## Deadlock Avoidance

OS needs advance information about each process's maximum resource needs.

### Safe State
A state is **safe** if there exists a sequence of process executions (safe sequence) where:
- Each process can obtain its maximum needed resources
- Process completes and releases all resources
- System never reaches deadlock

**Key insight**: If system is in safe state → no deadlock. If unsafe → deadlock possible (not guaranteed).

### Banker's Algorithm (Dijkstra)
Named because it's like a banker who won't grant loans that leave insufficient reserves.

**Data structures** (n processes, m resource types):
```
Available[m]: Available instances of each resource
Max[n][m]: Maximum demand of each process
Allocation[n][m]: Resources currently allocated
Need[n][m]: Remaining need = Max - Allocation
```

**Safety Algorithm**:
```
1. Work = Available, Finish[i] = false for all i
2. Find i such that Finish[i] == false AND Need[i] <= Work
3. If found: Work += Allocation[i], Finish[i] = true, goto 2
4. If all Finish[i] == true → system is SAFE
```

**Resource Request Algorithm** (when Pi requests Request[i]):
```
1. If Request[i] > Need[i] → ERROR (exceeded max claim)
2. If Request[i] > Available → WAIT (not enough resources)
3. Pretend to grant: Available -= Request, Allocation += Request, Need -= Request
4. Run Safety Algorithm:
   - Safe → grant request
   - Unsafe → deny, restore previous state, Pi must wait
```

### Banker's Algorithm Example
```
Available: [3, 3, 2]

Process   Allocation   Max       Need
P0        [0, 1, 0]    [7, 5, 3]  [7, 4, 3]
P1        [2, 0, 0]    [3, 2, 2]  [1, 2, 2]
P2        [3, 0, 2]    [9, 0, 2]  [6, 0, 0]
P3        [2, 1, 1]    [2, 2, 2]  [0, 1, 1]
P4        [0, 0, 2]    [4, 3, 3]  [4, 3, 1]

Safe sequence: P1 → P3 → P4 → P0 → P2  (verify each step has enough available)
```
Sequence verification:
- Work start = [3,3,2]; P1 Need [1,2,2] ≤ [3,3,2] ✓ → Work=[5,3,2]
- P3 Need [0,1,1] ≤ [5,3,2] ✓ → Work=[7,4,3]
- P4 Need [4,3,1] ≤ [7,4,3] ✓ → Work=[7,4,5]
- P0 Need [7,4,3] ≤ [7,4,5] ✓ → Work=[7,5,5]
- P2 Need [6,0,0] ≤ [7,5,5] ✓ → System is safe ✓

## Deadlock Detection & Recovery

Allow deadlocks to occur, detect them, then recover.

### Detection Algorithm
Similar to Safety Algorithm but checks if any process can finish:
```
1. Work = Available, Finish[i] = (Allocation[i] == 0)
2. Find i: Finish[i] == false AND Request[i] <= Work
3. Work += Allocation[i], Finish[i] = true
4. If any Finish[i] == false after algorithm → deadlock exists
```

### Recovery Strategies

| Strategy | Action | Trade-off |
|----------|--------|-----------|
| **Process Termination** | Kill all deadlocked processes | Simple but expensive (lose all work) |
|  | Kill one at a time until deadlock broken | Choose victim carefully |
| **Resource Preemption** | Take resource from one process, give to another | Need to rollback process state |
|  |  | May need to restart process |

### Victim Selection
- Priority of the process
- How long has it been running?
- How many resources does it hold?
- How many more does it need?
- Is it a batch or interactive process?

## Key Interview Questions

1. **Q: What are the four necessary conditions for deadlock?**
   A: (1) Mutual exclusion — resources not shareable. (2) Hold and wait — holding resources while waiting for more. (3) No preemption — resources can't be forcibly taken. (4) Circular wait — cycle in resource allocation graph. All four must hold simultaneously.

2. **Q: How does Banker's Algorithm prevent deadlock?**
   A: It only grants resource requests if the resulting state would be safe — meaning there exists a sequence where all processes can complete. It simulates granting the request and checks safety before actually granting.

3. **Q: How do you prevent deadlock in a multi-threaded program?**
   A: Enforce lock ordering (always acquire locks in same order across threads), use try_lock() with backoff, use lock-free data structures, or minimize the scope of locks (release before acquiring next).

4. **Q: What's the difference between deadlock prevention and avoidance?**
   A: Prevention breaks one of the four conditions a priori (at design time). Avoidance uses runtime information about maximum resource needs to decide dynamically whether to grant requests (e.g., Banker's algorithm).

5. **Q: Can deadlock occur with only one process?**
   A: No. Deadlock requires at least two processes by definition (circular wait among a set of processes). However, a single process could deadlock on itself if it needs to reacquire a lock it already holds (non-recursive mutex), but this is typically considered a programming error (self-deadlock) rather than a classic deadlock.

## Practice Exercises

### Beginner
1. Give one real OS/programming example for each Coffman condition.
2. Draw a resource allocation graph for two processes and two resources that deadlocks.
3. Draw a graph with a cycle that is not necessarily deadlocked because resources have multiple instances.
4. Explain the difference between deadlock prevention and deadlock avoidance.
5. Explain why killing one process can recover from deadlock.

### Banker's Algorithm
1. Given Allocation and Max matrices, calculate the Need matrix.
2. Run the safety algorithm and find one safe sequence.
3. Try granting a new request and decide if the resulting state is safe.
4. Explain why an unsafe state is not always a deadlocked state.
5. Explain why Banker's algorithm is uncommon in general-purpose operating systems.

### Hands-on
1. Write two threads that lock `A` then `B` and `B` then `A`; observe the deadlock.
2. Fix the program by enforcing a single lock ordering.
