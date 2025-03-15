---
tags:
  - Go
  - OS-Architecture
  - TODO
---

# Implementation

We can divide allocations into 2 buckets: small and large allocations

Most of the allocations are served from local thread caches, meaning that no locks are required, reducing contention and increasing the overall performance

## Tiny Allocations

Allocations with size <= 16B and no pointers

## Small Allocations

Allocations with size <= 32kB

1. The requested sized is rounded up to one of the 70 size classes
2. Allocator looks up at the local thread cache to see if there are free objects available for this size class:
	1. If there are free objects, it returns the first one from the list. This allocation required no locks as it was totally served from the local thread cache
	2. If there are no free objects, fetch new objects from the central cache
3. The central cache checks if it has free objects:
	1. If there are free objects, return a bunch to the local cache
	2. If there are no free objects, request a chunk of memory pages from the central heap, split it into objects for this size class and return them to the local cache

## Large Allocations

Large allocations are served directly by the central heap. They are rounded up to a page size 4K and a run of contiguous pages are called **spans**

The central heap maintains an array of [free lists](Heap%20Memory%20Manager.md). For k < 256, the kth entry is a free list of spans that consist of k pages. The 256th entry is a free list of spans that have length >= 256 pages.

1. The central heap will search for a span on the `k`-th free list
	1. If one is found, return it to the user
	2. If one is not found, look for it on the `k+1`-th free list, and so on…
2. If a free span is not found in any of the free lists, allocate a large chunk of memory from the OS
3. If the span found, or the memory requested from the OS, is larger than the user requested, split it into multiple spans and place then on the free lists before returning to the user.

# References

- [Go's Memory Allocator - Overview · André Carvalho](https://andrestc.com/post/go-memory-allocation-pt1/)
- [A visual guide to Go Memory Allocator from scratch (Golang) | by Ankur Anand | Medium](https://medium.com/@ankur_anand/a-visual-guide-to-golang-memory-allocator-from-ground-up-e132258453ed)
- [go/src/runtime/malloc.go at master · golang/go · GitHub](https://github.com/golang/go/blob/master/src/runtime/malloc.go)
- [Language Mechanics On Escape Analysis](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-escape-analysis.html)
