[← C++ Field Guide](../README.md)

# Overload Set

> Combine multiple lambdas into one callable by inheriting from all of them — overload resolution picks the right one, making exhaustive `std::variant` dispatch concise and compiler-checked.

---

## The Pattern

A variadic `overloaded` struct inherits from each of its template arguments and pulls all of their `operator()` definitions into scope. Normal overload resolution then selects the right one at the call site.

```cpp
template<typename... Fs>
struct overloaded : Fs... {
    using Fs::operator()...;
};
```

Class template argument deduction (CTAD, C++17) eliminates the need to specify template arguments at construction.

---

## With `std::variant` and `std::visit`

The primary use is exhaustive dispatch on a `std::variant` without writing a named visitor class:

```cpp
using Value = std::variant<int, double, std::string>;

Value v = 3.14;

std::visit(overloaded{
    [](int x)                { std::cout << "int: "    << x << "\n"; },
    [](double x)             { std::cout << "double: " << x << "\n"; },
    [](const std::string& x) { std::cout << "string: " << x << "\n"; },
}, v);
```

Each lambda handles exactly one alternative. If any alternative is not covered, the code does not compile.

---

## Generic Catch-All

A generic lambda at the end acts as a catch-all for alternatives that share the same handling:

```cpp
std::visit(overloaded{
    [](int x)  { process_int(x); },
    [](auto&)  { /* ignore all other alternatives */ },
}, v);
```

The specific overload (`int`) takes priority over the generic one when both match.

---

## Pros and Cons

**Pros**
- All dispatch logic is collocated at the `std::visit` call — no separate visitor class needed.
- Missing alternatives are a compile error; exhaustiveness is enforced by the compiler.
- Each lambda handles exactly one case; adding a new alternative means adding one lambda.

**Cons**
- `overloaded` is not in the standard library — it requires a one-time definition.
- Two lambdas with identical signatures are ambiguous and cause a compile error; they must be merged or disambiguated.

> [!WARNING]
> A generic lambda placed before a more specific one silently swallows it — ordering matters, and the compiler gives no error when a later, more specific lambda becomes unreachable.
