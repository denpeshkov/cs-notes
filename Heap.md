---
tags:
  - Architecture
  - OS
---

# Heap Usage

Heap is used instead of [[Stack and Subroutine Calling|stack]] because:

- Stack space is automatically reclaimed when function returns. Stack values can last for up to the lifetime of a procedure call, no longer
- Stack space used by a procedure doesn't vary substantially during its execution:
	- Set of local variables within each code block is fixed
	- Set of arguments passed to a procedure is also fixed
- Memory required often depends on the input values (can vary dramatically)
- Some data structures e.g. linked list, can't be implemented using [[Stack and Subroutine Calling|stack]]

# References

- [[References#Caltech CS24-Sp18 Introduction-To-Computing-System - YouTube]]
