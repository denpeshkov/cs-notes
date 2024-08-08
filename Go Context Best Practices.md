---
tags:
  - Go
---

# Context in Structs

Contexts should not be stored inside a struct type, but instead passed to each method as a first parameter

Storing context in a struct prevents callers from passing individual contexts to methods. This results in a server whose requests lack individual contexts, making it impossible to properly handle cancellation. It also prevents propagation of trace IDs or other values from the caller. Additionally, the API becomes more confusing since it is unclear which methods the stored context applies to

There are exceptions to this rule. For example, satisfying the interface `io.Reader` or `io.Writer` is a good reason to put a context in a struct.  But to avoid issues discussed above, binding to context should be done locally to the use:

```go
func (s *S) IO(ctx context.Context) io.ReadWriter {
	// return a struct that implements Read and Write on s using ctx
}
```

Then code can use `s.IO(ctx)` as an `io.Reader` or `io.Writer`:

```go
func Foo(ctx context.Context, s *S) {
	fmt.Fprintln(s.IO(ctx), "hello, world")
}
```

# Avoiding Allocations for Context Keys

The following values can be used as context key values to avoid allocation when passing a value as `any` in a `context.WithValue` call:

- Small integers. Using a small integer (0 to 255) as a context key doesn't cause allocation due to [interface conversion optimization](Go%20Interfaces%20Internals.md#Heap%20Allocations%20and%20Escape%20Analysis)
- Constant value
- Pointer value
- [Interface value](Go%20Interfaces%20Internals.md)

# References

- [Contexts and structs - The Go Programming Language](https://go.dev/blog/context-and-structs)
- [Why context.Context not in structs ?](https://groups.google.com/g/golang-nuts/c/xRbzq8yzKWI)
- [context package - Go Packages](https://pkg.go.dev/context)
- [Go Concurrency Patterns: Context - The Go Programming Language](https://go.dev/blog/context)
