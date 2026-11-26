[← C++ Field Guide](../README.md)

# Fixed-Width Integers vs Built-in Types

> Use fixed-width types like `std::int32_t` from `<cstdint>` when the exact bit width is part of a contract; use `int`, `long`, and friends for ordinary counters and arithmetic where only "big enough" matters.

---

## Built-in Integer Types

The standard only guarantees minimum widths for `char`, `short`, `int`, `long`, and `long long` — not exact ones. Two common data models disagree on how wide `long` actually is:

```cpp
long x = 10;   // 32 bits on Windows (LLP64), 64 bits on Linux/macOS (LP64)
```

Same source, same compiler vendor even, different width depending on the target. `int` is nearly always 32 bits in practice, but the standard only promises at least 16.

These types are fine for loop counters, indices, and general arithmetic, where the only requirement is "big enough to hold the values involved."

---

## Fixed-Width Integer Types

_C++11_

[Read more](https://en.cppreference.com/w/cpp/types/integer)

`<cstdint>` defines aliases with an exact, portable width regardless of platform or data model: `std::int8_t`, `std::int16_t`, `std::int32_t`, `std::int64_t`, and their `u`-prefixed unsigned counterparts.

```cpp
#include <cstdint>

std::int32_t offset   = -1024;
std::uint64_t fileSize = 0;
```

Use these whenever the width is part of a contract rather than an implementation detail — binary file formats, network protocols, hashing, bit manipulation, memory-mapped hardware registers.

The exact-width types are optional — a platform without a matching representation simply doesn't define them. If portability to such platforms matters, use `std::int_least32_t` (at least N bits, smallest such type) or `std::int_fast32_t` (at least N bits, fastest such type) instead. In practice, every mainstream platform provides the exact-width types.

### What They Alias on a Modern Compiler

On a typical 64-bit target:

| Fixed-width type | LP64 (Linux, macOS) | LLP64 (Windows) |
|---|---|---|
| `std::int8_t` / `std::uint8_t` | `signed char` / `unsigned char` | `signed char` / `unsigned char` |
| `std::int16_t` / `std::uint16_t` | `short` / `unsigned short` | `short` / `unsigned short` |
| `std::int32_t` / `std::uint32_t` | `int` / `unsigned int` | `int` / `unsigned int` |
| `std::int64_t` / `std::uint64_t` | `long` / `unsigned long` | `long long` / `unsigned long long` |

The 64-bit row is exactly where the two data models diverge: `std::int64_t` is `long` on Linux/macOS but `long long` on Windows.

> [!WARNING]
> Code should never depend on which underlying type a fixed-width alias resolves to — only that it's exactly 64 bits. Overload resolution, template specializations, or `printf` format specifiers written against `long` or `long long` specifically can pick the wrong path on the other platform.

---

## `std::size_t`

`std::size_t` (from [`<cstddef>`](https://en.cppreference.com/w/cpp/types/size_t)) is the unsigned integer type the standard library uses for sizes, counts, and indices — it's what `sizeof`, `.size()`, and container indexing operate on. Its width isn't fixed by name the way `std::uint32_t`'s is; it's only guaranteed large enough to hold the size of any object, which in practice means 64 bits on a 64-bit target and 32 bits on a 32-bit one.

```cpp
std::vector<int> v = {1, 2, 3};
std::size_t n = v.size();   // unsigned, platform's "natural" size width
```

---

## `std::int32_t` vs `::int32_t`

`<cstdint>` is only required to put these names in namespace `std`. C's `<stdint.h>` defines the same names at global scope, and most standard libraries also inject them into the global namespace as an extension when `<cstdint>` is included.

> [!WARNING]
> `::int32_t` compiles almost everywhere, but nothing guarantees it in standard C++ — code relying on the global-namespace name can fail to compile against a standard-conforming library that doesn't inject the extension.

```cpp
#include <cstdint>

std::int32_t a = 1;  // guaranteed by the C++ standard
int32_t b = 1;       // relies on a non-standard extension
```

Always write the `std::` prefix and include `<cstdint>`, not `<stdint.h>`.

---

## Functional-Style Casts

A fixed-width type is a single alias name, so it works in a functional-style cast like any other type:

```cpp
auto a = std::uint64_t(x);
```

The built-in type it aliases often doesn't have that property. Functional-cast syntax only accepts a single type-specifier token, and multi-keyword built-in types like `unsigned long long` are three:

```cpp
auto b = unsigned long long(x);  // error: expected primary-expression before 'unsigned'
```

Single-keyword types (`int`, `long`, `unsigned`) work fine in this form; anything spelled with more than one keyword needs `static_cast` or a C-style cast instead:

```cpp
auto c = static_cast<unsigned long long>(x);
auto d = (unsigned long long)x;
```

`std::uint64_t(x)` sidesteps the question entirely — it's always a single token, so it always works.

---

## Prefer

Use `std::int32_t`, `std::uint64_t`, and their siblings from `<cstdint>` whenever a type's exact bit width is part of a contract — file formats, network protocols, hashing, or bit manipulation. Use plain `int`, `long`, and friends for ordinary counters and arithmetic where only "big enough" matters, and always write the `std::` prefix rather than relying on the non-standard global-namespace names.
