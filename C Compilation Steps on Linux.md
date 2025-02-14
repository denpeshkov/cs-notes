---
tags:
  - OS-Architecture
---

# Steps

## Pre-processing

Expand compiler directives e.g. `#define`, `#include`

``` shell
gcc -E > out
```

## Compilation

Compiles the `.c` source code file to a `.s` assembly code file

```shell
gcc -S prog.c
```

## Assembly

Converts the `.s` assembly code file to a `.o` relocatable binary object code file. `gcc` on Unix and Linux produces [ELF](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) object files

```shell
gcc -c prog.c
objdump -d prog.o
```

## Linking

Links the `.o` files together along with `.a` and `.so` library files to create a `.out` executable file  
The `.o` files inside the `.a` **static library** archive get embedded inside the resulting `.out` file  
The `.so` files are not embedded but information about **dynamically loaded libs** is stored inside the resulting `.out` file

``` shell
gcc -o prog prog.c
objdump prog.out
```

[C Standard Library](C%20Standard%20Library.md) `libc.so` gets linked with every executable by `GCC` implicitly. Others libs need to use the `-l` option, e.g. `libmylib.so` or `libmylib.a`:

``` shell
gcc -o prog prog.c -lmylib
```

### Runtime Linking

Performs dynamic linking of `.so` library files during the execution of `.out` file  
The `ldd` utility lists an executable file's shared object dependencies:

``` shell
ldd prog.out
```

# References

- [Dive Into Systems A Gentle Introduction to Computer Systems. Suzanne J. Matthews, Tia Newhall, Kevin C. Webb](References.md#Dive%20Into%20Systems%20A%20Gentle%20Introduction%20to%20Computer%20Systems.%20Suzanne%20J.%20Matthews,%20Tia%20Newhall,%20Kevin%20C.%20Webb)
- [Computer Systems A Programmer's Perspective, Global Edition (3rd ed). Randal E. Bryant, David R. O'Hallaron](References.md#Computer%20Systems%20A%20Programmer's%20Perspective,%20Global%20Edition%20(3rd%20ed).%20Randal%20E.%20Bryant,%20David%20R.%20O'Hallaron)
