---
tags:
  - Arch
---

# I/O Devices Diagram

![IO devices access architecture.png|400](IO%20devices%20access%20architecture.png)

I/O devices are connected to the CPU via **I/O bus**

# Device Interaction

## Port-Mapped I/O (PMIO)

I/O devices use a separate address space from [main memory](Main%20Memory.md), either accomplished by an extra I/O pin on the CPU, or an entire bus dedicated to I/O

Uses explicit CPU instructions to perform I/O `in`/`out`

Such instructions are privileged and can be used only by the OS

## Memory-Mapped I/O (MMIO)

I/O devices use the same address space as [main memory](Main%20Memory.md)

The memory and registers of the I/O device are mapped to address values, so a memory address may refer to either a portion of [RAM](Main%20Memory.md) or to memory and [registers](#Memory%20Mapped%20Registers) of the device

Uses the same CPU instructions to access both RAM and devices `mov`

This memory region is managed by the OS and cannot be accessed directly by [user programs](User%20Space.md)

### Memory Mapped Registers

OS interacts with the device controller using **device registers**:

- **Status register** indicates the current status of the device
- **Command register** used to indicate the command to perform
- **Data register** stores data for reading and writing

There can be more registers: control register, error register, etc.

# Data Transfer Mechanisms

## Programmed I/O (PIO)

Each data item transfer is initiated by an instruction in the program, involving the CPU for every transaction

### Polling

1. Issue read/write command to I/O device
2. Repeatedly check (poll) status register until the device is ready
3. Perform read/write request between the device and [RAM](Main%20Memory.md)
4. If there is more data, repeat with step 1

Inefficient because it spends a lot of time polling and moving data, waisting CPU resources

### Interrupt-driven

1. Issue read/write command to I/O device
2. On [interrupt](Interrupts%20and%20Exceptions.md), perform read/write request between the device and [RAM](Main%20Memory.md)
3. If there is more data, repeat with step 1

Inefficient because it spends a lot of time moving data, waisting CPU resources

## Direct Memory Access (DMA)

#todo

CPU first initiates the transfer, then it does other operations while the transfer is in progress, and it finally receives an [interrupt](Interrupts%20and%20Exceptions.md) from the DMA controller (**DMAC**) when the operation is done

1. Issue read/write command to I/O device using DMA

Example read request:

1. CPU initiates read memory request by executing three store instructions to I/O device registers
	1. Sends a command word that tells the disk to initiate a read, along with other parameters such as whether to interrupt the CPU
	2. Indicates the logical block number that should be read
	3. Indicates the main memory address where the contents of the disk sector should be stored
2. The **disk controller** receives the read command, translates the logical block number to a sector address, reads the contents of the sector, and transfers the contents directly to main memory, without using the CPU
3. The disk controller notifies the CPU by sending an [interrupt](Interrupts%20and%20Exceptions.md) signal to the CPU

### Virtual Memory

The difficulties in having DMA in a [virtual memory](Virtual%20Memory.md) system arise because pages have both a physical and a virtual address

# Network I/O

Write about network I/O #TODO

# References

- [Computer Systems A Programmer's Perspective, Global Edition (3rd ed). Randal E. Bryant, David R. O'Hallaron](References.md#Computer%20Systems%20A%20Programmer's%20Perspective,%20Global%20Edition%20(3rd%20ed).%20Randal%20E.%20Bryant,%20David%20R.%20O'Hallaron)
- [[CS61C FA20] Lecture 31.1 - I/O: I/O Devices - YouTube](https://youtu.be/RwF5xnx5Bxo?si=qyVQSOBNpLbJMoDT)
- [Direct memory access - Wikipedia](https://en.wikipedia.org/wiki/Direct_memory_access)
- [Memory-mapped I/O and port-mapped I/O - Wikipedia](https://en.wikipedia.org/wiki/Memory-mapped_I/O_and_port-mapped_I/O)
- [Computer Organization and Design RISC-V Edition The Hardware Software Interface (2nd ed). David A. Patterson, John L. Hennessy](References.md#Computer%20Organization%20and%20Design%20RISC-V%20Edition%20The%20Hardware%20Software%20Interface%20(2nd%20ed).%20David%20A.%20Patterson,%20John%20L.%20Hennessy)
- [Programmed input–output - Wikipedia](https://en.wikipedia.org/wiki/Programmed_input–output)
- [I/O Techniques - Overview](http://inputoutput5822.weebly.com)
- [Operating Systems Three Easy Pieces. Remzi H Arpaci-Dusseau, Andrea C Arpaci-Dusseau](References.md#Operating%20Systems%20Three%20Easy%20Pieces.%20Remzi%20H%20Arpaci-Dusseau,%20Andrea%20C%20Arpaci-Dusseau)
