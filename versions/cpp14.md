# C++14

> C++14 is a focused refinement of C++11 — it relaxes restrictions on `constexpr`, extends lambdas, and fills in missing pieces like `std::make_unique`.

## Index
- [Language Features](#language-features)
  - [Generic Lambdas](#generic-lambdas)
  - [Lambda Init Captures](#lambda-init-captures)
  - [Return Type Deduction](#return-type-deduction)
  - [decltype(auto)](#decltypeauto)
  - [Variable Templates](#variable-templates)
  - [Relaxed constexpr](#relaxed-constexpr)
  - [Binary Literals](#binary-literals)
  - [Digit Separators](#digit-separators)
  - [deprecated Attribute](#deprecated-attribute)
- [Standard Library Additions](#standard-library-additions)
  - [std::make_unique](#stdmake_unique)
  - [std::exchange](#stdexchange)
  - [std::integer_sequence](#stdinteger_sequence)
  - [std::quoted](#stdquoted)
  - [Heterogeneous Lookup](#heterogeneous-lookup)
  - [std::chrono Literals](#stdchrono-literals)
  - [std::get by Type](#stdget-by-type)
  - [std::shared_timed_mutex](#stdshared_timed_mutex)
- [Deprecations & Removals](#deprecations--removals)

---

## Language Features

C++14 extends C++11 features rather than introducing new paradigms — generic lambdas, relaxed `constexpr`, and convenience syntax improvements.

### Generic Lambdas

[Read more](https://en.cppreference.com/w/cpp/language/lambda)

Lambda parameters can be `auto`, making the lambda a template without explicit type arguments:

```cpp
auto add = [](auto a, auto b) { return a + b; };
add(1, 2);       // int
add(1.5, 2.5);   // double
```

### Lambda Init Captures

[Read more](https://en.cppreference.com/w/cpp/language/lambda#Lambda_capture)

C++11 lambdas cannot capture by move. C++14 init captures allow arbitrary expressions in the capture list, including `std::move`:

```cpp
auto ptr = std::make_unique<Foo>();
auto f = [p = std::move(ptr)] { p->bar(); };  // ptr moved into the lambda
```

Also useful for renaming captures or computing a value once at capture time:
```cpp
auto f = [x = expensive_compute()] { use(x); };
```

### Return Type Deduction

[Read more](https://en.cppreference.com/w/cpp/language/function#Return_type_deduction)

Regular functions (not just lambdas) can use `auto` as the return type. The compiler deduces it from the `return` statement.

_Before (C++11 trailing return):_
```cpp
auto add(int a, int b) -> int { return a + b; }
```

_After:_
```cpp
auto add(int a, int b) { return a + b; }
```

All `return` statements in the function must deduce to the same type. Recursive functions require at least one non-recursive return to anchor the type.

### `decltype(auto)`

[Read more](https://en.cppreference.com/w/cpp/language/decltype)

`auto` return type always deduces a value type (stripping references). `decltype(auto)` preserves references and cv-qualifiers — useful for perfect-forwarding return types:

```cpp
int& get(std::vector<int>& v) { return v[0]; }

auto           f(std::vector<int>& v) { return get(v); }   // int  (auto strips the reference)
decltype(auto) g(std::vector<int>& v) { return get(v); }   // int& (return type of get preserved)
```

Typically used in generic wrapper functions that must forward the return type of an inner call unchanged.

### Variable Templates

[Read more](https://en.cppreference.com/w/cpp/language/variable_template)

Templates can now define variables, not just types and functions. The standard library uses this for `_v` type trait shortcuts:

```cpp
template <typename T>
constexpr bool is_integral_v = std::is_integral<T>::value;

// Usage (available in std:: from C++17, but the mechanism is C++14):
is_integral_v<int>;   // true
is_integral_v<float>; // false
```

### Relaxed `constexpr`

[Read more](https://en.cppreference.com/w/cpp/language/constexpr)

C++11 `constexpr` functions are limited to a single `return` expression. C++14 lifts most restrictions — loops, local variables, and conditionals are allowed:

_C++11 — single expression only:_
```cpp
constexpr int factorial(int n) {
    return n <= 1 ? 1 : n * factorial(n - 1);
}
```

_C++14 — imperative style allowed:_
```cpp
constexpr int factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; ++i)
        result *= i;
    return result;
}
```

### Binary Literals

[Read more](https://en.cppreference.com/w/cpp/language/integer_literal)

Integer literals can be written in binary with the `0b` or `0B` prefix:

```cpp
int mask = 0b1111'0000;   // 240 — digit separator ' for readability
int flags = 0b0000'0111;  // 7
```

### Digit Separators

[Read more](https://en.cppreference.com/w/cpp/language/integer_literal)

A single quote `'` can appear anywhere inside a numeric literal as a visual separator. It has no effect on the value.

```cpp
int million   = 1'000'000;
double pi     = 3.141'592'653;
int mask      = 0xFF'AA'BB'CC;
int binary    = 0b1010'0101;
```

### `[[deprecated]]` Attribute

[Read more](https://en.cppreference.com/w/cpp/language/attributes/deprecated)

Marks a declaration as deprecated. The compiler emits a warning when the entity is used.

```cpp
[[deprecated("use new_api() instead")]]
void old_api();

old_api();  // warning: 'old_api' is deprecated: use new_api() instead
```

Can be applied to functions, types, variables, enumerators, and more.

---

## Standard Library Additions

C++14 adds `std::make_unique` (the glaring omission from C++11), several utility functions, and quality-of-life improvements to existing types.

### `std::make_unique`

[Read more](https://en.cppreference.com/w/cpp/memory/unique_ptr/make_unique)

The missing counterpart to `std::make_shared` from C++11. Prefer it over `new` — it is exception-safe and avoids repeating the type.

_Before (C++11):_
```cpp
auto p = std::unique_ptr<Foo>(new Foo(arg1, arg2));
```

_After:_
```cpp
auto p = std::make_unique<Foo>(arg1, arg2);
```

### `std::exchange`

[Read more](https://en.cppreference.com/w/cpp/utility/exchange)

Replaces a value with a new one and returns the old value. Concise idiom for move constructors and swap-like operations:

```cpp
// move constructor pattern
Buffer(Buffer&& other) noexcept
    : data_(std::exchange(other.data_, nullptr)),
      size_(std::exchange(other.size_, 0)) {}
```

### `std::integer_sequence`

[Read more](https://en.cppreference.com/w/cpp/utility/integer_sequence)

Compile-time sequence of integers, used to unpack tuples or arrays into function arguments via parameter pack expansion:

```cpp
template <typename Tuple, std::size_t... I>
void print_tuple(const Tuple& t, std::index_sequence<I...>) {
    int arr[] = {(std::cout << std::get<I>(t) << ' ', 0)...};
    (void)arr;
}

auto t = std::make_tuple(1, 2.0, "three");
print_tuple(t, std::make_index_sequence<3>{});
```

### `std::quoted`

[Read more](https://en.cppreference.com/w/cpp/io/manip/quoted)

Stream manipulator that wraps a string in quotes and escapes embedded quotes. Useful for round-tripping strings through streams:

```cpp
std::string s = R"(hello "world")";
std::cout << std::quoted(s);        // "hello \"world\""

std::istringstream in{R"("hello \"world\"")"};
std::string out;
in >> std::quoted(out);             // out == hello "world"
```

### Heterogeneous Lookup

[Read more](https://en.cppreference.com/w/cpp/container/map/find)

`std::less<>` (void specialization) enables lookup in ordered containers without constructing the key type. Avoids temporary allocations when looking up by string literal in a `std::map<std::string, ...>`:

```cpp
std::map<std::string, int, std::less<>> m = {{"alice", 1}};
m.find("alice");  // no std::string constructed from the literal
```

The comparator must support mixed-type comparison (i.e., `operator<` between `std::string` and `const char*`).

### `std::chrono` Literals

[Read more](https://en.cppreference.com/w/cpp/chrono/operator%22%22h)

User-defined literals for duration types via `using namespace std::chrono_literals`:

```cpp
using namespace std::chrono_literals;

auto delay = 100ms;
auto timeout = 5s;
std::this_thread::sleep_for(500us);
```

Available suffixes: `h`, `min`, `s`, `ms`, `us`, `ns`.

### `std::get` by Type

[Read more](https://en.cppreference.com/w/cpp/utility/tuple/get)

In addition to index-based access, `std::get` can retrieve a tuple element by type (when the type appears exactly once):

```cpp
auto t = std::make_tuple(42, true, std::string{"hi"});
std::get<int>(t);         // 42
std::get<std::string>(t); // "hi"
```

### `std::shared_timed_mutex`

[Read more](https://en.cppreference.com/w/cpp/thread/shared_timed_mutex)

Reader-writer mutex with timed locking (`try_lock_for`, `try_lock_until`). The untimed `std::shared_mutex` was added in C++17.

```cpp
std::shared_timed_mutex mtx;
std::shared_lock<std::shared_timed_mutex> read_lock(mtx);   // shared read
std::unique_lock<std::shared_timed_mutex> write_lock(mtx);  // exclusive write
```

---

## Deprecations & Removals

C++14 has no significant removals. A few minor deprecations:

- `std::gets` — removed (was deprecated in C++11); use `std::fgets`.
- Implicit use of `std::bind` result as `std::function` argument — some edge cases deprecated.
