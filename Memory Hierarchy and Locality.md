---
tags:
  - OS_Arch
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

# References

- [Computer Systems A Programmer's Perspective, Global Edition (3rd ed). Randal E. Bryant, David R. O'Hallaron](References.md#Computer%20Systems%20A%20Programmer's%20Perspective,%20Global%20Edition%20(3rd%20ed).%20Randal%20E.%20Bryant,%20David%20R.%20O'Hallaron)
