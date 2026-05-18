# Joins in DBMS

## Table of Contents
1. [What is a Join?](#what-is-a-join)
2. [Types of Joins (by Result)](#types-of-joins-by-result)
   - [Inner Join](#inner-join)
   - [Left Outer Join](#left-outer-join)
   - [Right Outer Join](#right-outer-join)
   - [Full Outer Join](#full-outer-join)
   - [Cross Join](#cross-join)
   - [Self Join](#self-join)
   - [Natural Join](#natural-join)
   - [Equi Join vs Theta Join](#equi-join-vs-theta-join)
3. [Venn Diagram Summary](#venn-diagram-summary)
4. [Join Algorithms](#join-algorithms)
   - [Nested Loop Join](#nested-loop-join)
   - [Block Nested Loop Join](#block-nested-loop-join)
   - [Hash Join](#hash-join)
   - [Sort-Merge Join](#sort-merge-join)
5. [Join Algorithm Comparison](#join-algorithm-comparison)
6. [Key Interview Questions](#key-interview-questions)

---

## What is a Join?

A **join** combines rows from two or more tables based on a related column between them. Joins are the core relational operation that makes relational databases powerful — they allow data to be normalized across tables and recombined on demand.

### Sample Tables Used Throughout

```
Table: employees                     Table: departments
+----+----------+--------+           +--------+-------------+
| id | name     | dept_id|           | dept_id| dept_name   |
+----+----------+--------+           +--------+-------------+
| 1  | Alice    | 10     |           | 10     | Engineering |
| 2  | Bob      | 20     |           | 20     | Sales       |
| 3  | Carol    | 10     |           | 30     | Marketing   |
| 4  | Dave     | NULL   |           +--------+-------------+
+----+----------+--------+
```

---

## Types of Joins (by Result)

### Inner Join

Returns only rows where there is a **match in BOTH tables**. The most common join type.

```sql
SELECT employees.name, departments.dept_name
FROM employees
INNER JOIN departments ON employees.dept_id = departments.dept_id;
```

**Result:**
```
+-------+-------------+
| name  | dept_name   |
+-------+-------------+
| Alice | Engineering |
| Carol | Engineering |
| Bob   | Sales       |
+-------+-------------+
```
- Dave (dept_id=NULL) and Marketing (30) are excluded — no match.

### Left Outer Join

Returns **ALL rows from the left table**, plus matching rows from the right table. If no match, right-side columns are NULL.

```sql
SELECT employees.name, departments.dept_name
FROM employees
LEFT JOIN departments ON employees.dept_id = departments.dept_id;
```

**Result:**
```
+-------+-------------+
| name  | dept_name   |
+-------+-------------+
| Alice | Engineering |
| Bob   | Sales       |
| Carol | Engineering |
| Dave  | NULL        |
+-------+-------------+
```
- Dave is included with NULL dept_name (no matching department).

### Right Outer Join

Returns **ALL rows from the right table**, plus matching rows from the left table. If no match, left-side columns are NULL.

```sql
SELECT employees.name, departments.dept_name
FROM employees
RIGHT JOIN departments ON employees.dept_id = departments.dept_id;
```

**Result:**
```
+-------+-------------+
| name  | dept_name   |
+-------+-------------+
| Alice | Engineering |
| Carol | Engineering |
| Bob   | Sales       |
| NULL  | Marketing   |
+-------+-------------+
```
- Marketing (dept_id=30) has no employees, so name is NULL.

### Full Outer Join

Returns **ALL rows from BOTH tables**. Matching rows are combined; non-matching rows from either side get NULLs on the other side.

```sql
-- MySQL doesn't support FULL JOIN directly; use UNION of LEFT and RIGHT.
SELECT employees.name, departments.dept_name
FROM employees
FULL OUTER JOIN departments ON employees.dept_id = departments.dept_id;
```

**Result:**
```
+-------+-------------+
| name  | dept_name   |
+-------+-------------+
| Alice | Engineering |
| Carol | Engineering |
| Bob   | Sales       |
| Dave  | NULL        |
| NULL  | Marketing   |
+-------+-------------+
```

### Cross Join

Returns the **Cartesian product** — every row from the left table paired with every row from the right. No join condition.

- If left has N rows and right has M rows → result has N × M rows.
- Useful for generating combinations (e.g., all product-size variants).

```sql
SELECT employees.name, departments.dept_name
FROM employees
CROSS JOIN departments;
```

**Result (4 × 3 = 12 rows):**
```
+-------+-------------+
| name  | dept_name   |
+-------+-------------+
| Alice | Engineering |
| Alice | Sales       |
| Alice | Marketing   |
| Bob   | Engineering |
| Bob   | Sales       |
| ...   | ...         |
+-------+-------------+
```

### Self Join

A table is **joined with itself**. Requires table aliases to distinguish the two instances.

```sql
-- Find employees who share the same department
SELECT e1.name AS emp1, e2.name AS emp2, e1.dept_id
FROM employees e1
INNER JOIN employees e2 ON e1.dept_id = e2.dept_id AND e1.id < e2.id;
```

**Result:**
```
+------+------+---------+
| emp1 | emp2 | dept_id |
+------+------+---------+
| Alice| Carol| 10      |
+------+------+---------+
```

Common uses: hierarchical data (employee-manager), finding duplicates, comparing rows.

### Natural Join

Automatically joins on **all columns with the same name** in both tables. No explicit ON clause needed.

```sql
SELECT employees.name, departments.dept_name
FROM employees
NATURAL JOIN departments;
-- Joins on dept_id (the only common column name)
```

**Result:** Same as INNER JOIN on dept_id.

⚠️ **Caution**: Natural joins can be dangerous — if schema changes add new columns with the same name, the join behavior changes silently. Most teams prefer explicit ON clauses.

### Equi Join vs Theta Join

- **Equi Join**: Uses the **equality operator (=)** in the join condition. Inner, outer, and natural joins are typically equi joins.
  ```sql
  SELECT * FROM A JOIN B ON A.id = B.a_id;   -- Equi join
  ```

- **Theta Join**: Uses any **comparison operator** (=, <, >, <=, >=, !=, BETWEEN). A more general form; equi join is a special case of theta join.
  ```sql
  SELECT * FROM A JOIN B ON A.salary > B.min_salary;  -- Theta join (non-equi)
  SELECT * FROM A JOIN B ON A.start_date BETWEEN B.from_date AND B.to_date;
  ```

---

## Venn Diagram Summary

```
    ┌─────────┐     ┌─────────┐
    │  Left   │     │  Right  │
    │  Table  │     │  Table  │
    └─────────┘     └─────────┘

INNER JOIN:        LEFT JOIN:          RIGHT JOIN:         FULL OUTER JOIN:
┌───┬───┬───┐     ┌───┬───┬───┐      ┌───┬───┬───┐       ┌───┬───┬───┐
│   │   │   │     │███│   │   │      │   │   │   │       │███│███│   │
│   │███│   │     │███│███│   │      │   │███│   │       │███│███│███│
│   │   │   │     │███│   │   │      │   │   │   │       │███│   │███│
└───┴───┴───┘     └───┴───┴───┘      └───┴───┴───┘       └───┴───┴───┘
Only matching     All left +          All right +         All rows from
rows              matching right      matching left       both tables

CROSS JOIN: Returns every combination (Cartesian product).
```

---

## Join Algorithms

The DBMS query optimizer chooses a join algorithm based on table sizes, indexes, and available memory.

### Nested Loop Join

The simplest algorithm. For every row in the outer table, scan the entire inner table looking for matches.

```
Algorithm:
for each row r in outer_table:
    for each row s in inner_table:
        if join_condition(r, s) matches:
            output combined row
```

**Complexity**: O(N × M) — where N = outer rows, M = inner rows.

**When to use:**
- Small tables.
- Inner table has an index on the join column (Index Nested Loop Join) — inner scan becomes O(1) per outer row → O(N).
- One table is very small, the other is large with an index.

```
Index Nested Loop Join:
for each row r in outer_table:
    use index to find matching rows in inner_table  -- O(1) lookup
    output combined rows
```

**Performance:**
- No index: O(N×M) — terrible for large tables.
- With index on inner: O(N) — good when outer is small and inner is indexed.

### Block Nested Loop Join

Improvement: load **blocks** (pages) of the outer table into memory to reduce I/O.

```
Algorithm:
for each block B_r of outer table:
    for each block B_s of inner table:
        for each row r in B_r:
            for each row s in B_s:
                if join_condition(r, s) matches:
                    output combined row
```

- Reduces disk I/O by processing page-by-page rather than row-by-row.
- Still O(N×M) in worst case, but with better constant factors.

### Hash Join

Best for **equi-joins** on large tables. Uses a hash table to partition and match rows.

```
Algorithm (two phases):

Phase 1 — Build Phase:
    Build a hash table on the smaller table (build input) using hash(join_key).
    Store each row in bucket h(join_key).

Phase 2 — Probe Phase:
    For each row in the larger table (probe input):
        compute h(join_key)
        look up matching rows in that hash bucket
        output combined rows

Grace Hash Join (when hash table doesn't fit in memory):
    Partition both tables into N buckets using same hash function.
    Join each bucket pair one at a time (fits in memory).
```

```
Build Phase:                     Probe Phase:
Smaller Table (R):               Larger Table (S):
+-----+-----+                    +-----+-----+
| key | val |                    | key | val |
+-----+-----+                    +-----+-----+
| 10  | A   |---\               | 10  | X   |---\
| 20  | B   |---|---> Hash       | 30  | Y   |---|---> Hash
| 30  | C   |---/    Table      | 10  | Z   |---/    Lookup
+-----+-----+       +---+---+   +-----+-----+       in bucket
                    | 10| A |                        find A → output
                    | 20| B |                        find A → output
                    | 30| C |
                    +---+---+
```

**Complexity**: O(N + M) on average. Very efficient for large tables.

**When to use:**
- Equi-joins (requires equality condition).
- Both tables are large with no useful indexes.
- One table fits (or nearly fits) in memory.

### Sort-Merge Join

Sorts both tables on the join key, then merges them like merge sort.

```
Algorithm:
Step 1: Sort both tables on join key (if not already sorted).
Step 2: Merge — two pointers walk through sorted tables:
    while both have rows:
        if r.key < s.key: advance r pointer
        if r.key > s.key: advance s pointer
        if r.key == s.key:
            combine and output
            advance pointer on the table with more matching rows
```

```
Sorted R:                        Sorted S:
+-----+------+                   +-----+------+
| key | val  |                   | key | val  |
+-----+------+                   +-----+------+
| 10  | A    |--> ptr_R          | 10  | X    |--> ptr_S
| 20  | B    |                   | 10  | Y    |
| 30  | C    |                   | 50  | Z    |
+-----+------+                   +-----+------+

Merge: 10==10 → output(A,X), output(A,Y). Advance ptr_S.
       20  < 50 → advance ptr_R.  30 < 50 → advance ptr_R.
       ptr_R done → stop.
```

**Complexity**: O(N log N + M log M) for sorting + O(N+M) for merging.

**When to use:**
- Both tables are already sorted on the join key (e.g., clustered indexes).
- Range joins (non-equi: `<`, `>`, `BETWEEN`) — hash join only handles `=`.
- Very large tables where hash table won't fit in memory.

---

## Join Algorithm Comparison

| Algorithm | Best For | Worst For | Complexity |
|-----------|----------|-----------|------------|
| Nested Loop (no index) | Tiny tables | Large tables | O(N×M) |
| Index Nested Loop | Small outer, indexed inner | Unindexed inner | O(N × log M) |
| Hash Join | Large equi-joins | Non-equi joins, skewed data | O(N+M) |
| Sort-Merge Join | Sorted inputs, range joins | Unsorted, large unsorted | O(N log N + M log M) |

**Rule of thumb (simplified query optimizer logic):**
1. If one table is tiny → Nested Loop (especially with index).
2. If equi-join and memory sufficient → Hash Join.
3. If inputs sorted or range condition → Sort-Merge Join.

---

## Key Interview Questions

1. **Q: What's the difference between INNER JOIN and OUTER JOIN?**
   - INNER JOIN returns only matching rows. OUTER JOIN (LEFT/RIGHT/FULL) preserves all rows from one or both tables, filling with NULLs where there's no match.

2. **Q: When does Hash Join fail or perform poorly?**
   - Non-equi joins (no equality condition).
   - Highly skewed data (all keys hash to same bucket → hash table degrades to linked list).
   - Memory too small to hold hash table → spills to disk (Grace Hash Join needed, adds I/O).

3. **Q: Why would the optimizer choose Nested Loop over Hash Join?**
   - One table is very small (build cost negligible).
   - An index exists on the join column of the inner table (Index Nested Loop is very fast).
   - The query has a LIMIT clause (Nested Loop can stop early; Hash Join must build entire hash table first).

4. **Q: What is a "join explosion"?**
   - When joining tables with duplicate keys on both sides, rows multiply. Example: Table A has 100 rows with key=1, Table B has 200 rows with key=1 → join produces 20,000 rows for that key alone. Mitigate by deduplicating or aggregating before joining.

5. **Q: Explain the difference between NATURAL JOIN and INNER JOIN ... USING.**
   - NATURAL JOIN matches ALL common-named columns automatically. USING(col) explicitly specifies which column to join on. NATURAL JOIN can silently produce wrong results if schemas change; USING is explicit and safer.

6. **Q: How do you optimize a slow JOIN query?**
   - Add indexes on join columns (especially foreign keys).
   - Reduce the row count before joining (filter with WHERE, subquery, CTE).
   - Use EXPLAIN/EXPLAIN ANALYZE to see the join plan.
   - Consider denormalization if the join is always needed.
   - Ensure statistics are up to date (ANALYZE TABLE).
   - Increase work_mem (PostgreSQL) / join_buffer_size (MySQL) for hash joins.

7. **Q: Can you JOIN without a foreign key?**
   - Yes! A join is a logical operation based on any condition. Foreign keys enforce integrity but are not required for joins. You can join on any expression: `ON a.name = b.full_name`, `ON a.salary > b.threshold`, etc.

## Practice Exercises

### Beginner
1. Using `employees` and `departments`, write an `INNER JOIN` that lists only employees with departments.
2. Write a `LEFT JOIN` that also includes employees with no department.
3. Write a `RIGHT JOIN` or equivalent `LEFT JOIN` that lists departments with no employees.
4. Explain why a `CROSS JOIN` can produce a very large result.
5. Write a self join to find pairs of employees in the same department.

### Interview
1. Explain the difference between join type and join algorithm.
2. When is hash join better than nested loop join?
3. Why does an index nested loop join work well when the outer table is small?
4. What is join explosion and how can you reduce it?
5. Why is `NATURAL JOIN` risky in production code?

### Hands-on
1. Create the sample `employees` and `departments` tables from this note and run every join query.
2. Add an index on `employees(dept_id)`, run `EXPLAIN`, and compare the plan before and after indexing.
