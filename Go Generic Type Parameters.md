---
tags:
  - Go
---

# Usage of `~[]T`

Let's consider these functions:

```go
func Clone1[S ~[]E, E any](s S) S {...}
func Clone2[E any](s []E) []E {...}

func Reverse1[S ~[]E, E any](s S) {...}
func Reverse2[E any](s []E) {...}
```

## Clone

In the case of `Clone` functions we should use `Clone1` because `Clone2` doesn't allow to use named slice types:

```go
type MySlice []string
func (s MySlice) String() string { return strings.Join(s, "+") }
```

If we use `Clone2` we get a compiler error:

```Go
func PrintSorted(ms MySlice) string {
    c := Clone1(ms)
    slices.Sort(c)
    return c.String() // FAILS TO COMPILE
}
```

The [Go assignment rules](https://go.dev/ref/spec#Assignability) allow us to pass a value of type `MySlice` to a parameter of type `[]string`, so calling `Clone2` is fine. But `Clone2` will return a value of type `[]string`, not a value of type `MySlice`. The type `[]string` doesn't have a `String` method, so the compiler reports an error

## Reverse

In the case of `Reverse` functions we should also prefer `Reverse1`

`Reverse` doesn't return the slice, so it works even with named slice types such as `MySlice`

However, there is still a case where it makes a difference: When instantiating `Reverse1` you can only get a `func([]string)`, not a `func(MySlice)`. This matters when passing them to higher level functions:

```go
func Apply(ms MySlice) {
	Apply(ms, Reverse1) // Compiles
	Apply(ms, Reverse2) // Does not compile
	Apply[[]string](ms,Reverse2) // Compiles
}

func apply[T any](v T, f func(T)) {
	f(v)
}
```

We have a compile error for the second call to `Apply`: type `func[E any](s []E)` of `Reverse2` does not match inferred type `func(MySlice)` for `func(T)`

To solve this issue we must provide the type explicitly: `Apply[[]string](ms,Reverse2)`

# References

- [Deconstructing Type Parameters - The Go Programming Language](https://go.dev/blog/deconstructing-type-parameters)
- [slices: Use \~[]E throughout · Issue #60546 · golang/go · GitHub](https://github.com/golang/go/issues/60546)
- [slices: consistently use S \~[]E · golang/go@0a48e5c · GitHub](https://github.com/golang/go/commit/0a48e5cbfabd679eecdec1efa731692cd6babf83)
