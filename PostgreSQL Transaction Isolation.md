---
tags:
  - DB
---

# Isolation Levels

| Isolation level                     | Dirty Write | Dirty Read | Predicate-Many-Preceders | Lost Update | Read Skew | Write Skew |
| :---------------------------------- | :---------: | :--------: | :----------------------: | :---------: | :-------: | :--------: |
| `READ UNCOMMITTED`,`READ COMMITTED` |      ✓      |     ✓      |            —             |      —      |     —     |     —      |
| `REPEATABLE READ`                   |      ✓      |     ✓      |            ✓             |      ✓      |     ✓     |     —      |
| `SERIALIZABLE`                      |      ✓      |     ✓      |            ✓             |      ✓      |     ✓     |     ✓      |

# Read Uncommitted and Read Committed

In PostgreSQL's `READ UNCOMMITTED` mode behaves the same as `READ COMMITTED`

The transaction obtains a snapshot each time an SQL command is executed

A `SELECT` query (without a `FOR UPDATE/SHARE` clause) sees only data committed before the query began. However, `SELECT` does see the effects of previous uncommitted updates within its own transaction

An `UPDATE`, `DELETE`, `SELECT FOR UPDATE`, `SELECT FOR SHARE` commands search for target rows just like `SELECT`, only finding rows committed as of the command's start time. However, if a target row has already been modified by another transaction, the command waits for that transaction to finish. If the first transaction rolls back, its changes are discarded, and the second transaction can proceed with the originally found row. If the first transaction commits, the second transaction ignores the row if it was deleted; otherwise, it attempts to update the latest version. The `WHERE` condition is re-evaluated to ensure the updated row still matches before proceeding. For `SELECT FOR UPDATE/SHARE` this means it is the updated version of the row that is locked and returned to the client

An `INSERT` with an `ON CONFLICT DO UPDATE` clause behaves similarly. If a conflict is detected because another transaction has modified a row (even if those changes are not yet visible to the `INSERT`), the `UPDATE` clause will be applied to that row

An `INSERT` with an `ON CONFLICT DO NOTHING` clause might skip inserting a row due to the outcome of another transaction whose effects are not visible to the `INSERT` snapshot

Because of the above rules, an updating command might see an inconsistent snapshot: it can see the effects of concurrent updating commands on the same rows it is trying to update, but it does not see effects of those commands on other rows

# Repeatable Read

The transaction obtains a snapshot only at the start of the transaction

A `SELECT` query (without a `FOR UPDATE/SHARE` clause) sees only data committed before the transaction began. However, `SELECT` does see the effects of previous uncommitted updates within its own transaction

An `UPDATE`, `DELETE`, `SELECT FOR UPDATE/SHARE` commands search for target rows just like `SELECT`, only finding rows committed as of the transaction start time. However, if a target row has already been modified by another transaction, the command waits for that transaction to finish. If the first transaction rolls back, its changes are discarded, and the second transaction can proceed with the originally found row. If the first transaction commits (and actually updated or deleted the row, not just locked it) then the second transaction will be rolled back with error message, because a repeatable read transaction cannot modify or lock rows changed by other transactions after the transaction began

# Serializable

It works exactly the same as `REPEATABLE READ` except that it detects conditions that could cause concurrent transactions to produce outcomes inconsistent with any serial execution

To guarantee serializability PostgreSQL uses **predicate locking**. These locks do not cause any blocking, they are used to identify dependencies among concurrent `SERIALIZABLE` transactions which can lead to serialization anomalies

# References

- [PostgreSQL: Documentation: 17: 13.2. Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
- [Hironobu SUZUKI @ InterDB: The Internals of PostgreSQL](https://www.interdb.jp/pg/index.html)
- [PostgreSQL 14 Internals. Egor Rogov](References.md#PostgreSQL%2014%20Internals.%20Egor%20Rogov)
- [Designing Data-Intensive Applications: The Big Ideas Behind Reliable, Scalable, and Maintainable Systems. Martin Kleppmann](References.md#Designing%20Data-Intensive%20Applications%20The%20Big%20Ideas%20Behind%20Reliable,%20Scalable,%20and%20Maintainable%20Systems.%20Martin%20Kleppmann)
- [GitHub - ept/hermitage: What are the differences between the transaction isolation levels in databases? This is a suite of test cases which differentiate isolation levels.](https://github.com/ept/hermitage)
