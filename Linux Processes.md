---
tags:
  - OS-Architecture
  - TODO
---

# Overview

A process is an instance of a program in execution. Each program in the system runs in the context of some process

## Process Address Space

![Linux x86-64 process memory layout|400](Linux%20x86-64%20process%20memory%20layout.png)

# Context Switching

The kernel is not a separate process, but rather runs as part of some existing process

# State

- Running - ﻿Process is either executing, or waiting to be executed and will eventually be *scheduled* by the kernel
- Stopped - ﻿Process execution is *suspended* and will not be scheduled until further notice
- Terminated - Process is stopped permanently

Process becomes terminated for one of three reasons:

1. Receiving a signal whose default action is to terminate
2. Returning from the `main` routine
3. Calling the `exit` function

# Creating Processes

Parent process creates a new running child process by calling `int fork(void)`:

- Returns O to the child process, child's PID to parent process
- Child is almost identical to parent:
	- Child get an identical (but separate) copy of the parent's virtual address space.
	- Child gets identical copies of the parent's open file descriptors
	- Child has a different PID than the parent

# Reaping Child Processes

When process terminates OS keeps it around until it reaped by it's parent, because parent may want an exit status of the child. It still consumes system resources: exit status, various OS tables

Reaping:

- Performed by parent on terminated child (using `wait` or `waitpid`)
- Parent is given exit status information
- Kernel then deletes zombie child process

If any parent terminates without reaping a child, then the orphaned child will be reaped by `init` process (pid 1). ﻿﻿So, only need explicit reaping in long-running processes e.g., shells and servers

# Referencese

- [Computer Systems A Programmer's Perspective, Global Edition (3rd ed). Randal E. Bryant, David R. O'Hallaron](References.md#Computer%20Systems%20A%20Programmer's%20Perspective,%20Global%20Edition%20(3rd%20ed).%20Randal%20E.%20Bryant,%20David%20R.%20O'Hallaron)
- [14 ECF Exceptions & Processes - YouTube](https://youtu.be/H8PpoEAnB6k?si=WBsKecZQv2FL7ISM)
