# C++23

> C++23 is a moderate release that completes unfinished C++20 business — delivering `std::generator`, `import std`, and `std::expected` — while adding practical ergonomics throughout the library.

## Index
- [Language Features](#language-features)
  - [Deducing this](#deducing-this)
  - [if consteval](#if-consteval)
  - [Multidimensional operator[]](#multidimensional-operator)
  - [auto(x) Decay-copy](#autox-decay-copy)
  - [assume Attribute](#assume-attribute)
  - [static operator()](#static-operator)
  - [Size Literal Suffixes](#size-literal-suffixes)
  - [#elifdef / #elifndef](#elifdef--elifndef)
- [Standard Library Additions](#standard-library-additions)
  - [std::expected](#stdexpected)
  - [std::generator](#stdgenerator)
  - [std::print / std::println](#stdprint--stdprintln)
  - [std::flat_map / std::flat_set](#stdflat_map--stdflat_set)
  - [std::mdspan](#stdmdspan)
  - [std::stacktrace](#stdstacktrace)
  - [import std](#import-std)
  - [std::move_only_function](#stdmove_only_function)
  - [Monadic std::optional](#monadic-stdoptional)
  - [std::string::contains](#stdstringcontains)
  - [std::to_underlying](#stdto_underlying)
  - [std::unreachable](#stdunreachable)
  - [std::byteswap](#stdbyteswap)
  - [New Range Views](#new-range-views)
- [Deprecations & Removals](#deprecations--removals)

---

## Language Features

C++23's language changes are smaller in scope than C++20 but highly practical — deducing `this` alone unlocks several long-standing CRTP and builder pattern improvements.

### Deducing `this`

[Read more](https://en.cppreference.com/w/cpp/language/member_functions#Explicit_object_parameter)

An explicit object parameter lets a member function deduce the type and value category of `this`. The first parameter is marked with the `this` keyword.

Eliminates CRTP boilerplate for mixin bases:

_Before:_
```cpp
template <typename Derived>
struct Base {
    void interface() { static_cast<Derived*>(this)->impl(); }
};
```

_After:_
```cpp
struct Base {
    void interface(this auto& self) { self.impl(); }
};
```

Enables recursive lambdas without `std::function`:
```cpp
auto fib = [](this auto self, int n) -> int {
    return n < 2 ? n : self(n - 1) + self(n - 2);
};
fib(10);  // 55
```

Unifies const/non-const overloads into one function:

_Before:_
```cpp
      T& get()       { return data_; }
const T& get() const { return data_; }
```

_After:_
```cpp
auto& get(this auto& self) { return self.data_; }
```

### `if consteval`

[Read more](https://en.cppreference.com/w/cpp/language/if#Consteval_if)

Branches based on whether the function is currently being evaluated in a constant expression context. Fills the gap where `constexpr` functions need different behavior at compile vs runtime.

```cpp
constexpr int compute(int x) {
    if consteval {
        return compile_time_impl(x);  // only called at compile time
    } else {
        return runtime_impl(x);       // only called at runtime
    }
}
```

Complements `consteval` — where `consteval` forces compile-time evaluation, `if consteval` adapts behavior inside an otherwise `constexpr` function.

### Multidimensional `operator[]`

[Read more](https://en.cppreference.com/w/cpp/language/operator_member_access#Subscript_operator)

`operator[]` can now accept multiple arguments, enabling natural matrix/tensor indexing without proxy objects:

```cpp
struct Matrix {
    double operator[](std::size_t row, std::size_t col) const {
        return data_[row * cols_ + col];
    }
    // ...
};

Matrix m{4, 4};
m[2, 3];  // previously required m(2,3) or m[2][3] with a proxy
```

`std::mdspan` uses this operator.

### `auto(x)` Decay-copy

[Read more](https://en.cppreference.com/w/cpp/language/explicit_cast)

`auto(x)` and `auto{x}` create a decayed copy of `x` in a single expression. Previously required a named variable or `std::decay_t` cast.

```cpp
// erase-remove a copy of the front element without a dangling reference
v.erase(std::remove(v.begin(), v.end(), auto(v.front())), v.end());
```

Without `auto(v.front())`, `v.front()` is a reference that `erase` may invalidate before `remove` finishes.

### `[[assume]]` Attribute

[Read more](https://en.cppreference.com/w/cpp/language/attributes/assume)

Tells the optimizer that an expression is always true at that point. Undefined behavior if the assumption is violated at runtime.

```cpp
void process(int* p, int n) {
    [[assume(n > 0)]];
    [[assume(p != nullptr)]];
    // optimizer may eliminate null/bounds checks in the loop
    for (int i = 0; i < n; ++i) use(p[i]);
}
```

### `static operator()`

[Read more](https://en.cppreference.com/w/cpp/language/operators#Static_call_operator)

`operator()` and `operator[]` can be declared `static` on types that don't need access to `this`. Enables better compiler optimization for stateless function objects and allows `static` lambdas.

```cpp
struct Square {
    static int operator()(int x) { return x * x; }
};

// static lambda
auto square = [](int x) static { return x * x; };
```

### Size Literal Suffixes

[Read more](https://en.cppreference.com/w/cpp/language/integer_literal)

`uz` / `UZ` produces a `std::size_t` literal, and `z` / `Z` produces the corresponding signed type. Eliminates casts when comparing loop indices against container sizes.

```cpp
for (auto i = 0uz; i < v.size(); ++i) { /* ... */ }  // no signed/unsigned mismatch
```

### `#elifdef` / `#elifndef`

[Read more](https://en.cppreference.com/w/cpp/preprocessor/conditional)

Shorthand preprocessor conditionals — equivalent to `#elif defined(X)` and `#elif !defined(X)`:

_Before:_
```cpp
#ifdef A
    // ...
#elif defined(B)
    // ...
#endif
```

_After:_
```cpp
#ifdef A
    // ...
#elifdef B
    // ...
#endif
```

---

## Standard Library Additions

C++23's library is the most significant part of the release: `std::expected`, `std::generator`, and `std::print` are immediately practical; the range view additions build meaningfully on C++20's foundation.

### `std::expected`

[Read more](https://en.cppreference.com/w/cpp/utility/expected)

Holds either an expected value `T` or an unexpected error `E`. A typed, value-based alternative to exceptions for error handling.

```cpp
std::expected<int, std::string> parse(std::string_view s) {
    if (s.empty()) return std::unexpected{"empty input"};
    return std::stoi(std::string{s});
}

auto result = parse("42");
if (result) use(*result);
else        log(result.error());
```

Monadic operations chain error-propagating calls without nested `if` checks:
```cpp
parse("42")
    .and_then([](int n)  -> std::expected<int, std::string> { return n * 2; })
    .or_else ([](auto& e) -> std::expected<int, std::string> { return 0; });
```

### `std::generator`

[Read more](https://en.cppreference.com/w/cpp/coroutine/generator)

The coroutine generator type that was missing from C++20. Produces a lazily evaluated range via `co_yield`.

```cpp
std::generator<int> iota(int start = 0) {
    while (true) co_yield start++;
}

for (int x : iota() | std::views::take(5)) {
    std::cout << x;  // 0 1 2 3 4
}
```

### `std::print` / `std::println`

[Read more](https://en.cppreference.com/w/cpp/io/print)

Direct formatted output — `std::format` without the intermediate string. `std::println` appends a newline.

```cpp
std::println("Hello, {}!", name);              // to stdout
std::println(stderr, "Error: {}", msg);        // to a specific stream
std::print("{:.2f}", 3.14159);                 // "3.14"
```

### `std::flat_map` / `std::flat_set`

[Read more](https://en.cppreference.com/w/cpp/container/flat_map)

Sorted associative containers backed by a contiguous array instead of a tree. Better cache performance for lookup-heavy workloads; slower insertion/deletion than `std::map`.

```cpp
std::flat_map<std::string, int> scores;
scores["alice"] = 100;
scores["bob"]   = 90;
// internally stored as two parallel sorted vectors: keys and values
```

Choose `std::flat_map` over `std::map` when the map is built once and read many times.

### `std::mdspan`

[Read more](https://en.cppreference.com/w/cpp/container/mdspan)

Non-owning multidimensional view over contiguous data. Works with the new multidimensional `operator[]`.

```cpp
std::vector<int> data(4 * 4);
std::mdspan<int, std::extents<int, 4, 4>> m{data.data()};

m[1, 2] = 42;
m[1, 2];  // 42
```

Layout policies (`std::layout_left`, `std::layout_right`, custom) control memory traversal order.

### `std::stacktrace`

[Read more](https://en.cppreference.com/w/cpp/utility/basic_stacktrace)

Captures and formats a stack trace at the point of call:

```cpp
void on_error() {
    auto trace = std::stacktrace::current();
    std::println("{}", std::to_string(trace));
}
```

### `import std`

[Read more](https://en.cppreference.com/w/cpp/language/modules)

Standardizes importing the entire standard library as a module. Replaces all `#include <...>` headers with a single line, significantly reducing compilation overhead.

```cpp
import std;

int main() {
    std::println("Hello");
    std::vector<int> v = {1, 2, 3};
}
```

Compiler and build system support varies — check toolchain documentation.

### `std::move_only_function`

[Read more](https://en.cppreference.com/w/cpp/utility/functional/move_only_function)

Like `std::function` but accepts move-only callables (e.g., lambdas capturing `std::unique_ptr`). Not copyable itself.

```cpp
auto p = std::make_unique<Foo>();
std::move_only_function<void()> f = [p = std::move(p)] { p->run(); };
// std::function would reject this — unique_ptr is not copyable
```

### Monadic `std::optional`

[Read more](https://en.cppreference.com/w/cpp/utility/optional/and_then)

`std::optional` gains three monadic operations for chaining optional-returning calls without nested `if` checks.

```cpp
std::optional<User> find_user(int id);
std::optional<Address> get_address(const User&);
std::optional<std::string> format_city(const Address&);

auto city = find_user(id)
    .and_then(get_address)    // User  -> optional<Address>
    .and_then(format_city)    // Address -> optional<string>
    .or_else([]{ return std::optional<std::string>{"unknown"}; });
```

- `and_then(f)` — calls `f` if value is present, returns its `optional` result
- `transform(f)` — calls `f` if present, wraps result in `optional`
- `or_else(f)` — calls `f` if empty, returns its `optional` result

### `std::string::contains`

[Read more](https://en.cppreference.com/w/cpp/string/basic_string/contains)

Substring check — replaces `find() != npos`:

_Before:_
```cpp
s.find("needle") != std::string::npos;
```

_After:_
```cpp
s.contains("needle");
```

Also available on `std::string_view`.

### `std::to_underlying`

[Read more](https://en.cppreference.com/w/cpp/utility/to_underlying)

Converts a scoped enum to its underlying integer type. Replaces the `static_cast<std::underlying_type_t<E>>(e)` mouthful.

```cpp
enum class Color : uint8_t { Red = 1, Green = 2, Blue = 3 };

std::to_underlying(Color::Green);  // 2 (uint8_t)
```

### `std::unreachable`

[Read more](https://en.cppreference.com/w/cpp/utility/unreachable)

Marks a code path as unreachable. Undefined behavior if reached at runtime; lets the optimizer eliminate dead branches.

```cpp
switch (direction) {
case Direction::North: /* ... */ break;
case Direction::South: /* ... */ break;
case Direction::East:  /* ... */ break;
case Direction::West:  /* ... */ break;
default: std::unreachable();
}
```

Replaces `__builtin_unreachable()` (GCC/Clang) and `__assume(false)` (MSVC).

### `std::byteswap`

[Read more](https://en.cppreference.com/w/cpp/numeric/byteswap)

Reverses the bytes of an integer — endianness conversion:

```cpp
std::uint32_t be = std::byteswap(le_value);  // little → big endian
```

`constexpr` and works on any integral type.

### New Range Views

[Read more](https://en.cppreference.com/w/cpp/ranges)

C++23 adds several composable views to `std::views`:

```cpp
// enumerate — index + value pairs
for (auto [i, v] : std::views::enumerate(vec)) { /* i is the index */ }

// zip — iterate multiple ranges together
for (auto [a, b] : std::views::zip(v1, v2)) { /* ... */ }

// chunk — split into fixed-size subranges
for (auto chunk : v | std::views::chunk(3)) { /* chunk has up to 3 elements */ }

// slide — rolling window
for (auto window : v | std::views::slide(3)) { /* overlapping windows of 3 */ }

// adjacent — tuple of N consecutive elements
for (auto [a, b, c] : v | std::views::adjacent<3>) { /* ... */ }

// repeat — infinite or finite repetition
std::views::repeat(42, 5);  // {42, 42, 42, 42, 42}
```

`std::ranges::to` collects any view into a concrete container — the terminal operation missing from C++20 pipelines:
```cpp
auto evens = std::views::iota(0, 10)
           | std::views::filter([](int x) { return x % 2 == 0; })
           | std::ranges::to<std::vector>();  // std::vector<int>{0, 2, 4, 6, 8}
```

---

## Deprecations & Removals

C++23 removes several items that had been deprecated in C++17:

- `std::wstring_convert` / `std::wbuffer_convert` — removed (deprecated in C++17).
- `std::codecvt` UTF specializations — removed (deprecated in C++17).
- Mixed wide/narrow string literal concatenation — now ill-formed.
