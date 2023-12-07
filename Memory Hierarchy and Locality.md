---
tags:
  - Architecture
  - OS
---

# Memory Locality

**Spatial locality** - If a memory location is referenced once, the the program is likely to reference a nearby memory location in the near future

**Temporal locality** - A memory location that is referenced once is likely to be referenced again multiple times in the near future

# Memory Hierarchy And Cache Management

Upper layers [[Cache Memory#Cache Inclusion Policy|typically]] store smaller subsets of data from lower layers but have faster access. Due to locality, accesses are often satisfied by upper layers

Memory management:

![[memory hierarchy and cache management.png|700]]

# Data Transfer Units

Data is always copied back and forth between level $k$ and level $k + 1$ in block-size transfer units:

- From [[Cache Memory|cache]] to register in words
- From lower [[Cache Memory|cache]] to higher cache in cache lines
- From [[Random Access Memory|RAM]] to cache in cache lines
- From [[Input-Output Devices|disk]] to [[Random Access Memory|RAM]] in disk blocks

# References

- [[References#Computer Systems A Programmer's Perspective, Global Edition (3rd ed). Randal E. Bryant, David R. O'Hallaron]]
