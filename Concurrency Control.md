---
tags:
  - DB
  - TODO
---

# Transaction Isolation Levels

## Read Uncommitted

Not supported. Implemented as a Read Committed

## Read Committed

Implemented using MVCC. A snapshot is **recorded** at the start of each SQL statement

## Repeatable Read (Snapshot Isolation)

Implemented using MVCC. A **snapshot** is *recorded* at the start of the transaction

## Serializable

Implemented using MVCC. A snapshot is recorded at the start of the transaction

# Snapshot Isolation (MVCC)

Snapshot Isolation does not ensure serializability

A key principle of snapshot isolation is readers never block writers, and writers never block readers

A transaction is given a "snapshot" of the DB at the time when it begins its execution. The data values in the snapshot consist only of values written by committed transactions. Each transaction that updates a data item creates a new version of the data item

Transactions have two timestamps:

1. `StartTS(Ti)` - the time at which transaction `Ti` started
2. `CommitTS(Ti)` - the time when the transaction `Ti` requested validation, initially set to ∞

A version of the data item has two timestamps:

1. `CommitTS(Ti)` - the time at which the version was created by the transaction `Ti`
2. The timestamp when the next version was created (an invalidation timestamp for the current version). The current version has the invalidation timestamp set to ∞

## Reads

When a transaction `Ti` reads a data item, the latest version of the data item whose timestamp is `<= StartTS(Ti)` is returned to `Ti`

## Writes (First Updater Wins)

When a transaction `Ti` wants to update an item, it requests a write lock:

- If no other concurrent transaction holds the lock:
	- If another concurrent transaction has already updated the item, `Ti` aborts.
	- Otherwise, `Ti` proceeds and may commit.
- If another concurrent transaction `Tj` holds the lock:
	- `Ti` waits for `Tj` to finish [^1].
		- If `Tj` aborts, `Ti` acquires the released lock and rechecks for concurrent updates as described above
		- If `Tj` commits, `Ti` aborts.

[^1]: As an alternative to waiting to see if the first updater `Tj` aborts, a subsequent updater `Ti` can be aborted as soon as it finds that the write lock it wishes to obtain is held by `Tj`

Locks are released when the transaction commits or aborts

Integrity constraints, such as primary-key and foreign-key constraints, are validated on the current state of the database, rather than on the snapshot, as part of validation at the time of commit

## Ensuring Serializability

A developer can guard against **certain** snapshot anomalies by using a `SELECT … FOR UPDATE` query

# Snapshot Isolation (MVCC) CMU

Transactions have two timestamps:

1. Read timestamp - the time at which transaction started. Equal to the commit timestamp of the most recently committed transaction
2. Commit timestamp - the time when the transaction committed

A version of the tuple has one timestamp:

1. The commit timestamp of the transaction that created this version. The current version's timestamp is ∞, meaning this tuple is modified by a transaction that is not yet committed/aborted

## Watermark

The watermark is the lowest read timestamp among all transactions that have not yet committed or aborted. If there's no such a transaction, the watermark is the latest commit timestamp.

## Reads

When a transaction reads a tuple, the latest version of the tuple with timestamp less than or equal to transaction's read timestamp is returned

## Commit

1. Take the commit mutex.
2. Obtain a commit timestamp
3. Iterate through every tuple that has been modified by this transaction and set the timestamp of the base tuples to the commit timestamp.
4. Set the transaction to the `COMMITTED` state.
5. Update the commit timestamp of the transaction.
6. Update `last_committed_ts_` (you can do the `.fetch_add(1)` here).

## Writes

If a tuple is being modified by an uncommitted transaction, no other transactions are allowed to modify it. If they do, there will be a write-write conflict and the transaction conflicting with a previous transaction should be aborted

Transaction should be aborted with a write-write conflict, if there is a newer update that happens after the transaction read timestamp

## GC

Any version of a tuple with timestamp lower than the watermark can be GC-ed

Any version of a tuple of an aborted transaction can be GC-ed

## Index

#TODO

# Serializable Snapshot Isolation (SSI)

Serializable Snapshot Isolation ensures serializability

# References

- [Concurrency Control (Part 1) - YouTube](https://youtu.be/Z1rI4xtkbWw?si=EH-KIY0WSta3qn6E)
- [Concurrency Control (Part 2) - YouTube](https://youtu.be/B1Ge7QmyK_w?si=IxVV3vz2VROVY4tZ)
- [Implementing MVCC and major SQL transaction isolation levels \| notes.eatonphil.com](https://notes.eatonphil.com/2024-05-16-mvcc.html)
- [A Generalized MVCC Store (Part 1) - YouTube](https://youtu.be/wMm03mGJ4jw?si=RYKcKSwr5zs2RCq5)
- [A Generalized MVCC Store (Part 2) - YouTube](https://youtu.be/WUhalZgliW0?si=qEZ3hrcjgIZXAbeu)
- [Database System Concepts (7th ed). Abraham Silberschatz, Henry F. Korth, S. Sudarshan](References.md#Database%20System%20Concepts%20(7th%20ed).%20Abraham%20Silberschatz,%20Henry%20F.%20Korth,%20S.%20Sudarshan)
- [#18 - Optimistic Concurrency Control ✸ Weaviate Database Talk (CMU Intro to Database Systems) - YouTube](https://youtu.be/-TA0QIwFUnU?si=6jMuiugRU0-mABup)
- [CMU Advanced Database Systems - 02 Transaction Models & In-Memory Concurrency Control (Spring 2019) - YouTube](https://youtu.be/DyYO3yaA1wc?si=uVP6h96WLAsdKsTu)
- [CMU Advanced Database Systems - 03 Multi-Version Concurrency Control Design Decisions (Spring 2019) - YouTube](https://youtu.be/BShOt5gYiPs?si=XpeFoaag0NkbbnAs)
- [5. Concurrency Control :: Hironobu SUZUKI @ InterDB](https://www.interdb.jp/pg/pgsql05.html)
- [Project #4 - Concurrency Control \| CMU 15-445/645 :: Intro to Database Systems (Fall 2024)](https://15445.courses.cs.cmu.edu/fall2024/project4/)
- [PostgreSQL 14 Internals. Egor Rogov](References.md#PostgreSQL%2014%20Internals.%20Egor%20Rogov)

# References
