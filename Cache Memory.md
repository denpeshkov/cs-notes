---
tags:
  - OS_Arch
---

# Cache Hierarchy

![Intel Core i7 cache hierarchy.png|500](Intel%20Core%20i7%20cache%20hierarchy.png)

The L1 cache is usually split into two parts: the instruction cache (L1i) and the data cache (L1d). This division allows for a better utilization of locality since instructions and data are handled separately

# Cache Memory Organization

Caches save a *subset* of data from a lower level

| Parameter   | Description                                                               |
| ----------- | ------------------------------------------------------------------------- |
| $S=2^s$     | Number of sets                                                            |
| $s=\lg S$   | Number of set bits                                                        |
| $E$         | Number of cache lines per set                                             |
| $B=2^b$     | Block size (bytes)                                                        |
| $m=\lg M$   | Number of physical (main memory) address bits                             |
| $b=\lg B$   | Number of block offset bits                                               |
| $t=m-(s+b)$ | Number of tag bits                                                        |
| $C=B×E×S$   | Cache size (bytes), not including overhead such as the valid and tag bits |

![general organization of cache.png|700](general%20organization%20of%20cache.png)

Middle $s$ bits are used for the set index because it allows contiguous blocks to be spread more evenly among cache sets. Index­ing with the high­-order bits would cause contiguous memory blocks to all map to the same cache set, leaving the rest of the cache unused

The cache stores and loads the whole cache data block (cache line) from/to [lower levels](Memory%20Hierarchy%20and%20Locality.md)

## Cache Data Block Starting Address

Cache data blocks are [$B$-bytes aligned](Data%20Alignment.md), for example, for a block of size $B$=64 bytes the starting address can be 0, 64, 128, etc

Without this alignment requirement, set selection would be complex due to one cache data block spanning multiple set index bits. For example, if $B$=8, $S$=4, $E$=1, and block spans addresses $0x0002,\dotsc,0x000A$, then the block would occupy two set index bits: $0x00$ and $0x01$ #TODO

# Request Processing

## Set Selection

Determine which set contains the requested word $w$ that we want to read or write (update)

## Line Matching

Determine which cache line (if any) contains $w$. If some cache line contains $w$, we have a **cache hit**; otherwise, a **cache miss**

### Line Replacement (Eviction)

On **read misses** and **write misses** in **write-allocate** caches, we need to retrieve the line containing the requested word from the next level in the memory hierarchy

If the set is full, we need to **replace** (**evict**) some line based on the **replacement policy**

## Word Extraction

Cache data block size $B$ is larger than word size, so we need to determine where in a cache block the word $w$ is contained

# Direct-Mapped Cache Request Processing

A cache with one cache line per set: $E=1$

## Set Selection

Set index bits in the address of $w$ are interpreted as an unsigned integer that corresponds to a set number

![direct-mapped cache set selection.png|600](direct-mapped%20cache%20set%20selection.png)

## Line Matching and Word Extraction

The word $w$ is contained in the line if and only if the valid bit is set, and the tag bits in the cache line match the tag bits in the address of $w$

Block offset bits in the address of $w$ determine the offset of the *first byte* in the word $w$

![direct-mapped cache line matching word extraction.png|600](direct-mapped%20cache%20line%20matching%20word%20extraction.png)

### Line Replacement

There is only one cache line per set, so the current line is replaced by the newly fetched line

# Set Associative Cache Request Processing

A cache with several cache lines per set: $1 < E < \frac{C}{B}$

## Set Selection

Set index bits in the address of $w$ are interpreted as an unsigned integer that corresponds to a set number

![set associative cache set selection.png|600](set%20associative%20cache%20set%20selection.png)

## Line Matching and Word Extraction

The word $w$ is contained in the line if and only if the valid bit is set, and the tag bits in the cache line match the tag bits in the address of $w$

Because there are multiple lines in the set, check each line in parallel. Circuitry becomes more complex

Block offset bits in the address of $w$ determine the offset of the *first byte* in the word $w$

![set associative cache line matching word extraction.png|600](set%20associative%20cache%20line%20matching%20word%20extraction.png)

### Line Replacement

The line to be replaced is chosen by the replacement policy (**LRU**, **LFU**) among the lines in the set. Need to store additional information to support replacement: access counters, age bits, etc.

# Fully Associative Cache Request Processing

A cache with one set containing all cache lines: $E=\frac{C}{B}$

## Set Selection

There is only one set and no set bits in the address of $w$, so it is always selected

