---
tags:
  - Go
---

# When to Use Interfaces

Interfaces should be used in the following situations:

1. Common behavior. For example, `sort.Interface`, `io.Reader` that are used by multiple functions
2. Decoupling. Allows to use mocks in unit tests; use different algorithm without code changes
3. Restricting behavior. Interface on the client side restricts the set of methods that can be called

As a general rule:

1. We should create an interface when we *need* it, not when we *foresee* that we could need it
2. Interfaces should be as small as possible, because the bigger the interface, the weaker the abstraction

# Accept Interfaces and Return Structs

Accepting interfaces rather than concrete types makes functions more flexible and decoupled, allowing them to work with any type that satisfies the interface rather than a single specific type. It allows us to provide mocks for testing and swap implementations

In most cases we should return concrete implementations, not interfaces. Returning concrete types means that we can add new methods without [breaking backward compatibility](https://youtu.be/JhdL5AkH-AQ?si=Cs0TDV7SjeSYAEki&t=1430). Also, clients can use all the methods of the type, not just the one defined in the interface

## When to Return an Interface

There are exceptions in the standard library, for example `error` interface and `io.LimitReader` function that returns `io.Reader` instead of `io.LimitedReader`. Although, it seems, that returning `io.Reader` instead of a concrete `io.LimitReader` was a mistake by Go designers

If a function needs to be able to return multiple implementations, it must return an interface

If a type exists only to implement an interface and will never have exported methods beyond that interface, there is no need to export the type itself. Exporting just the interface makes it clear the value has no interesting behavior beyond what is described in the interface. It also avoids the need to repeat the documentation on every instance of a common method. For example, if we have a `Router` struct implementing `http.Handler`, it's better to return `http.Handler` rather than `Router`

# Interface on the Producer Side

Interfaces generally should be created on the client side, not the producer side

Defining the interface on the producer side forces clients to depend on all the methods of the interface. Instead, the client should define the interface with only the methods it needs. This relates to the concept of the [interface segregation principle](https://dave.cheney.net/2016/08/20/solid-go-design#Interface%20Segregation%20Principle), which states that no client should be forced to depend on methods it doesn't use

Note that interfaces sometimes can be used on the producer side. For example, `io` package exports interfaces because it also needs to export generic-use functions like `io.Copy`: `func Copy(dst Writer, src Reader) (int64, error)`

# Interface Embedding

## Interface Wrapping Method Erasure

#TODO

- [Fixing Interface Erasure in Go | doxsey.net](https://www.doxsey.net/blog/fixing-interface-erasure-in-go/)
- [Interface wrapping method erasure | by Jack Lindamood | Medium](https://medium.com/@cep21/interface-wrapping-method-erasure-c523b3549912)
- [The trouble with optional interfaces | Mero's Blog](https://blog.merovius.de/posts/2017-07-30-the-trouble-with-optional-interfaces/)

## Degrading Capability with a More Restrictive Interface

We can embed an interface in a struct that will act as a wrapper for the original value  
This technique can be used to restrict the interfaces the original value implements

For example, the `io.ReaderFrom` is defined as:

```go
type ReaderFrom interface {
    ReadFrom(r Reader) (n int64, err error)
}
```

The `os.File` type implements this interface, and inside its `ReadFrom` method, it invokes:

```go
func genericReadFrom(f *File, r io.Reader) (int64, error) {
	return io.Copy(onlyWriter{f}, r)
}
```

It uses `io.Copy` to copy from `r` to `f`, but instead of passing `f` directly, it wraps it in an `onlyWriter` struct:

```go
type onlyWriter struct {
	io.Writer
}
```

To understand why, we should look at what `io.Copy` does. If its destination implements `io.ReaderFrom`, it will invoke `ReadFrom`. But this brings us back in a circle, since we ended up in `io.Copy` when `File.ReadFrom` was called. This causes an infinite recursion

By wrapping `f` in the call to `io.Copy`, what `io.Copy` gets is not a type that implements `io.ReaderFrom`, but only a type that implements `io.Writer`. It will then call the `Write` method of our `File` and avoid the infinite recursion trap of `ReadFrom`

Note that, we can use an anonymous wrapper struct:

```go
return io.Copy(struct{ io.Writer }{f}, r)
```

## Method Interception

We can define a struct with an embedded interface to intercept its method(s)  
When such a struct is initialized with a proper value implementing the interface for the embedded field, it 'inherits' all the methods of that value  
The key insight is that we can intercept any method we wish, leaving all the others intact

For example, we can implement middleware to provide logging. To achieve this, we need to capture the HTTP status code from the handler:

```go
type statusRecorder struct {
	http.ResponseWriter
	status int
}

func (rec *statusRecorder) WriteHeader(code int) {
	rec.status = code
	rec.ResponseWriter.WriteHeader(code)
}

func logware(h http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		// Initialize the status to 200 in case WriteHeader is not called
		rec := statusRecorder{w, 200}
		h.ServeHTTP(&rec, r)
		log.Printf("response status: %v\n", rec.status)
	}
}
```

We can use the middleware as follows:

```go
func main() {
	http.ListenAndServe(":8080",
		logware(
			http.HandlerFunc(handle),
		),
	)
}

func handle(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(201)
	w.Write([]byte("Accepted"))
}
```

# References

- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
- [SOLID Go Design | Dave Cheney](https://dave.cheney.net/2016/08/20/solid-go-design)
- [Go Proverbs](https://go-proverbs.github.io)
- [GopherCon 2023: Dylan Bourque - Clean Up Your GOOOP: How to Break OOP Muscle Memory - YouTube](https://www.youtube.com/watch?v=tautMDOlEFs)
- [GopherCon 2019: Jonathan Amsterdam - Detecting Incompatible API Changes - YouTube](https://youtu.be/JhdL5AkH-AQ?si=NwJOVwfpPXFxmgBG)
- [Interface pollution in Go · rakyll.org](https://rakyll.org/interface-pollution/)
- [Embedding in Go: Part 3 - interfaces in structs - Eli Bendersky's website](https://eli.thegreenplace.net/2020/embedding-in-go-part-3-interfaces-in-structs/)
- [Golang Tip: Wrapping http.ResponseWriter for Middleware](https://upgear.io/blog/golang-tip-wrapping-http-response-writer-for-middleware/)
- [Designing Go Libraries | Abhinav Gupta](https://abhinavg.net/2022/12/06/designing-go-libraries/#interfaces-are-forever)
- [Avtok. Interface Upgrades in Go](https://avtok.com)
