---
tags: Concurrency 
---

# Coffman Conditions

Conditions that must be present for deadlock to arise:

1. **Mutual Exclusion**. A concurrent process holds exclusive rights to a resource at any one time
2. **Wait For Condition**. A concurrent process must simultaneously hold a resource and be waiting for an additional resource
3. **No Preemption**. A resource held by a concurrent process can only be released by that process, so it fulfills this condition
4. **Circular Wait**. A concurrent process `P1` must be waiting on a chain of other concurrent processes `P2`, which are in turn waiting on it `P1`, so it fulfills this final condition too

# References

- [Concurrency in Go. Katherine Cox-Buday](References.md#Concurrency%20in%20Go.%20Katherine%20Cox-Buday)
