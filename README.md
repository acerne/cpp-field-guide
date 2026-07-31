# C++ Field Guide

> A practical reference for C++ — pitfalls, patterns, hints, and version overviews.

## Floating-Point Arithmetic

- [Guide] [Floating-Point Comparison](guides/floating-point-comparison.md) — when to use `==`, absolute epsilon, or combined tolerance to compare floating-point values
- [Pitfall] [Float Addition Precision Loss](pitfalls/float-addition-precision-loss.md) — small values vanish when added to large ones
- [Pitfall] [Floating Point Associativity](pitfalls/floating-point-associativity.md) — `(a+b)+c` ≠ `a+(b+c)` for floats
- [Pitfall] [Floating Point Comparison](pitfalls/floating-point-comparison.md) — equality with `==` fails due to rounding error
- [Pitfall] [NaN in Sorted Containers](pitfalls/nan-sorted-containers.md) — NaN violates strict weak ordering, corrupting ordered containers
- [Pitfall] [std::round vs Truncation](pitfalls/round-vs-truncation.md) — rounding functions differ for negative numbers
- [Pitfall] [std::min / std::fmin and NaN](pitfalls/nan-min-fmin.md) — `std::min` result depends on argument order when NaN is present

## Integer Arithmetic

- [Pitfall] [char Signedness](pitfalls/char-signedness.md) — whether plain `char` is signed or unsigned is implementation-defined
- [Guide] [Fixed-Width Integers vs Built-in Types](guides/fixed-width-integers.md) — use `std::int32_t` and friends when the exact bit width is part of a contract
- [Pitfall] [INT_MIN Negation](pitfalls/int-min-negation.md) — negating `INT_MIN` is undefined behavior
- [Guide] [Integer Comparison](guides/integer-comparison.md) — use `std::cmp_equal` and its siblings to compare signed and unsigned integers without the implicit-conversion trap
- [Pitfall] [Integer Division Truncation](pitfalls/integer-division.md) — division truncates toward zero, not toward −∞
- [Pitfall] [Integer Promotion in Arithmetic](pitfalls/integer-promotion.md) — `char`/`uint8_t` arithmetic promotes to `int`; `~uint8_t(0xFF)` is `-256`, not `0`
- [Pitfall] [Signed Integer Overflow](pitfalls/signed-integer-overflow.md) — UB, not wrap-around; compilers exploit this
- [Pitfall] [Signed / Unsigned Comparison](pitfalls/signed-unsigned-comparison.md) — `-1 < 1u` is `false`

## Object Lifetime & Initialization

- [Guide] [Copy vs Move](guides/copy-vs-move.md) — when to copy an object and when to move it, including RVO and copy elision
- [Guide] [Initialization Syntax](guides/initialization.md) — when to use `=`, `()`, or `{}` to initialize a variable
- [Guide] [Rule of Five](guides/rule-of-five.md) — a class that manages a resource directly needs the destructor, copy, and move members defined together, or none at all
- [Idiom] [Copy-and-Swap](idioms/copy-and-swap.md) — implement copy assignment by copying into a temporary and swapping for exception safety and self-assignment correctness
- [Idiom] [RAII](idioms/raii.md) — bind resource lifetime to object lifetime for guaranteed cleanup
- [Idiom] [Scope Guard](idioms/scope-guard.md) — run cleanup code on scope exit regardless of how or when the resource was acquired
- [Pitfall] [Temporary Lifetime Extension](pitfalls/temporary-lifetime-extension.md) — binding `const&` to a temporary extends lifetime only when direct; member access or function returns still dangle
- [Pitfall] [Uninitialized Variables Are Undefined Behavior](pitfalls/uninitialized-variables.md) — reading an uninitialized local is UB, not just a garbage value
- [Pitfall] [Zero-Initializing a Scoped Enum](pitfalls/enum-zero-initialization.md) — `Number x{};` yields `0`, a value that may not match any named enumerator

## Inheritance & Polymorphism

- [Idiom] [CRTP](idioms/crtp.md) — a derived class passes itself as a template argument to its base, enabling mixins and static polymorphism
- [Idiom] [Non-Virtual Interface](idioms/nvi.md) — public non-virtual functions delegate to private virtual hooks, keeping the call protocol under base class control
- [Pitfall] [delete via Non-Virtual Base Pointer](pitfalls/delete-non-virtual-base.md) — UB if the base lacks a virtual destructor; derived destructor never called
- [Pitfall] [Passing this to External Code During Construction](pitfalls/this-escape-constructor.md) — registry or callback receives a partially-constructed object; virtual dispatch and dynamic_cast both misbehave
- [Pitfall] [Virtual Functions in Constructors and Destructors](pitfalls/virtual-in-constructors.md) — virtual calls during construction dispatch to the current class, bypassing overrides

## Class Design & Encapsulation

