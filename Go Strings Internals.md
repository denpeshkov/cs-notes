---
tags: Go
---

# Representation

A `string` is represented by a `stringStruct` struct:

```go
type stringStruct struct {
	str unsafe.Pointer
	len int
}
```

- `str` is a pointer to an **immutable** byte sequence
- `len` is a total number of bytes in this sequence

For example:

![string representation|200](string%20representation%20Go.png)

Strings are **immutable**, so there is no need for a capacity (you can't grow them)  

It is safe for multiple strings to share the same storage, so slicing `s` results in a new 2-word structure with a potentially different pointer and length that still refers to the same byte sequence  
This means that slicing can be done **without allocation** or copying, making string slices as efficient as passing around explicit indexes

# Casting and Memory Allocation

Because the underlying byte array is immutable, casting `[]byte` to `string` and vice versa results in a copy

However, there are some optimizations that the compiler makes to avoid copies:

- A conversion from `string` to `[]byte` which follows the `range` keyword in a range loop
- A conversion from `[]byte]` to `string` which is used as a map key in map element retrieval indexing syntax
- A conversion from `[]byte]` to `string` which is used in a comparison
- A conversion from `[]byte]` to `string` which is used in a string concatenation, and at least one of concatenated string values is a non-blank string constant

# References

- [research!rsc: Go Data Structures](https://research.swtch.com/godata)
- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
- [Source code: string.go](https://github.com/golang/go/blob/master/src/runtime/string.go#L232)
- [Strings in Go -Go 101](https://go101.org/article/string.html)
