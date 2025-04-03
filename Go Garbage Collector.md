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
- Tri-color: Objects are categorized into three sets: white, grey, and black
- Precise: Metadata `runtime.stackmap` tells GC exactly what is and isn't a pointer
- Mark-Sweep: The process consists of a mark phase, where live objects are identified, followed by a sweep phase, where the memory of unreachable objects is reclaimed

# When GC is Triggered

- Calls to `runtime.GC`
- Inside calls to `malloc` when the heap size reaches a certain threshold
- Forcefully triggered by the [`sysmon`](Go%20Goroutines%20and%20Scheduler%20Internals.md) every 2 minutes, to ensure periodic memory management even in low-allocation scenarios

# GC Algorithm

The GC algorithm starts with all nodes marked as white. During the marking phase, both collector goroutines and application goroutines (mutators) run concurrently to eventually mark all reachable objects as black. Any objects that remain white after marking are considered garbage

The GC algorithm must have two properties:

1. Termination guarantee: No object can become lighter during the marking phase. Ensures that a black object won't be scanned again, guaranteeing that the algorithm eventually scans all reachable objects and the marking phase terminates
2. Tri-color invariant: Any white object pointed to by a black object must be reachable via a chain of white pointers from the grey object.

To ensure this properties in concurrent GC, objects must be divided into three sets:

1. White – Objects not yet visited (unmarked); potential garbage
2. Grey – Objects discovered but not fully scanned for references to white objects
3. Black – Objects completely scanned (marked); not candidates for collection

In concurrent GC, three colors are needed instead of just two (marked and unmarked) because the collector and the mutator (application goroutines) run simultaneously. When a goroutine is about to create a pointer from a black object to a white one:

1. The white target object cannot remain white, as this would break the tri-color invariant
2. The black source object must remain black to guarantee termination
3. The white target object cannot be directly marked black, as this could violate the tri-color invariant if it has any white successor objects

# Write Barrier

Because the collector runs concurrently with the application, application goroutines can create new objects (potentially with pointers to white objects) and modify pointers in existing objects. This can break the tri-color invariant by "hiding" objects from GC. To prevent this, Go uses write barriers

The write barrier in Go is implemented as follows:

```
writePointer(slot, ptr):
    shade(*slot)
    if current stack is grey:
        shade(ptr)
    *slot = ptr

// slot is the destination in Go code
// ptr is the value that goes into the slot in Go code
// shade(*slot) marks the object whose reference is being overwritten grey if it is not already grey or black
// shade(ptr) mark the reference being installed grey if it is not already grey or black
```

Additionally, the write barrier requires that objects be allocated black

Given implementation doesn't require stack re-scanning and prevents violation of a tri-color invariant:

1. Allocating objects as black eliminates the possibility of a black object pointing to an unreachable white object
2. `shade(*slot)` prevents a mutator from hiding an object by moving the sole pointer to it from the heap to its stack. If it attempts to unlink an object from the heap, this will shade it grey
3. `shade(ptr)` prevents a mutator from hiding an object by moving the sole pointer to it from its stack into a black object in the heap. If it attempts to install the pointer into a black object, this will shade it grey
4. Once a goroutine's stack is black, the `shade(ptr)` becomes unnecessary. `shade(ptr)` prevents hiding an object by moving it from the stack to the heap, but this requires first having a pointer hidden on the stack. Immediately after a stack is scanned, it only points to shaded objects, so it's not hiding anything, and the `shade(*slot)` prevents it from hiding any other pointers on its stack

## Stack and Globals Writes

The compiler omits write barriers for writes to the current [frame](Call%20Stack.md), but if a stack pointer has been passed down the call stack, the compiler will generate a write barrier for writes through that pointer (because it doesn't know it's not a heap pointer)

The Go garbage collector requires write barriers when heap pointers are stored in globals, so that we don't have to rescan global during mark termination

# GC Phases

## Sweep Termination

GC not running; Sweeping in background; Write barrier disabled

1. Stop the world. This causes all [`P`s](Go%20Goroutines%20and%20Scheduler%20Internals.md) to reach a GC safe-point
2. Sweep any unswept spans, clear [`sync.Pool`s](Go%20sync.Pool.md) and caches

## Mark Phase

GC marking; Write barrier enabled

1. Prepare for the marking by enabling the write barrier and mutator assists, and enqueueing root mark jobs. No objects are scanned until every [`P`](Go%20Goroutines%20and%20Scheduler%20Internals.md) has the write barrier enabled, which is done using STW
2. Start the world. From this point, GC is done by mark workers started by the [scheduler](Go%20Goroutines%20and%20Scheduler%20Internals.md) and by assists performed as part of allocation. The write barrier shades both the overwritten pointer and the new pointer value as grey. Newly allocated objects are immediately marked black
3. GC marks roots by scanning stacks, globals, and heap pointers in off-heap structures. Scanning a stack stops a goroutine, shades any pointers found on its stack, and then resumes the goroutine
4. GC drains the work queue of grey objects, scanning each grey object to black and shading all pointers found in the object (which in turn may add those pointers to the work queue)
5. When there are no more root marking jobs or grey objects, GC transitions to mark termination

