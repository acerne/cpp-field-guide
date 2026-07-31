[← C++ Field Guide](../README.md)

# Numeric Literals

> A prefix selects an integer literal's base and a suffix fixes its type — sign, width, or floating-point precision — avoiding an implicit conversion after the fact.

---

## Integer Literals

An integer literal's base is set by a prefix: `0x`/`0X` for hexadecimal, a leading `0` for octal, `0b`/`0B` for binary.

```cpp
auto hex = 0x1A;    // 26
auto oct = 032;     // 26
auto bin = 0b11010; // 26
```

_C++14_ also allows a single quote as a digit separator, purely for readability — it carries no meaning:

```cpp
auto million = 1'000'000;
```

[Read more](https://en.cppreference.com/w/cpp/language/integer_literal)

A suffix fixes the literal's type: `u`/`U` for unsigned, `l`/`L` for `long`, `ll`/`LL` for `long long`. They combine in either order (`ul`, `lu`, `ull`, `llu`, ...), but the two letters of `ll`/`LL` must match case — `lL` doesn't compile.

```cpp
auto a = 1u;    // unsigned int
auto b = 1l;    // long
auto c = 1ll;   // long long
auto d = 1ull;  // unsigned long long
```

Without a suffix, an integer literal's type is the first of `int`, `long`, `long long` (or their unsigned equivalents, for hex/octal/binary literals or when `u` is present) that can represent its value — a literal larger than `INT_MAX` silently becomes `long` or wider.

> [!WARNING]
> There is no suffix for [fixed-width types](fixed-width-integers.md) — `5l` has the exact same width ambiguity as the bare `long` type does: 32-bit on Windows, 64-bit on Linux/macOS. When a literal's width must be exact, use a functional-style cast instead of a suffix:
>
> ```cpp
> auto a = std::int32_t(5);  // always exactly 32 bits
> ```

### `uz` / `z` — Sized to `std::size_t`

_C++23_

`uz`/`UZ` marks a literal as [std::size_t](fixed-width-integers.md); `z`/`Z` marks it the same width, signed (`std::ptrdiff_t`). Useful for matching a container's `size_type` without a cast:

```cpp
for (auto i = 0uz; i < container.size(); ++i) { /* ... */ }  // no signed/unsigned mismatch
```

---

## Floating-Point Literals

`f`/`F` marks a literal `float`; `l`/`L` marks it `long double`. Without a suffix, a floating-point literal is `double`.

```cpp
auto a = 1.0f;  // float
auto b = 1.0;   // double
auto c = 1.0l;  // long double
```

[Read more](https://en.cppreference.com/w/cpp/language/floating_literal)

Mixing `float` and `double` in an expression triggers implicit widening; writing `1.0f` where a `float` is the target avoids constructing a `double` intermediate just to narrow it back down.

_C++17_ also allows hexadecimal floating-point literals, useful when a value must match its binary representation exactly:

```cpp
double a = 0x1.8p3;  // hex 1.8 = 1 + 8/16 = decimal 1.5; p3 means *2^3; 1.5 * 8 = 12.0
```
