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

1. **Heap tuple** - A record data itself. Heap tuples are stacked in order from the bottom of the page
2. **Line pointer** - A pointer to each heap tuple. Line pointers form a simple array that plays the role of an index to the tuples. Each index is numbered sequentially from 1, and called **offset number**. When a new tuple is added to the page, a new line pointer is also pushed onto the array to point to the new tuple
3. **Header info** – A header contains general information about the page. The major variables:
	1. `pd_lsn` – Stores the LSN of `XLOG` record written by the last change of this page
	2. `pd_lower`, `pd_upper` – `pd_lower` points to the end of line pointers, and `pd_upper` to the beginning of the newest heap tuple

To identify a tuple within the table, a **tuple identifier** `tid`  `ctid ????? ` is used internally. A TID comprises a pair of values: the **block number** of the page that contains the tuple, and the **offset number** of the line pointer that points to the tuple

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

A backend process, also called a `postgres` process, is started by the postgres server process and handles all queries issued by one connected client. It communicates with the client using a single TCP connection and terminates when the client disconnects

Because each new client connection spawns a backend process, conection pools suchs as [PgBouncer](PgBouncer%20Usage%20and%20Best%20Practices.md) are used

### Background Processes

| process             | description                                                                                            |
| ------------------- | ------------------------------------------------------------------------------------------------------ |
| background writer   | This process writes dirty pages on the shared buffer pool to a disk on a regular basis gradually       |
| checkpointer        | This process performs the checkpoint process                                                           |
| autovacuum launcher | This process periodically postgres server to create the autovacuum workers for the VACUUM process      |
| WAL writer          | This process writes and flushes the WAL data on the WAL buffer to disk periodically                    |
| WAL Summarizer      | This process tracks changes to all database blocks and writes these modifications to WAL summary files |
| archiver            | This process executes archiving logging                                                                |

## Memory Architecture

![PostgreSQL memory architecture|300](PostgreSQL%20memory%20architecture.png)

### Local Memory Area

Each backend process allocates a local memory area for query processing, divided into several sub-areas

