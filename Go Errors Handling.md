---
tags: Go 
---

# Error Types

The table bellow summarizes options for declaring errors:

| Error matching | Error Message | Guidance                          |
| -------------- | ------------- | --------------------------------- |
| No             | static        | `errors.New`                      |
| No             | dynamic       | `fmt.Errorf`                      |
| Yes            | static        | top-level `var` with `errors.New` |
| Yes            | dynamic       | custom `error` type               |

# Error Wrapping

There are three main options for propagating errors if a call fails:

1. Return the original error as-is if there is no additional context to add
2. Add context with `fmt.Errorf` and the `%w` verb if the caller should have access to the underlying error. It creates potential coupling as it makes the source error available for the caller. So for cases where the wrapped error is a known `var` or type, document and test it as part of your function's contract
3. Add context with `fmt.Errorf` and the `%v` verb to obfuscate the underlying error. Callers will be unable to match it, but you can switch to `%w` in the future if needed

# Handle Errors Once

Handling an error multiple times, e.g. log and return, can lead to situations in which we have multiple log lines for a single error, making debugging more challenging, especially in concurrent execution

Therefore, in most situations, an error should be handled only once. You have to choose between logging or returning an error. In many cases, error wrapping is the solution as it allows you to provide additional context to an error and return the source error

# Equality of `errors.New` Errors

Separate but otherwise seemingly identical errors created by `errors.New` are not equal according to the `==` operator:

```go
const message = "A custom error message"

func main() {
	var e1, e2 error

	e1 = errors.New(message)
	e2 = errors.New(message)

	fmt.Println(e1 == e2) // false

	// fmt.Errorf delegates its error creation to errors.New and so shares the same behaviour.
	e1 = fmt.Errorf(message)
	e2 = fmt.Errorf(message)

	fmt.Println(e1 == e2) //false
}
```

The Language Specification says:

> - Interface values are comparable. Two interface values are equal if they have identical dynamic types and equal dynamic values or if both have value nil.
> - Struct values are comparable if all their fields are comparable. Two struct values are equal if their corresponding non-blank fields are equal.
> - Pointer values are comparable. Two pointer values are equal if they point to the same variable or if both have value nil. Pointers to distinct zero-size variables may or may not be equal. 

The *dynamic value* returned by `errors.New` is a *pointer* to a struct:

```go
// New returns an error that formats as the given text.
// Each call to New returns a distinct error value even if the text is identical.
func New(text string) error {
	return &errorString{text}
}

// errorString is a trivial implementation of error.
type errorString struct {
	s string
}

func (e *errorString) Error() string {
	return e.s
}
```

An `errors.New` method returns a pointer to a new variable, irrespective of whether the string argument is the same as a previously supplied argument, and pointers are not equal unless they point to the same variable

# References

- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
- [GitHub - uber-go/guide: The Uber Go Style Guide.](https://github.com/uber-go/guide/tree/master)
- [Don’t just check errors, handle them gracefully | Dave Cheney](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)
- [Why are Go's identical errors not equal? · GitHub](https://gist.github.com/fospathi/1e6f5aea622abb52bddc9bcb1ffee858)
