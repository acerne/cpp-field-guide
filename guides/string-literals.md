[← C++ Field Guide](../README.md)

# String and Character Literals

> A prefix before the opening quote selects a character or string literal's encoding, and a suffix after it can construct `std::string` or `std::string_view` directly.

---

## Character and String Encoding Prefixes

A prefix before the opening quote selects the character or string's encoding and element type. No prefix means `char`/`const char*`.

| Prefix | Character type | String type |
|---|---|---|
| (none) | `char` | `const char*` |
| `L` | `wchar_t` | `const wchar_t*` |
| `u8` | `char8_t` (C++20; `char` before) | `const char8_t*` (C++20; `const char*` before) |
| `u` | `char16_t` | `const char16_t*` |
| `U` | `char32_t` | `const char32_t*` |

```cpp
auto c0 = 'A';    // char
auto c1 = L'A';   // wchar_t
auto c2 = u'A';   // char16_t
auto c3 = U'A';   // char32_t
```

[Read more](https://en.cppreference.com/w/cpp/language/character_literal)

```cpp
auto s0 = "hi";    // const char*
auto s1 = L"hi";   // const wchar_t*
auto s2 = u8"hi";  // const char8_t* (C++20)
```

[Read more](https://en.cppreference.com/w/cpp/language/string_literal)

`R"(...)"` marks a raw string literal — backslashes and quotes inside it aren't escape sequences, useful for regexes or file paths. It combines with the encoding prefixes above: `LR"(...)"`, `u8R"(...)"`, `uR"(...)"`, `UR"(...)"`.

```cpp
auto path = R"(C:\Users\name)";  // no need to escape the backslashes
```

---

## String Suffixes

Standard-library string suffixes require an explicit `using namespace std::literals;` (or a narrower sub-namespace) — unlike the prefixes above, they aren't available by default.

### `s` — `std::string`

_C++14_

```cpp
using namespace std::literals;

auto a = "hi";    // const char*
auto b = "hi"s;   // std::string
```

[Read more](https://en.cppreference.com/w/cpp/string/basic_string/operator%22%22s)

Without the suffix, a string literal is `const char*`; the suffix matters mainly where `auto` deduction must produce a `std::string`, or to avoid `std::string("hi")` verbosity.

The same `s` suffix means `std::chrono::seconds` on a numeric literal — see [Suffix Collisions](chrono-literals.md#suffix-collisions) in the `std::chrono` literals guide.

### `sv` — `std::string_view`

_C++17_

```cpp
using namespace std::literals;

auto a = "hi"sv;  // std::string_view, no allocation
```

[Read more](https://en.cppreference.com/w/cpp/string/basic_string_view/operator%22%22sv)

Prefer `sv` over `s` when a view is enough — it doesn't allocate or copy the character data.
