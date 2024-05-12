---
tags:
  - Go
---

# Overview

We can have an unexported struct with exported fields, For example, consider the following declaration:

```go
package foo

type s struct {
	I int
}

func F() s {
	return s{}
}
```

This struct can be accessed in another package as follows:

```go
package main

func main() {
	a:=foo.F()
	a.I=1
	_=foo.F().I
}
```

---

This technique is commonly used, for instance, when declaring structs in handlers for encoding/decoding to/from JSON

# References

- [The Go Programming Language Specification - The Go Programming Language](https://tip.golang.org/ref/spec#Exported_identifiers)
