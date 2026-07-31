[← C++ Field Guide](../README.md)

# Integer Division Truncation

Integer division truncates toward zero — not toward negative infinity. This differs from mathematical floor division for negative operands.

```cpp
7 / 2;    //  3  (not 3.5)
-7 / 2;   // -3  (not -4 — truncates toward zero)
7 % 2;    //  1
-7 % 2;   // -1  (sign follows the dividend, not the divisor)
7 % -2;   //  1
```

The invariant is: `(a / b) * b + (a % b) == a`.

Before C++11, the sign of `%` with negative operands was implementation-defined. C++11 mandates truncation toward zero for both `/` and `%`.
