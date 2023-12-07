---
tags:
  - Architecture
  - OS
  - Concurrency
---

# Overview

A **memory consistency model** for a shared address space specifies constraints on the order in which memory operations must appear to be performed (to become visible to the CPU) with respect to one another
This includes operations to the same locations or to different locations and by the same process or different processes. In this sense, memory consistency subsumes [[Cache Coherency|coherence]].

# Sequential Consistency (SC)

A multiprocessor is **sequentially consistent** if the result of any execution is the same as if the operations of all the CPUs were executed in some sequential order, and the operations of each individual CPU occur in this sequence in the order specified by its program (**program order**)

Every process appears to issue and complete memory operations one at a time and **atomically** in **program order**. It should appear globally as if one operation in the consistent **interleaved order** executes and completes before the next one begins

Two constraints:

1. **Program order** - Memory operations of a process must appear to become visible (to itself and others) in program order
2. **Write atomicity** - All writes (to *any* location) are seen in the same order by all CPUs

## Synchronization

We still may need synchronization because **SC** works at the granularity of *individual* instructions

Synchronization is needed if we want to preserve **atomicity** (mutual exclusion) across *multiple* memory operations from a process or if we want to enforce constraints on the interleaving across processes

# Keyword `volatile`

The keyword `volatile` prevents the variable from being **register-allocated** or any memory operation on the variable from being reordered with respect to operations before or after it in program order

FIXME #TODO

# References

- [[References#Computer Organization and Design RISC-V Edition The Hardware Software Interface (2nd ed). David A. Patterson, John L. Hennessy]]
- [[References#Parallel Computer Architecture A Hardware/Software Approach (1st ed). David Culler, Jaswinder Pal Singh, Anoop Gupta]]
- [volatile (computer programming) - Wikipedia](https://en.wikipedia.org/wiki/Volatile_(computer_programming))
- [research!rsc: Memory Models](https://research.swtch.com/mm)
