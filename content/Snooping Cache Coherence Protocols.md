---
tags:
  - Arch
  - Concurrency
  - DistributedSystems
---

# Overview

All coherence-related activity is broadcasted to all cache controllers in the system

The caches are all accessible via [broadcast interconnect](Interconnection%20Network.md) (e.g., [bus](Main%20Memory.md) or [network](Network.md)), and all cache controllers monitor or **snoop** the interconnect to determine whether or not they have a copy of a block that is requested

Because the interconnect is shared, only one CPU can **acquire** medium access and have **exclusive** access for writing at a time. Medium arbitration and acquisition enforce [write serialization](Cache%20Coherency.md)

Broadcasting makes snooping easy to implement but limits **scalability** because every message may be broadcasted to all devices through the same medium. The medium becomes a **bottleneck**

---

Here we assume the following simplifications:

- Bus transactions are atomic: only one transaction is in progress on the bus at a time
- Memory operations are atomic: the CPU waits until its previous memory operation is complete before issuing another memory operation
- Transitions between states (**stable states**) are atomic

In reality, these simplifications are not true. See [Snoop-Based Multiprocessor Design](Snoop-Based%20Multiprocessor%20Design.md) for implementation details

# MSI

MSI is [write-back](Cache%20Memory.md), [write-invalidate](Cache%20Coherency.md), [snooping](Cache%20Coherency.md) coherency protocol

- **Modified/Exclusive (M)**: exclusive read/write access, *[dirty](Cache%20Memory.md)*
- **Shared (S)**: shared read access, *[clean](Cache%20Memory.md)*
- **Invalid (I)**: line is invalid

FSM for MSI (here "exclusive" means "modified"):

![MSI FSM.png|800](MSI%20FSM.png)

In contrast to SI **write-through** protocol, a line is flushed to memory only on **eviction** (**write-miss**, **read-miss**) and snooped **write-miss** and **read-miss**

## Satisfying Coherency

MSI satisfies [coherency conditions](Cache%20Coherency.md):

1. "**Program order**" is implicitly satisfied by uniprocessor
2. "**Write propagation**" is satisfied by the combination of invalidation and write-back actions
3. "**Write serialization**" is satisfied:
   1. Writes that appear on interconnect are ordered by the order they appear on interconnect
   2. Reads that appear on interconnect are ordered by the order they appear on interconnect
   3. Writes that don't appear on interconnect ([write-back](Cache%20Memory.md#Write-Back) cache):
	  1. Writes come between two [interconnect](Interconnection%20Network.md) transactions
	  2. All writes are performed by the same CPU and ordered due to the 1st condition ("**program order**")
	  3. All other CPUs observe writes only after the interconnect transaction.
	  4. So all CPUs see writes in the same order

# MESI

Modified, Shared, and Invalid states same as in MSI

- **Exclusive clean (E)** state: exclusive read/write access, *[clean](Cache%20Memory.md)*

---

MSI requires two [interconnect transactions](Interconnection%20Network.md) for the case of reading an address and then writing to it even if no sharing at all:

1. **Read miss on interconnect** on **CPU read** to move from **Invalid** to **Shared**
2. **Invalidate on interconnect** on **CPU write** to move from **Shared** to **Modified (Exclusive)**

In the uniprocessor scenario (no shared lines), only the first transaction is needed

With **Exclusive clean** state, MESI requires only one [interconnect transaction](Interconnection%20Network.md) to write to a line that is not shared:

1. **Read miss on interconnect** on **CPU read** to move from **Invalid** to **Exclusive clean** if no sharing in other caches
2. On **CPU write**, move from **Shared** to **Modified (Exclusive)**

Thus, the behavior is the same as in a uniprocessor scenario

For implementation details, see: [Snoop-Based Multiprocessor Design](Snoop-Based%20Multiprocessor%20Design.md).

# MOESI

"Modified," "Shared," "Invalid," "Exclusive clean" states same as in MESI

- **Owned (O)**: shared read access, *[dirty](Cache%20Memory.md)*

Allows sending dirty cache lines directly between caches instead of writing back to a shared lower cache (or memory) and then reading from there

# References

- [Cache coherence - Wikipedia](https://en.wikipedia.org/wiki/Cache_coherence)
- [Cache coherency protocols (examples) - Wikipedia](https://en.wikipedia.org/wiki/Cache_coherency_protocols_(examples))
- [Bus snooping - Wikipedia](https://en.wikipedia.org/wiki/Bus_snooping)
- [Directory-based cache coherence - Wikipedia](https://en.wikipedia.org/wiki/Directory-based_cache_coherence)
- [False sharing - Wikipedia](https://en.wikipedia.org/wiki/False_sharing#:~:text=False%20sharing%20is%20an%20inherent,is%20limited%20to%20RAM%20caches.)
- [High Performance Computer Architecture: Part 5 - YouTube](https://youtube.com/playlist?list=PLAwxTw4SYaPkr-vo9gKBTid_BWpWEfuXe&si=TH5JP0CVKX9_TSQJ)
- [Computer Organization and Design RISC-V Edition The Hardware Software Interface (2nd ed). David A. Patterson, John L. Hennessy](References.md#Computer%20Organization%20and%20Design%20RISC-V%20Edition%20The%20Hardware%20Software%20Interface%20(2nd%20ed).%20David%20A.%20Patterson,%20John%20L.%20Hennessy)
- [Snooping-Based Cache Coherence: CMU 15-418/618 Spring 2016](http://15418.courses.cs.cmu.edu/spring2016/lecture/snoopcoherence)
- [Directory-Based Cache Coherence: CMU 15-418/618 Spring 2016](http://15418.courses.cs.cmu.edu/spring2016/lecture/dircoherence)
- [A Basic Snooping-Based Multi-Processor Implementation: CMU 15-418/618 Spring 2016](http://15418.courses.cs.cmu.edu/spring2016/lecture/snoopimpl)
