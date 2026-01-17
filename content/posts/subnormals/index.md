+++
date = '2026-01-12T02:34:00+05:30'
draft = false
title = 'Subnormal numbers'
tags = ['C++', 'floating-point', 'subnormal numbers', 'performance']
description = 'Subnormal numbers and performance'
+++

Floating-point numbers represented using the usual
[IEEE 754](https://en.wikipedia.org/wiki/IEEE_754) use the following format for
a 32-bit `float`:

- 1 bit: sign
- 8 bits: exponent
- 23 bits: fraction

![float_repr](images/float_repr.png)

Exponent 0 is reserved for representing subnormal numbers and zero and exponent
255 is reserved for representing infinity and NaN.

Normalized numbers always have the leading bit as 1 (`1.xxxxxx * 2^exp`). This
means that the smallest normalized positive float is `1.0 * 2^-126` and we end
up missing smaller numbers closer to zero. Without subnormals, values very near
zero would abruptly underflow to zero, which may not be desirable for high
precision calculations.

Subnormal numbers help with this by always storing the exponent as 0 and using
-126 as actual exponent value, and having the leading bit as 0
(`0.xxxxxx * 2^-126`).

The largest positive subnormal float is `0.FFFFFE * 2^-126` (i.e., 23 fraction
bits set) which is very close to the smallest normalized float so we get
continuity and the smallest positive subnormal float is `0.000002 * 2^-126` (
i.e., only the least significant fraction bit set) which is also very close to
zero.

While subnormals are good and all, some processors do not handle them as well as
normalized numbers and operations involving subnormals can be significantly
slower than normalized numbers. Subnormal numbers trade precision for range -
the smaller the value, the fewer significant bits remain.

C++ provides [`std::isnormal`](https://en.cppreference.com/w/cpp/numeric/math/isnormal)
to determine if a floating-point number is normal or not. Flushing subnormals to
zero should be preferred when performance is desirable over precision. This can
be done using platform-specific methods.

---

## References

- [StackOverflow: What is a subnormal floating point number?](https://stackoverflow.com/questions/8341395/what-is-a-subnormal-floating-point-number)
- [Wikipedia: Subnormal number](https://en.wikipedia.org/wiki/Subnormal_number)
- [StackOverflow: Why does changing 0.1f to 0 slow down performance by 10x?](https://stackoverflow.com/questions/9314534/why-does-changing-0-1f-to-0-slow-down-performance-by-10x/9314926#9314926)
