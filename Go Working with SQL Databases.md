---
tags:
  - Go
  - DB
---

# Transactions

- Begin a transaction. `DB.BeginTx` begins a new transaction, returning an `sql.Tx` that represents it
- Perform database operations using an `sql.Tx`
- End the transaction with _one_ of the following:
	- Commit using `Tx.Commit`. If it succeeds, all queries and updates are confirmed and applied atomically. If it fails, discard all results from `Query` and `Exec` as invalid
	- Rollback using `Tx.Rollback`. Even if it fails, the transaction remains invalid and won't be committed. A `Tx.Rollback` call after a successful `Tx.Commit` is a no-op

# Prepared Statements

## Using `DB.Exec`

When executing `DB.Exec` without placeholder parameters, a prepared statement will not be utilized  
The connection will be released to the pool automatically

When executing `DB.Exec` with placeholder parameters, a new prepared statement will be used under the hood for each invocation  
The connection will be released to the pool automatically

## Using `DB.Query`

When executing `DB.Query` without placeholder parameters, a prepared statement will not be used  
The connection will be released to the pool automatically when the returned rows are iterated or after the call to `Rows.Close()`

When executing `DB.Query` with placeholder parameters, a new prepared statement will be used under the hood for each invocation  
The connection will be released to the pool automatically when the returned rows are iterated or after the call to `Rows.Close()`

## Using `DB.Prepare`

When executing `DB.Prepare`, a new prepared statement will be used once for all the invocations in the given connection  
The connection will be released to the pool after the call to `Stmt.Close()`

# Using `pgx`

TODO: [PGX Top to Bottom - YouTube](https://youtu.be/sXMSWhcHCf8?si=KSVdJzXAFaCqjEJr)

# References

- [Query vs Exec vs Prepare in Go. Things to know before calling a Query… | by alok sinha | Medium](https://aloksinhanov.medium.com/query-vs-exec-vs-prepare-in-golang-e7c49212c36c)
- [Executing transactions - The Go Programming Language](https://go.dev/doc/database/execute-transactions)
- [PGX Top to Bottom - YouTube](https://youtu.be/sXMSWhcHCf8?si=KSVdJzXAFaCqjEJr)
