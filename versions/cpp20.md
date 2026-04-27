# C++20

> C++20 is the largest revision since C++11 — it introduces concepts, coroutines, modules, and ranges as four major pillars alongside a substantially expanded standard library.

## Index
- [Language Features](#language-features)
  - [Concepts](#concepts)
  - [Coroutines](#coroutines)
  - [Modules](#modules)
  - [Three-way Comparison](#three-way-comparison)
  - [consteval](#consteval)
  - [constinit](#constinit)
  - [Designated Initializers](#designated-initializers)
  - [Template Lambdas](#template-lambdas)
  - [Abbreviated Function Templates](#abbreviated-function-templates)
  - [using enum](#using-enum)
  - [likely / unlikely Attributes](#likely--unlikely-attributes)
  - [no_unique_address Attribute](#no_unique_address-attribute)
- [Standard Library Additions](#standard-library-additions)
  - [std::ranges](#stdranges)
  - [std::span](#stdspan)
  - [std::format](#stdformat)
  - [std::jthread](#stdjthread)
  - [std::source_location](#stdsource_location)
  - [std::bit_cast](#stdbit_cast)
  - [std::erase / std::erase_if](#stderase--stderase_if)
  - [Synchronization Primitives](#synchronization-primitives)
  - [Mathematical Constants](#mathematical-constants)
  - [std::chrono Calendar](#stdchrono-calendar)
- [Deprecations & Removals](#deprecations--removals)

---

## Language Features

C++20's headline features are the four pillars: concepts (constrained templates), coroutines (async/generator support), modules (replacing headers), and ranges (composable algorithms). The release also fills many smaller gaps.

### Concepts

[Read more](https://en.cppreference.com/w/cpp/language/constraints)

Concepts are named boolean predicates on template arguments, checked at compile time. They make template constraints readable and produce better error messages.

Define a concept with `concept`:
```cpp
template <typename T>
concept Numeric = std::is_arithmetic_v<T>;
```

Use it to constrain a template:
```cpp
template <Numeric T>
T square(T x) { return x * x; }

square(3);      // ok
square("hi");   // error: "hi" does not satisfy Numeric
```

`requires` clause for inline constraints:
```cpp
template <typename T>
    requires std::is_integral_v<T>
T gcd(T a, T b) { return b == 0 ? a : gcd(b, a % b); }
```

`requires` expression to check a set of operations:
```cpp
template <typename T>
concept Printable = requires(T t) {
    { std::cout << t } -> std::same_as<std::ostream&>;
};
```

Standard library ships a set of ready-made concepts in `<concepts>`: `std::integral`, `std::floating_point`, `std::same_as`, `std::convertible_to`, `std::invocable`, and more.

### Coroutines

[Read more](https://en.cppreference.com/w/cpp/language/coroutines)

A coroutine is a function that can suspend and resume. Three new keywords control suspension: `co_await`, `co_yield`, and `co_return`.

C++20 provides the low-level machinery only — no ready-made `generator` or `task` types in the standard library (`std::generator` arrives in C++23; no standard task type exists yet). Coroutines require a hand-rolled or library-provided promise type.

Minimal generator pattern (illustrative — real implementations need more boilerplate):
```cpp
Generator<int> iota(int start) {
    while (true)
        co_yield start++;
}

for (int x : iota(0) | std::views::take(5)) {
    std::cout << x;  // 0 1 2 3 4
}
```

`co_await` suspends until an awaitable completes — the foundation for async I/O frameworks:
```cpp
Task<std::string> fetch(std::string url) {
    auto response = co_await http_get(url);
    co_return response.body();
}
```

### Modules

[Read more](https://en.cppreference.com/w/cpp/language/modules)

Modules replace `#include` with a compiled unit that exports only what is explicitly declared. They eliminate header guards, reduce compile times, and prevent macro leakage.

Define a module:
```cpp
// math.cppm
export module math;

export int square(int x) { return x * x; }
int helper() { return 42; }  // not exported — internal linkage
```

Import it:
```cpp
import math;

int main() {
    square(5);   // 25
    helper();    // error: not exported
}
```

Import the standard library as a module (C++23 standardizes `import std;`; C++20 support varies by compiler):
```cpp
import std.core;  // MSVC syntax — not yet standardized in C++20
```

Compiler and build system support for modules is still maturing as of C++20.

### Three-way Comparison

[Read more](https://en.cppreference.com/w/cpp/language/operator_comparison#Three-way_comparison)

The spaceship operator `<=>` returns an ordering category type instead of `bool`. A single `operator<=>` replaces all six comparison operators. Mark it `= default` to get all comparisons for free:

```cpp
struct Point {
    int x, y;
    auto operator<=>(const Point&) const = default;  // generates ==, !=, <, <=, >, >=
};

Point a{1, 2}, b{1, 3};
a < b;   // true
a == b;  // false
```

Return types reflect ordering strength:
- `std::strong_ordering` — total order, equal implies identical (integers)
- `std::weak_ordering` — total order, equal does not imply identical (case-insensitive strings)
- `std::partial_ordering` — not all pairs are comparable (floats — NaN)

### `consteval`

[Read more](https://en.cppreference.com/w/cpp/language/consteval)

An **immediate function** — must be evaluated at compile time. Calling it at runtime is a compile error. Stronger guarantee than `constexpr`, which may fall back to runtime.

```cpp
consteval int square(int x) { return x * x; }

constexpr int a = square(5);  // ok — compile time
int n = 5;
int b = square(n);            // error — n is not a constant expression
```

### `constinit`

[Read more](https://en.cppreference.com/w/cpp/language/constinit)

Asserts that a variable is initialized with a constant expression. Prevents the [static initialization order fiasco](https://en.cppreference.com/w/cpp/language/siof) by ensuring initialization happens at compile time. Does not make the variable immutable.

```cpp
constinit int counter = 0;  // initialized at compile time
counter++;                  // allowed — constinit does not mean const
```

Useful for global and `static` variables that must be zero-initialized or constant-initialized.

### Designated Initializers

[Read more](https://en.cppreference.com/w/cpp/language/aggregate_initialization#Designated_initializers)

Aggregate members can be initialized by name. Unmentioned members are value-initialized.

```cpp
struct Config {
    int width = 800;
    int height = 600;
    bool fullscreen = false;
};

Config c{ .width = 1920, .height = 1080 };  // fullscreen stays false
```

Designators must appear in declaration order. Skipping members is fine; out-of-order is not.

### Template Lambdas

[Read more](https://en.cppreference.com/w/cpp/language/lambda)

C++20 allows explicit template parameter lists on lambdas, giving more control than C++14 `auto` parameters:

```cpp
auto same_type = []<typename T>(T a, T b) { return a == b; };
same_type(1, 2);      // ok
same_type(1, 2.0);    // error — deduced T must match for both args
```

Also useful for constraining or forwarding:
```cpp
auto print_all = []<typename... Ts>(Ts&&... args) {
    ((std::cout << args << ' '), ...);
};
```

### Abbreviated Function Templates

[Read more](https://en.cppreference.com/w/cpp/language/function_template#Abbreviated_function_template)

`auto` in a function parameter makes it an unconstrained template. Combine with concepts for concise constrained templates:

```cpp
void print(auto x) { std::cout << x; }          // template on any type

void print(std::integral auto x) { std::cout << x; }  // constrained
```

Equivalent to:
```cpp
template <std::integral T>
void print(T x) { std::cout << x; }
```

### `using enum`

[Read more](https://en.cppreference.com/w/cpp/language/enum#Using-enum-declaration)

Imports enumerators from a scoped enum into the current scope, eliminating repetitive qualification:

```cpp
enum class Color { Red, Green, Blue };

void describe(Color c) {
    using enum Color;
    switch (c) {
    case Red:   std::cout << "red";   break;
    case Green: std::cout << "green"; break;
    case Blue:  std::cout << "blue";  break;
    }
}
```

### `[[likely]]` / `[[unlikely]]` Attributes

[Read more](https://en.cppreference.com/w/cpp/language/attributes/likely)

Hints to the optimizer which branch is expected to execute more often:

```cpp
if (x > 0) [[likely]] {
    fast_path();
} else [[unlikely]] {
    slow_path();
}
```

### `[[no_unique_address]]` Attribute

[Read more](https://en.cppreference.com/w/cpp/language/attributes/no_unique_address)

Allows the compiler to overlap a member's storage with other members or base classes if it has no state (empty type). Eliminates the size overhead of storing stateless allocators, comparators, or policies.

```cpp
struct MyContainer {
    [[no_unique_address]] MyAllocator alloc;  // takes 0 bytes if MyAllocator is empty
    int* data;
    std::size_t size;
};
```

---

## Standard Library Additions

C++20's library additions are anchored by ranges, `std::format`, and `std::span`, plus a complete set of synchronization primitives and calendar support in `std::chrono`.

### `std::ranges`

[Read more](https://en.cppreference.com/w/cpp/ranges)

The ranges library provides algorithm and view adaptors that operate on ranges (anything with `begin`/`end`) and compose with `|` pipes.

Range algorithms work directly on containers — no `begin`/`end` boilerplate:
```cpp
std::vector<int> v = {3, 1, 4, 1, 5};
std::ranges::sort(v);
```

Views are lazy, composable transformations:
```cpp
auto result = v
    | std::views::filter([](int x) { return x % 2 == 0; })
    | std::views::transform([](int x) { return x * x; });
```

Views do not own data and are cheap to copy.

### `std::span`

[Read more](https://en.cppreference.com/w/cpp/container/span)

Non-owning view over a contiguous sequence. Replaces the `(T* ptr, size_t len)` pair idiom:

```cpp
void process(std::span<int> data) {
    for (int x : data) { /* ... */ }
}

std::vector<int> v = {1, 2, 3};
int arr[] = {4, 5, 6};
process(v);    // works
process(arr);  // works
```

Fixed-size span: `std::span<int, 3>` — size encoded in the type.

### `std::format`

[Read more](https://en.cppreference.com/w/cpp/utility/format/format)

Type-safe string formatting with Python-style `{}` placeholders. Replaces `sprintf` and `std::ostringstream` in most cases.

```cpp
std::string s = std::format("Hello, {}! You are {} years old.", name, age);
std::format("{:.2f}", 3.14159);   // "3.14"
std::format("{:>10}", "right");   // "     right"
```

Custom types are supported by specializing `std::formatter<T>`.

### `std::jthread`

[Read more](https://en.cppreference.com/w/cpp/thread/jthread)

A `std::thread` that automatically joins on destruction and supports cooperative cancellation via `std::stop_token`.

```cpp
std::jthread worker([](std::stop_token st) {
    while (!st.stop_requested()) {
        do_work();
    }
});
// joins automatically when worker goes out of scope
worker.request_stop();  // signal cancellation
```

### `std::source_location`

[Read more](https://en.cppreference.com/w/cpp/utility/source_location)

Captures call-site information (file, line, function) at compile time. Replaces `__FILE__` / `__LINE__` macros in logging:

```cpp
void log(std::string_view msg,
         std::source_location loc = std::source_location::current()) {
    std::cout << loc.file_name() << ':' << loc.line() << ' ' << msg << '\n';
}

log("something happened");  // prints caller's file and line
```

### `std::bit_cast`

[Read more](https://en.cppreference.com/w/cpp/numeric/bit_cast)

Reinterprets the bit pattern of one type as another, safely and `constexpr`. Replaces the UB-prone `memcpy` / `reinterpret_cast` trick:

```cpp
float f = 1.0f;
uint32_t bits = std::bit_cast<uint32_t>(f);  // 0x3F800000
```

Both types must be the same size and trivially copyable.

### `std::erase` / `std::erase_if`

[Read more](https://en.cppreference.com/w/cpp/container/vector/erase2)

Free functions that implement the erase-remove idiom in one call:

_Before:_
```cpp
v.erase(std::remove(v.begin(), v.end(), 42), v.end());
```

_After:_
```cpp
std::erase(v, 42);                              // erase by value
std::erase_if(v, [](int x) { return x < 0; }); // erase by predicate
```

Works on all standard sequence and associative containers.

### Synchronization Primitives

C++20 adds higher-level synchronization types built on top of `std::mutex` and `std::atomic`.

`std::latch` — single-use countdown.

[Read more](https://en.cppreference.com/w/cpp/thread/latch)
```cpp
std::latch done{4};
for (int i = 0; i < 4; ++i)
    threads.emplace_back([&]{ do_work(); done.count_down(); });
done.wait();  // blocks until all 4 count down
```

`std::barrier` — reusable synchronization point across a fixed number of threads.

[Read more](https://en.cppreference.com/w/cpp/thread/barrier)
```cpp
std::barrier sync{4};
// each thread calls sync.arrive_and_wait() to meet at the barrier each iteration
```

`std::counting_semaphore` / `std::binary_semaphore` — limits concurrent access.

[Read more](https://en.cppreference.com/w/cpp/thread/counting_semaphore)
```cpp
std::counting_semaphore<4> pool{4};  // at most 4 threads concurrently
pool.acquire();
do_work();
pool.release();
```

### Mathematical Constants

[Read more](https://en.cppreference.com/w/cpp/numeric/constants)

`<numbers>` provides common mathematical constants as `constexpr` variable templates:

```cpp
std::numbers::pi;         // π
std::numbers::e;          // e
std::numbers::sqrt2;      // √2
std::numbers::log2e;      // log₂(e)
std::numbers::phi;        // golden ratio
```

All available as `double` by default; use `std::numbers::pi_v<float>` for other precisions.

### `std::chrono` Calendar

[Read more](https://en.cppreference.com/w/cpp/chrono)

C++20 extends `std::chrono` with calendar types and timezone support:

```cpp
using namespace std::chrono;

auto today = year_month_day{floor<days>(system_clock::now())};
today.year();   // e.g. 2026
today.month();  // e.g. April
today.day();    // e.g. 27

// Timezone-aware time
auto zt = zoned_time{"Europe/London", system_clock::now()};
```

---

## Deprecations & Removals

- `volatile` compound operators (`+=`, `-=`, etc. on `volatile` variables) — deprecated; use explicit loads/stores.
- Implicit capture of `*this` by `[=]` in lambdas — deprecated; use `[=, this]` explicitly.
- Array comparisons with `==` using pointer decay — deprecated.
- `std::strstream` — deprecated (had been deprecated since C++98; use `std::stringstream`).
- `std::codecvt` specializations for UTF conversions — deprecated.
