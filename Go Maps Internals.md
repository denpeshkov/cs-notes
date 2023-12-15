---
tags: Go
---

# Representation

A `map` value is a *pointer* to a `runtime.hmap` structure:

```go
// A header for a Go map.
type hmap struct {
    count     int // # live cells == size of map.  Must be first (used by len() builtin)
    flags     uint8
    B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
    noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
    hash0     uint32 // hash seed
    
    buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
    oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
    nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)
    
    extra *mapextra // optional fields
}
```

Note that `hash0` adds randomness to the hash function, making it adversary safe

The map runtime doesn't use generics to enable a generic map implementation. Instead, map lookups, insertion, and removal are implemented in the runtime package. During compilation, map operations are rewritten to calls to the runtime:

```go
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {...}
func mapaccess2(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, bool) {...}
func mapaccessK(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, unsafe.Pointer) {...}
func mapdelete(t *maptype, h *hmap, key unsafe.Pointer) {...}
...
```

Let's walk through the parameters:

- `key` is a pointer to the key, this is the value you provided as the key
- `h` is a pointer to a `runtime.hmap` structure
- `t` is a pointer to a `runtime.maptype` which is an alias to `abi.MapType`

# Internals

The Go compiler generates only one map implementation, regardless of the key/value types used. To enable generic behavior, the compiler generates a distinct `MapType` for each unique key/value pair

Each `MapType` contains details about the properties of this kind of map from key to element:

```go
type MapType struct {
	Type
	Key    *Type
	Elem   *Type
	Bucket *Type // internal type representing a hash bucket
	// function for hashing keys (ptr to key, seed) -> hash
	Hasher     func(unsafe.Pointer, uintptr) uintptr
	KeySize    uint8 // size of key slot
	ValueSize  uint8 // size of elem slot
	BucketSize uint16 // size of bucket
	Flags      uint32
}
```

The `Type` contains information about the Go type:

```go
// Type is the runtime representation of a Go type.
type Type struct {
	Size_       uintptr
	PtrBytes    uintptr // number of (prefix) bytes in the type that can contain pointers
	Hash        uint32 // hash of type; avoids computation in hash tables
	TFlag       TFlag // extra type information flags
	Align_      uint8 // alignment of variable with this type
	FieldAlign_ uint8 // alignment of struct field with this type
	Kind_       uint8 // enumeration for C
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

These structures allow the Go runtime to implement a generic map. For example, `Hasher` and `Equal` allow hashing and comparing types

# Addressability

We can't take an address of the map element, because if the map grows, the address is of the stale value

# References

- [If a map isn’t a reference variable, what is it? | Dave Cheney](https://dave.cheney.net/2017/04/30/if-a-map-isnt-a-reference-variable-what-is-it)
- [GopherCon 2016: Keith Randall - Inside the Map Implementation - YouTube](https://youtu.be/Tl7mi9QmLns?si=HOOEUzsmu2sLy9J2)
- [Source code: map.go](https://github.com/golang/go/blob/master/src/runtime/map.go)
- [How the Go runtime implements maps efficiently (without generics) | Dave Cheney](https://dave.cheney.net/2018/05/29/how-the-go-runtime-implements-maps-efficiently-without-generics)
