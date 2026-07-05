[← C++ Field Guide](../README.md)

# Idioms

> Short articles on idiomatic and correct ways to write C++ code.

## Control Flow

Patterns for structuring conditional logic and function exit paths.

- [Early Return](early-return.md) — guard clauses keep the happy path at the top level
- [Immediately Invoked Lambda](iife.md) — initialize a `const` variable with complex logic in place using a lambda called immediately

## Resource Management

Patterns for pairing acquisition and release so cleanup is guaranteed on every exit path.

- [RAII](raii.md) — bind resource lifetime to object lifetime for guaranteed cleanup
- [Scope Guard](scope-guard.md) — run cleanup code on scope exit regardless of how or when the resource was acquired

## Class Design

Patterns for controlling construction, access, and special member behavior within a single class.

- [Factory Method](factory-method.md) — a static member function that validates inputs and returns an object only when construction would succeed
- [Hidden Friend](hidden-friend.md) — define non-member functions inside the class body so they are reachable only via ADL
- [Pimpl](pimpl.md) — hide private members behind a pointer to a forward-declared struct to reduce compile-time coupling
- [Passkey](passkey.md) — restrict a public constructor to specific callers by requiring a private token type only those callers can construct
- [Copy-and-Swap](copy-and-swap.md) — implement copy assignment by copying into a temporary and swapping for exception safety and self-assignment correctness

## Inheritance

Patterns for structuring behavior across base and derived classes.

- [CRTP](crtp.md) — a derived class passes itself as a template argument to its base, enabling mixins and static polymorphism
- [Non-Virtual Interface](nvi.md) — public non-virtual functions delegate to private virtual hooks, keeping the call protocol under base class control

## Dispatch

Patterns for routing calls to the right implementation based on type information, at compile time or at runtime.

- [Type Erasure](type-erasure.md) — hide concrete types behind a uniform interface without requiring a shared base class
- [Tag Dispatch](tag-dispatch.md) — select a function overload at compile time by passing a trait-derived tag type
- [Overload Set](overload-set.md) — combine multiple lambdas into one callable for exhaustive and compiler-checked variant dispatch
- [if constexpr Dispatch](if-constexpr-dispatch.md) — branch on a compile-time condition inside a template without instantiating the discarded branch
