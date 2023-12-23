---
tags: Go
---

# Degrading Capability with a More Restrictive Interface

We can embed an interface in a struct that will act as a wrapper for the original value  
This technique can be used to restrict the interfaces the original value implements

For example, the `io.ReaderFrom` is defined as:

```go
type ReaderFrom interface {
    ReadFrom(r Reader) (n int64, err error)
}
```

The `os.File` type implements this interface, and inside its `ReadFrom` method, it invokes:

```go
func genericReadFrom(f *File, r io.Reader) (int64, error) {
	return io.Copy(onlyWriter{f}, r)
}
```

It uses `io.Copy` to copy from `r` to `f`, but instead of passing `f` directly, it wraps it in an `onlyWriter` struct:

```go
type onlyWriter struct {
	io.Writer
}
```

To understand why, we should look at what `io.Copy` does. If its destination implements `io.ReaderFrom`, it will invoke `ReadFrom`. But this brings us back in a circle, since we ended up in `io.Copy` when `File.ReadFrom` was called. This causes an infinite recursion

By wrapping `f` in the call to `io.Copy`, what `io.Copy` gets is not a type that implements `io.ReaderFrom`, but only a type that implements `io.Writer`. It will then call the `Write` method of our `File` and avoid the infinite recursion trap of `ReadFrom`

Note that, we can use an anonymous wrapper struct:

```go
return io.Copy(struct{ io.Writer }{f}, r)
```

# References

- [Embedding in Go: Part 3 - interfaces in structs - Eli Bendersky's website](https://eli.thegreenplace.net/2020/embedding-in-go-part-3-interfaces-in-structs/)
