---
tags: Go
---

# Error Types

The table bellow summarizes options for declaring errors:

| Error matching? | Error Message | Guidance |
| --- | --- | --- |
| No | static | [`errors.New`](https://golang.org/pkg/errors/#New) |
| No | dynamic | [`fmt.Errorf`](https://golang.org/pkg/fmt/#Errorf) |
| Yes | static | top-level `var` with [`errors.New`](https://golang.org/pkg/errors/#New) |
| Yes | dynamic | custom `error` type |

Note that if you export error variables or types from a package, they will become part of the public API of the package

# Error Wrapping

There are three main options for propagating errors if a call fails:

- return the original error as-is
- add context with `fmt.Errorf` and the `%w` verb
- add context with `fmt.Errorf` and the `%v` verb

Return the original error as-is if there is no additional context to add

Use `%w` if the caller should have access to the underlying error. It creates potential coupling as it makes the source error available for the caller. So for cases where the wrapped error is a known `var` or type, document and test it as part of your function's contract

Use `%v` to obfuscate the underlying error. Callers will be unable to match it, but you can switch to `%w` in the future if needed

# Handle Errors Once

Handling an error multiple times can cause situations where the same error is logged multiple times, making debugging harder (especially in concurrent execution)

Therefore, in most situations, an error should be handled only once. You have to choose between logging or returning an error

In many cases, error wrapping is the solution as it allows you to provide additional context to an error and return the source error

# Handling Deferred Errors

Don't ignore an error returned by a `defer` function. Either handle it directly or propagate it to the caller, depending on the context

For example:

```go
func getBalance(db *sql.DB, clientID string) (balance float32, err error) {
	rows, err := db.Query(query, clientID)
	if err != nil {
		return 0, err
	}
	defer func() {
		closeErr := rows.Close()
		if err != nil {
			if closeErr != nil {
				log.Printf("failed to close rows: %v", closeErr)
			}
			return
		}
		err = closeErr
	}()

	// Use rows
	return 0, nil
}
```

Here, we are using a **named returned value** to change the returned error from the enclosing function

# References

- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
- [GitHub - uber-go/guide: The Uber Go Style Guide.](https://github.com/uber-go/guide/tree/master)
- [Don’t just check errors, handle them gracefully | Dave Cheney](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)
- [Defer, Panic, and Recover - The Go Programming Language](https://go.dev/blog/defer-panic-and-recover)
