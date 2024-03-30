---
tags: Go AlgsDs
---

# Data Structure Representation

In Go a `map` is a [[Hash Map|hash map]], represented as an array of buckets, each of which contains an array of key/value pairs

![Go map internals|800](go%20map%20internals.png)

## The `hmap` Struct

A `map` value is a **pointer** to a `runtime.hmap` structure:

```go
// A header for a Go map.
type hmap struct {
    count     int // # live cells == size of map.
    flags     uint8
    B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
    noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
    hash0     uint32 // hash seed
    
    buckets    unsafe.Pointer // array of 2^B buckets. may be nil if count==0.
    oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
    nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)
    
    extra *mapextra // optional fields
}
```

A new `hmap` is created for **every** `map` instance in the program

Every `hmap` will have a different `hash0` random hash seed value. Therefore, the same key may have different hash values for different map instances

The number of buckets is always a power of 2, so the $\log_{2}$ of number of buckets is stored to reduce variable size

## The `maptype` Struct

The `runtime.maptype` struct is an alias for `abi.MapType`

```go
// alias to runtime.maptype
type MapType struct {
	Type
	Key    *Type
	Elem   *Type
	Bucket *Type // internal type representing a hash bucket
	Hasher     func(unsafe.Pointer, uintptr) uintptr // function for hashing keys (ptr to key, seed) -> hash 
	KeySize    uint8  // size of key slot
	ValueSize  uint8  // size of elem slot
	BucketSize uint16 // size of bucket
	Flags      uint32
}
```

`Key`, `Elem` and `Bucket` fields store the [type information](Go%20Type%20Internals.md) about key, value and bucket, respectively

A new `maptype` is only created for **every unique** (combination of key and value types) `map` declaration in the program

