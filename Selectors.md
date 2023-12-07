---
tags:
  - Go
---

# Defined Pointer Type

If the type of `x` is a [defined](https://go.dev/ref/spec#Type_definitions) pointer type and `(*x).f` is a valid selector expression denoting a field (but not a method), `x.f` is shorthand for `(*x).f`

For example, given:

```go
type V struct {
	i int
}
func (v V) f() {}

type P *V

var v V
var p *V = &v // pointer using standard pointer type
var p2 P = &v // pointer using defined type
```

These calls are legal:

```go
p.f()
(*p).f()
p.i
(*p).i

(*p2).f()
p2.i
(*p2).i
```

This call is illegal:

```go
p2.f()
```

The call is illegal because the method set of `P` does not inherit any methods bound to type `V`. However, we can access fields of type `V`

# References

- [The Go Programming Language Specification - The Go Programming Language](https://go.dev/ref/spec#Selectors)
- [Selectors in Go. Expression foo.bar can mean two things… | by Michał Łowicki | golangspec | Medium](https://medium.com/golangspec/selectors-in-go-c53a016702cf)
- [The Go Programming Language Specification - The Go Programming Language](https://go.dev/ref/spec#Type_definitions)
