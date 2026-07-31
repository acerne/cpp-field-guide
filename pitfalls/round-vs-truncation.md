[← C++ Field Guide](../README.md)

# `std::round` vs Truncation

The four rounding functions differ for negative numbers. Casting to `int` truncates — it does not round.

```cpp
std::floor(-1.5);    // -2.0  — toward -∞
std::ceil(-1.5);     // -1.0  — toward +∞
std::trunc(-1.5);    // -1.0  — toward zero
std::round(-1.5);    // -2.0  — away from zero ("round half away from zero")
(int)(-1.5);         // -1    — truncation, same as trunc
```

> [!WARNING]
> `std::round` and `std::rint` are not interchangeable: `std::rint` uses the current rounding mode (default: round-to-nearest-even, i.e. banker's rounding), while `std::round` always rounds half away from zero. Swapping one for the other silently changes results at `.5` boundaries.
