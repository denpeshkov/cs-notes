---
tags:
  - DB
  - TODO
---

# Output Format

## Index and Table Access

- `Seq Scan`: Scans the entire relation (table) as stored on disk
- `Index Scan`: Performs a B-tree traversal, walks through the leaf nodes to find all matching entries, and fetches the corresponding table data. It fetches one tuple-pointer at a time from the index, and immediately visits that tuple in the table
- `Index Only Scan`: Performs a B-tree traversal and walks through the leaf nodes to find all matching entries. There is no table access needed because the index has all columns to satisfy the query (exception: MVCC visibility information)
- `Bitmap Index Scan`, `Bitmap Heap Scan`, `Recheck Cond`: Fetches all the tuple-pointers from the index in one go, sorts them using an in-memory `bitmap` data structure, and then visits the table tuples in physical tuple-location order

## Join Operations

#TODO 

# References

- [How the PostgeSQL database executes queries](https://use-the-index-luke.com/sql/explain-plan/postgresql)
