---
tags:
  - Go
  - OS_Arch
  - AlgsDs
---

# Interface Values Representation

Interface values are represented as a two-word pair, defined by a struct `runtime.iface`:

```go
type iface struct {
	tab  *itab
	data unsafe.Pointer
}
```

The first word `tab` is a pointer to an **itable**, which contains information about the type of the interface, the type of the data it points to and a virtual method table

The second word `data` is a pointer to the value (copy of the original) held by the interface. Values stored in interfaces might be arbitrarily large, but only one word is dedicated to holding the value in the interface structure, so the assignment allocates a chunk of memory on the [heap](Heap%20Memory.md) and records the pointer in the `data` field

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

To check a type of the actual value an interface points to, compiler generates the code equivalent to `s.tab->_type`

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

If an interface has no methods, itable is dropped and the first word points at the type directly. That is, `runtime.eface` is used instead of `runtime.iface`:

```go
type eface struct {
	_type *_type
	data unsafe.Pointer
}
```

As an optimization, the Go runtime maintains an array of small integers. If an interface is instantiated from one of these small integers, it points to an element of this array, thereby preventing runtime allocation

In Russ Cox [blog post](https://research.swtch.com/interfaces) it is stated that if the actual value fits in a single word, it is stored in the second word directly without indirection or [heap](Heap%20Memory.md) allocation. However, this is no longer true, as it caused race conditions in concurrent GC and ambiguity about whether the data word holds a pointer or scalar ( #todo elaborate further). So interfaces **always** store the pointer in the `data` field

# Heap Allocations and Escape Analysis

When a method is called via an interface value instead of directly through a struct, the compiler **generally** lacks knowledge of the method's implementation at compile time. Consequently, escape analysis cannot confirm that the value doesn't escape, leading to [heap](Heap%20Memory.md) allocations (there are [optimizations](#Optimizations)) even for scalar types like `int`s, `float`s, `string`s

In some cases, the compiler can prove the concrete type of the value stored in the interface and **devirtualize** a method call, avoiding [heap](Heap%20Memory.md) allocations

Putting a [zero-width type](Go%20Zero-Sized%20Values.md), e.g. `struct{}`, in an interface value doesn't allocate

## Avoiding Allocation for Context Key

There are couple of ways to avoid allocation when passing context key to `any` in `context.WithValue` call:

```go
// small ints are shared by runtime, most of the keys won't cause allocation
type keyInt int
const (
	k1 = keyInt(iota + 1)
	k2
)

// strings are not shared by runtime
type keyString string
// use const to avoid allocations
const (
	k3 = keyString("k3")
	k4 = "k4"
)

type keyStruct struct {
	str string
}
// use pointers to avoid allocation
var (
	k5 = &keyStruct{"k5"}
	k6 = &keyStruct{"k6"}
)

// use interface to avoid allocation
type keyInter any
var (
	k7 = keyInter("k7")
	k8 = keyInter("k8")
)

```

# Addressability

The concrete value stored in an interface is not addressable (it's a copy), in the same way that a [map element is not addressable](Go%20Map%20Internals.md#Addressability)

Therefore, when you call a method on an interface, it must either have an identical receiver type or it must be directly discernible from the concrete type: 

- Pointer and value receiver methods can be called with pointers and values respectively
- Value receiver methods can be called with pointer values because they can be dereferenced first
- Pointer receiver methods cannot be called with values

# References

- [research!rsc: Go Data Structures: Interfaces](https://research.swtch.com/interfaces)
- [Source code: runtime2.go](https://github.com/golang/go/blob/master/src/runtime/runtime2.go#L205)
- [Source code: iface.go](https://github.com/golang/go/blob/master/src/runtime/iface.go)
- [How are interfaces implemented in Go? : r/golang](https://www.reddit.com/r/golang/comments/ehy75k/how_are_interfaces_implemented_in_go/)
- [Chapter II: Interfaces - Go Internals](https://cmc.gitbook.io/go-internals/chapter-ii-interfaces#anatomy-of-an-interface)
- [Internals of Interfaces in Golang | Intermediate level - YouTube](https://youtu.be/x87Cs9vU4Fk?si=xYrKUEtrWuPlMCTC)
- [Source code: type.go](https://github.com/golang/go/blob/master/src/internal/abi/type.go#L478)
- [Lec08 Allocation Strategies - YouTube](https://youtu.be/s0j8U-NsbqQ?si=XkwGYR3xzurHEp_j)
- [Lec09 Implicit Allocators Indirection And References - YouTube](https://youtu.be/GH7MGNAuwaQ?si=xxh8N3d80fmgN2qo)
- [GopherCon Europe 2023: Jonathan Amsterdam - A Fast Structured Logging Package - YouTube](https://www.youtube.com/watch?v=tC4Jt3i62ns)
- [Generics can make your Go code slower](https://planetscale.com/blog/generics-can-make-your-go-code-slower)
- [Go Wiki: Compiler And Runtime Optimizations](https://go.dev/wiki/CompilerOptimizations#zero-width-types-in-interface-values)
- [Go Wiki: MethodSets - The Go Programming Language](https://go.dev/wiki/MethodSets)
- [Fetching Title#t8ne](https://eli.thegreenplace.net/2022/interface-method-calls-with-the-go-register-abi/)