- [Guide] [Forward Declarations](guides/forward-declarations.md) — declare a type or function before defining it to break circular dependencies and reduce compile-time coupling
- [Idiom] [Factory Method](idioms/factory-method.md) — a static member function that validates inputs and returns an object only when construction would succeed
- [Idiom] [Hidden Friend](idioms/hidden-friend.md) — define non-member functions inside the class body so they are reachable only via ADL
- [Idiom] [Passkey](idioms/passkey.md) — restrict a public constructor to specific callers by requiring a private token type only those callers can construct
- [Idiom] [Pimpl](idioms/pimpl.md) — hide private members behind a pointer to a forward-declared struct to reduce compile-time coupling

## Templates & Generic Programming

- [Guide] [Deduction Guides](guides/deduction-guides.md) — write explicit deduction guides when the compiler cannot derive class template arguments from constructors alone
- [Guide] [std::variant and std::visit](guides/variant-visit.md) — type-safe union with exhaustive dispatch over its alternatives
- [Idiom] [if constexpr Dispatch](idioms/if-constexpr-dispatch.md) — branch on a compile-time condition inside a template without instantiating the discarded branch
- [Idiom] [Overload Set](idioms/overload-set.md) — combine multiple lambdas into one callable for exhaustive and compiler-checked variant dispatch
- [Idiom] [Tag Dispatch](idioms/tag-dispatch.md) — select a function overload at compile time by passing a trait-derived tag type
- [Idiom] [Type Erasure](idioms/type-erasure.md) — hide concrete types behind a uniform interface without requiring a shared base class

## Containers & Iteration

- [Guide] [Range-Based For Loop](guides/range-based-for.md) — iterate over any range-compatible container or array with a concise loop syntax
- [Guide] [Structured Bindings](guides/structured-bindings.md) — bind names to struct members, tuple elements, or pair components in a single declaration
- [Pitfall] [Iterator Invalidation](pitfalls/iterator-invalidation.md) — `push_back` may reallocate, silently invalidating all existing iterators
- [Pitfall] [Unchecked Access Is Undefined Behavior](pitfalls/unchecked-access.md) — `operator[]` and `operator*` are UB without a precondition check; `at()` and `value()` throw instead
- [Pitfall] [std::vector\<bool\> Is Not a Container](pitfalls/vector-bool.md) — bit-packing specialization returns a proxy reference, breaking generic code and `auto`

## Casting & Type Conversion

- [Guide] [Polymorphic Casting](guides/polymorphic-casting.md) — when to use `static_cast`, `dynamic_cast`, or `dynamic_pointer_cast` on class hierarchies
- [Guide] [Type Casting](guides/type-casting.md) — when to use `static_cast`, `reinterpret_cast`, or a C-style cast

## Operators & Function Interfaces

- [Guide] [Operator Forms](guides/operator-forms.md) — when to define an operator as a member method, free function, or hidden friend
- [Guide] [Parameter Passing](guides/parameter-passing.md) — when to pass by value, by const reference, or by pointer
- [Pitfall] [Non-const Member Operator](pitfalls/non-const-member-operator.md) — member comparison operators without `const` silently break for const objects and const references

## Control Flow

- [Idiom] [Early Return](idioms/early-return.md) — guard clauses keep the happy path at the top level
- [Idiom] [Immediately Invoked Lambda](idioms/iife.md) — initialize a `const` variable with complex logic in place using a lambda called immediately

## Concurrency

- [Guide] [std::future and std::promise](guides/future-promise.md) — set a value asynchronously and retrieve it from another thread

## Compiler & Portability

- [Pitfall] [MSVC /permissive-](pitfalls/msvc-permissive.md) — MSVC is non-conformant by default; the conformance flag name is counterintuitive

## Tools

### Code Quality
- [clang-format](tools/clang-format.md) — enforce consistent code style automatically
- [clang-tidy](tools/clang-tidy.md) — static analysis and modernization checks with auto-fix support

### Testing
- [Catch2](tools/catch2.md) — expressive testing framework with natural-language syntax; an alternative to GoogleTest
- [GoogleTest](tools/googletest.md) — Google's C++ unit testing framework

### Benchmarking
- [Google Benchmark](tools/google-benchmark.md) — microbenchmarking library for measuring C++ code performance

### Online Tools
- [Compiler Explorer](tools/compiler-explorer.md) — shows annotated assembly for C++ code across multiple compilers and optimization levels
- [OnlineGDB](tools/onlinegdb.md) — browser-based compiler and debugger for quick tests and sharing snippets
- [Quick-Bench](tools/quick-bench.md) — browser-based microbenchmarking built on Google Benchmark; compares variants without local setup

## Versions

- [C++11](versions/cpp11.md) — move semantics, lambdas, threading model
- [C++14](versions/cpp14.md) — refinements and fixes to C++11
- [C++17](versions/cpp17.md) — structured bindings, `if constexpr`, parallel algorithms
- [C++20](versions/cpp20.md) — concepts, ranges, coroutines, modules
- [C++23](versions/cpp23.md) — library completions and language cleanup

---

_AI assistance was used for generation of these documents._
