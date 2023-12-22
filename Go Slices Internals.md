---
tags: Go
---

# Representation

Slices are represented by a `runtime.slice` struct (*slice header*):

```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

- `array` is a pointer to the underlying array
- `len` represents the length of a slice
- `cap` indicates the capacity of a slice

Note that, slice is *not* a pointer to a struct

## `nil` Slice

`nil` slice is actually a zero value of the slice header:

```go
slice {
	array: nil,
	len:   0,
	cap:   0,
}
```

## Empty Slice

Empty slice is just a slice with `len` and `cap` equal to 0 and `array` pointing to 'zero-sized' array:

```go
slice {
	array: unsafe.Pointer(&zerobase),
	len:   0,
	cap:   0,
}
```

The value of the `array` is the address of the `runtime.zerobase`, the base address for all 0-byte allocations

# Type Information

[Type information](Go%20Type%20Internals.md) about the slice data type is represented by an `abi.SliceType` struct:

```go
type SliceType struct {
	Type
	Elem *Type // slice element type
}
```

# Invariance and Conversions

Slices are invariant. This is one of the reasons why we can't convert `[]T` to an `[]interface{}`  
Another, is that memory layout of these two slices is different:

- Each `interface{}` takes up [two words](Go%20Interfaces%20Internals.md). As a consequence, a slice with length `N` and with type `[]interface{}` is backed by a chunk of data that is `N*2` words long
- This is different than the chunk of data backing a slice with type `[]MyType` and the same length. Its chunk of data will be `N*sizeof(MyType)` words long

---

- Converting a slice to an *array* yields an array containing the elements of the underlying array of the slice
- Converting a slice to an *array pointer* yields a pointer to the underlying array of the slice

In both cases, if the length of the slice is less than the length of the array, a panic occurs

Note that converting a `nil` slice to an array pointer yields a `nil` pointer, while converting an empty slice to an array pointer yields a non-`nil` pointer

# References

- [null - nil slices vs non-nil slices vs empty slices in Go language - Stack Overflow](https://stackoverflow.com/questions/44305170/nil-slices-vs-non-nil-slices-vs-empty-slices-in-go-language)
- [Slices from the ground up | Dave Cheney](https://dave.cheney.net/2018/07/12/slices-from-the-ground-up)
- [Go Slices: usage and internals - The Go Programming Language](https://go.dev/blog/slices-intro)
- [research!rsc: Go Data Structures](https://research.swtch.com/godata)
- [InterfaceSlice · golang/go Wiki · GitHub](https://github.com/golang/go/wiki/InterfaceSlice)
- [Arrays, slices (and strings): The mechanics of 'append' - The Go Programming Language](https://go.dev/blog/slices)
- [Go internals: invariance and memory layout of slices - Eli Bendersky's website](https://eli.thegreenplace.net/2021/go-internals-invariance-and-memory-layout-of-slices/)
- [Source code: slice.go](https://github.com/golang/go/blob/master/src/runtime/slice.go)
- [Source code: malloc.go](https://github.com/golang/go/blob/master/src/runtime/malloc.go)
- [The Go Programming Language Specification - The Go Programming Language](https://go.dev/ref/spec)
