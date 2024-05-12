---
tags:
  - Go
---

# Allocation of Zero-Sized Values

Instantiating an object of size 0 (such as `struct{}{}`, `[42]struct{}`) doesn't result in an allocation. It utilizes the address of `runtime.zerobase`, the base address for all 0-byte allocations

Note that Go specification states:

> A struct or array type has size zero if it contains no fields (or elements, respectively) that have a size greater than zero. Two distinct zero-size variables may have the same address in memory.

So, in theory, the compiler doesn't guarantee this fact to be true, although it has always been and continues to be the case in the current implementation of the official Go compiler (`gc`)

# References

- [Chapter II: Interfaces - Go Internals](https://cmc.gitbook.io/go-internals/chapter-ii-interfaces#special-cases-and-compiler-tricks)
- [The Go Programming Language Specification - The Go Programming Language](https://tip.golang.org/ref/spec#Size_and_alignment_guarantees)
- [Why struct{}{} is zero-size? : r/golang](https://www.reddit.com/r/golang/comments/1alxwnt/why_struct_is_zerosize/)
