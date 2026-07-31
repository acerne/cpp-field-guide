[← C++ Field Guide](../README.md)

# `if constexpr` Dispatch

> Branch on a compile-time condition inside a template — the discarded branch is not instantiated, so it may reference operations that only exist for the taken branch's types.

---

## The Pattern

`if constexpr` evaluates its condition at compile time. The branch not taken is parsed but never instantiated — invalid code in the discarded branch is not an error as long as it depends on a template parameter.

```cpp
template<typename T>
void process(T value) {
    if constexpr (std::is_integral_v<T>) {
        handle_int(static_cast<long long>(value));
    } else if constexpr (std::is_floating_point_v<T>) {
        handle_float(static_cast<double>(value));
    } else {
        handle_other(value);  // handle_int/handle_float need not compile for this T
    }
}
```

---

## vs Tag Dispatch

The same selection was historically done with [tag dispatch](tag-dispatch.md) or `std::enable_if`, both of which spread the logic across multiple function overloads:

_Tag dispatch:_
```cpp
template<typename T> void process_impl(T v, std::true_type)  { handle_int(v); }
template<typename T> void process_impl(T v, std::false_type) { handle_float(v); }

template<typename T>
void process(T v) { process_impl(v, std::is_integral<T>{}); }
```

_`if constexpr`:_
```cpp
template<typename T>
void process(T v) {
    if constexpr (std::is_integral_v<T>) {
        handle_int(v);
    } else {
        handle_float(v);
    }
}
```

Prefer `if constexpr` for a closed set of branches within a single function. Prefer [tag dispatch](tag-dispatch.md) when the set of cases must be open for extension — new tags can be added by third parties without modifying existing code.

---

## Common Uses

**Type-specific serialization:**

```cpp
template<typename T>
std::string to_string(const T& v) {
    if constexpr (std::is_same_v<T, bool>) {
        return v ? "true" : "false";
    } else if constexpr (std::is_arithmetic_v<T>) {
        return std::to_string(v);
    } else {
        return v.toString();
    }
}
```

**Optional operation based on type capability (C++20 `requires`):**

```cpp
template<typename Container>
void reserve_if_possible(Container& c, std::size_t n) {
    if constexpr (requires { c.reserve(n); }) {
        c.reserve(n);
    }
}
```

---

## Pros and Cons

**Pros**
- All branches are co-located in one function body — readable and easy to maintain.
- The discarded branch is not instantiated; `handle_int` need not compile when `T` is `double`.
- No helper functions, tag types, or `enable_if` scaffolding required.

**Cons**
- The set of branches is closed — adding a new case requires modifying the function, unlike tag dispatch which is open for extension.
- A long chain of `else if constexpr` is hard to read; consider tag dispatch or virtual dispatch when the number of cases grows.
- The discarded branch must still be syntactically valid; only instantiation is skipped, not parsing.
