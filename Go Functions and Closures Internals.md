---
tags: Go
---

# Capturing Variables

Closures may refer to variables defined in a surrounding function. Those variables are then shared between the surrounding function and the closure, and they survive as long as they are accessible

Closed over variables **semantically** are always captured by address, meaning they escape to the heap  
However, internally, a closure can be passed a copy of a value (not the address) to reduce GC pressure. For example, if the value is a constant or is never modified

# References

- [Go internals: capturing loop variables in closures - Eli Bendersky's website](https://eli.thegreenplace.net/2019/go-internals-capturing-loop-variables-in-closures/)
- [Go internals: capturing loop variables in closures : r/golang](https://www.reddit.com/r/golang/comments/d6pwu1/go_internals_capturing_loop_variables_in_closures/)
- [How Golang Closures are layed out in memroy? : r/golang](https://www.reddit.com/r/golang/comments/on2vo9/how_golang_closures_are_layed_out_in_memroy/)
- [How are Go closures layed out in memory? - Stack Overflow](https://stackoverflow.com/questions/51489323/how-are-go-closures-layed-out-in-memory)
