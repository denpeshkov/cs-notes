---
tags:
  - OS
---

# Heap Usage

Heap is used instead of a [stack](Call%20Stack.md) because:

- Stack space is automatically reclaimed when function returns. Stack values can last for up to the lifetime of a procedure call, no longer
- Stack space used by a procedure doesn't vary substantially during its execution:
	- Set of local variables within each code block is fixed
	- Set of arguments passed to a procedure is also fixed
- Memory required (e.g. size of the result) often depends on the input values (can vary dramatically)
- Some data structures e.g. [[Linked List]], can't be implemented using a [stack](Call%20Stack.md)

# References

- [CS24-Sp18-Introduction-To-Computing-System - YouTube](https://youtube.com/playlist?list=PL3swII2vlVoXiqUBV524pKEsP1iBN4UBU&si=TPdYM8NrC0Zxm-2M)
