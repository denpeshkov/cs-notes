---
tags:
  - Arch
  - OS
aliases:
  - ABI
---

# Overview

ABI describes how data structures and computational routines are accessed in machine code. ABI includes the following:

- CPU instruction set, with details like register file structure, [stack organization](Call%20Stack.md), memory access types
- Sizes, layouts, and [alignments](Data%20Alignment.md) of basic data types that the CPU can directly access
- [Calling convention](Calling%20Convention.md)
- How an application should make [system calls](System%20Calls.md) to the OS
- Libraries and [object files](Executable%20File%20Format%20(ELF)) binary format

[System V AMD64 ABI](https://wiki.osdev.org/System_V_ABI) is the standard ABI used by Unix OSs such as Linux, the BSD systems, MacOS etc.

# References

- [Application binary interface - Wikipedia](https://en.wikipedia.org/wiki/Application_binary_interface)
- [compiler construction - What is an application binary interface (ABI)? - Stack Overflow](https://stackoverflow.com/questions/2171177/what-is-an-application-binary-interface-abi)
