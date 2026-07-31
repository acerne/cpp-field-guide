[← C++ Field Guide](../README.md)

# Floating-Point Comparison

> Avoid `==`; use named comparison functions with epsilon tolerance, and keep ordering operators consistent with the chosen equality semantics.

Floating-point numbers are stored in binary, and most decimal fractions have no exact binary representation. Each value is rounded to the nearest representable bit pattern, so arithmetic results rarely match what algebra predicts:

```cpp
double x = 0.1 + 0.2;
x == 0.3;   // false — x is 0.30000000000000004
```

The rounding applied to `0.1 + 0.2` produces a different bit pattern than the rounding applied to the literal `0.3`. Neither value is wrong — both are the closest representable double to their respective decimal inputs — but they are not equal.

---

## Exact Comparison

`==` is correct in a narrow set of cases: comparing against a literal that is exactly representable, comparing two variables produced by identical computation paths, or checking a float that holds an integer value.

```cpp
float x = 0.0f;
if (x == 0.0f) { /* fine — 0.0 is exact */ }

double a = std::sqrt(2.0);
double b = std::sqrt(2.0);
if (a == b) { /* fine — identical computation */ }
```

Arithmetic introduces rounding at each step, so two expressions that are mathematically equal rarely produce bit-identical results. Also note that `NaN != NaN` — `==` silently returns false for any NaN operand; use `std::isnan` to test for NaN explicitly.

---

## Absolute Epsilon

Compare the difference against a fixed threshold.

```cpp
bool equalsApprox(float a, float b, float eps = 1e-6f) {
    return std::abs(a - b) < eps;
}
```

Works when values are known to be near zero or at a fixed scale. Fails for large numbers: at magnitude 1e8, adjacent floats are ~8 units apart, so an epsilon of 1e-6 is far too tight.

---

## Relative Epsilon

Scale the threshold by the magnitude of the operands.

```cpp
bool equalsApprox(double a, double b, double rel = 1e-9) {
    return std::abs(a - b) <= rel * std::max(std::abs(a), std::abs(b));
}
```

Works for large values. Breaks down near zero — the threshold shrinks with the operands, making the comparison meaninglessly strict.

---

## Combined Tolerance

Use the larger of an absolute and a relative threshold to handle both extremes.

```cpp
bool equalsApprox(double a, double b,
                  double rel = 1e-9, double absEps = 1e-12) {
    return std::abs(a - b) <=
           std::max(absEps, rel * std::max(std::abs(a), std::abs(b)));
}
```

This is the robust general-purpose form. Pick `rel` to match expected accumulated rounding error and `absEps` to match the scale at which values are considered effectively zero.

`std::numeric_limits<double>::epsilon()` (~2.2e-16) is one ULP at 1.0 — a useful floor for `rel` when only a few rounding steps are involved. [cppreference: std::numeric_limits::epsilon](https://en.cppreference.com/w/cpp/types/numeric_limits/epsilon)

---

## Choosing a Tolerance

**Exact comparison** is correct when values come from the same sub-expression, are known-exact constants, or hold integer values. For everything else, use approximate equality.

**Absolute epsilon** is the simplest and often sufficient. Use it when values are bounded near a fixed scale — normalized weights in [0, 1], angles in radians, or quantities near zero. Set the tolerance to domain precision when it applies (e.g., `1e-3` for positions in metres), or to a small multiple of `std::numeric_limits<T>::epsilon()` (2–10×) for a handful of operations.

**Relative epsilon** suits quantities with wide dynamic range. Error accumulates with each operation — a few multiplications → `1e-5` for `float`, `1e-9` for `double`. Longer computations need more headroom. Never tighten below a few multiples of `std::numeric_limits<T>::epsilon()` — that is the resolution limit of the type.

**Combined tolerance** handles both extremes when values may be near zero or very large in the same code path. Set `absEps` as the near-zero threshold and `rel` by operation count independently.

---

## Named Functions for Class Types

For classes with floating-point members, `operator==` is a trap — it silently performs exact comparison. Delete it and provide named functions instead.

```cpp
struct Vec2 {
    float x, y;

    bool operator==(const Vec2&) const = delete;

    bool equals(const Vec2& other) const {
        return x == other.x && y == other.y;
    }

    bool equalsApprox(const Vec2& other, float eps = 1e-6f) const {
        return std::abs(x - other.x) < eps && std::abs(y - other.y) < eps;
    }
};
```

Named functions make intent visible at every call site. Deleting `operator==` turns accidental exact comparisons into compile errors.

---

## Consistent Ordering

When epsilon equality is in use, the ordering operators must match it. With exact `<` and epsilon `==`, both `equalsApprox(a, b)` and `a < b` can be true simultaneously when `a` is slightly below `b` but within epsilon — a value satisfies two mutually exclusive conditions.

Define ordering with the same tolerance so the boundary region is owned entirely by "equal":

```cpp
bool lessThanApprox(double a, double b, double eps = 1e-9) {
    return a < b - eps;
}

bool lessOrEqualApprox(double a, double b, double eps = 1e-9) {
    return a <= b + eps;
}
```

`lessThanApprox(a, b)` and `equalsApprox(a, b)` are then mutually exclusive: a value cannot be both within tolerance and strictly less.

---

## Prefer

Use `==` only for exact representations: zero, integer-valued floats, or outputs of identical sub-expressions. For classes with floating-point members, delete `operator==` and provide named `equals` and `equalsApprox` functions. When ordering is needed alongside epsilon equality, define epsilon-aware `lessThanApprox` and `lessOrEqualApprox` — otherwise a value near the boundary can satisfy two ordering conditions at once.