| sub-area               | description                                                                                                                         | reference                                                |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| `work_mem`             | The executor uses this area for sorting tuples, and for joining tables by [Merge Join](#Merge%20Join) and [Hash Join](#Hash%20Join) | [Chapter 3](https://www.interdb.jp/pg/pgsql03.html)      |
| `maintenance_work_mem` | Maintenance operations (e.g., [`VACUUM`](#Vacuum%20Processing), `REINDEX`) use this area                                            | [Section 6.1](https://www.interdb.jp/pg/pgsql06/01.html) |
| `temp_buffers`         | The executor uses this area for storing temporary tables                                                                            |                                                          |

### Shared Memory Area

A shared memory area is allocated by a PostgreSQL server when it starts up. This area is also divided into several fixed-sized sub-areas

| sub-area           | description                                                                                           | reference                                                |
| ------------------ | ----------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| shared buffer pool | PostgreSQL loads pages within tables and indexes from a disk to this area, and operates them directly | [Chapter 8](https://www.interdb.jp/pg/pgsql08.html)      |
| WAL buffer         | The WAL buffer is a buffering area of the WAL data before writing to a disk                           | [Chapter 9](https://www.interdb.jp/pg/pgsql09.html)      |
| commit log         | The [`clog`](#Commit%20Log%20(clog)) keeps the states of all transactions for the MVCC                | [Section 5.4](https://www.interdb.jp/pg/pgsql05/04.html) |

# Query Processing

## Join Operations

### Nested Loop Join

Nested loop join can be used in all join conditions

### Merge Join

#TODO

### Hash Join

#TODO

# MVCC

## Transaction ID

At the start of every transaction, the transaction manager assigns a unique identifier known as a transaction ID (`txid`)

## Tuple Structure

![PostgreSQL tuple structure|600](PostgreSQL%20tuple%20structure.png)

- `t_xmin` holds the `txid` of the transaction that inserted this tuple.
- `t_xmax` holds the `txid` of the transaction that deleted or updated this tuple. If this tuple has not been deleted or updated, `t_xmax` is set to 0, which means `INVALID`
- `t_ctid` holds the tuple identifier (`tid`) that points to itself or a new tuple. When this tuple is updated, the `t_ctid` of this tuple points to the new tuple; otherwise, the `t_ctid` points to itself

## Inserting, Deleting and Updating Tuples

### Insertion

A new tuple is inserted directly into a page of the target table

![PostgreSQL tuple insertion|600](PostgreSQL%20tuple%20insertion.png)

- `t_xmin` is set to the `txid` of the inserting transaction
- `t_xmax` is set to 0 because this tuple has not been deleted or updated
- `t_ctid` is set to `(0,1)`, which points to itself, because this is the latest tuple

### Deletion

The target tuple is deleted **logically**

![PostgreSQL tuple deletion|600](PostgreSQL%20tuple%20deletion.png)

- `t_xmax` is set to the `txid` of the deleting transaction

If this transaction is committed, tuple is no longer required, it becomes a **dead tuple**. Dead tuples are eventually removed by [VACUUM processing](#Vacuum%20Processing)

### Update

Logically a delete followed by update

![PostgreSQL tuple update|600](PostgreSQL%20tuple%20update.png)

As with the delete operation:

- If `txid` 100 is **committed**, Tuple_1 and Tuple_2 will be dead tuples
- If `txid` 100 is **aborted**, Tuple_2 and Tuple_3 will be dead tuples

### Free Space Map (FSM)

When inserting a heap or an index tuple, PostgreSQL uses the Free Space Map (FSM) of the corresponding table or index to select the page which can be inserted into

All tables and indexes have respective FSMs. Each FSM stores the information about the free space of each page within the corresponding table or index file

FSMs are loaded into [shared memory](#Shared%20Memory%20Area) if necessary. FSM are updated during the [VACUUM processing](#Vacuum%20Processing)

## Commit Log (clog)

Commit Log (`clog`) holds the statuses of transactions. It is stored in a [shared memory](#Shared%20Memory%20Area) and is used throughout transaction processing

![PostgreSQL clog|600](PostgreSQL%20clog.png)

PostgreSQL defines four transaction states: `IN_PROGRESS`, `COMMITTED`, `ABORTED`, `SUB_COMMITTED`

It logically forms an array, where the indices correspond to the respective `txid`, and each item holds the status of the corresponding `txid`

When the current `txid` advances and the `clog` can no longer store it, a new page is appended

When the status of a transaction is needed, internal functions are invoked to read the `clog` and return the status of the requested transaction

### Maintenance of the Clog

When PostgreSQL shuts down or whenever the checkpoint process runs, the data of the `clog` are written into files stored in the `pg_xact` subdirectory. When PostgreSQL starts up, the data stored in the `pg_xact` files are loaded to initialize the `clog`

[VACUUM processing](#Vacuum%20Processing) regularly removes unnecessary old data, both the `clog` pages and files

## Transaction Snapshot

A transaction snapshot stores information about which `txid`s are active at a certain point in time for an individual transaction. An active transaction means it is in progress or has not yet started

Snapshot textual representation is `xmin:xmax:xip_list`:

- `xmin` - Lowest `txid` that is still active. All `txid`s less than `xmin` are either committed and visible, or rolled back and dead
- `xmax` - One past the highest completed `txid`. All `txid`s greater than or equal to `xmax` had not yet completed as of the time of the snapshot, and thus are invisible
- `xip_list` - Transactions in progress at the time of the snapshot. A `txid` that greater than or equal to `xmin` and less than `xmax` and not in this list was already completed at the time of the snapshot, and thus is either visible or dead according to its commit status

Transaction snapshots are provided by the transaction manager:

- In the `READ COMMITTED` isolation level, the transaction obtains a snapshot whenever an SQL command is executed
- In `REPEATABLE READ` and `SERIALIZABLE` isolation levels, the transaction only gets a snapshot when the first SQL command is executed

The obtained transaction snapshot is used for a [visibility check](#Visibility%20Check) of tuples

## Visibility Check

#TODO

# Vacuum Processing

Vacuum processing performs the following tasks for specified tables or all tables in the database:

1. Removing dead tuples
	1. Remove dead tuples and defragment live tuples for each page
	2. Remove index tuples that point to dead tuples
2. Update the [FSM](#Free%20Space%20Map%20File) and [VM](#Visibility%20Map%20(VM)) of processed tables
3. Remove unnecessary parts of the [clog](#Commit%20Log%20(clog)) if possible
4. Update statistics
5. Freezing old `txids`

Its main task is removing dead tuples. To remove dead tuples, vacuum processing provides two modes:

- `VACUUM` removes dead tuples for each page of the table file, and other transactions can read the table while is is running
- `VACUUM FULL` removes dead tuples and defragments live tuples in the whole file, and other transactions cannot access tables while it is running

## Visibility Map (VM)

Each table has a Visibility Map to keep track of which pages contain only tuples that are known to be visible to all active transactions

`VACUUM` can skip a page that does not have dead tuples by using the corresponding VM

## Vacuum Full

The `VACUUM` is not sufficient, it cannot reduce the size of a table even if many dead tuples are removed

![PostgreSQL VACUUM|300](PostgreSQL%20VACUUM.png)

The dead tuples are removed, but the table size is not reduced. This is both a waste of disk space and has a negative impact on database performance

To deal with this situation, PostgreSQL provides the `VACUUM FULL` mode

![PostgreSQL VACUUM FULL|600](PostgreSQL%20VACUUM%20FULL.png)

1. Create a new table file, acquiring the exclusive lock for the table. Nobody can access the table when `VACUUM FULL` is processing
2. Copies live tuples from the old table file to the new table
3. Remove the old file, rebuild all associated table indexes, updates both the [FSM](#Free%20Space%20Map%20(FSM)) and [VM](#Visibility%20Map%20(VM)) of this table, and updates associated statistics and system catalogs

# HOT and Index-Only Scans

## Heap Only Tuples (HOT)

Without Heap Only Tuple updates, every version of a row in a version chain has its own index entry, even if all indexed columns are the same. As a result, every row update triggers updates of all indexes on the table

The HOT updates optimize this process in two ways:

1. When a row is updated, a new index entry is not created for the new row version. There is a single index entry for the entire version chain on the heap page, pointing to the line pointer of the head of the chain
2. Intermediate row versions, except for the oldest and newest, can be removed during normal operations (`SELECT`, `UPDATE`, `INSERT`, `DELETE`), instead of requiring periodic VACUUM operations. This process is called pruning, or single page vacuuming

The HOT optimization is available when two conditions are met:

1. An update doesn't modify any indexed columns of the row
2. An updated row (a new row version) is stored in the same table page as the old row version

Row versions that are not referenced from any index are tagged with the `HEAP_ONLY_TUPLE` bit in tuple header. If a version is included into the HOT chain, it is tagged with the `HEAP_HOT_UPDATED` bit and it's `t_ctid` field links to the newer version

The index search follows the HOT chain as long as the `HEAP_HOT_UPDATED` flag is set, checking each row version for visibility. Since the HOT chain lies within a single page, this requires no additional page fetches

If an update changes any indexed column, or if there is not room on the same page for the new tuple, then the HOT chain ends: the last member has a regular `t_ctid` link to the next version and is not marked `HEAP_HOT_UPDATED`. If further updates occur, the next version could become the root of a new HOT chain

### Pruning

Pruning process reduces the length of the HOT version chain, by collapsing out line pointers for intermediate dead tuples

Because when using HOT there is only one index entry for the chain, when reclaiming space occupied by dead tuples PostgreSQL can't delete the line pointer of the head of the version chain, as it's still referenced by the index, and we still need to be able to traverse the version chain. HOT handles this by turning this line pointer into a redirecting line pointer, which points to the line pointer of the never version tuple

Since HOT uses only one index entry for the entire version chain, PostgreSQL cannot delete the line pointer of the head of the chain when reclaiming space from dead tuples, as it is still referenced by the index and needed for traversing the version chain. To resolve this problem PostgreSQL redirects the line pointer that points to the old tuple to the line pointer that points to the new tuple

## Index-Only Scans

To reduce the I/O cost, index-only scans directly use the index key without accessing the corresponding table pages when all of the target entries of the `SELECT` statement are included in the index key

The index tuples do not have any information about transactions, such as the `t_xmin` and `t_xmax` of the heap tuples

Therefore, PostgreSQL has to access the table data to check the visibility of the data in the index tuples. This is like putting the cart before the horse

To avoid this dilemma, PostgreSQL uses the [VM](#Visibility%20Map%20(VM)) of the target table. If all tuples stored in a page are visible, PostgreSQL uses the key of the index tuple and does not access the table page that is pointed at from the index tuple to check its visibility. Otherwise, PostgreSQL reads the table tuple that is pointed at from the index tuple and checks the visibility of the tuple

Whenever PostgreSQL is looking at an index entry, it cannot tell whether or not that entry is visible to the current transaction. It could be a deleted entry or an entry that has not yet been committed. The canonical way to find out is to look into the table and check the `xmin`/`xmax` values

A consequence is that there is no such thing as an index-only scan in PostgreSQL. No matter how many columns you put into an index, PostgreSQL will always need to check the visibility, which is not available in the index

Yet there is an `Index Only Scan` operation in PostgreSQL—but that still needs to check the visibility of each row version by accessing data outside the index. Instead of going to the table, the `Index Only Scan` first checks the so-called visibility map (it's a [Bloom Filter](Bloom%20Filter.md)). This visibility map is very dense so the number of read operations is (hopefully) less than fetching `xmin`/`xmax` from the table. However, the visibility map does not always give a definite answer: the visibility map either states that that the row is known to be visible, or that the visibility is not known

# Write Ahead Log (WAL)

#todo

# References

- [Hironobu SUZUKI @ InterDB: The Internals of PostgreSQL](https://www.interdb.jp/pg/index.html)
- [Implementing MVCC and major SQL transaction isolation levels \| notes.eatonphil.com](https://notes.eatonphil.com/2024-05-16-mvcc.html)
- [The Part of PostgreSQL We Hate the Most // Blog // Andy Pavlo - Carnegie Mellon University](https://www.cs.cmu.edu/~pavlo/blog/2023/04/the-part-of-postgresql-we-hate-the-most.html)
- [Why Uber Engineering Switched from Postgres to MySQL \| Uber Blog](https://www.uber.com/en-TR/blog/postgres-to-mysql-migration/)
- [Improving join-performance of SQL databases](https://use-the-index-luke.com/sql/join)
- [15-721 Advanced Database Systems (Spring 2019) - YouTube](https://youtube.com/playlist?list=PLSE8ODhjZXja7K1hjZ01UTVDnGQdx5v5U&si=gvzy1RXaNBCxoSPf)
- [CMU Intro to Database Systems (15-445/645 - Fall 2024) - YouTube](https://youtube.com/playlist?list=PLSE8ODhjZXjYDBpQnSymaectKjxCy6BYq&si=C2YkUp1SEfntfPDg)
- [Database System Concepts (7th ed). Abraham Silberschatz, Henry F. Korth, S. Sudarshan](References.md#Database%20System%20Concepts%20(7th%20ed).%20Abraham%20Silberschatz,%20Henry%20F.%20Korth,%20S.%20Sudarshan)
- [HOT updates in PostgreSQL for better performance \| CYBERTEC PostgreSQL \| Services & Support](https://www.cybertec-postgresql.com/en/hot-updates-in-postgresql-for-better-performance/)
- [How Postgres Makes Transactions Atomic — brandur.org](https://brandur.org/postgres-atomicity)
- [README.HOT · postgres/postgres · GitHub](https://github.com/postgres/postgres/blob/master/src/backend/access/heap/README.HOT)
- [PostgreSQL: Documentation: 17: 65.7. Heap-Only Tuples (HOT)](https://www.postgresql.org/docs/current/storage-hot.html)
- [PostgreSQL 14 Internals. Egor Rogov](References.md#PostgreSQL%2014%20Internals.%20Egor%20Rogov)
