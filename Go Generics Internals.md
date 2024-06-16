---
tags:
  - Go
  - TODO
---

# Performance

- **DO** try to de-duplicate identical methods that take a `string` and a `[]byte` using an `interface {~string | ~[]byte}` constraint. The generated shape instantiation is very close to manually writing two almost-identical functions
- **DO** use generics in data structures. Removing the type assertions and storing types _unboxed_ in a type-safe way makes these data structures both easier to use and more performant
- **DO** attempt to parametrize functional helpers by their callback types. In some cases, it may allow the Go compiler to flatten them
- **DO NOT** attempt to use Generics to de-virtualize or inline method calls. It doesn't work because there's a single shape for all pointer types that can be passed to the generic function; the associated method information lives in a runtime dictionary
- **DO NOT** pass an interface to a generic function. Because of the way shape instantiation works for interfaces, instead of de-virtualizing, you're adding another virtualization layer that involves a global hash table lookup for every method call. When dealing with Generics in a performance-sensitive context, use only pointers instead of interfaces
- **DO NOT** rewrite interface-based APIs to use Generics. Given the current constraints of the implementation, any code that currently uses non-empty interfaces will behave more predictably, and will be simpler, if it continues using interfaces. When it comes to method calls, Generics devolve pointers into twice-indirect interfaces

# References

- [generics-implementation-dictionaries-go1.18.md](https://github.com/golang/proposal/blob/master/design/generics-implementation-dictionaries-go1.18.md)
- [Faster sorting with Go generics - Eli Bendersky's website](https://eli.thegreenplace.net/2022/faster-sorting-with-go-generics/)
- [Type Parameters Proposal](https://go.googlesource.com/proposal/+/refs/heads/master/design/43651-type-parameters.md)
- [Generics can make your Go code slower](https://planetscale.com/blog/generics-can-make-your-go-code-slower)
- [When To Use Generics - The Go Programming Language](https://go.dev/blog/when-generics)
