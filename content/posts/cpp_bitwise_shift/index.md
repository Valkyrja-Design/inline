+++
date = '2026-01-18T02:20:53+05:30'
draft = false
title = 'Bitwise shift operations with negative operands in C++'
description = 'Exploring how C++20 defines bitwise shifts with negative operands'
tags = ['C++', 'Bitwise Operations']
+++

For bitwise operation $lhs \ll rhs$ or $lhs \gg rhs$ in C++, the type of the
result is the type of $lhs$ (after promotions/conversions). The behavior is
undefined if $rhs$ is negative or greater than or equal to the number of bits in
$lhs$. The reason for the latter being that
[different CPUs handle shifts greater than bit-width differently](https://stackoverflow.com/questions/51145636/why-does-shifting-a-variable-by-more-than-its-width-in-bits-zeroes-out).

Since C++20 wherein [signed integers must use two's complement](https://stackoverflow.com/questions/57363324/ramifications-of-c20-requiring-twos-complement), the behavior for other cases is
defined as follows

- The value of $a \ll b$ is the _unique_ value congruent to $a * 2^{b}$
  modulo $2^{n}$, where $n$ is the number of bits in the type of $a$. That is,
  bitwise left shift is performed and the bits that get shifted out are
  discarded.
- The value of $a \gg b$ is $a / 2^{b}$, rounded towards negative infinity, i.e.,
  arithmetic right shift as opposed to logical right shift.

The results for positive $a$ are not very surprising, but things get weird for
negative $a$. For example,

[See demo on Godbolt](https://godbolt.org/z/1sjPh6dh9)

```cpp
#include <iostream>

int main() {
    // bit pattern: 10000000'00000000'00000000'00000001
    int32_t a = -2147483647;
    // bit pattern: 00000000'00000000'00000000'00000010
    int32_t b = a << 1; // Only well-defined from C++20 onwards

    std::cout << b << '\n'; // prints 2!

    std::cout << (-5 >> 1) << '\n'; // -3, rounded towards -infinity
}
```

Before C++20, the behavior of `a << b` where `a` is negative was undefined.

[JF Bastien](https://jfbastien.com/), the author of the paper that proposed two's complement for
C++20 also gave an amazing talk on it: [_Signed Integers are Two's Complement_](https://youtu.be/JhUxIVf1qok?si=F2Z4V18_IHqD5caz)

---

## References

- [cppreference: Built-in bitwise shift operators](https://en.cppreference.com/w/cpp/language/operator_arithmetic.html)
- [StackOverflow: Why does shifting a variable by more than its width in bits zeroes out?](https://stackoverflow.com/questions/51145636/why-does-shifting-a-variable-by-more-than-its-width-in-bits-zeroes-out)
- [StackOverflow: Ramifications of C++20 requiring two's complement](https://stackoverflow.com/questions/57363324/ramifications-of-c20-requiring-twos-complement)
- [YouTube: Signed Integers are Two's Complement](https://youtu.be/JhUxIVf1qok?si=F2Z4V18_IHqD5caz)
