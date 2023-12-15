---
tags: Go
---

# Representation

A `string` is represented in memory as a 2-word structure containing:

- A pointer to an **immutable** byte sequence
- The total number of bytes in this sequence

![string representation|200](string%20representation%20Go.png)

String are **immutable**, so there is no need for a capacity (you can't grow them)  

It is safe for multiple strings to share the same storage, so slicing `s` results in a new 2-word structure with a potentially different pointer and length that still refers to the same byte sequence  
This means that slicing can be done without allocation or copying, making string slices as efficient as passing around explicit indexes

Because the underlying byte array is immutable, casting `[]byte` to `string` and vice versa results in a copy

# References

- [research!rsc: Go Data Structures](https://research.swtch.com/godata)
- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
