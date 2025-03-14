---
tags:
  - Go
  - Data-Structures-Algorithms
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

The number of buckets is always a power of 2, so `hmap.B` stores $\log_{2}$ of number of buckets to reduce the variable size

## The `maptype` Struct

The `runtime.maptype` struct is an alias for `abi.MapType`:

```go
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

// Store ptr to key instead of key itself.
func (mt *MapType) IndirectKey() bool {
	return mt.Flags&1 != 0
}
// Store ptr to elem instead of elem itself.
func (mt *MapType) IndirectElem() bool {
	return mt.Flags&2 != 0
}
```

`Key`, `Elem` and `Bucket` fields store the [type information](Go%20Type%20Internals.md) about key, value and bucket, respectively

A new `maptype` is only created for **every unique** (combination of key and value types) `map` declaration in the program

The `maptype` is what makes a `map` [generic](#Generic%20Map%20Implementation) for every combination of key and value types. By separating `maptype` from `hmap` every `map` with same key-value types combination reuses the same `maptype` instance

## The `bmap` Struct

This struct describes a hash map bucket. A bucket is configured to store 8 key/value pairs

```go
// A bucket for a Go map.
type bmap struct {
	// tophash generally contains the top byte of the hash value for each key in this bucket.
	// If tophash[0] < minTopHash, tophash[0] is a bucket evacuation state instead.
	tophash [abi.MapBucketCount]uint8
	// Followed by abi.MapBucketCount keys and then abi.MapBucketCount elems.
	// NOTE: packing all the keys together and then all the elems together 
	// rather than alternating key/elem/key/elem/... allows us to eliminate 
	// padding which would be needed for, e.g., map[int64]int8.
	// Followed by an overflow pointer.
}
```

There is a large key/value optimization: if a key or value is over 128 bytes, it's allocated on the heap and a **pointer** to it is stored in the map bucket

The `tophash` is an 8 element array containing the high order byte (**HOB**) of the hash value for each key in this bucket. This array is used to speed up key lookup: if two HOB values are different, we can skip full key comparison

There are a couple of predefined `tophash` values:

```go
// Possible tophash values.
emptyRest      = 0 // this cell is empty, and there are no more non-empty cells at higher indexes or overflows.
emptyOne       = 1 // this cell is empty
evacuatedX     = 2 // key/elem is valid.  Entry has been evacuated to first half of larger table.
evacuatedY     = 3 // same as above, but evacuated to second half of larger table.
evacuatedEmpty = 4 // cell is empty, bucket is evacuated.
minTopHash     = 5 // minimum tophash for a normal filled cell.
```

Following `tophash` is an array of bytes that stores the key/value pairs. The byte array packs all the keys and then all the values together for the respective bucket. Packing allows eliminating padding which would be needed to maintain proper [alignment boundaries](Data%20Alignment.md). For example, `map[int64]int8` would have needed an extra 7 bit padding per key/value pair. By using packing the padding only has to be appended to the end of the byte array and not in between

The key and values are not directly represented in the `bmap` struct, because they can be of any type, the compiler can't know the size of the `bmap` struct in advance. For example:

```go
// Having this map:
map[string]int

// The corresponding bmap is represented as:
type bmap {
	tophash [8]uin8
	keys [8]string
	values [8]int
	overflow *bmap
}
```

## Overflow Buckets

When a bucket becomes full and a 9th key needs to be added, it would be inefficient to grow the map by doubling the number of buckets. Instead, Go creates an overflow bucket, linking it to the original one. The new key-value pair gets stored in this overflow bucket rather than forcing a full grow

# Generic Behavior

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

# Eviction

## Same Size Growth

##

A map in Go grows when one of two conditions are met: 

1. There are too many overflow buckets
2. The map is overloaded, meaning the load factor exceeds the threshold of 6.5

Because of these 2 conditions, there are also two kinds of growth:

- One that doubles the size of the buckets (when overloaded)
- One that keeps the same size but redistributes entries (when there are too many overflow buckets)

---

When the buckets start getting 80% full (load factor is 6.5), the map will trigger a growth, which might double the number of buckets

A `map` starts with one bucket. The map grows when buckets are about 80% full (load factor is 6.5). When `map` grows it allocates 2x number of buckets

Growing starts with assigning a pointer called the "old bucket" pointer to the current bucket array. Then a new bucket array is allocated to hold twice the number of existing buckets. This could result in large allocations, but the memory is not initialized so the allocation is fast

Once the memory for the new bucket array is available, the key/value pairs from the old bucket array can be moved or "**evacuated**" to the new bucket array

Evacuations happen **incrementally** as key/value pairs are added or removed from the map. This allows to amortize the cost of copying elements between multiple `map` operations. During evacuation `map` operation take longer because they need to check both old and new arrays

The key/value pairs that are together in an old bucket could be moved to different buckets inside the new bucket array. The evacuation algorithm attempts to distribute the key/value pairs evenly across the new bucket array

# Operations Implementation

## Map Creation

```go
// makemap implements Go map creation for make(map[k]v, hint).
// If the compiler has determined that the map or the first bucket
// can be created on the stack, h and/or bucket may be non-nil.
// If h != nil, the map can be created directly in h.
// If h.buckets != nil, bucket pointed to can be used as the first bucket.
func makemap(t *maptype, hint int, h *hmap) *hmap {
	// Initialize hmap.
	if h == nil {
		h = new(hmap)
	}
	// Randomize hash seed.
	h.hash0 = uint32(rand())

	// Find the size parameter B which will hold the requested # of elements.
	// For hint < 0 overLoadFactor returns false since hint < bucketCnt.
	B := uint8(0)
	for overLoadFactor(hint, B) {
		// hint items placed in 1<<B buckets is over loadFactor.
		B++
	}
	h.B = B

	// Allocate initial hash table.
	// If B == 0, the buckets field is allocated lazily later (in mapassign).
	if h.B != 0 {
		var nextOverflow *bmap
		h.buckets, nextOverflow = makeBucketArray(t, h.B, nil)
		if nextOverflow != nil {
			h.extra = new(mapextra)
			h.extra.nextOverflow = nextOverflow
		}
	}
	return h
}

// makeBucketArray initializes a backing array for map buckets.
// dirtyalloc is either nil or an array previously allocated by makeBucketArray with the same t and b parameters.
// If dirtyalloc is nil a new array will be allocated, otherwise dirtyalloc will be cleared and reused.
func makeBucketArray(t *maptype, b uint8, dirtyalloc unsafe.Pointer) (buckets unsafe.Pointer, nextOverflow *bmap) {
	base := bucketShift(b) // 1<<b
	// Number of buckets.
	nbuckets := base
	// For small b, overflow buckets are unlikely. Avoid the overhead of the calculation.
	if b >= 4 {
		// Add on the estimated number of overflow buckets required to insert
		// the median number of elements used with this value of b.
		nbuckets += bucketShift(b - 4) // 1<<(b-4)
		sz := t.Bucket.Size_ * nbuckets
		// Round up to mallocgc size class.
		up := roundupsize(sz, !t.Bucket.Pointers())
		if up != sz {
			nbuckets = up / t.Bucket.Size_
		}
	}

	if dirtyalloc == nil {
		// New array of nbuckets elements.
		buckets = newarray(t.Bucket, int(nbuckets))
	} else {
		// dirtyalloc was previously generated by the above newarray(t.Bucket, int(nbuckets))
		// but may not be empty. We should clear the memory.
		buckets = dirtyalloc
		size := t.Bucket.Size_ * nbuckets
		if t.Bucket.Pointers() {
			memclrHasPointers(buckets, size)
		} else {
			memclrNoHeapPointers(buckets, size)
		}
	}

	if base != nbuckets {
		// To keep the overhead of tracking overflow buckets to a minimum,
		// if a preallocated overflow bucket's overflow pointer is nil,
		// then there are more available by bumping the pointer.
		// A safe non-nil pointer for the last overflow bucket is buckets.
		nextOverflow = (*bmap)(add(buckets, base*uintptr(t.BucketSize)))
		last := (*bmap)(add(buckets, (nbuckets-1)*uintptr(t.BucketSize)))
		last.setoverflow(t, (*bmap)(buckets))
	}
	return buckets, nextOverflow
}
```

## Element Lookup

``` go
// mapaccess1 returns a pointer to h[key]. It returns a reference to the zero object 
// for the elem type if the key is not in the map.
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
	// Return zero value if map is empty.
	if h == nil || h.count == 0 {
		return unsafe.Pointer(&abi.ZeroVal[0])
	}
	if h.flags&hashWriting != 0 {
		fatal("concurrent map read and map write")
	}

	// Compute the hash value of the key.
	hash := t.Hasher(key, uintptr(h.hash0))
	// Compute the bucket index for the key: hash % (# of buckets).
	m := bucketMask(h.B)
	bucket := hash & m
	// Pointer to the starting address of the bucket: &h.buckets[bucket].
	b := (*bmap)(add(h.buckets, bucket*uintptr(t.BucketSize)))

	// If we have an old bucket array of half the size and are currently growing.
	if c := h.oldbuckets; c != nil {
		if !h.sameSizeGrow() {
			// There used to be half as many buckets; mask down one more power of two.
			m >>= 1
		}
		// Compute the bucket index for the key in the old bucket array: hash % (# of buckets).
		bucket := hash & m
		// Pointer to the starting address of the bucket in old bucket array: &h.oldbuckets[bucket].
		oldb := (*bmap)(add(c, bucket*uintptr(t.BucketSize)))
		// Bucket in the old array is not evacuated.
		if !evacuated(oldb) {
			// Use an old bucket onwards.
			b = oldb
		}
	}

	// Compute tophash value for the key: the high order byte (HOB) of the key hash value.
	top := tophash(hash)

bucketloop:
	// Iterate over the bucket and its overflow buckets.
	for ; b != nil; b = b.overflow(t) {
		// Iterate over key/element entries in the bucket.
		for i := uintptr(0); i < abi.MapBucketCount; i++ {
			// If the tophash values of the key and the ith key are different.
			if b.tophash[i] != top {
				// (Optimization): break if the rest is empty.
				if b.tophash[i] == emptyRest {
					break bucketloop
				}
				continue
			}
			// Pointer to the ith key in the bucket.
			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.KeySize))
			if t.IndirectKey() {
				k = *((*unsafe.Pointer)(k))
			}
			// Compare the key with ith key in the bucket.
			if t.Key.Equal(key, k) {
				// Pointer to the ith element in the bucket.
				e := add(unsafe.Pointer(b), dataOffset+abi.MapBucketCount*uintptr(t.KeySize)+i*uintptr(t.ValueSize))
				if t.IndirectElem() {
					e = *((*unsafe.Pointer)(e))
				}
				return e
			}
		}
	}
	// Return zero value.
	return unsafe.Pointer(&abi.ZeroVal[0])
}
```

## Element Insertion

``` go
// Like mapaccess, but allocates a slot for the key if it is not present in the map.
func mapassign(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer {
	if h == nil {
		panic(plainError("assignment to entry in nil map"))
	}
	if h.flags&hashWriting != 0 {
		fatal("concurrent map writes")
	}

	// Compute the hash value of the key.
	hash := t.Hasher(key, uintptr(h.hash0))

	// Set hashWriting after calling t.hasher, since t.hasher may panic,
	// in which case we have not actually done a write.
	h.flags ^= hashWriting

	// Allocate bucket array.
	if h.buckets == nil {
		h.buckets = newobject(t.Bucket) // newarray(t.Bucket, 1)
	}

again:
	// Compute the bucket index for the key: hash % (# of buckets).
	bucket := hash & bucketMask(h.B)
	if h.growing() {
		growWork(t, h, bucket)
	}

	// Pointer to the starting address of the bucket: &h.buckets[bucket].
	b := (*bmap)(add(h.buckets, bucket*uintptr(t.BucketSize)))
	// Compute tophash value for the key: the high order byte (HOB) of the key hash value.
	top := tophash(hash)

	var inserti *uint8
	var insertk unsafe.Pointer
	var elem unsafe.Pointer
bucketloop:
	for {
		// Iterate over key/element entries in the bucket.
		for i := uintptr(0); i < bucketCnt; i++ {
			// If the tophash values of the key and the ith key are different.
			if b.tophash[i] != top {
				// If bucket is empty remember insert position.
				if isEmpty(b.tophash[i]) && inserti == nil {
					// Pointer to ith tophash value.
					inserti = &b.tophash[i]
					// Pointer to the ith key in the bucket.
					insertk = add(unsafe.Pointer(b), dataOffset+i*uintptr(t.KeySize))
					// Pointer to the ith element in the bucket.
					elem = add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.KeySize)+i*uintptr(t.ValueSize))
				}
				// (Optimization): return if the rest is empty.
				if b.tophash[i] == emptyRest {
					break bucketloop
				}
				continue
			}

			// Pointer to the ith key in the bucket.
			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.KeySize))
			if t.IndirectKey() {
				k = *((*unsafe.Pointer)(k))
			}

			// Compare the key with ith key in the bucket.
			if !t.Key.Equal(key, k) {
				continue
			}

			// Already have a mapping for key. Update it.
			if t.NeedKeyUpdate() {
				typedmemmove(t.Key, k, key)
			}

			// Pointer to the ith element in the bucket.
			elem = add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.KeySize)+i*uintptr(t.ValueSize))
			goto done
		}
		
		// Check overflow buckets
		ovf := b.overflow(t)
		if ovf == nil {
			break
		}
		b = ovf
	}

	// Did not find mapping for key. Allocate new cell & add entry.

	// If we hit the max load factor or we have too many overflow buckets,
	// and we're not already in the middle of growing, start growing.
	if !h.growing() && (overLoadFactor(h.count+1, h.B) || tooManyOverflowBuckets(h.noverflow, h.B)) {
		hashGrow(t, h)
		// Growing the table invalidates everything, so try again.
		goto again
	}

	if inserti == nil {
		// The current bucket and all the overflow buckets connected to it are full, allocate a new one.
		newb := h.newoverflow(t, b)
		inserti = &newb.tophash[0]
		insertk = add(unsafe.Pointer(newb), dataOffset)
		elem = add(insertk, bucketCnt*uintptr(t.KeySize))
	}

	// Store new key/elem at insert position.
	if t.IndirectKey() {
		kmem := newobject(t.Key)
		*(*unsafe.Pointer)(insertk) = kmem
		insertk = kmem
	}
	if t.IndirectElem() {
		vmem := newobject(t.Elem)
		*(*unsafe.Pointer)(elem) = vmem
	}

	// Copy the key to the memory pointed to by insertk.
	typedmemmove(t.Key, insertk, key)
	*inserti = top
	h.count++

done:
	if h.flags&hashWriting == 0 {
		fatal("concurrent map writes")
	}
	h.flags &^= hashWriting
	if t.IndirectElem() {
		elem = *((*unsafe.Pointer)(elem))
	}

	// Return pointer to the element.
	return elem
}
```

## Element Deletion

```go
func mapdelete(t *maptype, h *hmap, key unsafe.Pointer) {
	if h == nil || h.count == 0 {
		return
	}
	if h.flags&hashWriting != 0 {
		fatal("concurrent map writes")
	}

	// Compute the hash value of the key.
	hash := t.Hasher(key, uintptr(h.hash0))

	// Flag that we are modifying the map.
	h.flags ^= hashWriting

	// Compute the bucket index for the key: hash % (# of buckets).
	bucket := hash & bucketMask(h.B)
	if h.growing() {
		growWork(t, h, bucket)
	}
	// Pointer to the starting address of the bucket: &h.buckets[bucket].
	b := (*bmap)(add(h.buckets, bucket*uintptr(t.BucketSize)))
	bOrig := b
	// Compute tophash value for the key: the high order byte (HOB) of the key hash value.
	top := tophash(hash)
search:
	// Iterate over the bucket and its overflow buckets.
	for ; b != nil; b = b.overflow(t) {
		// Iterate over key/element entries in the bucket.
		for i := uintptr(0); i < abi.MapBucketCount; i++ {
			// If the tophash values of the key and the ith key are different.
			if b.tophash[i] != top {
				// (Optimization): return if the rest is empty.
				if b.tophash[i] == emptyRest {
					break search
				}
				continue
			}
			
			// Pointer to the ith key in the bucket.
			k := add(unsafe.Pointer(b), dataOffset+i*uintptr(t.KeySize))
			k2 := k
			if t.IndirectKey() {
				k2 = *((*unsafe.Pointer)(k2))
			}
			// Compare the key with ith key in the bucket.
			if !t.Key.Equal(key, k2) {
				continue
			}
			// Clear key memory only if there are pointers in it.
			if t.IndirectKey() {
				*(*unsafe.Pointer)(k) = nil
			} else if t.Key.Pointers() {
				memclrHasPointers(k, t.Key.Size_)
			}
			
			// Pointer to the ith element in the bucket.
			e := add(unsafe.Pointer(b), dataOffset+abi.MapBucketCount*uintptr(t.KeySize)+i*uintptr(t.ValueSize))
			// Clear element memory. See: https://github.com/golang/go/commit/b080abf656feea5946922b2782bfeaa73cc317d4
			if t.IndirectElem() {
				*(*unsafe.Pointer)(e) = nil
			} else if t.Elem.Pointers() {
				memclrHasPointers(e, t.Elem.Size_)
			} else {
				memclrNoHeapPointers(e, t.Elem.Size_)
			}
			
			// Mark ith cell as empty
			b.tophash[i] = emptyOne
			// If the bucket now ends in a bunch of emptyOne states, change those to emptyRest states.
			if i == abi.MapBucketCount-1 {
				if b.overflow(t) != nil && b.overflow(t).tophash[0] != emptyRest {
					goto notLast
				}
			} else {
				if b.tophash[i+1] != emptyRest {
					goto notLast
				}
			}
			for {
				b.tophash[i] = emptyRest
				if i == 0 {
					if b == bOrig {
						break // beginning of initial bucket, we're done.
					}
					// Find previous bucket, continue at its last entry.
					c := b
					for b = bOrig; b.overflow(t) != c; b = b.overflow(t) {
					}
					i = abi.MapBucketCount - 1
				} else {
					i--
				}
				if b.tophash[i] != emptyOne {
					break
				}
			}
		notLast:
			// Decrement the size of the map.
			h.count--
			// Reset the hash seed to make it more difficult for attackers to repeatedly trigger hash collisions.
			if h.count == 0 {
				h.hash0 = uint32(rand())
			}
			break search
		}
	}

	if h.flags&hashWriting == 0 {
		fatal("concurrent map writes")
	}
	// Drop the flag that we are modifying the map.
	h.flags &^= hashWriting
}
```

### Memory Leaks

The number of buckets in a map cannot shrink. Therefore, removing elements from a map doesn't impact the number of existing buckets; it just zeroes the slots in the buckets. A map can only grow and have more buckets; it never shrinks

There are a couple of solutions if we want to delete unused buckets:

- Re-create a copy of the current map
- Store pointers as values. It doesn't solve the fact that we will have a significant number of buckets; however, each bucket entry will reserve only the size of a pointer for the value

# Addressability

Map elements are not addressable, because if the map grows, the address is of the stale value

Thus, we can't assign to `struct` field in the map:

``` go
type S struct {s int}
m := map[int]S{}
m[0] = S{0}
m[0].s = 1 // cannot assign to struct field m[0].s in map
```

Also, if the map is `nil` or does not contain an entry, `m[0]` would return the zero value. Therefore, attempting to assign to `m[0].s` would assign to a nonexistent zero value element

To overcome this problem, we need to copy the element and reassign:

``` go
v := m[0]
v.s = 1
m[0] = v
```

However, we can assign to a slice element because it is a [slice header](Go%20Slice%20Internals.md) containing a **pointer** to the underlying array:

``` go
m := map[int][]int{}
m[0] = []int{0}
m[0][0] = 1
```

# Map Iteration Guarantees

- The iteration order over maps is not specified and is not guaranteed to be the same from one iteration to the next
- If a map entry that has not yet been reached is removed during iteration, the corresponding iteration value will not be produced
- If a map entry is created during iteration, that entry may be produced during the iteration or may be skipped. The choice may vary for each entry created and from one iteration to the next
- If the map is `nil`, the number of iterations is 0

# References

- [If a map isn’t a reference variable, what is it? | Dave Cheney](https://dave.cheney.net/2017/04/30/if-a-map-isnt-a-reference-variable-what-is-it)
- [GopherCon 2016: Keith Randall - Inside the Map Implementation - YouTube](https://youtu.be/Tl7mi9QmLns?si=HOOEUzsmu2sLy9J2)
- [How the Go runtime implements maps efficiently (without generics) | Dave Cheney](https://dave.cheney.net/2018/05/29/how-the-go-runtime-implements-maps-efficiently-without-generics)
- [Maps in Golang | Intermediate level - YouTube](https://youtu.be/1LnzCtcfsFc?si=yK1iIsX4IF2nrtyU)
- [Internals of Maps in Golang - YouTube](https://youtu.be/ACQs6mdylxo?si=s3gHTZnxXz4rOR7y)
- [Source code: map.go](https://github.com/golang/go/blob/master/src/runtime/map.go)
- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
- [Go Maps Explained: How Key-Value Pairs Are Actually Stored](https://victoriametrics.com/blog/go-map/)
- [Go源码学习之map \| 极客熊生](https://www.kevinwu0904.top/blogs/golang-map/#map的扩容)
- [深度解密 Go 语言之 map \| qcrao 的博客](https://qcrao.com/post/dive-into-go-map/)
