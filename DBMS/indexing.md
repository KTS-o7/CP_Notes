# Indexing in DBMS

## Table of Contents
1. [What is Indexing?](#what-is-indexing)
2. [Why Indexing Matters](#why-indexing-matters)
3. [Types of Indexes](#types-of-indexes)
   - [Primary Index](#primary-index)
   - [Secondary Index](#secondary-index)
   - [Clustered Index](#clustered-index)
   - [Non-Clustered Index](#non-clustered-index)
   - [Dense vs Sparse Index](#dense-vs-sparse-index)
4. [B-Tree Indexing](#b-tree-indexing)
5. [B+ Tree Indexing](#b-tree-indexing-1)
6. [Hash Indexing](#hash-indexing)
7. [Bitmap Index](#bitmap-index)
8. [Composite Index](#composite-index)
9. [Advantages and Disadvantages](#advantages-and-disadvantages)
10. [When to Use Which Index](#when-to-use-which-index)
11. [Key Interview Questions](#key-interview-questions)

---

## What is Indexing?

Indexing is a data structure technique used to quickly locate and access data in a database. An index is like the index at the back of a book — instead of scanning every page (full table scan), you look up the keyword in the index and jump directly to the relevant pages.

- **Without Index**: Full table scan — DBMS reads every row sequentially. O(n).
- **With Index**: Uses the index structure to find rows in O(log n) or O(1) time.

### Key Terminology
- **Search Key**: The attribute(s) used to look up records.
- **Index Entry / Index Record**: A pair `(search-key, pointer-to-data-record)`.
- **Index File**: The file that stores index entries, usually smaller than the data file.

---

## Why Indexing Matters

1. **Speed**: Dramatically reduces I/O operations for SELECT queries.
2. **Efficiency**: Especially critical on large tables (millions of rows).
3. **Uniqueness**: Primary key indexes enforce uniqueness of key values.
4. **ORDER BY / GROUP BY**: Indexes help avoid sorting by returning data already in order.
5. **JOIN performance**: Speeds up join conditions on foreign keys.

> **Trade-off**: Indexes speed up reads but slow down writes (INSERT, UPDATE, DELETE) because the index must also be updated.

---

## Types of Indexes

### Primary Index

- Built on the **primary key** of a table.
- Data file is sorted on the primary key (ordered file).
- The index is sparse — one entry per block of the data file (anchor record for each block).
- Usually implemented as a **clustered index** (physically reorders rows).

```
Primary Index (Sparse) on ordered file:

Index File:                    Data File (sorted on PK):
+--------+---------+          +-----+-------------------+
|  PK    | Pointer |          | 101 | Alice | CS        |
+--------+---------+          | 102 | Bob   | Math      |
| 101 ---|-------> | --->     | 105 | Carol | Physics   |
| 201 ---|-------> |          +-----+-------------------+
| 301 ---|-------> |          | 201 | Dave  | English   |
+--------+---------+          | 205 | Eve   | History   |
                              +-----+-------------------+
                              | 301 | Frank | Art       |
                              +-----+-------------------+
```

### Secondary Index

- Built on **non-key attributes** (can have duplicates).
- Data file may not be ordered by this attribute.
- The index is **dense** — one entry per record in the data file.
- Multiple secondary indexes can exist on a table.
- Also called a **non-clustered index** when the data order is independent.

### Clustered Index

- The **data rows are physically stored in the order of the index key**.
- A table can have **only one** clustered index (because physical order can be only one).
- Leaf nodes of the index contain the actual data rows (or direct pointers to them in order).
- Typically the primary key index in most DBMS (MySQL InnoDB always clusters on PK).
- **Fast for range queries**: `WHERE id BETWEEN 100 AND 200` — all rows are adjacent on disk.

### Non-Clustered Index

- The **logical order** of the index does **not** match the physical order of data.
- A table can have **multiple** non-clustered indexes.
- Leaf nodes store pointers (row IDs or primary key values) to the actual data.
- Requires an **extra lookup** to fetch the actual row (bookmark lookup / key lookup).
- Slower for range scans compared to clustered indexes (random I/O due to scattered data).

**Clustered vs Non-Clustered Comparison:**

| Feature | Clustered Index | Non-Clustered Index |
|---------|----------------|---------------------|
| Physical order | Matches index order | Independent of data order |
| Count per table | Only 1 | Many (up to DB limits) |
| Leaf nodes | Actual data pages | Pointers to data rows |
| Speed for range queries | Very fast | Slower (extra lookups) |
| Storage | Smaller (data = leaf) | Larger (separate structure) |

### Dense vs Sparse Index

| Aspect | Dense Index | Sparse Index |
|--------|------------|--------------|
| Entries | One entry per search-key value in data file | One entry per block/page of data file |
| File requirement | Works on any file (ordered or unordered) | Only works on ordered (sorted) files |
| Size | Larger | Smaller |
| Lookup speed | Faster (direct pointer to record) | Slightly slower (scan within block) |
| Duplicate keys | Handles duplicates | For primary key columns only |
| Maintenance | More insert/delete overhead | Less overhead per entry |

```
Dense Index:                       Sparse Index:
Index:                             Index:
key|ptr                            key|ptr
101|R1                             101|block1_ptr
102|R2                             201|block2_ptr
103|R3                             301|block3_ptr
104|R4
105|R5
```

---

## B-Tree Indexing

A **B-Tree** (Balanced Tree) is a self-balancing tree data structure that maintains sorted data and allows searches, insertions, and deletions in **O(log n)** time.

### B-Tree Properties (order m):
1. All leaves are at the same level (perfectly balanced).
2. Root has at least 2 children (unless it's a leaf).
3. Each internal node (except root) has between ⌈m/2⌉ and m children.
4. Each node can store up to (m-1) keys.
5. Keys within a node are sorted.

### B-Tree Node Structure:
```
Internal Node (order m=5, so up to 4 keys, up to 5 children):
+-----+------+-----+------+-----+------+-----+------+-----+
| P1  | K1   | P2  | K2   | P3  | K3   | P4  | K4   | P5  |
+-----+------+-----+------+-----+------+-----+------+-----+
  |      |      |      |      |      |      |      |
  v      v      v      v      v      v      v      v
 subtree  subtree  subtree  subtree  subtree  subtree  subtree  subtree
  (<K1)  (K1<K<K2) ...                                       (>K4)
```

### B-Tree ASCII Diagram (order 3):
```
                [30, 60]
               /   |    \
              /    |     \
         [10,20] [40,50] [70,80]
         /  |  \  /  |  \  /  |  \
       [5] [15] [25] [35] [45] [55] [65] [75] [85]
```

### B-Tree in Databases:
- Internal nodes store **both keys AND data pointers**.
- Searching stops at any level if key is found.
- Higher fan-out → shallower tree → fewer disk I/Os.

---

## B+ Tree Indexing

A **B+ Tree** is a variant of B-Tree where:
1. **All keys reside in the leaves** (internal nodes only store routing keys).
2. **Leaf nodes are linked together** (doubly linked list) for efficient range scans.
3. Internal nodes only guide navigation; data pointers are only in leaves.

### B+ Tree Structure:
```
Internal Nodes (keys only, no data pointers):
                [50]
               /     \
              /       \
         [20, 35]     [65, 80]
        /   |   \     /   |   \
       /    |    \   /    |    \
     [10]  [25]  [40] [55]  [70]  [90]    <-- Leaf level (linked list)
      |      |     |    |     |     |
    data   data  data  data  data  data
      |______|_____|____|_____|_____|     <-- Doubly linked
      <================================>
```

### B+ Tree Properties:
- **High fan-out**: Internal nodes store only keys (no data), so each node fits more keys → shallower tree.
- **Range queries are efficient**: Once you reach a leaf, just follow the linked list forward.
- **All searches go to leaf level**: Even if key exists in internal node, you traverse to leaf for data.
- **Stable I/O cost**: Every search takes exactly O(log n) + scan within leaf.

### B-Tree vs B+ Tree:

| Feature | B-Tree | B+ Tree |
|---------|--------|---------|
| Data pointers | Internal AND leaf nodes | Only leaf nodes |
| Leaf linking | No linked list | Doubly linked list |
| Range queries | Requires tree traversal for each value | Follow linked list after first find |
| Tree height | Taller (less fan-out) | Shallower (higher fan-out) |
| Search time | Can stop early at internal nodes | Always goes to leaf |
| Common use | Some filesystems | Most DBMS (MySQL, PostgreSQL, Oracle) |

---

## Hash Indexing

Hash indexing uses a **hash function** h(k) to map a search key k to a bucket address. Ideal for **equality lookups** (point queries), not range queries.

```
Hash Function: h(k) = k mod N

Buckets:
+--------+---------------------------+
| Bucket | Records                   |
+--------+---------------------------+
|   0    | k=10 → ptr, k=20 → ptr   |
|   1    | k=11 → ptr, k=21 → ptr   |
|   2    | k=12 → ptr               |
|   3    | k=13 → ptr, k=23 → ptr   |
+--------+---------------------------+
```

### Types of Hash Indexing:
- **Static Hashing**: Fixed number of buckets. Performance degrades as data grows (overflow chains).
- **Dynamic/Extendable Hashing**: Buckets split dynamically as they fill up. Uses a directory of pointers. Better for growing data.
- **Linear Hashing**: Gradual bucket splitting without a directory.

### When to Use:
- ✅ Equality searches: `WHERE id = 12345`
- ✅ Hash joins in query execution
- ❌ Range queries: `WHERE id BETWEEN 100 AND 200`
- ❌ Pattern matching: `WHERE name LIKE 'A%'`
- ❌ Sorting / ORDER BY

---

## Bitmap Index

- Uses bit arrays (bitmaps) and bitwise operations.
- Each distinct value of an indexed column gets a bitmap.
- **Best for low-cardinality columns** (e.g., Gender: M/F, Status: Active/Inactive).
- Very fast for AND/OR/NOT combinations.
- Common in data warehousing (Oracle, PostgreSQL).

```
Bitmap on 'Gender' column:
        Row1 Row2 Row3 Row4 Row5
Male:     1    0    0    1    1
Female:   0    1    1    0    0
```

Bitwise AND for `Male AND Active`: 1-2 orders of magnitude faster than B-tree for such queries.

---

## Composite Index

An index on **multiple columns** (column1, column2, ..., columnN).

- Order matters: index is sorted first by column1, then by column2, etc.
- Supports queries filtering on prefix columns: `WHERE col1 = x` or `WHERE col1 = x AND col2 = y`.
- Does **NOT** efficiently support skipping prefix: `WHERE col2 = y` (without col1) won't use the index well.

```
CREATE INDEX idx_name ON employees(dept_id, salary);
-- Good for: WHERE dept_id = 5
-- Good for: WHERE dept_id = 5 AND salary > 50000
-- Bad for:  WHERE salary > 50000  (skips leading column)
```

---

## Advantages and Disadvantages

### Advantages
- **Faster SELECT queries**: Often reduces I/O by orders of magnitude.
- **Unique constraint enforcement**: Primary key and unique indexes prevent duplicates.
- **Faster ORDER BY/GROUP BY**: Pre-sorted data eliminates sorting.
- **Faster JOINs**: Indexes on foreign keys dramatically speed up joins.

### Disadvantages
- **Slower writes**: INSERT/UPDATE/DELETE must also update all indexes.
- **Extra storage**: Indexes consume disk space (can be significant with many indexes).
- **Query planner overhead**: Too many indexes can confuse the optimizer and lead to poor plan choices.
- **Maintenance**: Indexes become fragmented over time and need rebuilding.

---

## When to Use Which Index

| Scenario | Recommended Index |
|----------|-------------------|
| Primary key lookups | Clustered B+ Tree |
| Range queries on ordered data | Clustered B+ Tree |
| Equality lookups only (no ranges) | Hash Index |
| Secondary lookups on non-key columns | Non-Clustered B+ Tree |
| Low-cardinality columns (gender, status) | Bitmap Index |
| Multi-column filtering (prefix pattern) | Composite B+ Tree Index |
| Full-text search | GIN (PostgreSQL) / FULLTEXT (MySQL) |
| Geospatial queries | R-Tree / GiST |

---

## Key Interview Questions

1. **Q: Why B+ Tree over B-Tree in databases?**
   - Higher fan-out → shallower tree → fewer disk I/Os
   - Leaf linked list enables efficient range scans
   - All searches have uniform cost

2. **Q: How many clustered indexes can a table have?**
   - Only **one**, because data rows can be physically sorted in only one order.

3. **Q: What happens to indexes during INSERT?**
   - Every index on the table must be updated. The B+ Tree may split nodes. This is why too many indexes hurt write performance.

4. **Q: What is index fragmentation and how to fix it?**
   - Over time, page splits leave empty space in index pages. Fix with `REINDEX` or `ALTER INDEX ... REBUILD`.

5. **Q: When would you NOT use an index?**
   - On very small tables (full scan is faster).
   - On columns with very few distinct values (unless bitmap).
   - On tables with heavy write loads and few reads.
   - On columns frequently updated (high index maintenance cost).

6. **Q: What is a covering index?**
   - An index that contains all columns needed by a query, so the DBMS never needs to access the actual table rows. Faster because it avoids the bookmark/key lookup.

```sql
-- Covering index example:
CREATE INDEX idx_cover ON employees(dept_id, salary, name);
SELECT dept_id, salary, name FROM employees WHERE dept_id = 5;
-- All columns in the index → no table access needed.
```
