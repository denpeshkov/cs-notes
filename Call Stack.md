---
tags:
  - Arch
  - TODO
  - OS
---

# Usage

The call stack is implemented by the CPU and is used to save the state of the calling procedure, pass parameters to the called procedure, and store local variables for the currently executing procedure

- Return address is saved on the stack to support nested functions (e.g., recursion)
- On x86-64, up to 6 arguments are passed by registers; others are passed on the stack
- Local variables are saved on the stack if no registers are left or if the address of the variable needs to be taken

The stack grows toward lower addresses, and the [heap](Heap%20Memory.md) grows towards higher addresses, allowing the use of as much space as possible before a collision

The stack allows abstracting the subroutine implementation from the caller. Registers can be mutated by the subroutine, so without the stack, the caller has to prepare and be aware of subroutine interworking

![stack frame pointer.png|300](stack%20frame%20pointer.png)

# Kernel and User Stack

[Kernel space and user space](Interrupts%20and%20Exceptions.md) each have their own distinct call stacks. The OS switches between them when transferring control between kernel and user space, e.g. via [interrupts](Interrupts%20and%20Exceptions.md). This stack switching is done to prevent privileged procedures from crashing due to insufficient stack space. It also prevents less privileged procedures from interfering with more privileged procedures by sharing the same stack

# Stack Registers

The stack pointer (contained in `RSP` register) stores the address of the last stack frame - the top of the stack (lowest address)

The stack-frame base pointer (contained in `RBP` register) identifies a fixed reference point within the stack frame for the called procedure. On x86-64 `RBP` is used only when the stack frame can be of variable size. On i386, most compilers always used `EBP`, but recent versions of `GCC` have dropped this convention. The old `RBP` is saved because it's a [callee-saved](Application%20Binary%20Interface%20(ABI).md) register

![stack no frame pointer.png|300](stack%20no%20frame%20pointer.png)

# Stack Manipulation Instructions

Items are placed on the stack using the `PUSH` instruction and removed from the stack using the `POP` instruction:

- `PUSH` decrements the stack pointer (contained in the `RSP` register), and copies the source operand to the top of stack
- `POP` copies the value at the current top of stack (indicated by the `RBP` register) to the location specified with the destination operand. It then increments the `RBP` register to point to the new top of stack

# Calling and Return from Procedure

The `CALL` instruction is used to transfer control to the procedure:

1. Pushes the current value of the `RIP` register on the stack
2. Loads the offset of the called procedure in the `RIP` register
3. Begins execution of the called procedure

The `RET` instruction is used to return from the called procedure:

1. Pops the top-of-stack value (the return instruction pointer) into the `RIP` register
2. Resumes execution of the calling procedure

By convention:

- Once a function completes, the stack returns to the state it was in prior to the function call
- The return value from a subroutine is stored in a register
- Parameters that are passed on the stack are in reverse order. This allows sensible order - later parameters have bigger offsets from `RBP`. It also allows access to fixed arguments (e.g., the number of arguments passed in) when using variadic functions
- Old data on the stack from the previous call is usually not cleaned

# Stack Memory Management

#TODO Elaborate

By default Linux user space stacks use 8Mb of [virtual address space](Virtual%20Memory.md), divided into 4Kb physical pages. Those pages are allocated lazily, so in reality only a subset of 8Mb address space is used

There is a protected guard page at the end of the stack address space which causes a [trap](Interrupts%20and%20Exceptions.md) and process termination on access, which prevents stack overflow

1. The OS allocates 8MB of virtual memory for a stack by setting up the [MMU](Virtual%20Memory.md)'s page tables for a thread. This requires very little [RAM](Main%20Memory.md) to hold the page table entries only
2. When a thread runs and tries to access a [virtual address](Virtual%20Memory.md) on the stack that doesn't have a physical page assigned to it yet, a [page fault exception](Interrupts%20and%20Exceptions.md) is triggered by the MMU
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