The `maptype` makes a `map` [generic](#Generic%20Map%20Implementation) for every combination of key and value types. By separating `maptype` from `hmap` every `map` with same key-value types combination reuses the same `maptype` instance

## The `bmap` Struct

```go
// A bucket for a Go map.
type bmap struct {
	// tophash generally contains the top byte of the hash value for each key in this bucket.
	// If tophash[0] < minTopHash, tophash[0] is a bucket evacuation state instead.
	tophash [bucketCnt]uint8
	// Followed by bucketCnt keys and then bucketCnt elems.
	// NOTE: packing all the keys together and then all the elems together makes the
	// code a bit more complicated than alternating key/elem/key/elem/... but it allows
	// us to eliminate padding which would be needed for, e.g., map[int64]int8.
	// Followed by an overflow pointer.
}
```

The `tophash` is an 8 element array containing the high order byte (**HOB**) of the hash value for each key in this bucket. This array is used to speed up key lookup: if two **HOB** values are different, we can skip full key comparison

There are a couple of predefined `tophash` values:

```go
// Possible tophash values. We reserve a few possibilities for special marks.
// Each bucket (including its overflow buckets, if any) will have either all or none of its
// entries in the evacuated* states (except during the evacuate() method, which only happens
// during map writes and thus no one else can observe the map during that time).
emptyRest      = 0 // this cell is empty, and there are no more non-empty cells at higher indexes or overflows.
emptyOne       = 1 // this cell is empty
evacuatedX     = 2 // key/elem is valid.  Entry has been evacuated to first half of larger table.
evacuatedY     = 3 // same as above, but evacuated to second half of larger table.
evacuatedEmpty = 4 // cell is empty, bucket is evacuated.
minTopHash     = 5 // minimum tophash for a normal filled cell.
```

Second, there is an array of bytes that store the key/value pairs. The byte array packs all the keys and then all the values together for the respective bucket. Packing allows to eliminate padding which would be needed to maintain proper [alignment boundaries](Data%20Alignment.md)  
For example, `map[int64]int8` would have needed an extra 7 bit padding per key/value pair. By using packing the padding only has to be appended to the end of the byte array and not in between

A bucket is configured to store only 8 key/value pairs. If a 9th key needs to be added to a bucket that is full, an **overflow** bucket is created and referenced from inside the respective bucket

The key and values are not directly represented in the `bmap` struct, because they can be of any type. Therefore, the compiler can't know the size of the `bmap` struct

For example:

```go
// Having this map

map[string]int

// The corresponding bmap is represented as

type bmap {
	tophash [8]uin8
	keys [8]string
	values [8]int
	overflow *bmap
}
```

# Generic Map Implementation

The map runtime doesn't use generics to enable a generic map implementation. Instead, `map` lookups, insertions, and deletions are implemented in the runtime package. During compilation, `map` operations are rewritten to calls to the runtime:

```go
// v := m[k]
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {...}

// v,ok := m[k]
func mapaccess2(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, bool) {...}

// m := make(map[kT]vT, hint)
func makemap(t *maptype, hint int, h *hmap) *hmap {...}

// m[k] = v
func mapassign(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {...}

// delete m(k)
func mapdelete(t *maptype, h *hmap, key unsafe.Pointer) {...}
...
```

- `key` is a pointer to the key
- `h` is a pointer to the `runtime.hmap` structure
- `t` is a pointer to the `runtime.maptype` structure

# Addressability

Map elements are not addressable, because if the map grows, the address is of the stale value

Thus, we can't assign to `struct` field in the map:

``` go
type S struct {s int}
m := map[int]S{}
m[0] = S{0}
m[0].s = 1 // cannot assign to struct field m[0].s in map
```

Also, if the map is `nil` or does not contain an entry, `m[0]` would return the zero value. Therefore, attempting to assign to `m[0].s` would assign to a nonexistent 'zero value' element.

To overcome this problem, we need to copy the element and reassign:

``` go
v := m[0]
v.s = 1
m[0] = v
```

---

However, we can assign to a slice element because it is a [slice header](Go%20Slices%20Internals.md) containing a **pointer** to the underlying array:

``` go
m := map[int][]int{}
m[0] = []int{0}
m[0][0] = 1
```

# Eviction #TODO

A `map` starts with one bucket. The map grows when buckets are about 80% full (load factor is 6.5). When `map` grows it allocates 2x number of buckets.

Growing starts with assigning a pointer called the "old bucket" pointer to the current bucket array. Then a new bucket array is allocated to hold twice the number of existing buckets. This could result in large allocations, but the memory is not initialized so the allocation is fast

Once the memory for the new bucket array is available, the key/value pairs from the old bucket array can be moved or "**evacuated**" to the new bucket array. 

Evacuations happen **incrementally** as key/value pairs are added or removed from the map. This allows to amortize the cost of copying elements between multiple `map` operations. During evacuation `map` operation take longer because they need to check both old and new arrays

The key/value pairs that are together in an old bucket could be moved to different buckets inside the new bucket array. The evacuation algorithm attempts to distribute the key/value pairs evenly across the new bucket array

# Element Lookup #TODO

# Element Deletion #TODO

# Element Insertion #TODO

# References

- [If a map isn’t a reference variable, what is it? | Dave Cheney](https://dave.cheney.net/2017/04/30/if-a-map-isnt-a-reference-variable-what-is-it)
- [GopherCon 2016: Keith Randall - Inside the Map Implementation - YouTube](https://youtu.be/Tl7mi9QmLns?si=HOOEUzsmu2sLy9J2)
- [How the Go runtime implements maps efficiently (without generics) | Dave Cheney](https://dave.cheney.net/2018/05/29/how-the-go-runtime-implements-maps-efficiently-without-generics)
- [Maps in Golang | Intermediate level - YouTube](https://youtu.be/1LnzCtcfsFc?si=yK1iIsX4IF2nrtyU)
- [Internals of Maps in Golang - YouTube](https://youtu.be/ACQs6mdylxo?si=s3gHTZnxXz4rOR7y)
- [Source code: map.go](https://github.com/golang/go/blob/master/src/runtime/map.go)
