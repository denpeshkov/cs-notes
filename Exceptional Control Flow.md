---
tags:
  - Arch
  - OS
aliases:
  - ECF
---

# Overview

The x86-64 CPU provides two mechanisms for interrupting program execution, interrupts, and exceptions

## Interrupts

Interrupts occur asynchronously as a result of signals from [I/O devices](Input-Output%20Devices.md) that are external to the CPU. Software can also generate interrupts by executing the `INT` instruction

[I/O devices](Input-Output%20Devices.md) such as network adapters, disk controllers, and timer chips trigger interrupts by signaling a pin on the CPU chip and placing onto the system bus the exception number that identifies the device that caused the interrupt. CPU checks the interrupt pin after every instruction, reads the exception number from the system bus, and then calls the appropriate interrupt handler

### The Interrupt Enable Flag

The interrupt enable (`IF`) flag in the `RFLAGS` register controls whether maskable interrupts are served by the CPU. This flag can be set using `STI` instruction and cleared using `CLI` instruction

## Exceptions

### Traps

Traps are intentional exceptions that occur as a result of executing an instruction. Trap handlers return control to the next instruction

### Faults

Faults result from error conditions that a handler might be able to correct. When a fault occurs, the CPU transfers control to the fault handler. If the handler is able to correct the error condition, it returns control to the faulting instruction, thereby re-executing it. Otherwise, the handler returns to an `abort` routine in the kernel that terminates the application program that caused the fault

### Aborts

Aborts result from unrecoverable fatal errors, typically hardware errors. Abort handlers never return control to the application program, the handler returns control to an `abort` routine that terminates the application program

# CPU Privilege Levels

The CPU uses privilege levels to prevent a program or task operating at a lesser privilege level from accessing a segment with a greater privilege, except under controlled situations. When the CPU detects a privilege level violation, it generates a general-protection exception

The x86-64 recognizes four privilege levels (protection rings), numbered from 0 to 3, the greater number mean less privilege. Each privilege level has its own [stack](Call%20Stack.md).

Most OSs use only two levels: level 0 for kernel mode and level 3 for user mode, with [kernel and user stack](Call%20Stack.md) respectively

# Interrupt Descriptor Table

On x86-64 each interrupt and exception are assigned a number, called a vector number. The x86-64 CPU supports 256 vector numbers from 0 to 255:

- Vector numbers 0-31 are x86-64 defined exceptions and interrupts: divide by zero, page faults, memory access violations, breakpoints, general-protection exception etc.
- Vector numbers 32-255 are defined by the OS kernel: system calls, signals from external I/O devices etc.

The x86-64 uses the Interrupt Descriptor Table (IDT) which associates each vector number with a gate descriptor for the interrupt/exception handler. Each gate descriptor is 16-bytes long and IDT is an array of 256 descriptors, one for each vector number

The IDT resides in [RAM](Main%20Memory.md) and its address is stored in the `IDTR` register. At system boot time OS sets up IDT using `LIDT` and `SIDT` instructions to load and store the contents of the `IDTR` register. At run time, when an interrupt or exception with vector number N occurs, CPU retrieves the gate descriptor stored at the address 16N+`IDTR`, and uses it to invoke the handler

## IDT Gate Descriptors

Code in lower privilege level can only access code operating at higher privilege level by means of a tightly controlled and protected interface called a gate. Attempts to access higher privilege levels without going through a gate and without having sufficient access rights causes a general-protection exception

The IDT may contain either an interrupt-gate descriptor or a trap-gate descriptor. If an interrupt or exception handler is called through an interrupt gate, the CPU clears the interrupt enable (`IF`) flag in the `RFLAGS` register to prevent subsequent interrupts from interfering with the execution of the handler. When a handler is called through a trap gate, the state of the `IF` flag is not changed

A gate descriptor is 16-bytes long and contains both a handler address and a descriptor privilege level (DPL) field, used to control access rights. The DPL field is a 2-bit value which defines the CPU privilege levels which are allowed to access this gate via the `INT` instruction. This restriction prevents programs running at user level from using a software interrupt to access critical exception handlers, such as the page-fault handler, providing that those handlers are placed in kernel space. For hardware-generated interrupts and processor-detected exceptions, the CPU ignores the DPL of interrupt and trap gates

# Calling and Return from Exceptions and Interrupts

A call to an interrupt or exception handler is similar to a [procedure call](Call%20Stack.md):

1. Temporarily saves (internally) the current contents of the `RSP`, `RFLAGS` and `RIP` registers
2. Loads the stack pointer for the kernel stack into `RSP` register and switches to the kernel stack
3. Pushes the temporarily saved `RSP`, `RFLAGS`, `RIP` values onto the kernel stack
4. Loads the new instruction pointer from the gate into the `RIP` register
5. If the call is through an interrupt gate, clears the `IF` flag in the `RFLAGS` register
6. Begins execution of the handler procedure in the kernel mode

The `IRET` instruction is used to return from an interrupt or exception handler:

1. Performs a privilege check
2. Restores `RIP`, `RFLAGS`
3. Restores `RSP` register, resulting in a switch back to the user stack
4. Resumes execution of the interrupted procedure in the user mode

# References

- [Computer Systems A Programmer's Perspective, Global Edition (3rd ed). Randal E. Bryant, David R. O'Hallaron](References.md#Computer%20Systems%20A%20Programmer's%20Perspective,%20Global%20Edition%20(3rd%20ed).%20Randal%20E.%20Bryant,%20David%20R.%20O'Hallaron)
- [CMU 15-213: Exceptional Control Flow: Exceptions and Processes](https://scs.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=d2759175-d59e-4f80-ab9e-24c2f15c8adb)
- [Caltech CS24: Virtualization - YouTube](https://youtu.be/kdeLkd8-EdI?si=gtskw4sk3FE0CcGQ)
- [Intel 64 and IA-32 Architectures Software Developer's Manual](References.md#Intel%2064%20and%20IA-32%20Architectures%20Software%20Developer's%20Manual)
