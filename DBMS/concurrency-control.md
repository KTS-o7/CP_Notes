# Concurrency Control in DBMS

## Table of Contents
1. [Why Concurrency Control?](#why-concurrency-control)
2. [Problems Without Concurrency Control](#problems-without-concurrency-control)
3. [Lock-Based Protocols](#lock-based-protocols)
   - [Shared and Exclusive Locks](#shared-and-exclusive-locks)
   - [Two-Phase Locking (2PL)](#two-phase-locking-2pl)
   - [Strict 2PL](#strict-2pl)
   - [Rigorous 2PL](#rigorous-2pl)
   - [Lock Conversion / Lock Upgrading](#lock-conversion--lock-upgrading)
4. [Timestamp-Based Protocols](#timestamp-based-protocols)
   - [Timestamp Ordering (TO)](#timestamp-ordering-to)
   - [Thomas' Write Rule](#thomas-write-rule)
5. [Multi-Version Concurrency Control (MVCC)](#multi-version-concurrency-control-mvcc)
6. [Optimistic Concurrency Control](#optimistic-concurrency-control)
7. [Deadlock Handling](#deadlock-handling)
   - [Deadlock Prevention](#deadlock-prevention)
   - [Deadlock Detection](#deadlock-detection)
   - [Deadlock Avoidance](#deadlock-avoidance)
8. [Comparison Summary](#comparison-summary)
9. [Key Interview Questions](#key-interview-questions)

---

## Why Concurrency Control?

When multiple transactions run simultaneously, they may interfere with each other and produce inconsistent results. Concurrency control ensures that concurrent transactions execute correctly while maximizing throughput.

**Goals:**
1. **Serializability**: The concurrent execution is equivalent to some serial execution.
2. **Recoverability**: Committed transactions are not lost after a crash.
3. **High throughput**: Allow as much parallelism as possible.
4. **Fairness**: Avoid starvation of any transaction.

---

## Problems Without Concurrency Control

| Problem | Description | Example |
|---------|-------------|---------|
| **Dirty Read** | T2 reads data written by T1 before T1 commits. T1 later aborts → T2 used garbage. | T1: write(X=100). T2: read(X=100). T1: abort. |
| **Lost Update** | T1 and T2 read same value, both update, one overwrites the other. | T1: read(X=50), X=X+10, write(X=60). T2: read(X=50), X=X+20, write(X=70). T1's update is lost! |
| **Non-Repeatable Read** | T1 reads X, T2 updates X and commits, T1 reads X again → different value. | T1: read(X=50). T2: write(X=100), commit. T1: read(X=100). |
| **Phantom Read** | T1 reads a set of rows based on a condition. T2 inserts a new row matching that condition. T1 re-reads → sees a "phantom" row. | T1: SELECT * WHERE salary>5000 (3 rows). T2: INSERT salary=6000. T1: SELECT * WHERE salary>5000 (4 rows). |

---

## Lock-Based Protocols

Locks control access to data items. A transaction must acquire the appropriate lock before accessing an item.

### Shared and Exclusive Locks

| Lock Type | Symbol | Allows | Conflicts With |
|-----------|--------|--------|----------------|
| **Shared (S)** | Read lock | Reading the item | Exclusive lock (X) |
| **Exclusive (X)** | Write lock | Reading AND writing the item | Both S and X locks |

**Lock Compatibility Matrix:**

```
          Requested
          S       X
Held   ┌───────┬───────┐
  S    │  ✓    │  ✗    │
       ├───────┼───────┤
  X    │  ✗    │  ✗    │
       └───────┴───────┘
```

- Multiple transactions can hold S-locks on the same item simultaneously.
- Only one transaction can hold an X-lock; no other locks can coexist.
- **Lock Manager**: A DBMS component that grants, queues, and releases locks.

```
Example with two transactions:
Time    T1              T2
────────────────────────────────────
t1      S-Lock(A)  ✓
t2      read(A)
t3                      S-Lock(A)  ✓   (S compatible with S)
t4                      read(A)
t5      X-Lock(A)  ⏳ wait...       (X conflicts with T2's S)
t6                      unlock(A)   (T2 releases S)
t7      X-Lock(A)  ✓ granted!
t8      write(A)
t9      unlock(A)
```

### Two-Phase Locking (2PL)

A transaction follows two phases:
1. **Growing Phase**: Acquire locks only (no releasing).
2. **Shrinking Phase**: Release locks only (no acquiring).

Once a transaction releases its **first lock**, it enters the shrinking phase and cannot acquire any more locks.

```
Lock count
  ^
  |     /\
  |    /  \        Growing phase    → lock acquisition only
  |   /    \       Shrinking phase  → lock release only
  |  /      \
  | /        \
  +-------------> Time
```

**Guarantee**: 2PL ensures **conflict serializability**. All schedules produced by 2PL are conflict-serializable.

**Problems with basic 2PL:**
- **Cascading rollbacks**: If T1 aborts after T2 reads its uncommitted data, T2 must also abort, and so on.
- **Deadlocks**: Two transactions waiting for each other's locks.

### Strict 2PL

**Rule**: A transaction holds all its exclusive (X) locks **until it commits or aborts**. Shared locks can be released earlier.

```
Strict 2PL:
- Growing phase: Acquire S and X locks.
- Shrinking phase (after commit/abort): Release ALL locks.
- X-locks held until commit → no other transaction can read/write uncommitted data.

Benefit: Prevents cascading rollbacks (no dirty reads possible).
```

### Rigorous 2PL

**Rule**: A transaction holds **ALL locks (both S and X)** until commit/abort.

```
Rigorous 2PL:
- All locks released only at commit/abort.
- Even stricter than Strict 2PL.
- Easier to implement but reduces concurrency more.
```

### Lock Conversion / Lock Upgrading

A transaction can upgrade an S-lock to X-lock, or downgrade an X-lock to S-lock.

- **Upgrade (S → X)**: Only during the growing phase. Must wait until no other transaction holds S on the item.
- **Downgrade (X → S)**: Only during the shrinking phase (for basic 2PL). Not needed in strict/rigorous 2PL.

---

## Timestamp-Based Protocols

Each transaction is assigned a unique timestamp TS(T) when it starts (usually the system clock or a logical counter). The protocol uses timestamps to determine serialization order without locks.

**Rules:**
- Each data item X has two timestamps:
  - **R-TS(X)**: Largest timestamp of any transaction that successfully **read** X.
  - **W-TS(X)**: Largest timestamp of any transaction that successfully **wrote** X.

When transaction T tries to read or write X:

```
read(X):
    if TS(T) < W-TS(X):    -- T is too old; X has been written by a newer transaction
        ABORT T (or restart with new timestamp)
    else:
        Allow read
        R-TS(X) = max(R-TS(X), TS(T))

write(X):
    if TS(T) < R-TS(X):    -- A newer transaction already read X; T's write would be invisible
        ABORT T
    else if TS(T) < W-TS(X):  -- A newer transaction already wrote X
        ABORT T (or apply Thomas' Write Rule)
    else:
        Allow write
        W-TS(X) = TS(T)
```

```
Example:
Data item X: R-TS(X)=0, W-TS(X)=0

T1 (TS=10): write(X) → allowed. W-TS(X)=10
T2 (TS=5):  read(X)  → TS(5) < W-TS(10)? Yes → ABORT T2 (too old)
T3 (TS=15): read(X)  → allowed. R-TS(X)=15
T4 (TS=12): write(X) → TS(12) < R-TS(15)? Yes → ABORT T4
```

**Advantages:**
- Deadlock-free (no waiting for locks).
- Simple and deterministic.

**Disadvantages:**
- Transactions may abort and restart frequently (starvation risk for long transactions).
- Not recoverable without extra mechanisms.

### Thomas' Write Rule

A modification to timestamp ordering that **ignores obsolete writes** instead of aborting.

```
Original write(X) rule:
    if TS(T) < W-TS(X):
        ABORT T

Thomas' Write Rule:
    if TS(T) < W-TS(X):
        IGNORE the write (don't abort T)
        -- T's write is obsolete anyway; a newer transaction already wrote X
```

```
Example:
T1 (TS=10): write(X=100) → W-TS(X)=10
T2 (TS=20): write(X=200) → W-TS(X)=20
T3 (TS=15): write(X=150) → TS(15) < W-TS(20)?
    Without Thomas' rule: ABORT T3
    With Thomas' rule:    IGNORE write (X stays 200; T3 continues)
```

**Benefit**: More concurrency — fewer transaction aborts. Useful when T3's write would have been overwritten anyway.

---

## Multi-Version Concurrency Control (MVCC)

MVCC maintains **multiple versions** of each data item. Readers see a consistent snapshot without blocking writers, and writers don't block readers. This is the foundation of PostgreSQL, Oracle, MySQL InnoDB, and many modern DBMS.

### How MVCC Works

1. Each data item has a chain of versions: `X(v1) → X(v2) → X(v3) → ...`
2. Each version is tagged with a **creation timestamp** (or transaction ID) and a **deletion timestamp**.
3. A transaction with timestamp TS(T) sees the version that was current at TS(T).
4. Writers create new versions; old versions are retained for readers.

```
Transaction T (TS=50) reads X:
    X versions:  v1 (created by T10, deleted by T30)
                 v2 (created by T30, deleted by T60)  <-- T sees this one
                 v3 (created by T60, not deleted)
    
    T sees v2 because it was alive at TS=50.
```

```
Read/Write Rules in MVCC:
┌──────────────────────────────────────────────────────┐
│ READ(X):  Find the latest version of X with          │
│           creation_ts ≤ TS(T) < deletion_ts          │
│                                                      │
│ WRITE(X): Create a new version with                  │
│           creation_ts = TS(T)                        │
│           If there's a version with creation_ts      │
│           between [TS(T)+1, ∞) that hasn't committed,│
│           abort T (write-write conflict).             │
└──────────────────────────────────────────────────────┘
```

### MVCC Benefits
- **Readers never block writers; writers never block readers.**
- No read locks needed → high concurrency for read-heavy workloads.
- Consistent snapshots for long-running queries (no dirty/non-repeatable reads).
- Natural support for time-travel queries.

### MVCC Drawbacks
- **Storage overhead**: Old versions accumulate (need vacuum/cleanup).
- **Write-write conflicts**: Two writers to the same item still need resolution.
- **Long-running transactions**: Can prevent cleanup of old versions (bloat).

### MVCC in Practice

| DBMS | MVCC Implementation | Old Version Cleanup |
|------|--------------------|---------------------|
| PostgreSQL | Tuple versions in heap pages | VACUUM (autovacuum) |
| MySQL InnoDB | Undo log + rollback segments | Purge thread |
| Oracle | Undo segments | Automatic undo retention |

---

## Optimistic Concurrency Control

Assumes conflicts are **rare** and validates at commit time rather than during execution. Suitable for low-contention environments.

### Three Phases:

```
Phase 1 — Read Phase:
    Execute transaction privately. Read values into local workspace.
    All writes go to local copies (not the database).
    
Phase 2 — Validation Phase:
    Check if any conflict occurred with other concurrent transactions.
    If conflict detected → ABORT and restart.
    
Phase 3 — Write Phase:
    If validation succeeds → apply local writes to the database.
```

### Validation Techniques:

- **Backward Validation**: Check this transaction against all recently committed transactions.
- **Forward Validation**: Check this transaction against all currently active transactions.

```
Validation check (backward OCC):
    For each committed transaction Tc that committed during T's read phase:
        If T's read set intersects with Tc's write set:
            CONFLICT → ABORT T
        Else:
            VALID → proceed to write phase
```

**When to use:**
- Read-mostly workloads.
- Low contention (few transactions touch the same data).
- Mobile/apps with local writes synced later (SQLite uses optimistic locking).

---

## Deadlock Handling

A **deadlock** occurs when T1 waits for T2, and T2 waits for T1 (or a longer cycle). Both are stuck forever.

```
Deadlock Example:
    T1: Lock(A) → Lock(B)    (holds A, waits for B)
    T2: Lock(B) → Lock(A)    (holds B, waits for A)
    
    Wait-for graph: T1 → T2 → T1 (cycle → deadlock!)
    
    T1 has A, wants B       T2 has B, wants A
         ┌───┐                  ┌───┐
         │T1 │─────────────────>│T2 │
         └───┘   waiting for B  └───┘
           ^                      │
           └──────────────────────┘
                waiting for A
```

### Deadlock Prevention

Eliminate one of the **four necessary conditions** for deadlock:
1. **Mutual exclusion** (can't eliminate — needed for consistency).
2. **Hold and wait**: Transaction must acquire all locks at once, or release all locks before requesting new ones.
3. **No preemption**: DBMS can preempt locks (abort and restart a transaction to release its locks).
4. **Circular wait**: Impose a total ordering on data items; transactions must lock items in that order.

#### Wait-Die and Wound-Wait Schemes (Timestamp-based prevention)

Both use transaction timestamps to break deadlocks without detection.

| Scheme | Rule | Older Transaction (TS=10) | Younger Transaction (TS=20) |
|--------|------|---------------------------|----------------------------|
| **Wait-Die** | Older waits; younger dies | **Waits** for younger's lock | **Dies** (aborts/restarts) if needs older's lock |
| **Wound-Wait** | Older wounds; younger waits | **Wounds** younger (forces it to abort) | **Waits** for older's lock |

```
Wait-Die (non-preemptive):
    T_old requests lock held by T_young → T_old WAITS
    T_young requests lock held by T_old → T_young DIES (aborts)
    
    "Older can wait, younger must restart."

Wound-Wait (preemptive):
    T_old requests lock held by T_young → T_young is WOUNDED (aborted), T_old gets lock
    T_young requests lock held by T_old → T_young WAITS
    
    "Older wounds younger, younger waits for older."
```

**Key difference**: In Wait-Die, older transactions may wait. In Wound-Wait, older transactions never wait — they preempt younger ones. Wound-Wait has fewer aborts overall because older transactions are less likely to abort.

### Deadlock Detection

Allow deadlocks to happen, detect them, and recover.

**Wait-For Graph (WFG):**
- Nodes = active transactions.
- Edge Ti → Tj = Ti is waiting for Tj to release a lock.
- **Cycle in WFG → deadlock.**

```
Detection Algorithm:
    Periodically (or on every lock request that blocks):
        1. Build/update the wait-for graph.
        2. Run cycle detection (DFS or topological sort).
        3. If cycle found → pick a victim transaction to abort.
```

**Victim Selection Criteria:**
- Youngest transaction (fewest resources invested).
- Transaction with fewest locks held.
- Transaction that has been restarted the fewest times (avoid starvation).
- Transaction with the smallest rollback cost.

### Deadlock Avoidance

The DBMS checks whether granting a lock request **might** lead to a deadlock before granting it. If the request is unsafe, the transaction waits or aborts preemptively.

Methods:
- Conservative 2PL: Transaction declares all locks it will ever need at start. If all are available → grant all at once. Else → wait.
- Resource ordering: Assign a linear ordering to all data items. All transactions must request locks in this order (prevents circular wait).

---

## Comparison Summary

| Protocol | Deadlock? | Cascading Aborts? | Concurrency | Complexity |
|----------|-----------|-------------------|-------------|------------|
| Basic 2PL | Yes | Yes | Medium | Low |
| Strict 2PL | Yes | No | Medium | Low |
| Rigorous 2PL | Yes | No | Low | Low |
| Timestamp Ordering | No | Yes (can be fixed) | Medium | Low |
| MVCC | Rare (write-write) | No | High | High |
| Optimistic CC | Rare | No (validated at commit) | High (low contention) | Medium |

---

## Key Interview Questions

1. **Q: How does MVCC allow readers and writers to not block each other?**
   - Readers read old versions (snapshots) instead of the latest version. Writers create new versions. Since they operate on different versions, no lock conflict occurs.

2. **Q: Why does Strict 2PL prevent cascading rollbacks?**
   - Strict 2PL holds all exclusive locks until commit. No other transaction can read uncommitted data → if the writer aborts, no other transaction has seen its writes → no cascading rollback needed.

3. **Q: When would you use Optimistic Concurrency Control over locking?**
   - In low-contention environments (few conflicts). In read-heavy workloads. In distributed systems where locking overhead is high. In client-side databases (SQLite).

4. **Q: What is the difference between Wait-Die and Wound-Wait?**
   - Wait-Die: Older waits for younger; younger dies if it needs older's lock. (Non-preemptive toward younger.)
   - Wound-Wait: Older preempts younger (wounds it); younger waits for older. (Preemptive toward younger.)
   - Wound-Wait generally has fewer aborts because older transactions (which have done more work) are favored.

5. **Q: How does PostgreSQL implement MVCC?**
   - Each row has `xmin` (creating transaction ID) and `xmax` (deleting transaction ID). A transaction sees rows where xmin is committed before it started and xmax is either null or committed after it started. Old versions are cleaned by VACUUM.

6. **Q: What is snapshot isolation and how does it differ from serializability?**
   - Snapshot isolation: Each transaction sees a consistent snapshot of the database as of its start time. It prevents dirty reads, non-repeatable reads, and phantoms BUT allows **write skew** anomalies (two transactions read overlapping data, then update disjoint parts based on what they read). Full serializability prevents write skew.

7. **Q: How do you detect deadlocks in a database?**
   - Build a wait-for graph (nodes = transactions, edges = waiting relationships). If there's a cycle, a deadlock exists. In PostgreSQL, enable `log_lock_waits` and check `deadlock_timeout`. In MySQL InnoDB, check `SHOW ENGINE INNODB STATUS` or enable `innodb_print_all_deadlocks`.
