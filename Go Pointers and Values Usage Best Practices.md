---
tags:
  - Go
---

# Receiver Type

- If the receiver is a map, func or chan, don't use a pointer to them. If the receiver is a slice and the method doesn't reslice or reallocate the slice, don't use a pointer to it
- If the method needs to mutate the receiver, the receiver must be a pointer
- If the receiver is a struct that contains a sync.Mutex or similar synchronizing field, the receiver must be a pointer to avoid copying
- If the receiver is a large struct or array, a pointer receiver is more efficient. How large is large? Assume it's equivalent to passing all its elements as arguments to the method. If that feels too large, it's also too large for the receiver
- Can function or methods, either concurrently or when called from this method, be mutating the receiver? A value type creates a copy of the receiver when the method is invoked, so outside updates will not be applied to this receiver. If changes must be visible in the original receiver, the receiver must be a pointer
- If the receiver is a struct, array or slice and any of its elements is a pointer to something that might be mutating, prefer a pointer receiver, as it will make the intention clearer to the reader
- If the receiver is a small array or struct that is naturally a value type (for instance, something like the time.Time type), with no mutable fields and no pointers, or is just a simple basic type such as int or string, a value receiver makes sense. A value receiver can reduce the amount of garbage that can be generated; if a value is passed to a value method, an on-stack copy can be used instead of allocating on the heap. (The compiler tries to be smart about avoiding this allocation, but it can't always succeed.) Don't choose a value receiver type for this reason without profiling first
- Don't mix receiver types. Choose either pointers or struct types for all available methods
- Finally, when in doubt, use a pointer receiver

# References

- [go - Pointers vs. values in parameters and return values - Stack Overflow](https://stackoverflow.com/a/23551970/15600693)
- [Bad Go: slices of pointers · Phil Pearl's Blog](https://philpearl.github.io/post/bad_go_slice_of_pointers/)
- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
- [Go Wiki: Go Code Review Comments - The Go Programming Language](https://go.dev/wiki/CodeReviewComments#receiver-type)
- [Effective Go - The Go Programming Language](https://go.dev/doc/effective_go#pointers_vs_values)
- [Should methods be declared on T or \*T | Dave Cheney](https://dave.cheney.net/2016/03/19/should-methods-be-declared-on-t-or-t)
- [Someone on here asked “should I use slices of pointers” in response to my last post. So here is my view on it. : r/golang](https://www.reddit.com/r/golang/comments/15wb1ru/someone_on_here_asked_should_i_use_slices_of/)
