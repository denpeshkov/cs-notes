---
tags:
  - Go
---

# Overview

In Go, slicing with three indexes is done using `s[i:j:k]`

- Length: `j - i`
- Capacity: `k - i`

## Usage

This slicing technique is useful for preventing access to elements beyond a specified range in the backing array

When using a slice without the third index (`s[i:j]`), the `append` operation may reuse the backing array, modifying its content. By setting the capacity to 1 (e.g., `s[2:3:3]`), `append` allocates a new backing array, ensuring that the initial array remains unmodified

# References

- [Three-Index Slices in Go 1.2](https://www.ardanlabs.com/blog/2013/12/three-index-slices-in-go-12.html)
