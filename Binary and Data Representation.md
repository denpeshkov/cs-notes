---
tags:
  - Math
  - OS-Architecture
---

# Two's Complement

The bit sequence $b_{n-1}b_{n-2}\ldots b_0$ represents the number $-b_{n-1} \cdot 2^{n-1} + b_{n-2} \cdot 2^{n-2} + \ldots + b_0 \cdot 2^0$

If the most significant bit is 1, the overall value will be negative because that first bit contributes the largest absolute value to the sum

The value $-x$ is represented as $\neg x + 1$

# Fixed-Point

It uses the notation $b_mb_{m-1}\ldots b_1b_0.b_{-1}b_{-2}\ldots b_{-n+1}b_{-n}$ to represent the number $\sum_{i=-n}^m 2^i \cdot b_i$

It can represent numbers of similar magnitudes with the same precision. For example, numbers with 4 digits after the decimal point have precision up to $1/2^4 = 1/16$

It can **only** exactly represent numbers of the form $x \cdot 2^y$. Other rational numbers have repeating bit representations

The binary point has a fixed position, so a lot of digits are needed to represent very small or very large numbers

# Floating Point (IEEE-754)

It represents numbers of the form $x \cdot 2^y$ by specifying $x$ and $y$

The number is represented as $(-1)^S \cdot M \cdot 2^E$

The bit representation is divided into three fields to encode these values:

- The single sign bit $s$ directly encodes the **sign** $S$
- The $k$-bit exponent field $exp=e_{k-1}\ldots e_1e_0$ encodes the **exponent** $E$
- The $n$-bit fraction field $frac=f_{n-1}\ldots f_1f_0$ encodes the **significand** (**mantissa**) $M$

## Normalized Values

$exp$ is neither all zeros nor all ones

$E = exp - Bias$ where $Bias=2^{k-1}-1$

$M = 1.frac$

## Denormalized Numbers

$exp$ is all zeros

$E = 1 - Bias$ where $Bias=2^{k-1}-1$

$M = 0.frac$

These are used to represent $\pm 0$ and numbers close to zero

The smallest normalized value $2^{1-Bias} \cdot 1.0$ comes right after the biggest denormalized number $2^{1-Bias} \cdot 0.1\ldots 1$

## Special Values

$exp$ is all ones

- If $frac$ is all zeros, it represents $\pm\infty$ depending on sign $S$
- If $frac$ is nonzero, it represents $NaN$

## Comparison

The IEEE format was designed so that floating-point numbers could be sorted using an integer sorting routine. If we interpret the bit representations of the values as unsigned integers, they occur in ascending order, as do the values they represent as floating-point numbers

This is why we use $Bias$ to represent negative $E$ instead of two's-complement

## Conversions

- From `int` to `float`, the number cannot overflow, but it may be rounded
- From `int` or `float` to `double`, the exact numeric value can be preserved because `double` has both greater range (i.e., the range of representable values) and greater precision (i.e., the number of significant bits)
- From `double` to `float`, the value can overflow to $\pm\infty$, since the range is smaller. Otherwise, it may be rounded because the precision is smaller
- From `float` or `double` to `int`, the value will be rounded toward zero. Furthermore, the value may overflow
- From a smaller unsigned integer to a larger integer, the value is zero-extended
- From a smaller signed integer to a larger number, the value is sign-extended
- From a larger integer to a smaller integer, the value is truncated

# References

- [Exponent bias - Wikipedia](https://en.wikipedia.org/wiki/Exponent_bias)
- [The Floating-Point Guide - What Every Programmer Should Know About Floating-Point Arithmetic](https://floating-point-gui.de)
- [Computer Systems A Programmer's Perspective, Global Edition (3rd ed). Randal E. Bryant, David R. O'Hallaron](References.md#Computer%20Systems%20A%20Programmer's%20Perspective,%20Global%20Edition%20(3rd%20ed).%20Randal%20E.%20Bryant,%20David%20R.%20O'Hallaron)
- [IEEE floating-point standard - Wikipedia, the free encyclopedia](http://www.kramirez.net/Discretas/Material/Internet/IEEE_754.htm)
- [Single-precision floating-point format - Wikipedia](https://en.wikipedia.org/wiki/Single-precision_floating-point_format)
- [Floating-point arithmetic - Wikipedia](https://en.wikipedia.org/wiki/Floating-point_arithmetic)
- [IEEE 754 - Wikipedia](https://en.wikipedia.org/wiki/IEEE_754)
