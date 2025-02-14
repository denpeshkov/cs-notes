---
tags:
  - OS-Architecture
aliases:
  - ABI
---

# Overview

ABI describes a system interface for compiled application programs. It is a set of specifications that detail:

- CPU instruction set, with details like register file structure, [stack organization](Call%20Stack.md), memory access types
- Sizes, layouts, and [alignments](Data%20Alignment.md) of data types
- Calling convention
- [System calls](System%20Calls.md)
- [Object files](Compiling,%20Assembling,%20Linking%20and%20Loading.md) binary format

[System V AMD64 ABI](https://wiki.osdev.org/System_V_ABI) is the standard ABI used by Linux

# Calling Convention

Calling convention describes the interface of the called code:

- The order in which parameters are allocated
- How parameters are passed (pushed on the stack, placed in registers, or a mix of both)
- Which registers (if any) are used for parameters
- Which register is used for the return value
- Which registers are **caller-saved** and which are **callee-saved**
- How the task of preparing the stack for, and restoring after, a function call is divided between the caller and the callee

The calling convention of a program's language may differ from the calling convention of the underlying platform, OS, or library being linked to. The function declarations will include additional platform-specific keywords that indicate the calling convention to be used

## Arguments Passing

The System V x86-64 ABI passes arguments in registers, which is more efficient than the System V x86 ABI's stack-based argument passing. This method avoids the latency and extra instructions associated with storing arguments to the [cache](Cache%20Memory.md) and then loading them back again in the callee

## Caller-saved and Callee-saved Registers

Caller-saved registers can be overwritten by the called subroutine and it's responsibility of the caller to preserve them

Callee-saved registers can't be overwritten by the called subroutine and it's responsibility of the subroutine to restore them to the state before the call

# References

- [Application binary interface - Wikipedia](https://en.wikipedia.org/wiki/Application_binary_interface)
- [compiler construction - What is an application binary interface (ABI)? - Stack Overflow](https://stackoverflow.com/questions/2171177/what-is-an-application-binary-interface-abi)
- [x86 calling conventions - Wikipedia](https://en.wikipedia.org/wiki/X86_calling_conventions)
- [Calling convention - Wikipedia](https://en.wikipedia.org/wiki/Calling_convention)
- [System V ABI - OSDev Wiki](https://wiki.osdev.org/System_V_ABI)
