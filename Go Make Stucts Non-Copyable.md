---
tags: Go
---

# Overview

Define a `noCopy` type that implements type `sync.Locker`:

```go
// noCopy may be added to structs which must not be copied after the first use.
// Note that it must not be embedded, due to the Lock and Unlock methods.
type noCopy struct{}

// Lock is a no-op used by -copylocks checker from `go vet`.
func (*noCopy) Lock()   {}
func (*noCopy) Unlock() {}
```

# References

- [Source code: type.go](https://github.com/golang/go/blob/master/src/sync/atomic/type.go)
