---
tags: Go TODO
---

# Range Loops

This range loop:

```go
for i,v := range exp {...}
```

after compilation is similar to:

```go
expT := exp
lenT := len(exp)
for iT = 0; iT < leT; iT++ {
	i = iT
	vT := expT[iT]
	v = vT
	// original body
}
```

From this, we can note a couple of important details:

- The range loop expression `exp` is evaluated only once and is assigned to a temporary variable (a copy of the original)
- The variable `v` is a single variable that takes different values in each loop iteration
- The variable `v` is a copy of the loop element's value
- The variable `i` is a single variable that takes different values in each loop iteration
- The variable `i` is a copy of the loop index's value

# References

 - [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
 - [Go Range Loop Internals](https://garbagecollected.org/2017/02/22/go-range-loop-internals/)
 - [The Go Programming Language Specification - Range Clause](https://go.dev/ref/spec#RangeClause)
 - [Range · golang/go Wiki · GitHub](https://github.com/golang/go/wiki/Range)
 - [CommonMistakes · golang/go Wiki · GitHub](https://github.com/golang/go/wiki/CommonMistakes)
