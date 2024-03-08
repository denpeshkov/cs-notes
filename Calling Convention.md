---
tags: OS_Arch
---

# Overview

Calling convention describes the interface of the called code:

- The order in which parameters are allocated
- How parameters are passed (pushed on the stack, placed in registers, or a mix of both)
- Which registers (if any) are used for parameters
- Which register is used for the return value
- Which registers are **caller-saved** and which are **callee-saved**
- How the task of preparing the stack for, and restoring after, a function call is divided between the caller and the callee

The calling convention of a program's language may differ from the calling convention of the underlying platform, OS, or library being linked to. The function declarations will include additional platform-specific keywords that indicate the calling convention to be used

On Unix systems the calling convention is specified by [System V AMD 64 ABI](Application%20Binary%20Interface%20(ABI).md)

## Caller-saved and Callee-saved Registers

Caller-saved registers can be overwritten by the called subroutine and it's responsibility of the caller to preserve them

Callee-saved registers can't be overwritten by the called subroutine and it's responsibility of the subroutine to restore them to the state before the call

# References

- [x86 calling conventions - Wikipedia](https://en.wikipedia.org/wiki/X86_calling_conventions)
- [Calling convention - Wikipedia](https://en.wikipedia.org/wiki/Calling_convention)
- [System V ABI - OSDev Wiki](https://wiki.osdev.org/System_V_ABI)
