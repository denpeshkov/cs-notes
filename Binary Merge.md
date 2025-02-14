---
tags:
  - Data-Structures-Algorithms
---

# Pseudocode

``` go
func Merge(s1, s2 []int) []int {
	s := make([]int, len(s1)+len(s2))
	for i, j, k := 0, 0, 0; k < len(s); k++ {
		// ensures stability
		if i >= len(s1) || (j < len(s2) && s2[j] < s1[i] < 0) {
			s[k] = s2[j]
			j++
		} else {
			s[k] = s1[i]
			i++
		}
	}
	return s
}
```

# Properties

| Worst Time  | Average Time | Best Time   | Worst Space | Average Space | Stability |
| ----------- | ------------ | ----------- | ----------- | ------------- | --------- |
| $\theta(N)$ | $\theta(N)$  | $\theta(N)$ | $\theta(N)$ | $\theta(N)$   | Stable    |

# References

- [Merge algorithm - Wikipedia](https://en.wikipedia.org/wiki/Merge_algorithm)
