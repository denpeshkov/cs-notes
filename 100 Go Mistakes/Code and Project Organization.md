---
tags: Go
---

# Interface Usage

Interfaces should be used in the following situations:

1. Common behavior. For example `sort.Interface` that is used by multiple functions
2. Decoupling. Allows to use mocks in unit tests; use different algorithm without code changes
3. Restricting behavior. Interface on the client side restricts the set of methods that can be called

Interfaces should not be used in the following situations:

1. Interface on the producer side
2. Returning interfaces
3. `Any` interface

As a general rules:

1. We should create an interface when we *need* it, not when we *foresee* that we could need it
2. We should accept interfaces and return structs
3. Interfaces should be as small as possible, because *the bigger the interface, the weaker the abstraction*

## Interface on the Producer Side

Interfaces generally should be created on the client side, not the producer side (library)

It's not up to the producer to force a given abstraction for all the clients. Instead, it's up to the client to decide whether it needs some form of abstraction and then determine the best abstraction level for its needs  
It relates to the concept of the [Interface Segregation Principle](https://dave.cheney.net/2016/08/20/solid-go-design#Interface%20Segregation%20Principle), which states that no client should be forced to depend on methods it doesn't use

For example, instead of:

```go
// Save writes the contents of doc to the file f.
func Save(f *os.File, doc *Document) error
```

write:

```go
// Save writes the contents of doc to the supplied Writer.
func Save(w io.Writer, doc *Document) error
```

---

Note that interfaces sometimes can be used on the producer side. For example `encoding` package defined interfaces used by `encoding/json` and `encoding/binary`

It's not a mistake because Go designers *knew* (not foresee) that creating these abstractions up front was valuable

## Returning Interfaces

In most cases we should return concrete implementations, not interfaces

Returning concrete types means that we can add new functionality without [breaking compatibility](https://youtu.be/JhdL5AkH-AQ?si=Cs0TDV7SjeSYAEki&t=1430) . Also, clients can use all the methods of the type, not just the one defined in interface

---

There are exceptions, for example `error` interface and `io.LimitReader` function that returns `io.Reader` instead of `io.LimitedReader`

The `io.Reader` is an up-front abstraction. It's not one defined by clients, but it's one that is forced because the language designers *knew* in advance that this level of abstraction would be helpful

## `Any` Interface

`Any` shouldn't generally be used, because we can loose benefits of static checking

It can be used when we do need any type, for example: `encoding/json`, `sql`, `fmt`

# Type Embedding

We shouldn't embed types that are private, otherwise their method get promoted and are accessible by the client

We should avoid embedding types in public structs  
Either with an embedded struct or an embedded interface, the embedded type places limits on the evolution of the type:

- Adding methods to an embedded interface is a breaking change
- Removing methods from an embedded struct is a breaking change
- Removing the embedded type is a breaking change
- Replacing the embedded type, even with an alternative that satisfies the same interface, is a breaking change

# Functional Options Pattern

Let's consider the example of setting a port for a server:

```go
func NewServer(addr string, port int) (*http.Server, error) {...}
```

- If the port isn't set, it uses the default one
- If the port is negative, it returns an error
- If the port is equal to 0, it uses a random port
- Otherwise, it uses the port provided by the client

There are couple of ways to deal with this configurations:

1. Config struct
2. Builder pattern
3. Using setters or exported fields directly
4. Functional options pattern

## Config Struct

We could use a config struct to represent optional configuration parameters:

```go
type Config struct {
	Port int
}

func NewServer(addr string, cfg Config) (*http.Server, error) {...}
```

However, this approach has some drawbacks:

- It can't distinguish between port that is equal to 0 and port that isn't set.
- Clients need to pass an empty struct for a default configuration

We could use pointers:

```go
type Config struct {
	Port *int
}

func NewServer(addr string, cfg Config) (*http.Server, error) {…}
```

That approach also has downsides:

- Clients need to create a variable to pass it's address as a pointer
- Clients need to pass an empty struct for a default configuration

## Using Setters or Exported Fields Directly

We could use directly exported fields (or setters) to config the server after instantiation. For example like an `http.Server` in the standard library

That approach also has downsides:

- The server may be in inconsistent state during configuration
- It also precludes the possibility of making a type immutable
- No validation at the instantiation time. Need to use a separate `Valid` method
- We can't return a different instance depending on the provided configuration
- Need setters to distinguish between port that is equal to 0 and port that isn't set
- It can be inconvenient to configure an object after creation, e.g. creating a middleware

## Builder Pattern

We could use the Builder Pattern:

```go
type Config struct {
	Port int
}

type ConfigBuilder struct {
	port *int
}

func (b *ConfigBuilder) Port(port int) *ConfigBuilder {
	b.port = &port
	return b
}

func (b *ConfigBuilder) Build() (Config, error) {
	cfg := Config{}

	if b.port == nil {
		cfg.Port = defaultHTTPPort
	} else {
		if *b.port == 0 {
			cfg.Port = randomPort()
		} else if *b.port < 0 {
			return Config{}, errors.New("port should be positive")
		} else {
			cfg.Port = *b.port
		}
	}

	return cfg, nil
}
```

This approach also has downsides:

- Client needs to pass an empty struct for a default configuration
- If builder method returns an error, we can't chain calls together and need to delay validation in the `Build` method
- Builder pattern allows the user to leave the object in an 'unfinished' state

## Functional Options Pattern

The main idea is as follows:

- An unexported struct holds the configuration: `options`
- Each option is a function that returns the same type: `type Option func(options *options) error`

```go
type options struct {
	port *int
}

type Option func(options *options) error

func WithPort(port int) Option {
	return func(options *options) error {
		if port < 0 {
			return errors.New("port should be positive")
		}
		options.port = &port
		return nil
	}
}

func NewServer(addr string, opts ...Option) (*http.Server, error) {
	var options options
	for _, opt := range opts {
		err := opt(&options)
		if err != nil {
			return nil, err
		}
	}

	var port int
	if options.port == nil {
		port = defaultHTTPPort
	} else {
		if *options.port == 0 {
			port = randomPort()
		} else {
			port = *options.port
		}
	}
	...
}
```

# Utility Packages

We should avoid packages like `common`, `utility`, `base` and so on

If we decide to have a `client` and a `server` package, where should we put the common types?  
The solution is to **combine** the `client`, the `server`, and the common code into a **single package**

For example, the `net/http` package does not have `client` and `server` packages, instead it has `client.go` and `server.go` files, each holding their respective types. `transport.go` holds for the common message transport code used by both HTTP clients and servers

# References

- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
- [SOLID Go Design | Dave Cheney](https://dave.cheney.net/2016/08/20/solid-go-design)
- [Go Proverbs](https://go-proverbs.github.io)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md)
- [Avoid package names like base, util, or common | Dave Cheney](https://dave.cheney.net/2019/01/08/avoid-package-names-like-base-util-or-common)
- [GopherCon 2019: Jonathan Amsterdam - Detecting Incompatible API Changes - YouTube](https://youtu.be/JhdL5AkH-AQ?si=SBWb9xaVFlDiBOhQ)
