[← C++ Field Guide](../README.md)

# C++17

> C++17 polishes the language with quality-of-life syntax improvements and expands the standard library with vocabulary types and concurrency primitives.

## Index
- [Language Features](#language-features)
  - [Nested Namespaces](#nested-namespaces)
  - [Structured Bindings](#structured-bindings)
  - [If / Switch with Initializer](#if--switch-with-initializer)
  - [If Constexpr](#if-constexpr)
  - [Fold Expressions](#fold-expressions)
  - [Class Template Argument Deduction](#class-template-argument-deduction)
  - [Attributes](#attributes)
- [Standard Library Additions](#standard-library-additions)
  - [std::optional](#stdoptional)
  - [std::variant](#stdvariant)
  - [std::any](#stdany)
  - [std::string_view](#stdstring_view)
  - [std::shared_mutex / std::shared_lock](#stdshared_mutex--stdshared_lock)
  - [std::scoped_lock](#stdscoped_lock)
  - [std::filesystem](#stdfilesystem)
  - [Parallel Algorithms](#parallel-algorithms-stdexecution)
- [Deprecations & Removals](#deprecations--removals)

---

## Language Features

C++17 brings several syntax improvements that reduce boilerplate and enable cleaner generic code.

### Nested Namespaces

[Read more](https://en.cppreference.com/w/cpp/language/namespace)

Allows declaring nested namespaces in a single declaration, reducing indentation.

_Before:_
```cpp
namespace engine {
namespace graphics {
namespace shapes {
    class Rectangle;
}
}
}
```

_After:_
```cpp
namespace engine::graphics::shapes {
    class Rectangle;
}
```

### Structured Bindings

[Read more](https://en.cppreference.com/w/cpp/language/structured_binding)

Eliminates `.first`/`.second` when iterating maps:
```cpp
std::map<int, std::string> numbers{ { 1, "one" }, { 2, "two" }, { 3, "three" } };
for (const auto& [key, value] : numbers) {
    // ...
}
```

Return multiple values directly into named variables:
```cpp
std::tuple<int, bool, std::string> example() {
    return {30, true, "John"};
}

auto [age, male, name] = example();
```

Decompose aggregate structs:
```cpp
struct Rectangle { int x, y, w, h; };

Rectangle box{10, 20, 30, 40};
auto [x, y, w, h] = box;
```

### If / Switch with Initializer

[Read more](https://en.cppreference.com/w/cpp/language/if)

Scopes the init variable to the if/else block, keeping it out of the surrounding scope:
```cpp
if (const auto it = std::find(names.begin(), names.end(), "John");
    it != names.end()) {
    return true;
}
```

The same syntax works for `switch`, with a separate init and condition divided by `;`:
```cpp
switch (auto result = process(); result.code()) {
case 0:  handle_success(result); break;
default: handle_error(result);   break;
}
```

### If Constexpr

[Read more](https://en.cppreference.com/w/cpp/language/if#Constexpr_if)

Branch is selected at compile time; the discarded branch is not instantiated.
```cpp
template <typename T>
auto length(const T& value) {
    if constexpr (std::is_integral_v<T>) {
        return value;
    } else {
        return value.length();
    }
}
```

### Fold Expressions

[Read more](https://en.cppreference.com/w/cpp/language/fold)

Reduces a variadic parameter pack with a binary operator — no manual recursion needed:
```cpp
template <typename... Args>
auto sum(Args... args) {
    return (args + ...);
}

// sum(1, 2, 3, 4, 5) == 15
```

### Class Template Argument Deduction

[Read more](https://en.cppreference.com/w/cpp/language/class_template_argument_deduction)

Template arguments are deduced from constructor arguments — no explicit `<T>` annotation needed:
```cpp
std::pair p{42, "hello"};           // std::pair<int, const char*>
std::vector v{1, 2, 3};             // std::vector<int>
std::optional opt = std::string{};  // std::optional<std::string>
```

### Attributes

[Read more](https://en.cppreference.com/w/cpp/language/attributes)

C++17 standardizes three new attributes for expressing intent and suppressing spurious warnings:
```cpp
[[nodiscard]] int compute();          // warn if return value is discarded

[[maybe_unused]] int debug_val = 0;  // suppress unused-variable warning

switch (x) {
  case 1:
    handle();
    [[fallthrough]];                  // suppress implicit-fallthrough warning
  case 2:
    also_handle();
}
```

---

## Standard Library Additions

C++17 adds vocabulary types that express intent more clearly than raw pointers or pairs, and improves concurrency primitives.

### `std::optional`

[Read more](https://en.cppreference.com/w/cpp/utility/optional)

Represents a value that may or may not be present — replaces sentinel values and `bool` out-params.
```cpp
std::optional<std::string> find_name(int id) {
    if (id == 42) return "Alice";
    return std::nullopt;
}

if (auto name = find_name(id)) {
    use(*name);
}
```

### `std::variant`

[Read more](https://en.cppreference.com/w/cpp/utility/variant)

Type-safe union. Knows which type it currently holds; throws on invalid access.
```cpp
std::variant<int, std::string> v = "hello";
std::get<std::string>(v);          // "hello"
std::get<int>(v);                  // throws std::bad_variant_access
```

`std::visit` dispatches to a callable based on the active type:
```cpp
std::visit([](const auto& val) { std::cout << val; }, v);
```

### `std::any`

[Read more](https://en.cppreference.com/w/cpp/utility/any)

Type-erased container for a single value of any copy-constructible type.
```cpp
std::any a = 42;
a = std::string{"now a string"};
std::any_cast<std::string>(a);     // "now a string"
```

### `std::string_view`

[Read more](https://en.cppreference.com/w/cpp/string/basic_string_view)

Non-owning reference to a character sequence. Avoids copies when reading strings.
```cpp
void print(std::string_view sv) { /* ... */ }

print("literal");           // no allocation
print(some_std_string);     // no copy
```

Does not own or extend the lifetime of its source.

### `std::shared_mutex` / `std::shared_lock`

[Read more](https://en.cppreference.com/w/cpp/thread/shared_mutex)

Reader-writer lock: multiple concurrent readers, exclusive writer.
```cpp
std::shared_mutex mtx;

// reader
std::shared_lock read_lock(mtx);    // shared (read) access

// writer
std::unique_lock write_lock(mtx);   // exclusive (write) access
```

### `std::scoped_lock`

[Read more](https://en.cppreference.com/w/cpp/thread/scoped_lock)

Locks multiple mutexes atomically, avoiding deadlock from lock-order differences.
```cpp
std::scoped_lock lock(mutex_a, mutex_b);
```

### `std::filesystem`

[Read more](https://en.cppreference.com/w/cpp/filesystem)

Portable API for path manipulation, directory traversal, and file status queries.
```cpp
namespace fs = std::filesystem;

fs::path p = "/home/user/docs";
for (const auto& entry : fs::directory_iterator(p)) {
    if (entry.is_regular_file())
        process(entry.path());
}
```

### Parallel Algorithms (`std::execution`)

[Read more](https://en.cppreference.com/w/cpp/algorithm/execution_policy_tag_t)

Execution policies let standard algorithms run in parallel or vectorized with a single flag:
```cpp
std::sort(std::execution::par_unseq, v.begin(), v.end());
```

---

## Deprecations & Removals

C++17 removes several pre-standard and long-deprecated features.

- `std::auto_ptr` — removed; use `std::unique_ptr`
- `register` keyword — removed as a storage-class specifier
- `std::random_shuffle` — removed; use `std::shuffle` with an explicit RNG
- Trigraphs — removed
- Dynamic exception specifications (`throw(type)`) — removed; use `noexcept`
