---
tags: Go
---

# Usage

Interfaces should be used in the following situations:

1. Common behavior. For example `sort.Interface` that is used by multiple functions
2. Decoupling. Allows to use mocks in unit tests; use different algorithm without code changes
3. Restricting behavior. Interface on the client side restricts the set of methods that can be called

Interfaces should not be used in the following situations:

1. Interface on the producer side
2. Returning interfaces
3. `Any` interface

As a general rules:

1. We should create an interface when we *need* it, not when we *foresee* that we could need it
2. We should accept interfaces and return structs
3. Interfaces should be as small as possible, because *the bigger the interface, the weaker the abstraction*

# Interface on the Producer Side

Interfaces generally should be created on the client side, not the producer side (library)

It's not up to the producer to force a given abstraction for all the clients. Instead, it's up to the client to decide whether it needs some form of abstraction and then determine the best abstraction level for its needs  
It relates to the concept of the [Interface Segregation Principle](https://dave.cheney.net/2016/08/20/solid-go-design#Interface%20Segregation%20Principle), which states that no client should be forced to depend on methods it doesn't use

For example, instead of:

```go
// Save writes the contents of doc to the file f.
func Save(f *os.File, doc *Document) error
```

write:

```go
// Save writes the contents of doc to the supplied Writer.
func Save(w io.Writer, doc *Document) error
```

---

Note that interfaces sometimes can be used on the producer side. For example `encoding` package defined interfaces used by `encoding/json` and `encoding/binary`

It's not a mistake because Go designers *knew* (not foresee) that creating these abstractions up front was valuable

# Returning Interfaces

In most cases we should return concrete implementations, not interfaces

Returning concrete types means that we can add new functionality without [breaking compatibility](https://youtu.be/JhdL5AkH-AQ?si=Cs0TDV7SjeSYAEki&t=1430) . Also, clients can use all the methods of the type, not just the one defined in interface

---

There are exceptions, for example `error` interface and `io.LimitReader` function that returns `io.Reader` instead of `io.LimitedReader`

The `io.Reader` is an up-front abstraction. It's not one defined by clients, but it's one that is forced because the language designers *knew* in advance that this level of abstraction would be helpful

# `Any` Interface

`Any` shouldn't generally be used, because we can loose benefits of static checking

It can be used when we do need any type, for example: `encoding/json`, `sql`, `fmt`

# References

- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
- [SOLID Go Design | Dave Cheney](https://dave.cheney.net/2016/08/20/solid-go-design)
- [Go Proverbs](https://go-proverbs.github.io)
