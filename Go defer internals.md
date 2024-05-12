---
tags: Go TODO
---

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

The solution is to add another surrounding function to execute the `defer` calls during each iteration. For example, using a closure:

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

# Defer Arguments Evaluation

Each time a `defer` statement executes, the function value and parameters to the call are **evaluated right-away** and saved anew but the actual function is not invoked

Deferred functions are invoked immediately before the surrounding function returns, in the reverse order they were deferred. That is, deferred functions are executed **after** any result parameters are set by that return statement but **before** the function returns to its caller

If a deferred function value evaluates to `nil`, execution panics when the function is invoked, not when the "defer" statement is executed

If the deferred function has any return values, they are discarded when the function completes

If we want to evaluate arguments to `defer` **during** deferred function execution we call a closure as a `defer` statement. The arguments passed to a `defer` function are still evaluated right away. But the variables referenced by a `defer` closure are evaluated during the closure execution

For example:

```go
var v int
defer func() {
	f(v) // initial deffered function call
}()
```

Here, we wrap the call to both `f` within a closure. This closure references the `v` from outside its body. Therefore, `v` is evaluated once the closure is executed, not when we call `defer`

# References

- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
