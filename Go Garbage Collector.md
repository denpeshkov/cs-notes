---
tags:
  - Go
  - TODO
aliases:
  - Go GC
---

# Overview

Go uses non-generational non-compacting concurrent tri-color precise mark-and-sweep GC

- Non-generational: GC traverses the entire [heap](Heap%20Memory.md) when performing a collection pass, and its performance therefore degrades as the heap expands. The heap is not broken into generations based on objects age
- Non-compacting: Objects are not relocated towards the beginning of the heap, thus heap can be fragmented
- Concurrent: Most of the GC work proceeds concurrently with normal program execution, leading to minimal GC pause
- Tri-color: Objects are categorized into three sets: white, grey, and black; representing objects that are unprocessed, currently being processed, and processed
- Precise: Metadata `runtime.stackmap` tells GC exactly what is and isn't a pointer
- Mark-Sweep: The process consists of a mark phase, where live objects are identified, followed by a sweep phase, where the memory of unreachable objects is reclaimed

# Phases

![GC algorithm phases|600](GC%20algorithm%20phases.png)

## Sweep Termination

1. **Stop the world** (**STW**). This causes all [`p`s](Go%20Goroutines%20and%20Scheduler%20Internals.md) to reach a GC safe-point
2. Sweep any unswept [spans](Go%20Memory%20Allocator.md). There will only be unswept spans if this GC cycle was forced before the expected time

## Mark Phase

1. Prepare for the mark phase by enabling the write barrier, enabling mutator assists, and enqueueing root mark jobs. No objects may be scanned until all [`p`s](Go%20Goroutines%20and%20Scheduler%20Internals.md) have enabled the write barrier, which is accomplished using STW
2. **Start the world**. From this point, GC work is done by mark workers started by the [scheduler](Go%20Goroutines%20and%20Scheduler%20Internals.md) and by assists performed as part of allocation. The write barrier shades both the overwritten pointer and the new pointer value for any pointer writes. Newly allocated objects are immediately marked black
3. GC performs root marking jobs. This includes scanning all [stacks](Call%20Stack.md), shading all globals, and shading any heap pointers in off-heap runtime data structures. Scanning a stack stops a goroutine, shades any pointers found on its stack, and then resumes the goroutine
4. GC drains the work queue of grey objects, scanning each grey object to black and shading all pointers found in the object (which in turn may add those pointers to the work queue)
5. Because GC work is spread across local caches, GC uses a distributed termination algorithm to detect when there are no more root marking jobs or grey objects. At this point, GC transitions to [mark termination](#Mark%20Termination)

Go dedicates 25% of `GOMAXPROCS` CPUs to background marking

### Tri-Color Marking

Objects are divided into 3 sets:

1. White set objects are not marked
2. Grey set (shaded) objects are marked, but we have not yet scanned all their referents (pointers)
3. Black set objects are marked along with all their referents (pointers)

Compiler generates `runtime.stackmap`s metadata for each call site to tell the runtime exactly which stack bytes are pointers. Note that, GC also has a conservative stack scanning, treating any pointer-like value in the block as a pointer, used for [loop preemption](Go%20Scheduler%20WIP.md#Loop%20Preemption)

GC must maintain a tri-color invariant: there are no direct pointers from black set objects to white set objects. It's because at the end of the mark phase all white set objects are considered unreachable and eligible for deletion. Having pointers to them from black set objects will cause dangling pointers

This invariant is maintained by the write barrier by shading (marking grey) objects and pointers

### Mark Assist

As stated above, in the preparation for the [mark phase](#Mark%20Phase) we enable mark assist. If the GC Pacer determines that it needs to slow down allocations, it will recruit the application goroutines to assist with the marking work. The amount of time any application goroutine will be placed in a mark assist is proportional to the amount of data it's adding to heap memory

## Mark Termination

1. **Stop the world**
2. Disable workers and assists
3. Perform housekeeping like flushing [memory caches](Go%20Memory%20Allocator.md)

## Sweep Phase

1. Prepare for the sweep phase by setting up sweep state and disabling the write barrier
2. **Start the world**. From this point on, newly allocated objects are white, and allocating sweeps spans before use if necessary
3. GC does concurrent sweeping in the background and in response to allocation

The sweep phase proceeds concurrently with normal program execution. The heap is swept [span-by-span](Go%20Memory%20Allocator.md) both lazily (when a goroutine needs another [span](Go%20Memory%20Allocator.md)) and concurrently in a background goroutine (this helps programs that are not CPU bound)

At the end of STW [mark termination](#Mark%20Termination) all [spans](Go%20Memory%20Allocator.md) are marked as "needs sweeping"

# GC Pacing

GC has a **pacing algorithm** which determines when to start a collection. Pacing uses feedback loop to find the right time to trigger a GC cycle so that it hits the target [heap](Heap%20Memory.md) size goal

Next GC is after we've allocated an extra amount of memory proportional to the amount already in use. The proportion is controlled by `GOGC` environment variable (100 by default). 

If `GOGC` is 100 and we're using 4M, we'll GC again when we get to 8M. This keeps the GC cost in linear proportion to the allocation cost. Adjusting `GOGC` just changes the linear constant (and also the amount of extra memory used).

The `GOMEMLIMIT` variable sets a soft limit on a total amount of memory that the Go runtime can use. It is a soft limit and runtime undertakes several processes to try to respect this memory limit, including adjustments to the frequency of garbage collections and returning memory to the OS more aggressively

# Optimizations

Garbage collector does not scan underlying buffers of [slices](Go%20Slice%20Internals.md), [channels](Go%20Channels%20Internals.md) and [maps](Go%20Map%20Internals.md) when element type does not contain pointers (both key and value for maps). This allows to hold large data sets in memory without paying high price during garbage collection

For example, the following map won't visibly affect GC time:

```go
type Key [64]byte // SHA-512 hash
type Value struct {
    Name      [32]byte
    Balance   uint64
    Timestamp int64
}
m := make(map[Key]Value, 1e8)
```

# References

- [A Guide to the Go Garbage Collector - The Go Programming Language](https://tip.golang.org/doc/gc-guide) #todo
- [Garbage Collection In Go : Part I - Semantics](https://www.ardanlabs.com/blog/2018/12/garbage-collection-in-go-part1-semantics.html) #todo
- [Go's garbage collector · agrim mittal](https://agrim123.github.io/posts/go-garbage-collector.html)
- [Source code: mbarier.go](https://github.com/golang/go/blob/master/src/runtime/mbarrier.go) #todo
- [Why golang garbage-collector not implement Generational and Compact gc?](https://groups.google.com/g/golang-nuts/c/KJiyv2mV2pU) #todo
- [Source Code: mgc.go](https://github.com/golang/go/blob/master/src/runtime/mgc.go#L5-L127)
- [Source code: mwbbuf.go](https://github.com/golang/go/blob/master/src/runtime/mwbbuf.go) #TODO 
- [Go Wiki: Compiler And Runtime Optimizations - The Go Programming Language](https://go.dev/wiki/CompilerOptimizations#non-scannable-objects)
