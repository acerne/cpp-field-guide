# C++11

> C++11 is the most transformative revision of the language — it introduced move semantics, lambdas, and a threading model that effectively made modern C++ a different language from its predecessor.

## Index
- [Language Features](#language-features)
  - [auto](#auto)
  - [Range-based for](#range-based-for)
  - [Lambda Expressions](#lambda-expressions)
  - [Move Semantics](#move-semantics)
  - [nullptr](#nullptr)
  - [constexpr](#constexpr)
  - [static_assert](#static_assert)
  - [enum class](#enum-class)
  - [Default and Deleted Functions](#default-and-deleted-functions)
  - [override and final](#override-and-final)
  - [noexcept](#noexcept)
  - [Uniform Initialization](#uniform-initialization)
  - [Type Aliases](#type-aliases)
  - [decltype](#decltype)
  - [Variadic Templates](#variadic-templates)
- [Standard Library Additions](#standard-library-additions)
  - [std::unique_ptr](#stdunique_ptr)
  - [std::shared_ptr / std::weak_ptr](#stdshared_ptr--stdweak_ptr)
  - [std::thread](#stdthread)
  - [std::future / std::async](#stdfuture--stdasync)
  - [std::atomic](#stdatomic)
  - [std::function](#stdfunction)
  - [std::array](#stdarray)
  - [std::tuple](#stdtuple)
  - [std::unordered_map / std::unordered_set](#stdunordered_map--stdunordered_set)
  - [std::chrono](#stdchrono)
- [Deprecations & Removals](#deprecations--removals)

---

## Language Features

C++11 overhauls the core language with type inference, lambda expressions, move semantics, and a range of ergonomic improvements that reduce boilerplate and eliminate common error patterns.

### `auto`

[Read more](https://en.cppreference.com/w/cpp/language/auto)

Deduces the type of a variable from its initializer. Especially useful with iterators and complex template types.

```cpp
auto x = 42;                            // int
auto it = some_map.begin();             // std::map<K,V>::iterator
auto ptr = std::unique_ptr<Foo>(new Foo());  // std::unique_ptr<Foo>
```

`auto` deduces the value type — it strips references and cv-qualifiers. Use `auto&` or `const auto&` to preserve them:
```cpp
for (auto& elem : vec)   { /* modifies */ }
for (const auto& elem : vec) { /* reads */ }
```

### Range-based `for`

[Read more](https://en.cppreference.com/w/cpp/language/range-for)

Iterates over any range (container, array, initializer list) without index or iterator boilerplate:
```cpp
std::vector<int> v = {1, 2, 3};
for (const auto& x : v) {
    // ...
}
```

### Lambda Expressions

[Read more](https://en.cppreference.com/w/cpp/language/lambda)

Anonymous function objects with an optional capture list. Syntax: `[capture](params) -> return_type { body }`.

```cpp
auto add = [](int a, int b) { return a + b; };
add(2, 3);  // 5
```

Capture local variables by value (`=`) or by reference (`&`):
```cpp
int offset = 10;
auto by_val  = [offset](int x) { return x + offset; };  // captures offset by value
auto by_ref  = [&offset](int x) { return x + offset; }; // captures by reference
auto all_val = [=](int x) { return x + offset; };       // captures all locals by value
auto all_ref = [&](int x) { return x + offset; };       // captures all locals by reference
```

Commonly used with standard algorithms:
```cpp
std::sort(v.begin(), v.end(), [](int a, int b) { return a > b; });
```

### Move Semantics

[Read more](https://en.cppreference.com/w/cpp/language/move_constructor)

Rvalue references (`T&&`) enable transferring resources instead of copying them. A move leaves the source in a valid but unspecified state.

```cpp
std::vector<int> a = {1, 2, 3};
std::vector<int> b = std::move(a);  // transfers ownership; a is now empty
```

`std::move` is a cast — it does not move anything itself. The actual move happens in the move constructor or move assignment operator of the target type.

Implement move constructor and move assignment to support efficient moves in your own types:
```cpp
class Buffer {
    Buffer(Buffer&& other) noexcept
        : data_(other.data_), size_(other.size_) {
        other.data_ = nullptr;
        other.size_ = 0;
    }
    // ...
};
```

See also: [rule of three/five/zero](https://en.cppreference.com/w/cpp/language/rule_of_three).

### `nullptr`

[Read more](https://en.cppreference.com/w/cpp/language/nullptr)

Replaces `NULL` and `0` as the null pointer constant. Has type `std::nullptr_t`, avoiding ambiguity in overload resolution:

```cpp
void f(int);
void f(int*);

f(NULL);     // ambiguous — could be either overload
f(nullptr);  // unambiguously calls f(int*)
```

### `constexpr`

[Read more](https://en.cppreference.com/w/cpp/language/constexpr)

Declares that a value or function can be evaluated at compile time. C++11 `constexpr` functions are limited to a single return expression.

```cpp
constexpr int square(int x) { return x * x; }

int arr[square(4)];         // ok — evaluated at compile time
constexpr int s = square(5); // 25 at compile time
```

(Restrictions were significantly relaxed in C++14 and C++17.)

### `static_assert`

[Read more](https://en.cppreference.com/w/cpp/language/static_assert)

Compile-time assertion. Fails to compile with a diagnostic if the condition is false.

```cpp
static_assert(sizeof(int) == 4, "expected 32-bit int");

template <typename T>
void serialize(T) {
    static_assert(std::is_trivially_copyable<T>::value, "T must be trivially copyable");
}
```

### `enum class`

[Read more](https://en.cppreference.com/w/cpp/language/enum#Scoped_enumerations)

Scoped, strongly-typed enums. Unlike plain `enum`, enumerators do not leak into the enclosing scope and do not implicitly convert to integers.

_Before:_
```cpp
enum Color { Red, Green, Blue };
int x = Red;            // implicit conversion — compiles
```

_After:_
```cpp
enum class Color { Red, Green, Blue };
Color c = Color::Red;   // must be qualified
int x = c;              // error — no implicit conversion
int x = static_cast<int>(c);  // explicit cast required
```

### Default and Deleted Functions

[Read more](https://en.cppreference.com/w/cpp/language/function#Deleted_functions)

`= default` asks the compiler to generate the default implementation. `= delete` explicitly removes a function from the overload set.

```cpp
struct NonCopyable {
    NonCopyable() = default;
    NonCopyable(const NonCopyable&) = delete;
    NonCopyable& operator=(const NonCopyable&) = delete;
};
```

`= delete` works on any function, not just special members — useful for blocking unintended conversions:
```cpp
void f(int);
void f(double) = delete;  // prevents implicit conversion from double
```

### `override` and `final`

[Read more](https://en.cppreference.com/w/cpp/language/override)

`override` makes the compiler verify that a virtual function actually overrides a base class method. `final` prevents further overriding (or further derivation when applied to a class).

```cpp
struct Base {
    virtual void draw() const;
};

struct Derived : Base {
    void draw() const override;  // error if Base::draw signature doesn't match
};

struct Leaf : Derived {
    void draw() const final;     // prevents further overriding in subclasses
};
```

Without `override`, a signature mismatch silently creates a new virtual function instead of overriding.

### `noexcept`

[Read more](https://en.cppreference.com/w/cpp/language/noexcept_spec)

Declares that a function does not throw. The standard library only moves elements during reallocation (e.g. `std::vector` resize) when the move constructor is `noexcept` — without it, it falls back to copying.

```cpp
void close() noexcept;

class Buffer {
    Buffer(Buffer&&) noexcept;
    Buffer& operator=(Buffer&&) noexcept;
};
```

### Uniform Initialization

[Read more](https://en.cppreference.com/w/cpp/language/list_initialization)

Brace `{}` syntax works uniformly across all types and prevents narrowing conversions:

```cpp
int x{5};
std::vector<int> v{1, 2, 3};
std::map<int, std::string> m{{1, "one"}, {2, "two"}};
```

Narrowing is a compile error with braces:
```cpp
int x = 3.7;    // ok — truncates silently
int x{3.7};     // error — narrowing conversion
```

Watch out: if a type has an `initializer_list` constructor, it takes priority. This is the `std::vector` trap:
```cpp
std::vector<int> v(3, 0);   // 3 elements, all 0
std::vector<int> v{3, 0};   // 2 elements: 3 and 0
```

### Type Aliases

[Read more](https://en.cppreference.com/w/cpp/language/type_alias)

`using` replaces `typedef` and extends it to support template aliases:

_Before:_
```cpp
typedef std::map<std::string, std::vector<int>> StringVecMap;
```

_After:_
```cpp
using StringVecMap = std::map<std::string, std::vector<int>>;
```

Template alias — not possible with `typedef`:
```cpp
template <typename T>
using Vec = std::vector<T>;

Vec<int> v;  // std::vector<int>
```

### `decltype`

[Read more](https://en.cppreference.com/w/cpp/language/decltype)

Deduces the type of an expression without evaluating it. Used in templates and trailing return types where the return type depends on arguments:

```cpp
template <typename A, typename B>
auto add(A a, B b) -> decltype(a + b) {
    return a + b;
}
```

(Trailing return types became largely unnecessary with `auto` return type deduction in C++14.)

### Variadic Templates

[Read more](https://en.cppreference.com/w/cpp/language/parameter_pack)

Templates that accept any number of type arguments. The pack is expanded with `...`. C++11 requires recursive unpacking with an explicit base-case overload:

```cpp
void print() {}  // base case

template <typename T, typename... Rest>
void print(T head, Rest... tail) {
    std::cout << head << " ";
    print(tail...);
}
```

C++17 fold expressions eliminate the recursive base case — see [Fold Expressions](cpp17.md#fold-expressions).

---

## Standard Library Additions

C++11 adds smart pointers, a threading model, type-erased function wrappers, and several new containers that replace common hand-rolled patterns.

### `std::unique_ptr`

[Read more](https://en.cppreference.com/w/cpp/memory/unique_ptr)

Sole-ownership smart pointer. Resource is freed when the `unique_ptr` goes out of scope. Not copyable, only movable.

```cpp
std::unique_ptr<Foo> p(new Foo());
p->bar();
// freed automatically at end of scope
```

`std::make_unique<Foo>()` is preferred over `new` and is available from C++14.

Use `std::move` to transfer ownership:
```cpp
auto q = std::move(p);  // p is now null
```

### `std::shared_ptr` / `std::weak_ptr`

[Read more](https://en.cppreference.com/w/cpp/memory/shared_ptr)

`std::shared_ptr<T>` is a reference-counted shared ownership pointer. `std::weak_ptr<T>` holds a non-owning reference that does not affect the count, used to break cycles.

```cpp
auto a = std::make_shared<Foo>();
auto b = a;           // ref count: 2
// freed when both a and b go out of scope
```

```cpp
std::weak_ptr<Foo> w = a;
if (auto locked = w.lock()) {  // null if the object is already freed
    locked->bar();
}
```

### `std::thread`

[Read more](https://en.cppreference.com/w/cpp/thread/thread)

Launches a new OS thread. Must be either joined or detached before destruction.

```cpp
std::thread t([]{ do_work(); });
t.join();
```

Protect shared data with `std::mutex` and `std::unique_lock`:
```cpp
std::mutex mtx;
std::unique_lock<std::mutex> lock(mtx);
shared_data.modify();
```

### `std::future` / `std::async`

[Read more](https://en.cppreference.com/w/cpp/thread/future)

`std::async` launches a task and returns a `std::future` that holds the result. Calling `.get()` blocks until the task completes.

```cpp
std::future<int> result = std::async(std::launch::async, compute);
// ... do other work ...
int val = result.get();
```

`std::promise` provides the write end of the channel for manually signalling a result between threads.

### `std::atomic`

[Read more](https://en.cppreference.com/w/cpp/atomic/atomic)

Lock-free atomic operations on scalar types. Avoids data races without a mutex for simple counters and flags.

```cpp
std::atomic<int> counter{0};
counter++;              // atomic increment
counter.fetch_add(5);   // atomic add, returns old value
```

### `std::function`

[Read more](https://en.cppreference.com/w/cpp/utility/functional/function)

Type-erased wrapper for any callable: function pointer, lambda, functor, or `std::bind` result. Heavier than templates but useful when the callable type must be stored or passed at runtime.

```cpp
std::function<int(int, int)> op = [](int a, int b) { return a + b; };
op(2, 3);  // 5
op = std::multiplies<int>{};
op(2, 3);  // 6
```

### `std::array`

[Read more](https://en.cppreference.com/w/cpp/container/array)

Fixed-size array with value semantics and standard container interface. Replaces raw C arrays.

```cpp
std::array<int, 3> a = {1, 2, 3};
a.size();   // 3
a.at(5);    // throws std::out_of_range
```

### `std::tuple`

[Read more](https://en.cppreference.com/w/cpp/utility/tuple)

Heterogeneous fixed-size collection. Generalizes `std::pair` to N elements.

```cpp
auto t = std::make_tuple(42, true, "hello");
std::get<0>(t);  // 42
std::get<2>(t);  // "hello"
```

(Structured bindings in C++17 make tuple unpacking much more ergonomic.)

### `std::unordered_map` / `std::unordered_set`

[Read more](https://en.cppreference.com/w/cpp/container/unordered_map)

Hash-based associative containers. O(1) average lookup vs O(log n) for `std::map`.

```cpp
std::unordered_map<std::string, int> scores;
scores["alice"] = 100;
scores.count("bob");  // 0
```

Custom key types require a hash specialization or a hash functor passed as template argument.

### `std::chrono`

[Read more](https://en.cppreference.com/w/cpp/chrono)

Type-safe time and duration library. Durations carry their unit in the type, preventing accidental unit mixing.

```cpp
using namespace std::chrono_literals;  // C++14; in C++11, use duration constructors

auto start = std::chrono::steady_clock::now();
do_work();
auto elapsed = std::chrono::steady_clock::now() - start;
auto ms = std::chrono::duration_cast<std::chrono::milliseconds>(elapsed).count();
```

---

## Deprecations & Removals

C++11 deprecates several pre-standard and unsafe idioms; most removals happen in C++17.

- `std::auto_ptr` — deprecated; use `std::unique_ptr`. (`auto_ptr` was removed in C++17.)
- `register` storage class — deprecated as a hint (the keyword is reserved for future use).
- Dynamic exception specifications (`throw(Type)`) — deprecated; use `noexcept`. (Removed in C++17.)
- `std::bind1st` / `std::bind2nd` — deprecated; use lambdas or `std::bind`.
