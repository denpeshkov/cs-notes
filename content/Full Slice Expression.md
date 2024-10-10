---
tags:
  - Go
---

# Overview

In Go, slicing with three indexes is done using `s[i:j:k]`

- Length: `j - i`
- Capacity: `k - i`

# Usage

This slicing technique is useful for preventing access to elements beyond a specified range in the backing array

When using a slice without the third index (`s[i:j]`), the `append` operation may reuse the backing array, modifying its content. By setting the capacity to 1 (e.g., `s[2:3:3]`), `append` allocates a new backing array, ensuring that the initial array remains unmodified

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

Here, the third element of the slice is modified, even though we pass only two elements. To protect the third element we can use full slice expression like so:

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

# Memory Usage

The inaccessible space (elements in the backing array with indices larger than capacity) is not reclaimed by [GC](Go%20Garbage%20Collector.md)

# References

- [Three-Index Slices in Go 1.2](https://www.ardanlabs.com/blog/2013/12/three-index-slices-in-go-12.html)
- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
