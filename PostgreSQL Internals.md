---
tags:
  - TODO
  - DB
---

# Database Cluster, Databases and Tables

## Internal Layout of a Heap Table File

A data file is divided into **pages** (or **blocks**) of fixed length. The pages within each file are numbered sequentially from 0, and these numbers are called **block numbers**

### Heap Table File

![PostgreSQL heap table file|700](PostgreSQL%20heap%20table%20file.png)

A page within a table file is a **slotted page** and contains three kinds of data:

1. **Heap tuple** - A record data itself. Heap tuples are stacked in order from the bottom of the page. The internal structure of tuple is described in [Section 5.2](https://www.interdb.jp/pg/pgsql05/02.html) and [Chapter 9](https://www.interdb.jp/pg/pgsql09.html)
2. **Line pointer** - A pointer to each heap tuple. Line pointers form a simple array that plays the role of an index to the tuples. Each index is numbered sequentially from 1, and called **offset number**. When a new tuple is added to the page, a new line pointer is also pushed onto the array to point to the new tuple
3. **Header info** – A header contains general information about the page. The major variables:
	1. `pd_lsn` – Stores the LSN of XLOG record written by the last change of this page. The details are described in [Section 9.1.2](https://www.interdb.jp/pg//pgsql09/01.html#912-insertion-operations-and-database-recovery).
	2. `pd_lower`, `pd_upper` – `pd_lower` points to the end of line pointers, and `pd_upper` to the beginning of the newest heap tuple

To identify a tuple within the table, a **tuple identifier (TID)**  `ctid ????? ` is used internally. A TID comprises a pair of values: the **block number** of the page that contains the tuple, and the **offset number** of the line pointer that points to the tuple

A large heap tuples are stored and managed using a method called [TOAST](https://www.postgresql.org/docs/current/storage-toast.html)

### Index File

#TODO 

### Free Space Map File

#TODO 

### Visibility Map File

#TODO 

## Writing and Reading Tuples

### Writing Heap Tuples

![PostgreSQL writing heap tuple|700](PostgreSQL%20writing%20heap%20tuple.png)

When the second tuple is inserted, it is placed after the first one. The second line pointer is appended to the first one, and it points to the second tuple. The `pd_lower` changes to point to the second line pointer, and the `pd_upper` to the second heap tuple

### Reading Heap Tuple

- **Sequential scan** – It reads all tuples in all pages sequentially by scanning all line pointers in each page
- **B-tree index scan** – It reads an index file that contains index tuples, each of which is composed of an index key and a TID that points to the target heap tuple. If the index tuple with the key that you are looking for has been found, PostgreSQL reads the desired heap tuple using the obtained TID value

# Process And Memory Architecture

## Process Architecture

![PostgreSQL process architecture|600](PostgreSQL%20process%20architecture.png)

PostgreSQL server contains the following types of processes:

- The **postgres server process** is the parent of all processes related to database cluster management
- Each **backend process** handles all queries and statements issued by a connected client
- Various **background processes** perform processes perform tasks for database management, such as VACUUM and CHECKPOINT processing
- **Replication-associated processes** perform streaming replication. More details are described in [Chapter 11](https://www.interdb.jp/pg/pgsql11.html)

### Postgres Server Process

Postgres server process is the parent of all processes within a PostgreSQL server

It allocates a shared memory area, initiates various background processes, starts replication-related processes and background worker processes as needed, and waits for connection requests from clients. When a connection request is received, it spawns a backend process to handle the client session

### Backend Processes

A backend process, also called a _postgres_ process, is started by the postgres server process and handles all queries issued by one connected client. It communicates with the client using a single TCP connection and terminates when the client disconnects

Because each new client connection spawns a backend process, conection pools suchs as [PgBouncer Usage and Best Practices](PgBouncer%20Usage%20and%20Best%20Practices.md) are used

### Background Processes

| process                    | description                                                                                            | reference                                                                                                          |
| -------------------------- | ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| background writer          | This process writes dirty pages on the shared buffer pool to a disk on a regular basis gradually       | [Section 8.6](https://www.interdb.jp/pg/pgsql08/06.html)                                                           |
| checkpointer               | This process performs the checkpoint process                                                           | [Section 8.6](https://www.interdb.jp/pg/pgsql08/06.html), [Section 9.7](https://www.interdb.jp/pg/pgsql09/07.html) |
| autovacuum launcher        | This process periodically postgres server to create the autovacuum workers for the VACUUM process      | [Section 6.5](https://www.interdb.jp/pg/pgsql06/05.html)                                                           |
| WAL writer                 | This process writes and flushes the WAL data on the WAL buffer to disk periodically                    | [Section 9.6.1](https://www.interdb.jp/pg/pgsql09/06.html#961-wal-writer-process)                                  |
| WAL Summarizer             | This process tracks changes to all database blocks and writes these modifications to WAL summary files | [Section 9.6.2](https://www.interdb.jp/pg/pgsql09/06.html#962-wal-summarizer-process)                              |
| archiver                   | This process executes archiving logging                                                                | [Section 9.10](https://www.interdb.jp/pg/pgsql09/10.html)                                                          |

## Memory Architecture

![PostgreSQL memory architecture|300](PostgreSQL%20memory%20architecture.png)

### Local Memory Area

Each backend process allocates a local memory area for query processing, divided into several sub-areas

| sub-area             | description                                                                                                                         | reference                                                |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| work_mem             | The executor uses this area for sorting tuples, and for joining tables by [Merge Join](#Merge%20Join) and [Hash Join](#Hash%20Join) | [Chapter 3](https://www.interdb.jp/pg/pgsql03.html)      |
| maintenance_work_mem | Maintenance operations (e.g., VACUUM, REINDEX) use this area                                                                        | [Section 6.1](https://www.interdb.jp/pg/pgsql06/01.html) |
| temp_buffers         | The executor uses this area for storing temporary tables                                                                            |                                                          |

### Shared Memory Area

A shared memory area is allocated by a PostgreSQL server when it starts up. This area is also divided into several fixed-sized sub-areas

| sub-area           | description                                                                                           | reference                                                |
| ------------------ | ----------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| shared buffer pool | PostgreSQL loads pages within tables and indexes from a disk to this area, and operates them directly | [Chapter 8](https://www.interdb.jp/pg/pgsql08.html)      |
| WAL buffer         | The WAL buffer is a buffering area of the WAL data before writing to a disk                           | [Chapter 9](https://www.interdb.jp/pg/pgsql09.html)      |
| commit log         | The commit log (CLOG) keeps the states of all transactions for the MVCC                               | [Section 5.4](https://www.interdb.jp/pg/pgsql05/04.html) |

# Join Operation

## Nested Loop Join

Nested loop join can be used in all join conditions

## Merge Join

#TODO

## Hash Join

#TODO

# Index-Only Scan and Visibility Map

Whenever PostgreSQL is looking at an index entry, it cannot tell whether or not that entry is visible to the current transaction. It could be a deleted entry or an entry that has not yet been committed. The canonical way to find out is to look into the table and check the `xmin`/`xmax` values

A consequence is that there is no such thing as an index-only scan in PostgreSQL. No matter how many columns you put into an index, PostgreSQL will always need to check the visibility, which is not available in the index

Yet there is an `Index Only Scan` operation in PostgreSQL—but that still needs to check the visibility of each row version by accessing data outside the index. Instead of going to the table, the `Index Only Scan` first checks the so-called visibility map (it's a [Bloom Filter](Bloom%20Filter.md)). This visibility map is very dense so the number of read operations is (hopefully) less than fetching `xmin`/`xmax` from the table. However, the visibility map does not always give a definite answer: the visibility map either states that that the row is known to be visible, or that the visibility is not known

# References

- [Hironobu SUZUKI @ InterDB: The Internals of PostgreSQL](https://www.interdb.jp/pg/index.html)
- [Implementing MVCC and major SQL transaction isolation levels \| notes.eatonphil.com](https://notes.eatonphil.com/2024-05-16-mvcc.html)
- [The Part of PostgreSQL We Hate the Most // Blog // Andy Pavlo - Carnegie Mellon University](https://www.cs.cmu.edu/~pavlo/blog/2023/04/the-part-of-postgresql-we-hate-the-most.html)
- [Why Uber Engineering Switched from Postgres to MySQL \| Uber Blog](https://www.uber.com/en-TR/blog/postgres-to-mysql-migration/)
- [Improving join-performance of SQL databases](https://use-the-index-luke.com/sql/join)
- [#19 - Multi-Version Concurrency Control (CMU Intro to Database Systems) - YouTube](https://youtu.be/niLwbfE3V9Q?si=MHtevskboGgkdB9x)
- [CMU Advanced Database Systems - 03 Multi-Version Concurrency Control Design Decisions (Spring 2019) - YouTube](https://youtu.be/BShOt5gYiPs?si=IaVr3niyQIsATXdP)
- [CMU Advanced Database Systems - 05 MVCC Garbage Collection (Spring 2019) - YouTube](https://youtu.be/AD1HW9mLlrg?si=ohxG66y3lIKw6A8U)