![fully associative cache set selection.png|600](fully%20associative%20cache%20set%20selection.png)

## Line Matching and Word Extraction

The word $w$ is contained in the line if and only if the valid bit is set, and the tag bits in the cache line match the tag bits in the address of $w$

We need to check all cache lines in parallel. Circuitry becomes very complex

Block offset bits in the address of $w$ determine the offset of the *first byte* in the word $w$

![fully associative cache line matching word extraction.png|600](fully%20associative%20cache%20line%20matching%20word%20extraction.png)

### Line Replacement

The line to be replaced is chosen by the replacement policy (**LRU**, **LFU**) among all the lines. Need to store additional information to support replacement: access counters, age bits, etc.

# Write-Hit Policies

When we write the word $w$ to the cache that already has its copy cached (**write hit**), we must update the copy of the data in the next lower level of the memory hierarchy

## Write-Through

Write is done synchronously both to the cache and to the [next lower level](Memory%20Hierarchy%20and%20Locality.md)

Inefficient because of a very high bandwidth requirement

## Write-Back

Write is initially done only to the cache. Write to the [next lower level](Memory%20Hierarchy%20and%20Locality.md) is postponed for as long as possible and done only when the line is **evicted** from the cache by the replacement algorithm

The cache maintains a **dirty bit** for each cache line that indicates whether or not it has been modified

### Write Buffer

A write miss in a **write-back** cache can involve two bus transactions:

1. Incoming line - the line requested by the CPU
2. Outgoing line - **evicted dirty** line that must be flushed

Ideally, we would like the CPU to continue as soon as possible without waiting for the flush to the lower level

The solution is a **write buffer**:

1. Save the line to be flushed in a write buffer
2. Immediately load the requested line, which allows the CPU to continue
3. Flush the write-back buffer at a later time

# Write-Miss Policies

When we write the word $w$ to the cache that doesn't have it cached (**write miss**), we must decide whether to load the data to the cache since no data is returned on write operations

## Write Allocate

Data at the missed-write location is loaded into the cache, followed by a **write-hit** operation

In this approach, **write misses** are similar to **read misses**

## No-Write Allocate

Data at the missed-write location is not loaded into the cache and is written directly to the [next lower level](Memory%20Hierarchy%20and%20Locality.md)

In this approach, data is loaded into the cache on **read misses** only

# Cache Inclusion Policy

## Inclusive

A lower-level cache is **inclusive** of a higher-level cache if it contains all entries present in a higher-level cache

- If the block is found in L1, it's returned to the CPU
- If the block is not found in L1 but found in L2, it's returned to the CPU and placed in L1
- If the block is not found in L1 and L2, it's fetched from memory and placed in both L1 and L2
- Eviction from L1 doesn't involve L2
- Eviction from L2 is propagated to L1 if the evicted block also exists in L1

## Exclusive

A lower-level cache is **exclusive** of a higher-level cache if it contains only entries not present in a higher-level cache

- If the block is found in L1, it's returned to the CPU
- If the block is not found in L1 but found in L2, it's moved from L2 to L1
- If the block is not found in L1 and L2, it's fetched from memory and placed in L1 only
- A block evicted from L1 is placed into L2. It's the only way L2 gets populated
- Eviction from L2 doesn't involve L1

## NINE

A lower-level cache is **non-inclusive non-exclusive** if it's neither strictly inclusive nor exclusive

- If the block is found in L1, it's returned to the CPU
- If the block is not found in L1 but found in L2, it's returned to the CPU and placed in L1
- If the block is not found in L1 and L2, it's fetched from memory and placed in both L1 and L2
- Eviction from L1 doesn't involve L2
- Eviction from L2 doesn't involve L1

# Addressing

Instructions use [virtual addresses](Virtual%20Memory.md), caches can use virtual or physical addresses for the index and tag

## Physically Indexed, Physically Tagged (PIPT)

Use the physical address for both the index and the tag

Simple but slow because the need to translate to a virtual address, which may involve a [TLB](Virtual%20Memory.md) miss and [RAM](Main%20Memory.md) access

## Virtually Indexed, Virtually Tagged (VIVT)

Use the [virtual address](Virtual%20Memory.md) for both the index and the tag

Fast because MMU is not used to translate to a physical address

Complex design:

- Different virtual addresses can refer to the same physical address (**aliasing**), which can cause coherency problems
- The same virtual address can map to different physical addresses (**homonyms**)
- Mapping can change, requiring cache flushing

