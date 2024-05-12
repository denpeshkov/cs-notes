---
tags:
  - Arch
  - OS
---

# Memory Locality

**Spatial locality** - If a memory location is referenced once, the the program is likely to reference a nearby memory location in the near future

**Temporal locality** - A memory location that is referenced once is likely to be referenced again multiple times in the near future

Memory locality means that to increase performance, accesses should be satisfied by upper memory layers

# Memory Hierarchy And Management

Upper layers [typically](Cache%20Memory.md#Cache%20Inclusion%20Policy) store smaller subsets of data from lower layers but have faster access. Due to locality, accesses are often satisfied by upper layers

Memory management:

![memory hierarchy and cache management.png|700](memory%20hierarchy%20and%20cache%20management.png)

# Data Transfer Units

Data is always copied back and forth between level $k$ and level $k + 1$ in block-size transfer units:

- From L1 [cache](Cache%20Memory.md) to registers in 64-bits words
- From lower [cache](Cache%20Memory.md) to higher level cache in 64-bytes cache data blocks
- From [RAM](Main%20Memory.md) to cache in 64-bytes cache data blocks
- From [disk](Input-Output%20Devices.md) to [RAM](Main%20Memory.md) in disk blocks

# Example of a Memory Flow

Memory flow during execution of the statement `int x = 10`:

1. [Virtual address](Virtual%20Memory.md) to physical address conversion (TLB lookup)
2. [TLB](Virtual%20Memory.md) miss
3. TLB update (might involve OS)
4. OS may need to swap in [page](Virtual%20Memory.md) to get the appropriate page table (load from disk to physical address)
5. [Cache lookup](Cache%20Memory.md) (tag check)
6. Determine line not in cache (need to generate read-miss)
7. Arbitrate for bus
8. Win bus, place address, command on bus
9. All caches [perform snoop](Snoop-Based%20Multiprocessor%20Design.md) (e.g., invalidate their local copies of the relevant line)
10. Another cache or [memory](Main%20Memory.md) decides it must respond (let's assume it's memory)
11. Memory request sent to memory controller
12. Memory controller is itself a scheduler
13. Memory controller checks active row in [DRAM row buffer](Main%20Memory.md). (May need to activate new DRAM row. Let's assume it does.)
14. DRAM reads values into row buffer
15. Memory arbitrates for data bus
16. Memory wins bus
17. Memory puts data on bus
18. Requesting cache grabs data, updates cache line and tags, moves line into exclusive state
19. Processor is notified that data exists
20. Instruction proceeds

# References

- [Computer Systems A Programmer's Perspective, Global Edition (3rd ed). Randal E. Bryant, David R. O'Hallaron](References.md#Computer%20Systems%20A%20Programmer's%20Perspective,%20Global%20Edition%20(3rd%20ed).%20Randal%20E.%20Bryant,%20David%20R.%20O'Hallaron)
- [Snooping-Based Cache Coherence: CMU 15-418/618 Spring 2016](http://15418.courses.cs.cmu.edu/spring2016/lecture/snoopcoherence)
- [Directory-Based Cache Coherence: CMU 15-418/618 Spring 2016](http://15418.courses.cs.cmu.edu/spring2016/lecture/dircoherence)
- [A Basic Snooping-Based Multi-Processor Implementation: CMU 15-418/618 Spring 2016](http://15418.courses.cs.cmu.edu/spring2016/lecture/snoopimpl)
