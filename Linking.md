---
tags:
  - OS_Arch
  - TODO
---

# Compilation Steps

![compilation steps|600](compilation%20steps.png)

A `GCC`, provides a compiler driver that invokes the language *preprocessor*, *compiler*, *assembler*, and *linker*, as needed on behalf of the user:

1. The driver first runs the C *preprocessor* `cpp` which translates the C source file `hello.c` into an intermediate file `hello.i`
2. Next, the driver runs the C *compiler* `cc1`, which translates `hello.i` into an assembly file `hello.s`
3. Then, the driver runs the *assembler* `as`, which translates `hello.s` into a binary relocatable object file `hello.o`
4. Finally, it runs the *linker* `ld`, which combines `hello.o` and `printf.o`, along with the necessary system object files, to create the binary executable object file `hello`

# References

- [Computer Systems A Programmer's Perspective, Global Edition (3rd ed). Randal E. Bryant, David R. O'Hallaron](References.md#Computer%20Systems%20A%20Programmer's%20Perspective,%20Global%20Edition%20(3rd%20ed).%20Randal%20E.%20Bryant,%20David%20R.%20O'Hallaron)
- [Dive Into Systems A Gentle Introduction to Computer Systems. Suzanne J. Matthews, Tia Newhall, Kevin C. Webb](References.md#Dive%20Into%20Systems%20A%20Gentle%20Introduction%20to%20Computer%20Systems.%20Suzanne%20J.%20Matthews,%20Tia%20Newhall,%20Kevin%20C.%20Webb)
