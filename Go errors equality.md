---
tags: Go
---

# Equality of Errors in Go

## Errors Created by `errors.New` Function

Separate but otherwise seemingly identical errors created by `errors.New` are not equal according to the `==` operator

Consider the following program:

```go
package main

import (
	"errors"
	"fmt"
)

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

## Custom Errors

Now consider the following program:

```go
package main

import "fmt"

type CustomError struct {
	Message string
}

func (ce CustomError) Error() string {
	return ce.Message
}

const message = "Custom error message 1"

func main() {
	var e1, e2 error

	e1 = CustomError{Message: message}
	e2 = CustomError{Message: message}

	fmt.Println(e1 == e2) // true
	fmt.Println(error(e1) == error(e2)) //true
}
```

# Explanation

The Language Specification says:

> - Interface values are comparable. Two interface values are equal if they have identical dynamic types and equal dynamic values or if both have value nil.
> - Struct values are comparable if all their fields are comparable. Two struct values are equal if their corresponding non-blank fields are equal.
> - Pointer values are comparable. Two pointer values are equal if they point to the same variable or if both have value nil. Pointers to distinct zero-size variables may or may not be equal. 

The *dynamic value* of our custom error type is a **struct**, so `==` works fine

The *dynamic value* returned by `errors.New` is a **pointer to a struct**:

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

`New` returns a pointer to a new variable, irrespective of whether the string argument is the same as a previously supplied argument, and pointers are not equal unless they point to the same variable

# References

- [Why are Go's identical errors not equal? · GitHub](https://gist.github.com/fospathi/1e6f5aea622abb52bddc9bcb1ffee858)
