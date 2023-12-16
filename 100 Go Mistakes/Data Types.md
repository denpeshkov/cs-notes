---
tags: Go
---

# Side Effects Using Slice `append`

Consider this example:

```go
func main() {
	s := []int{1, 2, 3}
	f(s[:2])
	fmt.Println(s) // [1,2,10]
}

func f(s []int) {
	_ = append(s, 10)
}
```

Here, the third element of the slice is modified, even though we pass only two elements

To protect the third element we can use [full slice expression](Full%20Slice%20Expression.md) like so:

```go
func main() {
	s := []int{1, 2, 3}
	f(s[:2:2])
	fmt.Println(s) // [1,2,3]
}

func f(s []int) {
	_ = append(s, 10)
}
```

# Declaring Empty Slices

We should prefer `nil` slices to empty slices

When designing interfaces, avoid making a distinction between a `nil` slice and a non-nil, zero-length slice, as this can lead to subtle programming error

Empty and `nil` slices have the [same memory footprint](Go%20Slices%20Internals.md)

# Maps and Memory Leaks

The number of buckets in a map cannot shrink. Therefore, removing elements from a map doesn't impact the number of existing buckets; it just zeroes the slots in the buckets

A map can only grow and have more buckets; it never shrinks

There are a couple of solutions if we want to delete unused buckets:

- Re-create a copy of the current map
- Store pointers as values. It doesn't solve the fact that we will have a significant number of buckets; however, each bucket entry will reserve only the size of a pointer for the value

# References

- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
- [Go Code Review Comments - The Go Programming Language](https://go.dev/wiki/CodeReviewComments#declaring-empty-slices)
