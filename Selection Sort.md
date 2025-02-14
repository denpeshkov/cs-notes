---
tags:
  - Data-Structures-Algorithms
---

# Pseudocode

```go
func Selection(s []int) {
	for i := 0; i < len(s); i++ {
		min := s[i]
		for j := i + 1; j < len(s); j++ {
			if s[j] < min {
				min = s[j]
			}
		}
		s[i] = min
	}
}
```

## Relation to Heapsort

[[Heapsort]] is just a selection sort backed by a [[Heap|heap]] rather than array

# Properties

| Worst Time    | Average Time  | Best Time   | Worst Space | Average Space | Stability  | Adaptive     |
| ------------- | ------------- | ----------- | ----------- | ------------- | ---------- | ------------ |
| $\theta(N^2)$ | $\theta(N^2)$ | $\theta(N)$ | $\theta(1)$ | $\theta(1)$   | Not Stable | Not Adaptive |

# References

- [Selection sort - Wikipedia](https://en.wikipedia.org/wiki/Selection_sort)
- [Algorithms (4th ed). Robert Sedgewick, Kevin Wayne](References.md#Algorithms%20(4th%20ed).%20Robert%20Sedgewick,%20Kevin%20Wayne)
- [Introduction to Algorithms (4th ed). Thomas H. Cormen, Charles E. Leiserson, Ronald L. Rivest, Clifford Stein](References.md#Introduction%20to%20Algorithms%20(4th%20ed).%20Thomas%20H.%20Cormen,%20Charles%20E.%20Leiserson,%20Ronald%20L.%20Rivest,%20Clifford%20Stein)
- [Сортировка выбором - Алгоритмика](https://ru.algorithmica.org/cs/sorting/selection/)
- [29.2 Selection Sort & Heapsort | CS61B Textbook](https://cs61b-2.gitbook.io/cs61b-textbook/29.-basic-sorts/29.2-selection-sort-and-heapsort)
