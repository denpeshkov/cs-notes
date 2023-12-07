---
tags:
  - Architecture
  - OS
---

# Memory Locality

**Spatial locality** - If a memory location is referenced once, the the program is likely to reference a nearby memory location in the near future

**Temporal locality** - A memory location that is referenced once is likely to be referenced again multiple times in the near future

# Memory Hierarchy And Cache Management

Upper layers [typically](Cache%20Memory.md#Cache%20Inclusion%20Policy) store smaller subsets of data from lower layers but have faster access. Due to locality, accesses are often satisfied by upper layers

Memory management:

![memory hierarchy and cache management.png|700](memory%20hierarchy%20and%20cache%20management.png)

# Data Transfer Units

Data is always copied back and forth between level $k$ and level $k + 1$ in block-size transfer units:

- From [cache](Cache%20Memory.md) to register in words
- From lower [cache](Cache%20Memory.md) to higher cache in cache lines
- From [RAM](Random%20Access%20Memory.md) to cache in cache lines
- From [disk](Input-Output%20Devices.md) to [RAM](Random%20Access%20Memory.md) in disk blocks

# References

- [Computer Systems A Programmer's Perspective, Global Edition (3rd ed). Randal E. Bryant, David R. O'Hallaron](References.md#Computer%20Systems%20A%20Programmer's%20Perspective,%20Global%20Edition%20(3rd%20ed).%20Randal%20E.%20Bryant,%20David%20R.%20O'Hallaron)
