---
tags:
  - DB
  - TODO
---

# Overview

Two main components:

1. Memtable - an in-memory data structure that stores key-value pairs in sorted order (e.g. [trie](Trie.md), [skip list](Skip%20List.md)). It is basically a buffer
2. SSTable - a segment containing key-value pair sorted by keys which is stored on disk. Storing tuples sorted by key allows for fast merging the segments using N-way merge algorithms

To avoid scanning all the files when searching for a key, we need an index. Because segments are sorted by the key, index can be a sparsed-index which stores only a subset of the keys

To achieve fault-tolerance every modification made to the Memtable is written to the WAL log, which is just an unsorted write-only log, before being written to the Memtable

LSM tree based storage works as follows:

- Updates are recorded in Memtable
- When Memtable size crosses the threshold it is flushed to the disk as a SSTable. This operation is fast because Memtable is already sorted so SSTable can be created by a simple in-order traversal of a Memtable
- SSTables are periodically merged in the background using a [[k-Way Merge]], this process is called compaction. By merging multiple segments into one, we speed up search operations and reduce required on disk space

## Reads

First search Memtable, than SSTables Level 0, than Level 1 …

To speed up lookup [bloom filter](Bloom%20Filter.md) are used to skip files that don't contain a key

## Downsides

- Write amplification, the same tuple is written multiple times: first to memtable, that to level 0, than to level 1 …
- Compaction process is expensive

# References

- [Designing Data-Intensive Applications: The Big Ideas Behind Reliable, Scalable, and Maintainable Systems. Martin Kleppmann](References.md#Designing%20Data-Intensive%20Applications%20The%20Big%20Ideas%20Behind%20Reliable,%20Scalable,%20and%20Maintainable%20Systems.%20Martin%20Kleppmann)
- [#04 - Database Storage: Log-Structured Merge Trees & Tuples (CMU Intro to Database Systems) - YouTube](https://youtu.be/IHtVWGhG0Xg?si=8GTwd05Zwr6Xiblu)
- [Igor Canadi + Mark Callaghan - RocksDB \[The Databaseology Lectures - CMU Fall 2015\] - YouTube](https://youtu.be/jGCv4r8CJEI?si=qIamBMvkM-llfA47)
