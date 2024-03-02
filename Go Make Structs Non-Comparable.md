---
tags: Go 
---

# Overview

Use an unexported, zero-width, non-comparable field to prevent clients from comparing a `struct`:

```go
type S struct {
	_ [0]func()
	...
}
```

Another way is to define a `nocmp` type and embed it into structs:

```go
type nocmp [0]func()

type S struct {
	nocmp
	...
}
```

# References

- [GopherCon 2019: Jonathan Amsterdam - Detecting Incompatible API Changes - YouTube](https://youtu.be/JhdL5AkH-AQ?si=xcfoMoBtkerctD7O)
- [atomic/nocmp.go at master · uber-go/atomic · GitHub](https://github.com/uber-go/atomic/blob/master/nocmp.go)
