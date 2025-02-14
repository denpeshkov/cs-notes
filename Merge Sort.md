---
tags:
  - Data-Structures-Algorithms
---

## Top-down Merge Sort

Pseudocode:

```go
func Merge(s []int) {
	aux := make([]int, len(s))
	var sort func(s S)
	sort = func(s S) {
		if len(s) <= 1 {
			return
		}
		m := len(s) / 2
		sort(s[:m])
		sort(s[m:])
		merge.MergeInto(s[:m], s[m:], cmp, aux)
		copy(s, aux)
	}
	sort(s)
}
```

The function `merge.MergeInto` is an implementation of [binary merge algorithm](Binary%20Merge.md), that stores the result into the provided `aux`

## Bottom-up Merge Sort

Bottom-up merge sort eliminates recursion by using an iterative approach, which avoids the risk of stack overflow

Pseudocode:

```go
func MergeBU(s []int) {
	n := len(s)
	aux := make([]int, n)
	for sz := 1; sz < n; sz *= 2 {
		for l := 0; l < n-sz; l += 2 * sz {
			m, r := l+sz, min(l+2*sz, n)
			merge.MergeInto(s[l:m], s[m:r], cmp, aux)
			copy(s[l:r], aux)
		}
	}
}
```

The function `merge.MergeInto` is an implementation of [binary merge algorithm](Binary%20Merge.md), that stores the result into the provided `aux`

## Sorting Linked Lists

A bottom-up mergesort is the method of choice for sorting data organized in a [linked list](Linked%20List.md) . Consider the list to be sorted sublists of size 1, then pass through to make sorted subarrays of size 2 linked together, then size 4, and so forth. This method rearranges the links to sort the list in place (without creating any new list nodes)

# Properties

| Worst Time         | Average Time       | Best Time          | Worst Space | Average Space | Stability | Adaptive     |
| ------------------ | ------------------ | ------------------ | ----------- | ------------- | --------- | ------------ |
| $\theta(N\log{N})$ | $\theta(N\log{N})$ | $\theta(N\log{N})$ | $\theta(N)$ | $\theta(N)$   | Stable    | Non Adaptive |

# References

- [Merge sort - Wikipedia](https://en.wikipedia.org/wiki/Merge_sort)
- [Algorithms (4th ed). Robert Sedgewick, Kevin Wayne](References.md#Algorithms%20(4th%20ed).%20Robert%20Sedgewick,%20Kevin%20Wayne)
- [Introduction to Algorithms (4th ed). Thomas H. Cormen, Charles E. Leiserson, Ronald L. Rivest, Clifford Stein](References.md#Introduction%20to%20Algorithms%20(4th%20ed).%20Thomas%20H.%20Cormen,%20Charles%20E.%20Leiserson,%20Ronald%20L.%20Rivest,%20Clifford%20Stein)
