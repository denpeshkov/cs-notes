---
tags: Go
---

# Useless String Conversion

When deciding between using `[]byte` and `string`, we should opt for `[]byte` because it's used by the I/O methods

All the exported functions of the `strings` package also have alternatives in the `bytes` package

By using `[]byte` we can avoid unnecessary `string` conversions that cause [memory allocations](Go%20Strings%20Internals.md)

# Substrings and Memory Leaks

When performing a substring operation, the Go specification doesn't specify whether the resulting string and the one involved in the substring operation should share the same data  
However, the standard Go compiler does allow them [share the same backing array](Go%20Strings%20Internals.md)

Thus, there can be memory leaks, as the string returned by a substring operation will be backed by the same byte array

The solution is to make a copy of the string:

```go
string([]byte(s[:ind]))
// or
strings.Clone(s[:ind])
```

# References

- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
