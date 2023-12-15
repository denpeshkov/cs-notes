---
tags: Go
---

# Representation

A `string` is represented in memory as a 2-word structure containing a pointer to the string data and a length

![string representation|200](string%20representation%20Go.png)

String are **immutable**, so there is no need for a capacity (you can't grow them)  

It is safe for multiple strings to share the same storage, so slicing `s` results in a new 2-word structure with a potentially different pointer and length that still refers to the same byte sequence  
This means that slicing can be done without allocation or copying, making string slices as efficient as passing around explicit indexes

# References

- [research!rsc: Go Data Structures](https://research.swtch.com/godata)
