---
tags:
  - TODO
  - OS-Architecture
aliases:
  - JIT
---

# Definition

JIT compilation is compilation of a computer code during execution of a program (at run time). Whenever a program, while running, creates and runs some new executable code which was not part of the program when it was stored on disk, it's a JIT

# Overview

JIT can be divided into two distinct phases:

1. Create [machine code](http://en.wikipedia.org/wiki/Machine_code) at program run-time
2. Execute that machine code, also at program run-time

## Machine Code Creation

It is where 99% of the challenges of JITing are. This is exactly what a compiler does. The machine code is emitted into an output stream, but it could be just kept in memory, both `gcc` and `clang` can keep the code in memory for JIT execution

## Machine Code Execution

Suppose we want to dynamically create this `C` function in memory and execute it:

```c
long add4(long num) {
  return num + 4;
}
```

Here is the `C` code:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/mman.h>

// Allocates RWX memory of given size and returns a pointer to it. On failure,
// prints out the error and returns NULL.
void* alloc_executable_memory(size_t size) {
  void* ptr = mmap(0, size,
                   PROT_READ | PROT_WRITE | PROT_EXEC,
                   MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
  if (ptr == (void*)-1) {
    perror("mmap");
    return NULL;
  }
  return ptr;
}

void emit_code_into_memory(unsigned char* m) {
  unsigned char code[] = {
    0x48, 0x89, 0xf8,                   // mov %rdi, %rax
    0x48, 0x83, 0xc0, 0x04,             // add $4, %rax
    0xc3                                // ret
  };
  memcpy(m, code, sizeof(code));
}

const size_t SIZE = 1024;
typedef long (*JittedFunc)(long);

// Allocates RWX memory directly.
void run_from_rwx() {
  void* m = alloc_executable_memory(SIZE);
  emit_code_into_memory(m);

  JittedFunc func = m;
  int result = func(2);
  printf("result = %d\n", result);
}
```

The main 3 steps performed by this code are:

1. Use `mmap` to allocate a readable, writable and executable chunk of memory on the heap
2. Copy the machine code implementing `add4` into this chunk
3. Execute code from this chunk by casting it to a function pointer and calling through it

Note that step 3 can only happen because the memory chunk containing the machine code is _executable_. Without setting the right permission, that call would result in a runtime error from the OS. This would happen if, for example, we allocated `m` with a regular call to `malloc`, which allocates readable and writable, but not executable memory

# References

- [How to JIT - an introduction - Eli Bendersky's website](https://eli.thegreenplace.net/2013/11/05/how-to-jit-an-introduction)
- [Just-in-time compilation - Wikipedia](https://en.wikipedia.org/wiki/Just-in-time_compilation)