Marking using tri-color marking algorithm

GC doesn't scan object that don't contain pointers

### Mark Assist

If the GC pacer determines that it needs to slow down allocations, it will recruit the application goroutines to assist with the marking work. The amount of time any application goroutine will be placed in a mark assist is proportional to the amount of data it's adding to heap memory

One goal of the GC is to eliminate the need for mark assists. If any given collection ends up requiring a lot of mark assist, the GC can start the next cycle earlier. This is done in an attempt to reduce the amount of mark assist that will be necessary on the next collection

## Mark Termination

GC mark termination; [`P`s](Go%20Goroutines%20and%20Scheduler%20Internals.md) help GC; Write barrier enabled

1. Stop the world
2. Disable workers and assists
3. Perform housekeeping like flushing `mcaches`
4. Calculate the next collection goal

Newly allocated objects are black

## Sweep Phase

GC not running; Sweeping in background; Write barrier disabled

1. Start the world. From this point on, newly allocated objects are white, and allocating sweeps spans before use if necessary
2. GC does concurrent sweeping in the background and in response to allocation

The sweep phase proceeds concurrently with normal program execution

The heap is swept both when a goroutine allocates a new memory (needs another span) and concurrently in a background sweeper goroutine. To avoid requesting more OS memory while there are unswept spans, when a goroutine needs another span, it first attempts to reclaim that much memory by sweeping

# Pacing

The Go GC uses a pacing algorithm to decide when to start a new collection cycle

Its goal is to balance two factors:

1. Keeping the heap size proportional to the live [heap](Heap%20Memory.md) at the end of the last cycle, controlled by the `GOGC` variable
2. Maintaining a mark phase utilization target of 25% of `GOMAXPROCS`

The next GC cycle starts after allocating extra memory proportional to the amount already in use, controlled by `GOGC` variable. For example, if `GOGC` is 100 and 4MB in use, GC runs again at 8MB. This keeps GC cost proportional to allocation

The `GOMEMLIMIT` variable sets a soft limit on total memory usage. It's a soft limit and runtime tries respect it by adjusting the frequency of GCs and returning memory to the OS more aggressively

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

- [A Guide to the Go Garbage Collector - The Go Programming Language](https://tip.golang.org/doc/gc-guide)
- [Garbage Collection In Go : Part I - Semantics](https://www.ardanlabs.com/blog/2018/12/garbage-collection-in-go-part1-semantics.html)
- [Go's garbage collector · agrim mittal](https://agrim123.github.io/posts/go-garbage-collector.html)
- [Why golang garbage-collector not implement Generational and Compact gc?](https://groups.google.com/g/golang-nuts/c/KJiyv2mV2pU) 
- [go/src/runtime/mgc.go at master · golang/go · GitHub](https://github.com/golang/go/blob/master/src/runtime/mgc.go#L5-L127)
- [go/src/runtime/mbarrier.go at master · golang/go · GitHub](https://github.com/golang/go/blob/master/src/runtime/mbarrier.go)
- [Go Wiki: Compiler And Runtime Optimizations - The Go Programming Language](https://go.dev/wiki/CompilerOptimizations#non-scannable-objects)
- [GopherCon 2015: Go GC: Solving the Latency Problem - Rick Hudson - YouTube](https://youtu.be/aiv1JOfMjm0?si=rbefmmOhxpQGB6wG)
- [Golang UK Conference 2017 \| Will Sewell & Jim Fisher - Golang's Realtime GC in Theory and Practice - YouTube](https://youtu.be/bMujSVMarqY?si=St76bli8DhHGNTeY)
- [Ozon Tech Go Meetup - YouTube](https://www.youtube.com/live/PB4vA5eId4c?si=eHToLE_jZSZu9ggR)
- [Как устроен garbage collector в Go 1.9 - Андрей Дроздов, Avito - YouTube](https://youtu.be/CX4GSErFenI?si=TrZ2A3j9jduOuQRe)
- [Tricolor Abstraction to build concurrent Garbage Collectors - YouTube](https://youtu.be/lhrRwjVPXPo?si=wrHi1hpYiEEEz0R3)
- [On-the-Fly Garbage Collection: An Exercise in Cooperation. Dijkstra, Lamport](https://lamport.azurewebsites.net/pubs/garbage.pdf)
- [Basics of Golang GC Explained: Tri-color Mark and Sweep and Stop the World \| by Vadim Inshakov \| Stackademic](https://blog.stackademic.com/basics-of-golang-gc-explained-tri-color-mark-and-sweep-and-stop-the-world-cc832f99164c)
- [proposal/design/17503-eliminate-rescan.md at master · golang/proposal · GitHub](https://github.com/golang/proposal/blob/master/design/17503-eliminate-rescan.md)
- [proposal/design/44167-gc-pacer-redesign.md at master · golang/proposal · GitHub](https://github.com/golang/proposal/blob/master/design/44167-gc-pacer-redesign.md)
