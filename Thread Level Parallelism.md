---
tags:
  - Concurrency
  - OS-Architecture
---

# Threads

There are two types of threads:

- **Physical threads**: Threads that are executing on a hardware core
- **Logical threads**: Thread abstractions managed by the OS. Multiple logical threads can map to a single physical thread using **time sharing**

# Hyper-threading

Hyper-threading is a form of **SMT** that allows one **physical core** to appear as multiple **logical cores**

To achieve this, each core has multiple copies of registers, `PC`, but a single **ALU**

# Context Switch

A [context switch](Context%20Switch.md) involves saving and restoring the process control block (**PCB**), including:

- Registers
- [Stack pointer](Call%20Stack.md)
- Program counter
- [Page table info](Virtual%20Memory.md)

Imagine a single-threaded program being relocated from one core to another; from a programmer's perspective, the program should behave as if nothing changed (program order). Because registers are saved in [RAM](Main%20Memory.md), [cache coherency](Cache%20Coherency.md) ensures that another core will see all changes (even to registers), so the program behaves as expected

# References

- [[CS61C FA20] Lecture 33.4 - Thread-Level Parallelism I: Multithreading - YouTube](https://youtu.be/_ZL8Z81yI5w?si=wNehxisw9x353Vvl)
- [Context switch - Wikipedia](https://en.wikipedia.org/wiki/Context_switch)
