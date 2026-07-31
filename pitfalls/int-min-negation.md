[← C++ Field Guide](../README.md)

# `INT_MIN` Negation

`INT_MIN` has no positive counterpart in two's complement — its negation overflows, which is undefined behavior.

```cpp
int x = std::numeric_limits<int>::min();  // typically -2147483648
-x;           // UB — 2147483648 does not fit in int
std::abs(x);  // UB — same issue, even through the standard library
```

Comes up in generic absolute-value functions and hash computations. Guard explicitly:
```cpp
if (x == std::numeric_limits<int>::min()) { /* handle separately */ }
```

---

Example: Negation and `std::abs` wrapping silently
```cpp
#include <cstdint>
#include <cstdlib>
#include <iostream>
#include <limits>

int main()
{
    std::int32_t x = std::numeric_limits<std::int32_t>::min();

    std::cout << "32-bit int:\n";
    std::cout << "  x         = " << x          << "\n";
    std::cout << "  -x        = " << -x         << "\n";  // UB: wraps back to INT_MIN
    std::cout << "  std::abs  = " << std::abs(x) << "\n"; // UB: same

    std::int64_t safe = x;  // widen first, then negate
    std::cout << "64-bit int:\n";
    std::cout << "  x         = " << safe           << "\n";
    std::cout << "  -x        = " << -safe          << "\n";
    std::cout << "  std::abs  = " << std::abs(safe) << "\n";

    return 0;
}
```
Output (no sanitizers):
```
32-bit int:
  x         = -2147483648
  -x        = -2147483648
  std::abs  = -2147483648
64-bit int:
  x         = -2147483648
  -x        = 2147483648
  std::abs  = 2147483648
```
[Try it!](https://godbolt.org/z/MrPa7oKzT)