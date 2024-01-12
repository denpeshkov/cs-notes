---
tags: Go
---

# Range Loops

## Before Go 1.22

This range loop:

```go
for i, v := range exp {...}
```

after compilation is semantically equivalent to:

```go
var i Type
var v Type

expT := exp
lenT := len(exp)
for iT := 0; iT < lenT; iT++ {
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

## After Go 1.22 (Loopvar Experiment)

This 3-clause for loop:

```go
for i := 0; i < N; i++ {...}
```

after compilation is semantically equivalent to:

```go
iT := 0
first := true
for {
	i := iT
	if first {
		first = false
	} else {
		i++
	}
	if !(i < N) {
		break
	}
	// original body
	iT = i
}
```

This range loop:

```go
for i, v := range exp {...}
```

after compilation is semantically equivalent to:

```go
var iT Type
var vT Type

for iT, vT = range exp {
	i := iT
	v := vT
	// original body
}
```

From this, we can note a couple of important details:

- The range loop expression `exp` is evaluated only once and is assigned to a temporary variable (a copy of the original)
- The variable `v` is a newly declared variable at each iteration
- The variable `v` is a copy of the loop element's value
- The variable `i` is a newly declared variable at each iteration
- The variable `i` is a copy of the loop index's value

# Map Iteration Guarantees

- The iteration order over maps is not specified and is not guaranteed to be the same from one iteration to the next
- If a map entry that has not yet been reached is removed during iteration, the corresponding iteration value will not be produced
- If a map entry is created during iteration, that entry may be produced during the iteration or may be skipped. The choice may vary for each entry created and from one iteration to the next
- If the map is `nil`, the number of iterations is 0

# Defer in Loops

The `defer` statement delays a call's execution until the surrounding function returns

For example, in this code:

```go
func f(ch <-chan string) error {
    for path := range ch {
        file, err := os.Open(path)
        if err != nil {
            return err
        }

        defer file.Close()

        // Do something with file
    }
    return nil
}
```

The `defer` calls are executed not during each loop iteration but when the `readFiles` function returns  
If `readFiles` doesn't return, the file descriptors will be kept open forever, causing leaks

The solution is to add another surrounding function to execute the `defer` calls during each iteration

For example, using closure:

```go
func readFiles(ch <-chan string) error {
    for path := range ch {
        err := func() error {
            // ...
            defer file.Close()
            // ...
        }()
        if err != nil {
	        return err
	    }
	}
	return nil
}
```

# References

 - [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
 - [Go Range Loop Internals](https://garbagecollected.org/2017/02/22/go-range-loop-internals/)
 - [Range · golang/go Wiki · GitHub](https://github.com/golang/go/wiki/Range)
 - [CommonMistakes · golang/go Wiki · GitHub](https://github.com/golang/go/wiki/CommonMistakes)
 - [LoopvarExperiment · golang/go Wiki · GitHub](https://github.com/golang/go/wiki/LoopvarExperiment)
 - [Fixing For Loops in Go 1.22 - The Go Programming Language](https://go.dev/blog/loopvar-preview)
 - [CommonMistakes · golang/go Wiki · GitHub](https://github.com/golang/go/wiki/CommonMistakes)
 - [The Go Programming Language Specification - The Go Programming Language](https://tip.golang.org/ref/spec#For_statements)
 - [Proposal: Less Error-Prone Loop Variable Scoping](https://go.googlesource.com/proposal/+/master/design/60078-loopvar.md)
 - [The Go Programming Language Specification - The Go Programming Language](https://tip.golang.org/ref/spec)
