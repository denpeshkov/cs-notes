---
tags: OS_Arch Concurrency
---

# Overview

ABA problem occurs during synchronization, when a location is read twice, has the same value for both reads, and the read value being the same twice is used to conclude that nothing has happened in the interim

The problem arises if this new value, which looks exactly like the old value, has a different meaning: for instance, it could be a recycled address, or a wrapped version counter

# Lock-Free Data Structures

#todo

# Workarounds

## Tagged State Reference

Add extra "tag" bits to the value. For example, when using [Double-Width CAS](Atomic%20Instructions.md#Double-Width%20CAS%20(DW-CAS)) the second half is used to hold a counter. The compare part of the operation compares the previously read value of the pointer **and** the counter, with the current pointer and counter. If they match, the swap occurs - the new value is written - but the new value has an incremented counter

Tagged state references are also used in [transactional memory](Transactional%20Memory.md)

## Alternate Instructions

Architectures that support [LL/SC](Atomic%20Instructions.md#Load-Link/Store-Conditional) are immune to the ABA problem, since the store to the location is performed only when there are no other stores of the indicated location and these instructions provide atomicity using the address rather than the value

# References

- [ABA problem - Wikipedia](https://en.wikipedia.org/wiki/ABA_problem)
