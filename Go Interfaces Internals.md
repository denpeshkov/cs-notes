---
tags:
  - Go
  - OS
  - Architecture
  - AlgsDs
---

# Interface Values Representation

Interface values are represented by a struct `runtime.iface`:

```go
type iface struct {
	tab  *itab
	data unsafe.Pointer
}
```

- `tab` is a pointer to an **itable**, which contains information about the type of the interface, as well as the type of the data it points to
- `data` is a pointer to the value (copy of the original) held by the interface

Note that `data` points to the **copy of a value** used in the assignment. For example, copying an interface value (passing interface as a parameter) makes a copy of the value stored in the interface  
A copy is used because if a variable later changes, the pointer should have the old value, not the new one

## Example

Given:

```go
type Stringer interface {
    String() string
}

type Binary uint64
func (i Binary) String() string {...}
func (i Binary) Get() uint64 {...}
```

On a 32-bit word size machine the interface `Stringer` storing the `Binary` is:

![interface value representation|400](interface%20value%20representation.png)

# Itable

The itable is represented by a struct `runtime.itab`:

```go
type itab struct {
	inter *interfacetype
	_type *_type
	hash  uint32 // copy of _type.hash. Used for type switches.
	_     [4]byte
	fun   [1]uintptr // variable sized. fun[0]==0 means _type does not implement inter.
}
```

- `inter` describes type information of the interface itself
- `_type` describes the [type information](Go%20Type%20Internals.md) of the value an interface contains
- `fun` is a variable-sized array of function pointers - the dispatch table of the interface

[Type information](app://obsidian.md/Go%20Type%20Internals.md) about the interface data type is represented by an `runtime.interfacetype` which is an alias for `abi.InterfaceType` struct:

```go
type InterfaceType struct {
	Type
	PkgPath Name // import path
	Methods []Imethod // sorted by hash
}
```

---

Note that, the itable corresponds to the *interface type*, not the dynamic type. In our example, it contains pointer only to `String` method, not `Get`

To check a type of the actual value an interface points to, compiler generates the code equivalent to `s.tab->type`

To call `s.String()`, compiler generates the code equivalent to `s.tab->fun[0](s.data)`  
Note that, the function in itable is being passed the pointer, not the actual value. Thus the function pointer in our example is `(*Binary).String` not `Binary.String`

## Computing the Itable

The itable gets computed during the assignment (conversion) of the value to the interface variable: `s := any.(Stringer)`

There is a **compile-time** generated [type description](Go%20Type%20Internals.md) struct for each concrete type like `Binary`  
Among other metadata, the type description struct contains a list of $nt$ methods implemented by that type

Similarly, there is **compile-time** generated `abi.InterfaceType` struct for each interface type like `Stringer`  
It too contains a list of $ni$ methods

At **run-time** the itable is computed by looking for each method listed in the interface type's method table in the concrete type's method table  
It then caches it, so the itable is computed only once

Note that, both tables are sorted, so the mapping is found in $O(ni+nt)$ instead of naive $O(ni*nt)$

## Optimizations

If an interface has no method, itable is dropped and the first word points at the type directly. That is, `runtime.eface` is used instead of `runtime.iface`:

```go
type eface struct {
	_type *_type
	data unsafe.Pointer
}
```

If the actual value fits in a one word, it is stored in the second word directly without indirection

# Heap Allocations and Escape Analysis

#TODO 

# References

- [research!rsc: Go Data Structures: Interfaces](https://research.swtch.com/interfaces)
- [Source code: runtime2.go](https://github.com/golang/go/blob/master/src/runtime/runtime2.go#L205)
- [Chapter II: Interfaces - Go Internals](https://cmc.gitbook.io/go-internals/chapter-ii-interfaces#anatomy-of-an-interface)
- [Internals of Interfaces in Golang | Intermediate level - YouTube](https://youtu.be/x87Cs9vU4Fk?si=xYrKUEtrWuPlMCTC)
- [Source code: type.go](https://github.com/golang/go/blob/master/src/internal/abi/type.go#L478)
