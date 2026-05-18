# Operating Systems Notes

## Beginner Reading Path

Follow this order if you are learning OS for interviews:

1. [Introduction and Processes](./intro-and-processes.md) - OS goals, process model, states, PCB, context switching, system calls, and IPC.
2. [Process Scheduling](./process-scheduling.md) - schedulers, criteria, FCFS, SJF, SRTF, priority, Round Robin, and real-time scheduling.
3. [Threads and Concurrency](./threads-and-concurrency.md) - threads, race conditions, critical sections, mutexes, semaphores, monitors, and classic problems.
4. [Deadlocks](./deadlocks.md) - Coffman conditions, resource allocation graphs, prevention, avoidance, detection, and Banker's algorithm.
5. [Memory Management](./memory-management.md) - paging, segmentation, virtual memory, page replacement, and thrashing.
6. [File Systems and Disk](./file-systems-and-disk.md) - file allocation, directories, free-space management, disk scheduling, and RAID.

## Interview Checklist

You should be able to explain:

- Program vs process vs thread.
- User mode vs kernel mode and why system calls are needed.
- Process states and what a PCB stores.
- Context switching cost and why too many switches hurt throughput.
- FCFS, SJF, SRTF, Priority, Round Robin, and Multilevel Feedback Queue.
- Race condition, data race, critical section, mutex, semaphore, monitor, and condition variable.
- Deadlock vs starvation vs livelock.
- Coffman conditions and how to break each one.
- Paging, segmentation, TLB, page fault, virtual memory, and thrashing.
- FIFO, LRU, Optimal, and Clock page replacement.
- File allocation methods and disk scheduling algorithms.

## Hands-on Practice

Try these commands while reading:

```bash
ps aux
top
time sleep 1
ulimit -a
df -h
du -sh .
```

Small C/C++ practice tasks:

1. Create a child process using `fork()` and print parent/child PIDs.
2. Create two threads that increment a shared counter, first without a mutex and then with a mutex.
3. Implement producer-consumer using a bounded queue and semaphores.
4. Simulate Round Robin scheduling for a list of arrival and burst times.
5. Simulate FIFO and LRU page replacement for a reference string.

## Worked Practice Set

1. Scheduling: Given `P1(AT=0,BT=5)`, `P2(AT=1,BT=3)`, `P3(AT=2,BT=1)`, draw Gantt charts for FCFS, SJF, SRTF, and RR with quantum 2.
2. Deadlock: Given Allocation, Max, and Available matrices, calculate Need and find a safe sequence.
3. Memory: For page size 4 KB and logical address 12345, calculate page number and offset.
4. Page replacement: For reference string `1 2 3 4 1 2 5 1 2 3 4 5`, count FIFO and LRU faults with 3 frames.
5. Disk scheduling: For requests `98, 183, 37, 122, 14, 124, 65, 67` and head at 53, compare FCFS, SSTF, SCAN, and C-SCAN.

## References

- Operating System Concepts by Silberschatz, Galvin, and Gagne.
- Modern Operating Systems by Andrew S. Tanenbaum.
- Linux manual pages: `fork`, `exec`, `wait`, `pthread_create`, `mmap`, and `open`.
- OSTEP: Operating Systems: Three Easy Pieces.
