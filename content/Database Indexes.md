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

# References

- [Introduction to indexes — MySQL for Developers — PlanetScale](https://planetscale.com/learn/courses/mysql-for-developers/indexes/introduction-to-indexes)
- [Learning SQL - Generate, Manipulate, and Retrieve Data (3rd ed). Alan Beaulieu](References.md#Learning%20SQL%20-%20Generate,%20Manipulate,%20and%20Retrieve%20Data%20(3rd%20ed).%20Alan%20Beaulieu)
