---
tags:
  - Go
---

# When to Use Interfaces

Interfaces should be used in the following situations:

1. Common behavior. For example, `sort.Interface`, `io.Reader` that are used by multiple functions
2. Decoupling. Allows to use mocks in unit tests; use different algorithm without code changes
3. Restricting behavior. Interface on the client side restricts the set of methods that can be called

As a general rule:

1. We should create an interface when we *need* it, not when we *foresee* that we could need it
2. We should accept interfaces and return structs
3. Interfaces should be as small as possible, because *the bigger the interface, the weaker the abstraction*
4. We should return structs and accept interfaces

# Interface on the Producer Side Anti-Pattern

Interfaces generally should be created on the client side, not the producer side

Defining the interface on the producer side forces clients to depend on all the methods of the interface. Instead, the client should define the interface with only the methods it needs. This relates to the concept of the [interface segregation principle](https://dave.cheney.net/2016/08/20/solid-go-design#Interface%20Segregation%20Principle), which states that no client should be forced to depend on methods it doesn't use

Note that interfaces sometimes can be used on the producer side. For example `encoding` package defines interfaces used by `encoding/json` and `encoding/binary`. It's not a mistake because Go designers *knew* (not foresee) that creating these abstractions up front was valuable

# Returning Interface Anti-Pattern

In most cases we should return concrete implementations, not interfaces

Returning concrete types means that we can add new methods without [breaking backward compatibility](https://youtu.be/JhdL5AkH-AQ?si=Cs0TDV7SjeSYAEki&t=1430). Also, clients can use all the methods of the type, not just the one defined in interface

There are exceptions in standard library, for example `error` interface and `io.LimitReader` function that returns `io.Reader` instead of `io.LimitedReader`. Although, it seems, that returning `io.Reader` instead of a concrete `io.LimitReader` was a mistake by Go designers

# References

- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
- [SOLID Go Design | Dave Cheney](https://dave.cheney.net/2016/08/20/solid-go-design)
- [Go Proverbs](https://go-proverbs.github.io)
- [GopherCon 2023: Dylan Bourque - Clean Up Your GOOOP: How to Break OOP Muscle Memory - YouTube](https://www.youtube.com/watch?v=tautMDOlEFs)
- [GopherCon 2019: Jonathan Amsterdam - Detecting Incompatible API Changes - YouTube](https://youtu.be/JhdL5AkH-AQ?si=NwJOVwfpPXFxmgBG)
