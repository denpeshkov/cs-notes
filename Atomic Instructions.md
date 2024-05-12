---
tags: Concurrency OS_Arch TODO 
---

# Compare-and-Swap (CAS)

Compare-And-Swap (**CAS**) is an atomic instruction used to achieve synchronization: it compares the contents of a memory location with a given value and, if so, changes it to a new value. It returns a flag indicating whether it changed the value

CAS behaves like this function, but atomically:

```go
func cas(val *int32, old, new int32) bool {
	if *val == old {
		*val = new
		return true
	}
	return false
}
```

CAS is implemented using special hardware instructions provided by the CPU

- On x86-64 CAS is supported directly by the `CMPXCHG` instruction
- On ARM64 and RISC-V, CAS can be implemented using [LL/SC](#Load-Link/Store-Conditional) instructions

For example, on RISC-V:

```asm
# a0 holds address of memory location
# a1 holds expected value
# a2 holds desired value
# a0 holds return value, 0 if successful, !0 otherwise
cas:
    lr.w t0, (a0)       # Load original value.
    bne t0, a1, fail    # Doesn’t match, so fail.
    sc.w t0, a2, (a0)   # Try to update.
    bnez t0, cas        # Retry if store-conditional failed.
    li a0, 0            # Set return to success.
    jr ra               # Return.
fail:
    li a0, 1            # Set return to failure.
    jr ra               # Return.
```

## Double-Width CAS (DW-CAS)

Double-Width Compare-and-Swap (DW-CAS) is a CPU instruction performing a CAS operation on a memory location that's double the native word size. For example, it gives you 128-bit atomic lock-free variables on a 64-bit CPU

On x86-64, DW-CAS is supported directly by the `CMPXCHG16B` instruction

# Fetch-and-Add

Fetch-and-Add (FAA) is an atomic instruction used to increment the contents of a memory location by a specified value and return the original value

FAA is implemented using special hardware instructions provided by the CPU:

- On x86-64, FAA is implemented by the `XADD` instruction
- On RISC-V, FAA is implemented using the `AMOADD` instruction, while on ARM64, it is implemented using the `LDADDAL` instruction

FAA can be implemented using CAS on architectures without FAA support:

```c
long fetchadd(long *addr, long delta) {
  long v;
  for (;;) {
    v = *addr;
    if (cas(addr, v, v + delta)) {
	  return v + delta;
    }
  }
}
```

# Load-Link/Store-Conditional

Load-Link (**LL**) (also known as Load-Reserved (**LR**)) and Store-Conditional (**SC**) are a pair of atomic instructions used to achieve synchronization:

- Load-Link (LL) returns the current value of a memory location
- Store-Conditional (SC) writes to a memory location only if (1) it was the memory location used in the last LL instruction, and (2) that location has not been changed since the LL

Real implementations of LL/SC do not always succeed even if there are no concurrent updates to the memory location in question. Any exceptional events between the two operations, such as a [context switch](Context%20Switch.md), another LL, or even another load or store operation, will cause the SC to spuriously fail

Typically, CPUs track the LL address at a [cache-line](Cache%20Memory.md) granularity, such that any modification to any portion of the cache line (whether via another core's SC or merely by an ordinary store) is sufficient to cause the SC to fail

LL/SC are supported by both ARM64 `ldxr/stxr` and RISC-V `lr/sc` instructions

## Double-Width LL/SC

Double-Width LL/SC are LL/SC that act on double words. For example on RISC-V these instructions are `lr.d` and `sc.d`

## Compared to CAS

If any updates have occurred, the SC is guaranteed to fail, even if the value read by the LL has since been restored. As such, an LL/SC pair is stronger than a read followed by a CAS, which will not detect updates if the old value has been restored, thus eliminating the [ABA problem](ABA%20Problem.md)

One advantage of CAS is that it guarantees that some hardware thread eventually makes progress, whereas an LR/SC atomic sequence could livelock indefinitely on some systems, due to spurious fails. To avoid this concern, RISC-V added an architectural guarantee of livelock freedom for certain LR/SC sequences

# Test-and-Set

# Test and Test-and-Set

# References

- [Compare-and-swap - Wikipedia](https://en.wikipedia.org/wiki/Compare-and-swap)
- [Test-and-set - Wikipedia](https://en.wikipedia.org/wiki/Test-and-set)
- [Test and test-and-set - Wikipedia](https://en.wikipedia.org/wiki/Test_and_test-and-set)
- [Implementing Synchronization: CMU 15-418/618 Spring 2016](http://15418.courses.cs.cmu.edu/spring2016/lecture/synchronization)
- [Load-link/store-conditional - Wikipedia](https://en.wikipedia.org/wiki/Load-link/store-conditional)
- [Fetch-and-add - Wikipedia](https://en.wikipedia.org/wiki/Fetch-and-add)
- [Specifications – RISC-V International](https://riscv.org/technical/specifications/)
