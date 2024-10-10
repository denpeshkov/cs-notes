---
tags:
  - Go
  - DB
---

# Using `DB.Exec`

When executing `DB.Exec` without placeholder parameters, a prepared statement will not be utilized  
The connection will be released to the pool automatically

When executing `DB.Exec` with placeholder parameters, a new prepared statement will be used under the hood for each invocation  
The connection will be released to the pool automatically

# Using `DB.Query`

When executing `DB.Query` without placeholder parameters, a prepared statement will not be used  
The connection will be released to the pool automatically when the returned rows are iterated or after the call to `Rows.Close()`

When executing `DB.Query` with placeholder parameters, a new prepared statement will be used under the hood for each invocation  
The connection will be released to the pool automatically when the returned rows are iterated or after the call to `Rows.Close()`

# Using `DB.Prepare`

When executing `DB.Prepare`, a new prepared statement will be used once for all the invocations in the given connection  
The connection will be released to the pool after the call to `Stmt.Close()`.

# References

- [Query vs Exec vs Prepare in Go. Things to know before calling a Query… | by alok sinha | Medium](https://aloksinhanov.medium.com/query-vs-exec-vs-prepare-in-golang-e7c49212c36c)
