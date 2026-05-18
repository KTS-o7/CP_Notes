# DBMS Notes

## Beginner Reading Path

Follow this order if you are starting DBMS from scratch:

1. [Database Introduction](./DB_Intro.md) - why databases exist, DBMS users, and basic architecture.
2. [RDBMS](./rdbms.md) - tables, relations, keys, and relational database basics.
3. [Relational Algebra](./RelationalAlgebra.md) - selection, projection, joins, and set operations.
4. [SQL](./sql.md) - query syntax, DDL, DML, constraints, and common commands.
5. [Normalization](./normalization.md) - functional dependencies and normal forms.
6. [Transactions](./transactions.md) - ACID, schedules, recoverability, and isolation.
7. [Indexing](./indexing.md) - B+ trees, hash indexes, clustered indexes, and index trade-offs.
8. [Joins](./joins.md) - SQL join types and physical join algorithms.
9. [Concurrency Control](./concurrency-control.md) - 2PL, timestamp ordering, MVCC, OCC, and deadlocks.
10. [NoSQL](./nosql.md) - non-relational models and when they are useful.

## What To Practice First

Start with SQL before memorizing theory. Create two or three small tables, insert sample rows, and practice:

- `SELECT`, `WHERE`, `ORDER BY`, `GROUP BY`, and `HAVING`.
- `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL OUTER JOIN`, and `SELF JOIN`.
- Primary key, foreign key, unique, check, and not-null constraints.
- Normalizing one messy table into 1NF, 2NF, and 3NF.
- Reading an `EXPLAIN` plan before and after adding an index.

## Interview Checklist

You should be able to explain these without notes:

- Difference between DBMS and RDBMS.
- Primary key vs foreign key vs candidate key.
- Normalization and why denormalization is sometimes acceptable.
- ACID properties with a money-transfer example.
- Conflict serializability using a precedence graph.
- B-Tree vs B+ Tree and why databases prefer B+ Trees.
- Clustered vs non-clustered indexes.
- Join result types vs join algorithms.
- Dirty read, non-repeatable read, phantom read, and write skew.
- 2PL, Strict 2PL, MVCC, and optimistic concurrency control.

## Mini Project

Build a small library database:

1. Tables: `books`, `members`, `loans`, and `authors`.
2. Add primary keys and foreign keys.
3. Write queries for active loans, overdue books, most borrowed books, and members with no loans.
4. Add indexes on foreign keys and common filters.
5. Compare query plans before and after indexing.

## References

- Database System Concepts by Silberschatz, Korth, and Sudarshan.
- Fundamentals of Database Systems by Elmasri and Navathe.
- PostgreSQL documentation: indexes, transactions, isolation, and `EXPLAIN`.
- MySQL documentation: InnoDB indexes, transactions, and locking.
