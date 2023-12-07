---
tags:
  - Architecture
  - OS
aliases:
  - RAM
---

# DRAM Chip Access

![DRAM chip architecture.png|700](DRAM%20chip%20architecture.png)

The **memory controller** reads and writes data from each DRAM chip

1. The memory controller sends a **RAS** (row access strobe) request. The entire row is copied to the **internal row buffer**
2. The memory controller sends a **CAS** (column access strobe) request. The contents of the column in the internal row buffer are sent to the memory controller

RAS and CAS use the same address pins - so RAS request occurs before the CAS request.

# Main Memory Access

![RAM access architecture.png|400](RAM%20access%20architecture.png)

- The **system bus** connects the CPU to the I/O bridge
- The **memory bus** connects the I/O bridge to RAM
- The **I/O bridge** translates system bus signals to memory bus signals and also contains the **memory controller**
- The **bus interface** initiates read and write **bus transactions**

## Read Transaction

1. The CPU places the address on the system bus
	1. The I/O bridge passes the signal along to the memory bus
	2. Main memory senses the address signal on the memory bus, reads the address from the memory bus, fetches the data from the DRAM, and writes the data to the memory bus
2. The I/O bridge translates the memory bus signal into a system bus signal and passes it to the system bus
3. Finally, the CPU senses the data on the system bus, reads the data from the bus, and copies the data to the register

## Write Transaction

1. The CPU places the address on the system bus
	1. The memory reads the address from the memory bus and waits for the data to arrive.
2. The CPU copies the data in the register to the system bus
3. Main memory reads the data from the memory bus and stores the bits in the DRAM

# References

- [Computer Systems A Programmer's Perspective, Global Edition (3rd ed). Randal E. Bryant, David R. O'Hallaron](References.md#Computer%20Systems%20A%20Programmer's%20Perspective,%20Global%20Edition%20(3rd%20ed).%20Randal%20E.%20Bryant,%20David%20R.%20O'Hallaron)
