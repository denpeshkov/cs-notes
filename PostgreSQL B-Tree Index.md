---
tags:
  - DB
  - TODO
---

# Internals

Indexes are commonly represented by [B+ trees](B+%20tree.md)

Indexes are a separate data structure that maintain a copy of part of the data. That means that if you have multiple indexes, there will be multiple copies of different parts of the data

Moreover, indexes have a pointer back to the row (the primary key). It's necessary for an index to know how to get back to the table because it maintains a copy of part of your data

# Primary Keys

The primary key is one or more columns that can be used as a unique identifier for each row in a table

The primary key determines how data is stored on disk. It is a *clustered index* which means that the data is physically ordered based on the values in the primary key column. This makes primary key lookups incredibly fast, but it also means that every time you insert a new row the tree structure has to be updated

# Secondary Keys

# Foreign Keys

A foreign key is a column or set of columns in a table that references the primary key of another table. This enables related data to be linked together in separate tables

A foreign key constraint is a condition that ensures the referential integrity of the data by enforcing a relationship between the foreign key and the referenced primary key. This means that the constraint will guarantee that all data references are valid and consistent

Foreign keys can exist without constraints to reduce performance penalties due to constrain checks and possible cascading

# What Every Developer Needs to Know

#todo

1. We can't create an index on every column because it will slow down the writes, as an index needs to be updated on every write
2. An index can't be used when filtering on function (e.g. `WHERE YEAR(PURCAHSED_DATE) = 2013`), because a function results can have no correlation with data indexed. PostgreSQL supportQs functional indexes, but many databases not. We can create a generated column based on the result of the function. We can restructure or index `WHERE PURCHASED_DATE BETWEEN …`
3. A full table scan is slower than full index scan, because index contains much fewer data, thus requires to load and walk less data in memory
4. A composite index is searched from left to right (in order of definition) and stops on first missing column, or first inequality operator

# References

- [Introduction to indexes — MySQL for Developers — PlanetScale](https://planetscale.com/learn/courses/mysql-for-developers/indexes/introduction-to-indexes)
- [Things every developer absolutely, positively needs to know about database indexing - Kai Sassnowski - YouTube](https://youtu.be/HubezKbFL7E?si=-JZ1MlNECztgGmLU)
- [07 - Tree Indexes I (CMU Databases Systems / Fall 2019) - YouTube](https://youtu.be/JHZFc4hMGhk?si=m2RTiG1Ys0dT53HF)
- [08 - Tree Indexes II (CMU Databases Systems / Fall 2019) - YouTube](https://youtu.be/Y_h1bH2FJ7M?si=4I6yYMVW-WhufPvF)
- [SQL Indexing and Tuning e-Book for developers: Use The Index, Luke covers Oracle, MySQL, PostgreSQL, SQL Server, ...](https://use-the-index-luke.com)
