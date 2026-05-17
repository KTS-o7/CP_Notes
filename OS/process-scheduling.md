# Process Scheduling

## Table of Contents
1. [Scheduling Queues](#scheduling-queues)
2. [Types of Schedulers](#types-of-schedulers)
3. [CPU Scheduling Criteria](#cpu-scheduling-criteria)
4. [Scheduling Algorithms](#scheduling-algorithms)
5. [Multilevel Queue & Feedback](#multilevel-queue--feedback)
6. [Multiple Processor Scheduling](#multiple-processor-scheduling)
7. [Real-Time Scheduling](#real-time-scheduling)
8. [Key Interview Questions](#key-interview-questions)

## Scheduling Queues

The OS maintains several queues of PCBs:

```
Job Queue: All processes in the system
  ↓ (Long-term scheduler admits to memory)
Ready Queue: Processes in main memory, ready to run
  ↓ (Short-term / CPU scheduler dispatches)
CPU
  ↓ (I/O request or event wait)
I/O Queue (Device Queues): Waiting for specific I/O device
  ↓ (I/O completion)
Back to Ready Queue
```

| Queue | Contains | Purpose |
|-------|----------|---------|
| **Job Queue** | All processes | Master list of all processes |
| **Ready Queue** | Processes ready to execute | CPU scheduler picks next process |
| **Device Queue** | Processes waiting for I/O | Per-device list of waiting processes |

## Types of Schedulers

| Scheduler | Frequency | Speed needed | Function |
|-----------|-----------|-------------|----------|
| **Long-Term (Job)** | Minutes/hours | Slow | Admit new processes to memory; control degree of multiprogramming |
| **Medium-Term** | Seconds | Medium | Swap processes in/out of memory (suspended states) |
| **Short-Term (CPU)** | Milliseconds | Very fast | Dispatch next process to CPU |

### Degree of Multiprogramming
- Number of processes in memory ready to execute
- Long-term scheduler controls this: too few = idle CPU, too many = thrashing

### CPU-Bound vs I/O-Bound
| Type | CPU bursts | I/O bursts | Priority preference |
|------|-----------|-----------|-------------------|
| **CPU-bound** | Long, few | Short | Lower priority |
| **I/O-bound** | Short, many | Frequent | Higher priority (for responsiveness) |

## CPU Scheduling Criteria

| Criterion | Goal | Description |
|-----------|------|-------------|
| **CPU Utilization** | Maximize | Keep CPU as busy as possible (40%-90% realistic) |
| **Throughput** | Maximize | Processes completed per time unit |
| **Turnaround Time** | Minimize | Interval from submission to completion |
| **Waiting Time** | Minimize | Time spent in ready queue (not running, not blocked) |
| **Response Time** | Minimize | Time from submission to first response (interactive) |

### Important Distinction
- **Arrival Time (AT)**: When process enters ready queue
- **Burst Time (BT)**: CPU time needed (estimated or known)
- **Completion Time (CT)**: When process finishes
- **Turnaround Time (TAT)** = CT - AT
- **Waiting Time (WT)** = TAT - BT
- **Response Time (RT)** = First time scheduled - AT

## Scheduling Algorithms

### 1. FCFS (First-Come, First-Served)
- Simplest — processes run in arrival order
- **Non-preemptive**: Once CPU assigned, process keeps it until completion/blocking
- **Convoy effect**: Short process behind long process

Example with AT and BT:
```
Process   AT   BT
P1         0   24
P2         1    3
P3         2    3

Gantt: |   P1 (0-24)  | P2 (24-27) | P3 (27-30) |
WT: P1=0, P2=23, P3=25 → Avg WT = 16
```

### 2. SJF (Shortest Job First)
- Run process with shortest next CPU burst
- Can be **preemptive** (SRTF — Shortest Remaining Time First) or **non-preemptive**
- Provably optimal for minimizing average waiting time
- **Problem**: Need to know/estimate future CPU burst length
- **Starvation**: Long processes may never run

Burst prediction: **Exponential averaging**
```
τ_n+1 = α * t_n + (1 - α) * τ_n
where: τ_n = predicted burst, t_n = actual burst, α (typically 0.5)
```

Example (non-preemptive SJF):
```
Process   AT   BT
P1         0    8
P2         1    4
P3         2    1

Gantt: | P1 (0-1) | P3 (1-2) | P2 (2-6) | P1 (6-14) |   Wait! At t=1, P2(4) and P3(1) arrive
Actually: | P1 (0-8) | P3 (8-9) | P2 (9-13) |       Non-preemptive: P1 runs to completion
WT: P1=0, P3=6, P2=7 → Avg WT = 4.33
```

### 3. SRTF (Shortest Remaining Time First) — Preemptive SJF
```
Process   AT   BT
P1         0    8
P2         1    4
P3         2    1

Gantt: | P1 (0-1) | P2 (1-2) | P3 (2-3) | P2 (3-6) | P1 (6-14) |
  t=1: P1 remaining=7, P2=4 → P2 runs
  t=2: P1=7, P2=3, P3=1 → P3 runs
  t=3: P1=7, P2=3 → P2 runs
  t=6: P1=7 → P1 runs to completion
WT: P1=6, P2=1, P3=0 → Avg WT = 2.33  (much better!)
```

### 4. Priority Scheduling
- Each process assigned a priority (lower number = higher priority often)
- Can be preemptive or non-preemptive
- **Starvation**: Low-priority processes wait forever
- **Solution**: **Aging** — gradually increase priority of waiting processes

### 5. Round Robin (RR)
- Preemptive, designed for time-sharing systems
- Each process gets a fixed **time quantum** (q), typically 10-100ms
- If process doesn't finish in q, it goes to end of ready queue
- **Choice of q**: Too small = excessive context switches; too large = degrades to FCFS
- Rule of thumb: q should be larger than 80% of CPU bursts

```
Process   BT   (q=4)
P1        24
P2         3
P3         3

Gantt: | P1(0-4) | P2(4-7) | P3(7-10) | P1(10-14) | P1(14-18) |
        | P1(18-22) | P1(22-26) | P1(26-30) |
WT: P1=6, P2=4, P3=7 → Avg WT = 5.67
```

### Algorithm Comparison

| Algorithm | Preemptive? | Avg WT | Starvation | Implementation |
|-----------|-----------|--------|-----------|----------------|
| **FCFS** | No | High | No | Simple queue |
| **SJF** | No | Low | Yes (short jobs starve long ones) | Sort by burst |
| **SRTF** | Yes | Lowest | Yes | Compare remaining times |
| **Priority** | Optional | Varies | Yes | Priority queue |
| **RR** | Yes | Medium-high | No (equal share) | Circular queue + timer |

## Multilevel Queue & Feedback

### Multilevel Queue
- Ready queue partitioned into separate queues:
  - **Foreground (interactive)**: Higher priority, RR scheduling
  - **Background (batch)**: Lower priority, FCFS scheduling
- Fixed assignment of process to a queue
- Scheduling between queues: fixed priority preemptive OR time slice between queues

### Multilevel Feedback Queue
- Processes can move between queues
- Different scheduling algorithms per queue
- Parameters: number of queues, algorithm per queue, promotion/demotion rules

```
Priority ↑    Queue 0 (high priority): RR, q=8
        │     │ Demote: uses full quantum → go to Queue 1
        │    Queue 1: RR, q=16
        │     │ Demote: uses full quantum → go to Queue 2
        ↓    Queue 2 (low priority): FCFS
```

- **Aging**: Promote process if it waits too long
- Used in: Linux (CFS), Windows, macOS

## Multiple Processor Scheduling

### Approaches
| Approach | Description |
|----------|-------------|
| **Asymmetric Multiprocessing** | One processor (master) handles scheduling; others run OS code |
| **Symmetric Multiprocessing (SMP)** | Each processor self-schedules from a common/private ready queue |

### Issues
- **Processor affinity**: Keep process on same CPU (cache reuse)
  - **Soft affinity**: Best-effort
  - **Hard affinity**: Explicit pinning (sched_setaffinity)
- **Load balancing**: Distribute work across CPUs
  - **Push migration**: Check load periodically, move tasks
  - **Pull migration**: Idle CPU pulls from busy CPU

## Real-Time Scheduling

| Type | Latency | Priority inversion handling |
|------|---------|--------------------------|
| **Soft RT** | Best-effort (miss occasional deadline OK) | Not critical |
| **Hard RT** | Must meet all deadlines (failure = system failure) | Critical |

### Real-Time Algorithms
- **Rate Monotonic (RM)**: Static priority = 1/period (shorter period = higher priority). Optimal for static priorities.
- **Earliest Deadline First (EDF)**: Dynamic — closest deadline gets CPU. Optimal for dynamic priorities.

### Priority Inversion
High-priority task blocked by low-priority task holding a resource.
**Solution**: **Priority inheritance** — low-priority task temporarily inherits the priority of the highest waiting task.

## Key Interview Questions

1. **Q: Why is SJF optimal for average waiting time?**
   A: Moving a shorter job before a longer one decreases the waiting time of the short job more than it increases the waiting time of the long job (since long job waits once for short job but short job would have waited for long job otherwise).

2. **Q: How do you choose the time quantum for Round Robin?**
   A: Balance context switch overhead vs responsiveness. Rule of thumb: q should be > 80% of CPU bursts (so most processes finish in one quantum). Too small (< 1ms): excessive context switching. Too large (> 100ms): degrades to FCFS, poor response time.

3. **Q: What is the convoy effect?**
   A: In FCFS, short I/O-bound processes queue up behind a long CPU-bound process. The CPU stays busy, but I/O devices sit idle, and many short processes experience high waiting time. Like slow vehicle holding up traffic.

4. **Q: SJF vs SRTF — which gives better average waiting time?**
   A: SRTF (preemptive SJF) gives equal or better average waiting time than non-preemptive SJF, because a shorter arriving process can preempt a running longer process immediately.

5. **Q: What is priority inversion and how is it solved?**
   A: High-priority task waits for low-priority task holding a shared resource, while medium-priority tasks run (starving the high-priority task). Solved by priority inheritance: low-priority task gets temporarily boosted to the highest waiter's priority.