## Virtually Indexed, Physically Tagged (VIPT)

Use the [virtual address](Virtual%20Memory.md) for the index and the physical address in the tag

The advantage over PIPT is lower latency, as the cache line can be looked up in parallel with the [TLB](Virtual%20Memory.md) translation, but the tag cannot be compared until the physical address is available

The advantage over VIVT is that since the tag has the physical address, the cache can detect homonyms

## Physically Indexed, Virtually Tagged (PIVT)

Useless and not used in practice

# Types of Misses

## Compulsory (Cold) Miss

The very first access to a block cannot be in the cache, so the block must be brought into the cache

## Capacity Miss

If the cache cannot contain all the blocks needed during the execution of a program, capacity misses (in addition to compulsory misses) will occur because of blocks being discarded and later retrieved

## Conflict Miss

If the block placement strategy is not fully associative, conflict misses (in addition to compulsory and capacity misses) will occur because a block may be discarded and later retrieved if multiple blocks map to its set and accesses to the different blocks are intermingled

## Coherency Miss

Misses due to [invalidations](Cache%20Coherency.md) when using [invalidation cache coherency protocol](Cache%20Coherency.md)

# Cache Coloring

[Cache coloring - Wikipedia](https://en.wikipedia.org/wiki/Cache_coloring) #TODO

# Prefetching

A technique used by CPU to boost execution performance by fetching instructions or data into caches before it is actually needed

Cache prefetching can be accomplished either by hardware or by software:

- **Hardware based prefetching** is accomplished by having a **prefetcher** in the CPU that watches the stream of instructions or data, recognizes the next few elements that the program might need based on this stream and prefetches into the cache
- **Software based prefetching** is accomplished by inserting *prefetch instructions* in the program during compilation. For example, intrinsic function `__builtin_prefetch` in *GCCoco*

# References

- [Computer Systems A Programmer's Perspective, Global Edition (3rd ed). Randal E. Bryant, David R. O'Hallaron](References.md#Computer%20Systems%20A%20Programmer's%20Perspective,%20Global%20Edition%20(3rd%20ed).%20Randal%20E.%20Bryant,%20David%20R.%20O'Hallaron)
- [Dive Into Systems A Gentle Introduction to Computer Systems. Suzanne J. Matthews, Tia Newhall, Kevin C. Webb](References.md#Dive%20Into%20Systems%20A%20Gentle%20Introduction%20to%20Computer%20Systems.%20Suzanne%20J.%20Matthews,%20Tia%20Newhall,%20Kevin%20C.%20Webb)
- [What every programmer should know about memory](https://lwn.net/Articles/252125/)
- [Cache replacement policies - Wikipedia](https://en.wikipedia.org/wiki/Cache_replacement_policies)
- [Cache (computing) - Wikipedia](https://en.wikipedia.org/wiki/Cache_(computing))
- [Cache inclusion policy - Wikipedia](https://en.wikipedia.org/wiki/Cache_inclusion_policy)
- [CPU cache - Wikipedia](https://en.wikipedia.org/wiki/CPU_cache)
- [Snooping-Based Cache Coherence: CMU 15-418/618 Spring 2016](http://15418.courses.cs.cmu.edu/spring2016/lecture/snoopcoherence)
- [Directory-Based Cache Coherence: CMU 15-418/618 Spring 2016](http://15418.courses.cs.cmu.edu/spring2016/lecture/dircoherence)
- [A Basic Snooping-Based Multi-Processor Implementation: CMU 15-418/618 Spring 2016](http://15418.courses.cs.cmu.edu/spring2016/lecture/snoopimpl)
- [High Performance Computer Architecture: Part 3 - YouTube](https://youtube.com/playlist?list=PLAwxTw4SYaPnhRXZ6wuHnnclMLfg_yjHs&si=_V-bFdZZY_2ESA8Z)
- [High Performance Computer Architecture: Part 4 - YouTube](https://youtube.com/playlist?list=PLAwxTw4SYaPn79fsplIuZG34KwbkYSedj&si=qiq0hc8ubwKI-j54)
- [Cache prefetching - Wikipedia](https://en.wikipedia.org/wiki/Cache_prefetching)
- [Cache Block Start Addr - Georgia Tech - HPCA: Part 3 - YouTube](https://www.youtube.com/watch?v=XPgT3arcoTg&list=PLAwxTw4SYaPnhRXZ6wuHnnclMLfg_yjHs&index=130)
