---
tags: Go 
---

# Overview

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

# Config Struct

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

# Using Setters or Exported Fields Directly

We could use directly exported fields (or setters) to config the server after instantiation. For example like an `http.Server` in the standard library

That approach also has downsides:

- No validation at the instantiation time. Need to use a separate `Valid` method
- The server may be in inconsistent state during configuration
- It also precludes the possibility of making a type immutable
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

# Functional Options Pattern

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

# References

- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md
- [Functional options for friendly APIs | Dave Cheney](https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis)
