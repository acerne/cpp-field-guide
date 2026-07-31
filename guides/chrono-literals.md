[← C++ Field Guide](../README.md)

# `std::chrono` Literals

> A suffix on a numeric literal constructs a `std::chrono` duration or calendar value directly, without spelling out the type.

---

## Duration Suffixes

_C++14_

[Read more](https://en.cppreference.com/w/cpp/chrono/duration)

`<chrono>` defines a suffix for every built-in duration unit, in `std::chrono_literals` (also reachable via `std::literals`):

| Suffix | Type |
|---|---|
| `h` | `std::chrono::hours` |
| `min` | `std::chrono::minutes` |
| `s` | `std::chrono::seconds` |
| `ms` | `std::chrono::milliseconds` |
| `us` | `std::chrono::microseconds` |
| `ns` | `std::chrono::nanoseconds` |

```cpp
using namespace std::chrono_literals;

auto timeout  = 500ms;
auto interval = 5min + 30s;
```

---

## Calendar Suffixes

_C++20_

`d` and `y` build calendar values rather than durations:

```cpp
using namespace std::chrono_literals;

auto date = 2024y / 1 / 15;  // std::chrono::year_month_day
```

`2024y` is a `std::chrono::year`, not a duration — combined with a month and day via `operator/`, it builds a `std::chrono::year_month_day`.

---

## Suffix Collisions

The same suffix can mean different things depending on what it's attached to. `s` on a numeric literal is `std::chrono::seconds` (from `std::literals::chrono_literals`); on a string literal it's [std::string](string-literals.md) (from `std::literals::string_literals`):

```cpp
using namespace std::literals;

auto text = "5"s;  // std::string, holds "5"
auto time = 5s;     // std::chrono::seconds, holds 5
```

Both live under `std::literals`, so pulling in the whole namespace is safe — overload resolution picks the right operator based on the literal's own type. Pulling in only `std::literals::chrono_literals`, though, leaves `""s` on a string literal without a matching operator.
