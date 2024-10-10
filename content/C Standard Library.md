---
tags:
  - OS
aliases:
  - libc
  - ISO C Library
---

# Overview

The **C standard library** is the standard library for the C language, as specified in the **ISO C** (**ANSI C**) standard

The API of the C standard library is declared in a number of [header files](https://en.wikipedia.org/wiki/C_standard_library#Header_files). Each header file contains one or more function declarations, data type definitions, and macros

On Unix-like systems, the documentation of the API is provided by `man` pages

## Standardization

**POSIX** is a family of standards that defines both the system and user-level API, along with shells and utility interfaces, for software compatibility with variants of UNIX and other operating systems

The **Single UNIX Specification** (**SUS**) is a standard for operating systems compliance, with which being required to qualify for using the UNIX trademark

The **C POSIX library** is a part of POSIX, a specification of a C standard library for POSIX compatible systems. It is a superset of the C standard library, which adds several nonstandard C headers for Unix-specific functionality. For example, [`unistd.h`](https://en.wikipedia.org/wiki/Unistd.h) provides access to the POSIX OS API

## Freestanding and Hosted Implementations

There are two kinds of the C implementations: **hosted**, where the C standard library is available; and **freestanding**, where only a few headers are usable that contains only definitions and types

An OS kernel is an example of a program running in a freestanding environment; a program using the facilities of an OS is an example of a program running in a hosted environment

# Implementation

Unix typically has a C library (**`libc`**) in a shared library form, and is linked automatically into every executable

The C library is considered part of the operating system on Unix systems; in addition to functions specified by the C standard, it includes other functions that are part of the operating system API, such as functions specified in the POSIX standard

There are couple of implementations of the `libc`: BSD `libc`, `glibc`, `musl` etc.

## Compiler Built-in Functions

GCC provides built-in versions of many of the functions in the `libc`; that is, the implementations of the functions are written into the compiled object file, and the program calls the built-in versions instead of the functions in the C library shared object file

# Linux API and ABI

A C standard library for Linux includes **wrappers** around the [system calls](System%20Calls.md) of the Linux kernel. Many library functions don't make any use of [system calls](System%20Calls.md) (e.g., the string-manipulation functions). On the other hand, some library functions are layered on top of system calls

The combination of the Linux kernel system call interface and a C standard library is what builds the Linux API

Linux API provides additional capabilities that are not part of POSIX. For example:

- [`cgroups`](Docker%20Architecture.md) subsystem,
- The system calls [`futex`](https://en.wikipedia.org/wiki/Futex), [`epoll`](https://en.wikipedia.org/wiki/Epoll), [`splice`](https://en.wikipedia.org/wiki/Splice_(system_call)), [`dnotify`](https://en.wikipedia.org/wiki/Dnotify), [`fanotify`](https://en.wikipedia.org/wiki/Fanotify), [`inotify`](https://en.wikipedia.org/wiki/Inotify)

# References

- [C standard library - Wikipedia](https://en.wikipedia.org/wiki/C_standard_library)
- [C Library - OSDev Wiki](https://wiki.osdev.org/C_Library)
- [POSIX - Wikipedia](https://en.wikipedia.org/wiki/POSIX)
- [ANSI C - Wikipedia](https://en.wikipedia.org/wiki/ANSI_C)
- [unistd.h - Wikipedia](https://en.wikipedia.org/wiki/Unistd.h)
- [Calling convention - Wikipedia](https://en.wikipedia.org/wiki/Calling_convention)
