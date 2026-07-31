[← C++ Field Guide](../README.md)

# Integer Comparison

> Use `std::cmp_equal` and its siblings when comparing integers of different signedness; casting by hand must be re-derived correctly for every pair of types.

Comparing a signed and an unsigned integer triggers the usual arithmetic conversions: the signed operand is converted to unsigned before the comparison runs. A negative value wraps around to a huge positive one, so the comparison silently gives the wrong answer — see [Signed / Unsigned Comparison](../pitfalls/signed-unsigned-comparison.md).

```cpp
int a = -1;
unsigned b = 1u;
a < b;  // false — a converts to 4294967295
```

---

## Manual Casting

Casting one operand to match the other avoids the conversion trap, but only if the cast is written correctly for the specific pair of types involved:

```cpp
bool lessThan(int a, unsigned b) {
    if (a < 0) return true;               // negative always less
    return static_cast<unsigned>(a) < b;  // safe now — a >= 0
}
```

This has to be rewritten, and re-verified, for every combination of widths and signedness (`int`/`unsigned`, `long`/`size_t`, `int`/`long long`, ...).

> [!WARNING]
> Widening to a bigger signed type is not a general fix: the cast is only safe if that type can represent every value of the unsigned operand, which fails once the unsigned operand is already the widest integer type (e.g. `uint64_t`).

---

## `std::cmp_equal` and Related Functions

_C++20_

[Read more](https://en.cppreference.com/w/cpp/utility/intcmp)

`<utility>` provides `std::cmp_equal`, `std::cmp_not_equal`, `std::cmp_less`, `std::cmp_greater`, `std::cmp_less_equal`, and `std::cmp_greater_equal`. Each compares its arguments by mathematical value regardless of signedness or width, with no implicit-conversion bug:

```cpp
#include <utility>

int a = -1;
unsigned b = 1u;

a < b;                  // false — wraps to a large unsigned value
std::cmp_less(a, b);    // true — compares as -1 < 1
```

Both arguments must be integer types; a call with a non-integer type is a compile error, which rules out accidental misuse.

`std::in_range<T>(x)` checks whether `x` fits in `T` without converting it first, useful before a narrowing cast:

```cpp
long x = -1;
std::in_range<std::size_t>(x);  // false — no conversion, no UB
```

---

## Prefer

Use `std::cmp_equal` and friends whenever comparing integers that may differ in signedness. They give the mathematically correct answer for any pair of integer types, unlike a hand-rolled cast that must be re-derived — and re-verified — for each new combination of types.
