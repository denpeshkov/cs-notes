---
tags:
  - Go
---

# Representation

An `abi.Type` struct represents **type information** common for all data types:

```go
// Type is the runtime representation of a Go type.
type Type struct {
	Size_       uintptr
	PtrBytes    uintptr // number of (prefix) bytes in the type that can contain pointers
	Hash        uint32  // hash of type; avoids computation in hash tables
	TFlag       TFlag   // extra type information flags
	Align_      uint8   // alignment of variable with this type
	FieldAlign_ uint8   // alignment of struct field with this type
	Kind_       uint8   // enumeration for C
	// function for comparing objects of this type
	// (ptr to object A, ptr to object B) -> ==?
	Equal func(unsafe.Pointer, unsafe.Pointer) bool
	// GCData stores the GC type data for the garbage collector.
	// If the KindGCProg bit is set in kind, GCData is a GC program.
	// Otherwise it is a ptrmask bitmap. See mbitmap.go for details.
	GCData    *byte
	Str       NameOff // string form
	PtrToThis TypeOff // type for pointer to this type, may be zero
}
```

Data types requiring additional information are represented by their respective structs: [slice](Go%20Slices%20Internals.md), struct, [interface](Go%20Interfaces%20Internals.md), [map](Go%20Map%20Internals.md), etc. All this structs embed the `abi.Type` struct and add additional fields

For example, `abi.UncommonType` stores information about methods defined on a type:

```go
// UncommonType is present only for defined types or types with methods
// (if T is a defined type, the uncommonTypes for T and *T have methods).
// Using a pointer to this struct reduces the overall size required
// to describe a non-defined type with no methods.
type UncommonType struct {
	PkgPath NameOff // import path; empty for built-in types like int, string
	Mcount uint16 // number of methods
	Xcount uint16 // number of exported methods
	Moff uint32 // offset from this uncommontype to [mcount]Method
	_ uint32 // unused
}
```

It's important to note that type information structures are computed at **compile-time**

# References

- [Source code: type.go](https://github.com/golang/go/blob/master/src/internal/abi/type.go)
- [Internals of Interfaces in Golang | Intermediate level - YouTube](https://youtu.be/x87Cs9vU4Fk?si=xYrKUEtrWuPlMCTC)
