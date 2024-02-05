---
tags:
  - AlgsDs
---

# Pseudocode

```
i ← 1
while i < length(A)
    j ← i
    while j > 0 and A[j-1] > A[j]
        swap A[j] and A[j-1]
        j ← j - 1
    end while
    i ← i + 1
end while
```

# Properties

Time complexity:

| Worst | Average | Best |
| --- | --- | --- |
| $\theta(N^2)$ | $\theta(N^2)$ | $\theta(N)$ |

Space complexity auxiliary:

| Worst | Average | Best |
| --- | --- | --- |
| $\theta(1)$ | $\theta(1)$ | $\theta(1)$ |

Stability: stable

Adaptivity: adaptive

Online: yes

## Relation to Number of Inversions

The number of exchanges used by insertion sort is equal to the number of inversions in the array

The number of compares is at least equal to the number of inversions and at most equal to the number of inversions plus the array size minus 1

# References

- [Insertion sort - Wikipedia](https://en.wikipedia.org/wiki/Insertion_sort)
