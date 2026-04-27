[← Pitfalls](README.md)

# `std::min` / `std::fmin` and NaN

`std::min` and `std::max` use `operator<`, and any comparison involving NaN returns `false`. This makes the result depend on argument order — the function is not commutative when NaN is involved.

```cpp
double a   = 5.0;
double nan = std::numeric_limits<double>::quiet_NaN();

std::isnan(std::min(a, nan));   // false — returns a (5.0)
std::isnan(std::min(nan, a));   // true  — returns nan
```

The root cause: `std::min(x, y)` returns `y` if `y < x`, otherwise `x`. Since `nan < a` is `false`, it always returns the first argument — silently propagating or hiding NaN depending on order.

`std::fmin` / `std::fmax` follow IEEE 754: if one argument is NaN, the other is returned. Order does not matter.

```cpp
std::isnan(std::fmin(a, nan));  // false — returns 5.0
std::isnan(std::fmin(nan, a));  // false — returns 5.0
```

Use `std::fmin` / `std::fmax`  when NaN is a possible input. Use `std::min` / `std::max` only when inputs are guaranteed NaN-free.

---

Example: argument-order sensitivity of `std::min` vs. `std::fmin` with NaN
```cpp
#include <algorithm>
#include <cmath>
#include <cstdio>
#include <limits>

int main()
{
    double a   = 5.0;
    double nan = std::numeric_limits<double>::quiet_NaN();

    double r1 = std::min(a, nan);
    double r2 = std::min(nan, a);
    double r3 = std::fmin(a, nan);
    double r4 = std::fmin(nan, a);

    std::printf("std::min(a, nan)  = %g\n", r1);
    std::printf("std::min(nan, a)  = %g\n", r2);
    std::printf("\n");
    std::printf("std::fmin(a, nan) = %g\n", r3);
    std::printf("std::fmin(nan, a) = %g\n", r4);

    return 0;
}
```
Output:
```
std::min(a, nan)  = 5
std::min(nan, a)  = nan

std::fmin(a, nan) = 5
std::fmin(nan, a) = 5
```
[Try it!](https://godbolt.org/z/4YnqYGTWP)
