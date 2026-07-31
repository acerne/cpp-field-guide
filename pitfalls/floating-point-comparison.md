[← C++ Field Guide](../README.md)

# Floating Point Comparison

Floating point values accumulate rounding error. Exact equality comparison with `==` is almost always wrong.

```cpp
double a = 0.1 + 0.2;
double b = 0.3;
a == b;  // false
```

Use an epsilon comparison instead:
```cpp
std::fabs(a - b) < std::numeric_limits<double>::epsilon();
```

`epsilon()` is the smallest difference distinguishable from 1.0 — it is only appropriate for values near 1. For larger magnitudes, use a relative epsilon:
```cpp
std::fabs(a - b) <= std::numeric_limits<double>::epsilon() * std::fabs(a + b);
```

Once you introduce epsilon equality, strict ordering (`<`, `>`) must also account for it — otherwise comparisons become logically inconsistent. With `a = 0.1 + 0.2` and `b = 0.3`:

```cpp
bool eq = std::fabs(a - b) < Eps;  // true - epsilon-equal
bool gt = a > b;                   // true - a is also greater?
```

`a` literally is the larger value, so `a > b` is correct in isolation. But if epsilon equality says they are equal, `a > b` also returning `true` is a contradiction. Non-strict operators (`<=`, `>=`) have the same problem: the embedded equality test fails for values within epsilon.

Define all six operators consistently in terms of relative epsilon:

```cpp
constexpr double Eps = std::numeric_limits<double>::epsilon();

bool eq(double a, double b) { return std::fabs(a-b) <= Eps * std::fabs(a+b); }
bool lt(double a, double b) { return b - a >  Eps * std::fabs(a+b); }
bool gt(double a, double b) { return a - b >  Eps * std::fabs(a+b); }
bool le(double a, double b) { return a - b <= Eps * std::fabs(a+b); }
bool ge(double a, double b) { return b - a <= Eps * std::fabs(a+b); }
```

**Disclaimer**: There are many different approaches to address floating-point comparison. This presents just the most basic ones.

---

Example: Exact equality vs. epsilon comparison
```cpp
#include <cmath>
#include <cstdio>
#include <limits>

constexpr double Eps = std::numeric_limits<double>::epsilon();

int main()
{
    double a = 0.1 + 0.2;
    double b = 0.3;

    bool exact = (a == b);
    bool abs_eq = std::fabs(a - b) < Eps;

    std::printf("a = %.20f\n", a);
    std::printf("b = %.20f\n", b);
    std::printf("\n");
    std::printf("a == b (exact): %s\n", exact ? "true" : "false");
    std::printf("abs epsilon:    %s\n", abs_eq ? "true" : "false");

    return 0;
}
```
Output:
```
a = 0.30000000000000004441
b = 0.29999999999999998890

a == b (exact): false
abs epsilon:    true
```

[Try it!](https://godbolt.org/z/876jenKKr)

---

Example: Absolute epsilon vs. relative epsilon
```cpp
#include <cmath>
#include <cstdio>
#include <limits>

constexpr double Eps = std::numeric_limits<double>::epsilon();

int main()
{
    double a = 100000.1 + 200000.2;
    double b = 300000.3;

    bool abs_eq = std::fabs(a - b) < Eps;
    bool rel_eq = std::fabs(a - b) <= Eps * std::fabs(a + b);

    std::printf("a = %.20f\n", a);
    std::printf("b = %.20f\n", b);
    std::printf("\n");
    std::printf("abs epsilon: %s\n", abs_eq ? "true" : "false");
    std::printf("rel epsilon: %s\n", rel_eq ? "true" : "false");

    return 0;
}
```
Output:
```
a = 300000.30000000004656612873
b = 300000.29999999998835846782

abs epsilon: false
rel epsilon: true
```

[Try it!](https://godbolt.org/z/1nM988oYz)

---

Example: Inconsistent ordering when mixing strict and epsilon comparisons
```cpp
#include <cmath>
#include <cstdio>
#include <limits>

constexpr double Eps = std::numeric_limits<double>::epsilon();

bool eps_eq(double a, double b) { return std::fabs(a-b) <= Eps * std::fabs(a+b); }
bool eps_gt(double a, double b) { return a - b >  Eps * std::fabs(a+b); }
bool eps_le(double a, double b) { return a - b <= Eps * std::fabs(a+b); }

int main()
{
    double a = 0.1 + 0.2;
    double b = 0.3;

    std::printf("Strict:\n");
    std::printf("  a == b : %s\n", (a == b) ? "true" : "false");
    std::printf("  a >  b : %s\n", (a >  b) ? "true" : "false");  // both false and greater?
    std::printf("  a <= b : %s\n", (a <= b) ? "true" : "false");  // should be true, is false
    std::printf("\n");
    std::printf("Epsilon-aware:\n");
    std::printf("  eq(a,b): %s\n", eps_eq(a, b) ? "true" : "false");
    std::printf("  gt(a,b): %s\n", eps_gt(a, b) ? "true" : "false");
    std::printf("  le(a,b): %s\n", eps_le(a, b) ? "true" : "false");

    return 0;
}
```
Output:
```
Strict:
  a == b : false
  a >  b : true
  a <= b : false

Epsilon-aware:
  eq(a,b): true
  gt(a,b): false
  le(a,b): true
```

[Try it!](https://godbolt.org/z/YTa1vrafx)