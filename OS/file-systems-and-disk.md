# File Systems & Disk Scheduling

## Table of Contents
1. [File Concepts](#file-concepts)
2. [Access Methods](#access-methods)
3. [Directory Structure](#directory-structure)
4. [Allocation Methods](#allocation-methods)
5. [Free Space Management](#free-space-management)
6. [Disk Structure & Scheduling](#disk-structure--scheduling)
7. [Disk Scheduling Algorithms](#disk-scheduling-algorithms)
8. [Key Interview Questions](#key-interview-questions)

## File Concepts

A **file** is a named collection of related information stored on secondary storage.

### File Attributes
| Attribute | Description |
|-----------|-------------|
| **Name** | Human-readable identifier (file.txt) |
| **Type** | Needed by systems that support multiple file types |
| **Location** | Pointer to device and location on device |
| **Size** | Current size in bytes |
| **Protection** | Access control (rwx bits, ACLs) |
| **Time/Date/User** | Creation, last modification, last access |

### File Operations
| Operation | Description |
|-----------|-------------|
| **Create** | Allocate space, add directory entry |
| **Open** | Load file metadata into memory (returns file descriptor) |
| **Read** | Read bytes from current position |
| **Write** | Write bytes at current position |
| **Seek** | Reposition file pointer |
| **Delete** | Release space, remove directory entry |
| **Truncate** | Reset file size to zero, keep attributes |
| **Close** | Flush buffers, free file descriptor |

### File Types
- **Regular file**: Contains user data (text, binary)
- **Directory**: Container for files and subdirectories
- **Special files**: Device files (character, block), pipes, sockets, symlinks

## Access Methods

### Sequential Access
- Read bytes in order from beginning
- Simple, like a tape
- **Operations**: read_next(), write_next(), reset()

### Direct (Random) Access
- Read/write at any position using byte offset
- Based on disk model (random-access device)
- **Operations**: read(n), write(n), seek(position)
- Used by databases for quick record lookup

### Indexed Access
- Index file contains pointers to blocks
- Index maps key → block address
- Used by large databases (ISAM — Indexed Sequential Access Method)

## Directory Structure

A **directory** is a symbol table mapping file names to their directory entries.

### Common Directory Structures

```
Single-Level:          Two-Level:              Tree-Structured:
/root                   /root                   /root
├── file1              ├── user1/              ├── home/
├── file2              │   ├── file1           │   ├── user1/
└── file3              │   └── file2           │   │   └── file1
                       └── user2/              │   └── user2/
                           └── file3           ├── etc/
                                               │   └── config
                                               └── usr/
                                                   └── bin/

Acyclic Graph:         General Graph:
(shared subdirectories) (allows cycles)
/home/user/docs ──→ /shared/docs      /a → /b → /c → /a (cycle!)
```

| Structure | Pros | Cons |
|-----------|------|------|
| **Single-level** | Simple | Naming collisions, no grouping |
| **Two-level** | Per-user isolation | No group sharing |
| **Tree** | Hierarchical grouping, efficient search | Path names can be long |
| **Acyclic Graph** | Allows sharing (symlinks, hard links) | Complex deletion (dangling pointers) |
| **General Graph** | Flexible cycles allowed | Risk of infinite loops in traversal |

### Directory Operations
- Search, Create, Delete, List, Rename, Traverse

## Allocation Methods

How to allocate disk blocks to files.

### 1. Contiguous Allocation
- File occupies consecutive blocks on disk
- Directory entry: (start_block, length)

```
Pros:
  ✓ Fast sequential access (minimal seek)
  ✓ Direct access easy (start + offset)
Cons:
  ✗ External fragmentation
  ✗ Must pre-allocate or reallocate (difficult to grow)
  ✗ Compaction needed
```

### 2. Linked Allocation
- Each block contains pointer to next block
- Directory entry: (first_block, last_block)

```
Pros:
  ✓ No external fragmentation
  ✓ File can grow dynamically
  ✓ Only need first block to access
Cons:
  ✗ Sequential access only (random access slow)
  ✗ Space wasted on pointers (1-4 bytes per block)
  ✗ Reliability: one bad pointer = lost rest of file
```

### 3. File Allocation Table (FAT)
- Variation of linked allocation
- All pointers gathered in a single table at disk start
- **FAT entry[i]** = next block for file using block i (or EOF)

```
Pros:
  ✓ Random access faster (FAT is cached in memory)
  ✓ Simple and widely supported
Cons:
  ✗ FAT size can be large
  ✗ FAT corruption = catastrophic data loss
  ✗ Limited scalability (FAT32 max file = 4GB)
```

### 4. Indexed Allocation
- Each file has an **index block** (array of block pointers)
- Directory entry: (index_block)

```
Index Block:
+----------+
| ptr[0] → block0
| ptr[1] → block1
| ptr[2] → block2
| ...
+----------+

Pros:
  ✓ Direct access (i-th entry = i-th block)
  ✓ No external fragmentation
  ✓ Dynamic growth
Cons:
  ✗ Wasted space for small files (entire index block)
  ✗ Index block size limits max file size
```

### Handling Large Files with Indexed Allocation

**Linked Scheme**: Link multiple index blocks together.
**Multilevel Index**: Index block points to second-level index blocks, etc.
**Combined Scheme (Unix inode)**:
```
+-------------------+
| 12 Direct pointers | → 12 direct blocks (small files)
+-------------------+
| 1 Single indirect  | → → 1 index block → N blocks
+-------------------+
| 1 Double indirect  | → → N index blocks → N² blocks
+-------------------+
| 1 Triple indirect  | → → N² index blocks → N³ blocks
+-------------------+
For N=1024: max file = (12 + 1024 + 1024² + 1024³) × block_size
                    = ~16TB with 4KB blocks
```

## Free Space Management

### Bitmap (Bit Vector)
- Each block represented by 1 bit: 0 = free, 1 = allocated
- Fast to find contiguous free blocks
- Size: disk_size / (8 × block_size) bytes

### Free List (Linked List)
- Link all free blocks together
- Each free block contains pointer to next
- Inefficient: must traverse to find many free blocks

### Grouping
- First free block stores addresses of `n` free blocks
- Last address points to next grouping block
- Finds many free blocks quickly

### Counting
- Store (start_block, count) for contiguous free runs
- Very compact for large contiguous free space

## Disk Structure & Scheduling

### Disk Anatomy
```
Platter (circular disk, magnetically coated)
  ↓
Track (concentric circle on platter)
  ↓
Sector (arc of track, typically 512B or 4KB)
  ↓
Cylinder (same track across all platters)
```

### Disk Access Time
```
T_access = T_seek + T_rotational_latency + T_transfer

T_seek: Time to move arm to correct cylinder (slowest — ~5-10ms)
T_rotational: Time for desired sector to rotate under head (~2-4ms at 7200 RPM)
T_transfer: Time to read/write data (fast — depends on RPM and sectors/track)
```

### Disk Performance Metrics
| Metric | Typical Value |
|--------|--------------|
| **Seek time** | 3-15 ms |
| **Rotational latency** | 4.2 ms (7200 RPM average) |
| **Transfer rate** | 100-250 MB/s (modern HDD) |
| **IOPS** (Random 4KB) | ~100-200 (HDD) vs 100K-1M (SSD) |

### SSD vs HDD
| Feature | HDD | SSD |
|---------|-----|-----|
| **Access time** | ms (mechanical) | μs (electronic) |
| **Random access** | Slow (seek + rotation) | Fast (no moving parts) |
| **Sequential** | Good | Excellent |
| **Cost per GB** | Low | Higher |
| **Write endurance** | Unlimited | Limited (but high for modern SSDs) |
| **Scheduling** | Important (minimize seek) | Less critical |

## Disk Scheduling Algorithms

Goal: Minimize seek time by reordering disk requests.

### Request Queue Example
Pending requests at cylinders: **98, 183, 37, 122, 14, 124, 65, 67**
Head starts at **53**.

### 1. FCFS (First-Come, First-Served)
Service in arrival order.
```
53 → 98 → 183 → 37 → 122 → 14 → 124 → 65 → 67
Total head movement: 640 cylinders
Simple but worst performance — no optimization.
```

### 2. SSTF (Shortest Seek Time First)
Service closest request first.
```
53 → 65 → 67 → 37 → 14 → 98 → 122 → 124 → 183
Total head movement: 236 cylinders
May cause starvation of far-away requests.
```

### 3. SCAN (Elevator Algorithm)
Head moves in one direction, services all requests, then reverses.
```
Direction: toward 0 (left)
53 → 37 → 14 → 0 → 65 → 67 → 98 → 122 → 124 → 183
Total head movement: 236 cylinders
Fairer than SSTF, no starvation (as long as head keeps moving).
```

### 4. C-SCAN (Circular SCAN)
Head moves in one direction only; when reaching end, jumps to beginning.
```
Direction: toward 199 (right)
53 → 65 → 67 → 98 → 122 → 124 → 183 → 199 → 0 → 14 → 37
Total head movement: 382 cylinders
More uniform wait time than SCAN (treats cylinders as circular list).
```

### 5. LOOK
Like SCAN but reverses at the last request (not end of disk).
```
53 → 37 → 14 → 65 → 67 → 98 → 122 → 124 → 183
Total head movement: 208 cylinders
```

### 6. C-LOOK
Like C-SCAN but jumps from last request to first.
```
53 → 65 → 67 → 98 → 122 → 124 → 183 → 14 → 37
Total head movement: 322 cylinders
```

### Summary

| Algorithm | Seek Movement | Starvation | Fairness |
|-----------|--------------|-----------|----------|
| **FCFS** | 640 | No | Good |
| **SSTF** | 236 | **Yes** (far cylinders) | Poor |
| **SCAN** | 236 | No | Better |
| **C-SCAN** | 382 | No | Best |
| **LOOK** | 208 | No | Better |
| **C-LOOK** | 322 | No | Best |

## Key Interview Questions

1. **Q: What are the different file allocation methods?**
   A: Contiguous (fast but external fragmentation), Linked (no fragmentation but sequential only), Indexed (direct access via index block, no fragmentation — used in Unix inodes).

2. **Q: How does a Unix inode map file blocks?**
   A: Using a combined scheme: 12 direct pointers for small files, then single indirect, double indirect, and triple indirect pointers for larger files. This balances space efficiency for small files with scalability for large files.

3. **Q: Why does disk scheduling matter less for SSDs?**
   A: SSDs have no mechanical parts — no seek time, no rotational latency. Access time is nearly uniform regardless of location. Disk scheduling is designed to minimize mechanical movement, which is irrelevant for SSDs.

4. **Q: Which disk scheduling algorithm causes starvation?**
   A: SSTF can starve requests at the edges of the disk because it always picks the closest request. A continuous stream of requests near the center can indefinitely delay an edge request.

5. **Q: What's the difference between SCAN and LOOK?**
   A: SCAN goes to the physical end of the disk (cylinder 0 or N) before reversing. LOOK only goes as far as the last request in that direction and reverses earlier. LOOK is more practical/efficient.
