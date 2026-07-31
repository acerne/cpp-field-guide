[← C++ Field Guide](../README.md)

# Zero-Initializing a Scoped Enum

A scoped enum is not restricted to its named enumerators — any value representable in the underlying type is valid. `Number x{};` zero-initializes `x` to `0`, even when no enumerator equals `0`.

```cpp
enum class Number {
    One = 1,
    Two = 2,
    Three = 3,
};

Number x{};  // x == 0, matches none of One, Two, Three
```

A chain of equality checks against every enumerator falls through silently:
```cpp
if (x == Number::One) { /* ... */ }
else if (x == Number::Two) { /* ... */ }
else if (x == Number::Three) { /* ... */ }
// none of these match when x == 0 — no compiler warning
```

A `switch` covering all enumerators has the same gap; only a `default:` label catches the zero value, and `-Wswitch` won't flag a missing case for it since `0` isn't a declared enumerator.

If a "no value" state is intended, reserve it explicitly so it has a name:
```cpp
enum class Number {
    None = 0,   // explicit default state
    One = 1,
    Two = 2,
    Three = 3,
};
```

If every value should carry meaning, `Number` has no safe default. Potential alternative is to use `std::optional<Number>`.

Avoid using zero initialization with enum, every enum initialization should be explicit. 

---

Example: switch falls through to `default` for the zero-initialized value
```cpp
#include <cstdio>

enum class Number {
    One = 1,
    Two = 2,
    Three = 3,
};

const char* to_string(Number n)
{
    switch (n)
    {
        case Number::One:   return "One";
        case Number::Two:   return "Two";
        case Number::Three: return "Three";
    }
    return "Unknown";
}

int main()
{
    Number x{};  // zero-initialized, not One, Two, or Three
    std::printf("x = %s\n", to_string(x));
}
```
Output:
```
x = Unknown
```

[Try it!](https://godbolt.org/z/7aEbhTPnn)
