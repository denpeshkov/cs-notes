---
tags:
  - Arch
  - TODO
  - OS
---

# Usage

The stack is implemented by the CPU and is used to save the **return address**, **arguments**, and **local variables** prior to calling the subroutine

- Return address is saved on the stack to support nested functions (e.g., recursion)
- On `x86-64`, up to 6 arguments are passed by **registers**; others are passed on the stack
- Local variables are saved on the stack if no registers are left or if the address of the variable needs to be taken

The stack grows toward lower addresses, and the [heap](Heap%20Memory.md) grows towards higher addresses, allowing the use of as much space as possible before a collision

The stack allows abstracting the subroutine implementation from the caller. Registers can be mutated by the subroutine, so without the stack, the caller has to prepare and be aware of subroutine interworking

![stack frame pointer.png|300](stack%20frame%20pointer.png)

# `push` And `pop` Assembler Instructions

- `push V` pushes a copy of `V` onto the top of the stack
- `pop Addr` pops the top element off the stack and places it in location `Addr`

Often, you can use `mov` instead of `push` and `pop`. For example, `sub SP` to store all values and use `mov` to store values using addresses directly

# Registers

## Stack Pointer

The `SP` register stores the address of the last stack frame - the top of the stack (lowest address)

## Frame (Base) Pointer

On `x86-64`, `BP` is used only when the stack frame can be of variable size. On `IA32`, most compilers always used `BP`. Recent versions of `GCC` have dropped this convention. `BP` is not really necessary to support VLA but used as a convention. See: [Is BP register really necessary to support Variable-Size Stack Frames? - Stack Overflow](https://stackoverflow.com/a/37584112/15600693)

`BP` is used to have a fixed point to reference variables on the stack. The old `BP` is saved because it's a [callee-saved](Calling%20Convention.md) register

![stack no frame pointer.png|300](stack%20no%20frame%20pointer.png)

# `call`, `ret`, `leave` Assembler Instructions

- `call` pushes the address of the next instruction (**return address**) onto the stack and sets `PC` to the address of the called subroutine
- `ret` pops the **return address** from the stack and stores it into `PC`
- `leave` deallocates the stack's frame space and restores the old `BP` value. Used only when `BP` is used

By convention:

- Once a function completes, the stack returns to the state it was in prior to the function call
- The return value from a subroutine is stored in a register
- Parameters that are passed on the stack are in reverse order. This allows sensible order - later parameters have bigger offsets from `BP`. It also allows access to fixed arguments (e.g., the number of arguments passed in) when using variadic functions

Old data on the stack from the previous call is usually not cleaned

# Stack Memory Management

#TODO Elaborate

By default Linux user space stacks use 8Mb of [virtual address space](Virtual%20Memory.md), divided into 4Kb physical pages. Those pages are allocated lazily, so in reality only a subset of 8Mb address space is used

There is a protected guard page at the end of the stack address space which causes a [trap](Exceptional%20Control%20Flow.md) and process termination on access, which prevents stack overflow

1. The OS allocates 8MB of virtual memory for a stack by setting up the [MMU](Virtual%20Memory.md)'s page tables for a thread. This requires very little [RAM](Main%20Memory.md) to hold the page table entries only
2. When a thread runs and tries to access a [virtual address](Virtual%20Memory.md) on the stack that doesn't have a physical page assigned to it yet, a [page fault exception](Exceptional%20Control%20Flow.md) is triggered by the MMU
3. The CPU core responds to the page fault exception by switching to a privileged execution mode (which has its own stack) and calling the page fault exception handler function inside the kernel
4. The kernel allocates a page of physical RAM to that virtual memory page and returns back to the user space thread

# References

- [Calling Conventions - OSDev Wiki](https://wiki.osdev.org/Calling_Conventions)
- [Calling convention - Wikipedia](https://en.wikipedia.org/wiki/Calling_convention)
- [CS24-Sp18-Introduction-To-Computing-System - YouTube](https://youtube.com/playlist?list=PL3swII2vlVoXiqUBV524pKEsP1iBN4UBU&si=B_w5UOwuIXVU-pq_)
- [Web Aside ASM:IA32](http://csapp.cs.cmu.edu/3e/waside/waside-ia32.pdf)
- [Computer Systems A Programmer's Perspective, Global Edition (3rd ed). Randal E. Bryant, David R. O'Hallaron](References.md#Computer%20Systems%20A%20Programmer's%20Perspective,%20Global%20Edition%20(3rd%20ed).%20Randal%20E.%20Bryant,%20David%20R.%20O'Hallaron)
- [Is BP register really necessary to support Variable-Size Stack Frames? - Stack Overflow](https://stackoverflow.com/a/37584112/15600693)
- [Dive Into Systems A Gentle Introduction to Computer Systems. Suzanne J. Matthews, Tia Newhall, Kevin C. Webb](References.md#Dive%20Into%20Systems%20A%20Gentle%20Introduction%20to%20Computer%20Systems.%20Suzanne%20J.%20Matthews,%20Tia%20Newhall,%20Kevin%20C.%20Webb)
- [Dmitry Vyukov — Go scheduler: Implementing language with lightweight concurrency - YouTube](https://www.youtube.com/watch?v=-K11rY57K7k&t=824s)
