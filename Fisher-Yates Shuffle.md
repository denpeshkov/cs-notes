---
tags:
  - AlgsDs
---

# Algorithm

```
for i = 0 to n-2 do {
	j <- random integer such that 0 <= j < n
	swap(a[i], a[j])
}
```

# The "inside-out" Algorithm

```
for i = 0 to n-1 do {
	j <- random integer such that 0 <= j <= i
	swap(a[i], a[j])
}
```

The advantage of this technique is that **n**, the number of elements in the array, does not need to be known in advance

# Proof

#TODO

# Properties

Time complexity:

| Worst | Average | Best |
| --- | --- | --- |
| $\theta(N)$ | $\theta(N)$ | $\theta(N)$ |

Space complexity auxiliary:

| Worst | Average | Best |
| --- | --- | --- |
| $\theta(1)$ | $\theta(1)$ | $\theta(1)$ |

Online: Yes, for "inside-out" version

# References

- [Introduction to Algorithms (4th ed). Thomas H. Cormen, Charles E. Leiserson, Ronald L. Rivest, Clifford Stein](References.md#Introduction%20to%20Algorithms%20(4th%20ed).%20Thomas%20H.%20Cormen,%20Charles%20E.%20Leiserson,%20Ronald%20L.%20Rivest,%20Clifford%20Stein)
- [Fisher–Yates shuffle - Wikipedia](https://en.wikipedia.org/wiki/Fisher–Yates_shuffle)
