---
tags: Go
---

# Receiver Type

A receiver should be a value:

- If the receiver is a map, function, channel or slice that is not resliced/reallocated
- If the receiver is a small array or struct that is naturally a value type without mutable fields, e.g. `time.Time`. A value receiver can reduce the amount of garbage that can be generated; if a value is passed to a value method, an on-stack copy can be used instead of allocating on the heap
- If we have to enforce a receiver's immutability
- If the receiver is a basic type

A receiver should be a pointer:

- If the method needs to mutate the receiver
- If the receiver contains a field that cannot be copied, e.g. `sync.Mutex`
- If the receiver is a struct, array or slice and any of its elements is a pointer to something that might be mutating, as it will make the intention clearer to the reader
- If the receiver is a large object

When in doubt opt for the pointer receiver
Do not mix

# References

- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
- [Go Code Review Comments - The Go Programming Language](https://go.dev/wiki/CodeReviewComments)
- [receivers | Dave Cheney](https://dave.cheney.net/tag/receivers)
