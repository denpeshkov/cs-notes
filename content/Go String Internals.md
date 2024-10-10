---
tags:
  - Go
---

# Overview

A string is a read-only [slice](Go%20Slice%20Internals.md) of **arbitrary** bytes. It is not required to hold Unicode, UTF-8 text, or any other predefined format

A string literal, absent byte-level escapes, always holds valid UTF-8 sequences

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

1. For a map `m` of type `map[string]T` and `[]byte b`, `m[string(b)]` doesn't allocate
2. No allocation when converting a `string` into a `[]byte` for ranging over the bytes
3. No allocation when converting a `[]byte` into a `string` for comparison purposes
4. A conversion from `[]byte]` to `string` which is used in a string concatenation, and at least one of concatenated string values is a non-blank string constant

# Substrings and Memory Leaks

When performing a substring operation, the Go specification doesn't specify whether the resulting string and the one involved in the substring operation should share the same data. However, the standard Go compiler does allow them share the same backing array

Thus, there can be memory leaks, as the string returned by a substring operation will be backed by the same byte array. The solution is to make a copy of the string:

```go
string([]byte(s[:ind]))
// or
strings.Clone(s[:ind])
```

# References

- [research!rsc: Go Data Structures](https://research.swtch.com/godata)
- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
- [Source code: string.go](https://github.com/golang/go/blob/master/src/runtime/string.go#L232)
- [Strings in Go -Go 101](https://go101.org/article/string.html)
- [Go Wiki: Compiler And Runtime Optimizations - The Go Programming Language](https://go.dev/wiki/CompilerOptimizations)
- [Strings, bytes, runes and characters in Go - The Go Programming Language](https://go.dev/blog/strings)
