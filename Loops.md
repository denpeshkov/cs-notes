---
tags:
  - Go
---

# For Loop

The iterator variable `i` is a single variable that takes different values in each loop iteration

# For-each Loop

The range loop expression is evaluated only once (if only index is used and `len(x)` is constant, it is not evaluated at all)

The for-range loop after compilation is similar to:

```Go
// The loop we generate:
len_temp := len(range_expr)
range_temp := range_expr
for index_temp = 0; index_temp < len_temp; index_temp++ {
	value_temp = range_temp[index_temp]
	index = index_temp
	value = value_temp
	original body
}
```

Three important details:

- The range expression we iterate over is assigned to a temporary variable (a copy of the original)
- The variable `v` used in a body is a single variable that takes different values in each loop iteration
- The iterator variable `i` is a single variable that takes different values in each loop iteration

# References

- [Go Range Loop Internals](https://garbagecollected.org/2017/02/22/go-range-loop-internals/)
- [The Go Programming Language Specification - Range Clause](https://go.dev/ref/spec#RangeClause)
- [Range · golang/go Wiki · GitHub](https://github.com/golang/go/wiki/Range)
- [CommonMistakes · golang/go Wiki · GitHub](https://github.com/golang/go/wiki/CommonMistakes)
