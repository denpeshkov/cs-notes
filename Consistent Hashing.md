---
tags:
  - Data-Structures-Algorithms
  - Distributed-Systems
---

# Traditional Hashing

Traditional hashing, for example using modulo of the number of nodes, maps data to a fixed set of nodes. Adding or removing a node may cause significant redistribution of keys, as they will be hashed to a different node

# Consistent Hashing

Consistent Hashing solves this problem by using the hash function that is independent of the number of nodes. It uses a huge constant hash space (ring) e.g. `[0, 2^128 - 1]` and the nodes and items both map to one of the slots on the ring

Instead of directly associating items with nodes, it uses a relative association: an item is associated to the node immediately to its right in the ring

Consistent Hashing on average requires only $k/n$ units of data to be migrated during scale up and down; where $k$ is the total number of keys and $n$ is the number of nodes in the system

# Implementation

We are using two sorted arrays:

- A `nodes` array holds the nodes
- A `keys` array holds the positions of the nodes in the hash space

Each node `nodes[i]` is present at position `keys[i]` in the hash space. Both arrays are sorted based on the `keys` array to enable [binary search](Binary%20Search.md) lookup

## Hash Function

We define `total_slots` as the size of entire hash space (ring), typically of the order `2^256` and the hash function could be implemented by taking SHA-256 modulo `total_slots`

## Adding a Node

Adding a new node affects only the items hashed to the position directly to its left, which were previously associated with the node to its right

To add a new node:

1. Calculate the key (position in the hash space) for the node using the hash function
2. Find the index of the smallest key in `keys` that is greater than the computed key using [binary search](Binary%20Search.md)
3. Insert the computed key at the found index in `keys`
4. Insert the new node at the same index in `nodes`

## Removing a Node

When a node is removed, only its associated items are affected, minimizing data migration and changes to the mapping

To remove a node:

1. Calculate the key (position in the hash space) for the node using the hash function
2. Find the index of the key in `keys` using [binary search](Binary%20Search.md)
3. Remove the computed key at the found index in `keys`
4. Remove the node at the same index in `nodes`

## Associating an Item to a Node

1. Calculate the key (position in the hash space) for the item using the hash function
2. Return the index of the smallest key in `keys` that is greater than the computed key using [binary search](Binary%20Search.md). If no such key exists, wrap around and return the 0th index

# Virtual Nodes

#TODO 

# References

- [Consistent Hashing: Algorithmic Tradeoffs \| by Damian Gryski \| Medium](https://dgryski.medium.com/consistent-hashing-algorithmic-tradeoffs-ef6b8e2fcae8) #TODO
- [Consistent Hashing - explanation and implementation](https://arpitbhayani.me/blogs/consistent-hashing/)
- [F2023 #21 - Intro to Distributed Databases (CMU Intro to Database Systems) - YouTube](https://youtu.be/fyDeu_5svbg?si=TSCv9zMYKzDk6shs)
- [GitHub - golang/groupcache: groupcache is a caching and cache-filling library, intended as a replacement for memcached in many cases.](https://github.com/golang/groupcache)
- [Database Internals: A Deep Dive into How Distributed Data Systems Work. Alex Petrov](References.md#Database%20Internals%20A%20Deep%20Dive%20into%20How%20Distributed%20Data%20Systems%20Work.%20Alex%20Petrov)
- [The Simple Magic of Consistent Hashing \| Mathias Meyer](https://www.paperplanes.de/2011/12/9/the-magic-of-consistent-hashing.html) #TODO
- [Consistent Hashing: Beyond the basics \| by Omar Elgabry \| OmarElgabry's Blog \| Medium](https://medium.com/omarelgabrys-blog/consistent-hashing-beyond-the-basics-525304a12ba) #TODO
- [Consistent Hashing \| Algorithms You Should Know #1 - YouTube](https://youtu.be/UF9Iqmg94tk?si=ZTxCqwhviqnWBYPZ) #TODO 
