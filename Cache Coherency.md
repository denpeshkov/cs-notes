---
tags:
  - OS_Arch
  - Concurrency
  - Distributed_Systems
---

# The Cache Coherence Problem

## Multiprocessors

We aim for the results of a program using multiple processes to be consistent whether the processes run on different CPUs or on the same CPU (interleaved or multiprogrammed)

However, in systems with multiple [caches](Cache%20Memory.md), individual caches can have a different view of the data at the same addresses

Cache coherence is implemented in hardware because a software implementation is too slow

## Uniprocessors

Cache coherence problems also arise in uniprocessors when [I/O operations](Input-Output%20Devices.md) occur

Most I/O occurs via [DMA](Input-Output%20Devices.md#Direct%20Memory%20Access%20(DMA)) that moves data between memory and the peripheral component without involving the CPU

- Writes by DMA may not be visible to the CPU
- DMA may read a stale value if [write-back](Cache%20Memory.md#Write-Back) caches are used

Several software solutions exist:

1. Mark [virtual memory pages](Virtual%20Memory.md) used for I/O as **uncacheable**
2. Use uncached load and store operations for I/O memory segments
3. Explicitly **flush** pages from the cache when I/O completes
4. Direct all I/O traffic to flow through the cache hierarchy. [Cache coherency](Cache%20Coherency.md) mechanisms are used to maintain coherency

Because I/O operations are less frequent than memory operations, these heavyweight software solutions are acceptable

# Definition

A memory system is **coherent** if the results of *any* execution of a program are such that, for **each location**, it is possible to construct a hypothetical serial order of all operations to the location (put all reads/writes issued by all CPUs into a **total order**) that is consistent with the **results** of the execution and in which:

1. Operations issued by any particular process occur in the order in which they were issued to the memory system by that process (**program order**)
2. The value returned by each read operation is the value written by the last write to that location in the serial order

---

A memory system is coherent if:

1. A read by CPU `C1` to a location `X` that follows a write by `C1` to `X`, with no writes of `X` by another CPU occurring between the write and the read by `C1`, always returns the value written by `P` (**program order**)
2. A read by CPU `C1` to a location `X` that follows a write by CPU `C2` to `X` returns the written value if the read and write are sufficiently separated in time, and no other writes to `X` occur between the two accesses (**write propagation**)
3. Two writes to the **same** location `X` by any two CPUs are seen in the same order by all CPUs (**write serialization**)

---

Three properties:

1. **Program order** - memory operations of a single process must appear to become visible (to itself and others) in program order (uniprocessor respects program order)
2. **Write propagation** - writes *eventually* become visible to other CPUs (when precisely is not specified)
3. **Write serialization** - all writes to the *same* location (from the same or different CPU) are seen in the same order by all CPUs

The **results** of a program can be viewed as the values returned by the read operations in it (perhaps augmented with an implicit set of reads to all locations at the end of the program)

Coherency applies individually to each cache line. Each line acts as having its own [FSM](Snooping%20Cache%20Coherence%20Protocols.md)

Coherence says nothing about when the write to a location will become visible (**write propagation**) in respect to other operations. It also says nothing about the order in which writes to different locations become visible

## Memory Consistency

We assume the following [memory consistency](Memory%20Models.md) properties:

- A write does not complete (and allow the next write to occur) until all CPUs have seen the effect of that write
- A CPU does not change the order of any write with respect to any other memory access

These two conditions mean that if a CPU writes location `X` followed by location `Y`, any CPU that sees the new value of `Y` must also see the new value of `X`

These restrictions allow the CPU to reorder reads but force the processor to finish a write in program order

# Coherence Mechanisms

Two mechanisms can be used to satisfy **"write serialization"**:

- [Snooping](Snooping%20Cache%20Coherence%20Protocols.md)
- [Directory-Based](Directory-Based%20Cache%20Coherence%20Protocols.md)

Two mechanisms can be used to satisfy **"write propagation"**:

1. **Write-Update** - CPU updates copies in other caches on a write, ensuring that all caches have the same data. This approach is similar to [write-through policy](Cache%20Memory.md#Write-Through). Often lower miss-rate compared to write-invalidate but not used in practice because of MUCH higher interconnect traffic
2. **Write-Invalidate** - CPU invalidates copies in other caches on a write, ensuring that it has exclusive access to a data item before it writes that item. This approach is similar to [write-back policy](Cache%20Memory.md#Write-Back)

# False Sharing

Write about false sharing #TODO

See:

- [False Sharing in Java](https://jenkov.com/tutorials/java-concurrency/false-sharing.html)
- [5.4. Multithreading Problems](http://www.nic.uoregon.edu/~khuck/ts/acumem-report/manual_html/multithreading_problems.html)
- [A Guide to False Sharing and @Contended | Baeldung](https://www.baeldung.com/java-false-sharing-contended)

# References

- [Cache coherence - Wikipedia](https://en.wikipedia.org/wiki/Cache_coherence)
- [Cache coherency protocols (examples) - Wikipedia](https://en.wikipedia.org/wiki/Cache_coherency_protocols_(examples))
- [Bus snooping - Wikipedia](https://en.wikipedia.org/wiki/Bus_snooping)
- [Directory-based cache coherence - Wikipedia](https://en.wikipedia.org/wiki/Directory-based_cache_coherence)
- [False sharing - Wikipedia](https://en.wikipedia.org/wiki/False_sharing#:~:text=False%20sharing%20is%20an%20inherent,is%20limited%20to%20RAM%20caches.)
- [High-Performance Computer Architecture: Part 5 - YouTube](https://youtube.com/playlist?list=PLAwxTw4SYaPkr-vo9gKBTid_BWpWEfuXe&si=TH5JP0CVKX9_TSQJ)
- [Computer Organization and Design RISC-V Edition The Hardware Software Interface (2nd ed). David A. Patterson, John L. Hennessy](References.md#Computer%20Organization%20and%20Design%20RISC-V%20Edition%20The%20Hardware%20Software%20Interface%20(2nd%20ed).%20David%20A.%20Patterson,%20John%20L.%20Hennessy)
- [CMU 15-418/15-618 Parallel Computer Architecture and Programming](References.md#CMU%2015-418/15-618%20Parallel%20Computer%20Architecture%20and%20Programming)
- [Dive Into Systems A Gentle Introduction to Computer Systems. Suzanne J. Matthews, Tia Newhall, Kevin C. Webb](References.md#Dive%20Into%20Systems%20A%20Gentle%20Introduction%20to%20Computer%20Systems.%20Suzanne%20J.%20Matthews,%20Tia%20Newhall,%20Kevin%20C.%20Webb)
- [Computer Architecture A Quantitative Approach (6th ed). John L. Hennessy, David A. Patterson](References.md#Computer%20Architecture%20A%20Quantitative%20Approach%20(6th%20ed).%20John%20L.%20Hennessy,%20David%20A.%20Patterson)
- [code::dive conference 2014 - Scott Meyers: Cpu Caches and Why You Care - YouTube](https://youtu.be/WDIkqP4JbkE?si=TWwpFRPxMXU9oYJx)
- [Parallel Computer Architecture A Hardware/Software Approach (1st ed). David Culler, Jaswinder Pal Singh, Anoop Gupta](References.md#Parallel%20Computer%20Architecture%20A%20Hardware/Software%20Approach%20(1st%20ed).%20David%20Culler,%20Jaswinder%20Pal%20Singh,%20Anoop%20Gupta)
