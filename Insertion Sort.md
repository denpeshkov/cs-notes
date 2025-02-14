---
tags:
  - Data-Structures-Algorithms
---

# Pseudocode

```go
func Insertion(s []int) {
	for i := 1; i < len(s); i++ {
		for j := i; j > 0 && s[j-1] > s[j]; j-- {
			s[j-1], s[j] = s[j], s[j-1]
		}
	}
}
```

# Properties

| Worst Time    | Average Time  | Best Time   | Worst Space | Average Space | Stability | Adaptive |
| ------------- | ------------- | ----------- | ----------- | ------------- | --------- | -------- |
| $\theta(N^2)$ | $\theta(N^2)$ | $\theta(N)$ | $\theta(1)$ | $\theta(1)$   | Stable    | Adaptive |

Insertion sort is adaptive: efficient for data sets that are already substantially sorted, the time complexity is $\theta(kN)$ when each element in the input is no more than $k$ places away from its sorted position

## Relation to Number of Inversions

The number of exchanges used by insertion sort is equal to the number of inversions

The number of compares is at least equal to the number of inversions and at most equal to the number of inversions plus the array size minus 1

**Proof:** Every exchange involves two inverted adjacent entries and thus reduces the number of inversions by one, and the array is sorted when the number of inversions reaches zero. Every exchange corresponds to a compare, and an additional compare might happen for each value of `i` from 1 to `N-1` (when `a[i]` does not reach the left end of the array).

# References

- [Insertion sort - Wikipedia](https://en.wikipedia.org/wiki/Insertion_sort)
- [Сортировка вставками - Алгоритмика](https://ru.algorithmica.org/cs/sorting/insertion/)
- [29.4 Insertion Sort | CS61B Textbook](https://cs61b-2.gitbook.io/cs61b-textbook/29.-basic-sorts/29.4-insertion-sort)
- [Algorithms (4th ed). Robert Sedgewick, Kevin Wayne](References.md#Algorithms%20(4th%20ed).%20Robert%20Sedgewick,%20Kevin%20Wayne)
- [Introduction to Algorithms (4th ed). Thomas H. Cormen, Charles E. Leiserson, Ronald L. Rivest, Clifford Stein](References.md#Introduction%20to%20Algorithms%20(4th%20ed).%20Thomas%20H.%20Cormen,%20Charles%20E.%20Leiserson,%20Ronald%20L.%20Rivest,%20Clifford%20Stein)
