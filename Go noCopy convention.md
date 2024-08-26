---
tags:
  - Go
---

# Overview

The noCopy convention is a way to tell linters, e.g. `go vet`, that a structure should not be copied

Define a `noCopy` type that implements type `sync.Locker`:

```go
// noCopy may be added to structs which must not be copied after the first use.
// Note that it must not be embedded, due to the Lock and Unlock methods.
type noCopy struct{}

// Lock is a no-op used by -copylocks checker from `go vet`.
func (*noCopy) Lock()   {}
func (*noCopy) Unlock() {}
```

Add `noCopy` as a field to the struct that must no be copied:

```go
type S struct {
	_ noCopy
	...
}
```

# References

- [Source code: type.go](https://github.com/golang/go/blob/master/src/sync/atomic/type.go)
- [noCopy convention - Unskilled](https://unskilled.blog/posts/nocopy-convention/)
