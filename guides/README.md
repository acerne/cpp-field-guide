[← C++ Field Guide](../README.md)

# Guides

> Practical guides on using language and library features, and on choosing between approaches when C++ offers more than one.

## Features

How to use specific language features and standard library types.

- [Structured Bindings](structured-bindings.md) — bind names to struct members, tuple elements, or pair components in a single declaration
- [Range-Based For Loop](range-based-for.md) — iterate over any range-compatible container or array with a concise loop syntax
- [Forward Declarations](forward-declarations.md) — declare a type or function before defining it to break circular dependencies and reduce compile-time coupling
- [std::variant and std::visit](variant-visit.md) — type-safe union with exhaustive dispatch over its alternatives
- [std::future and std::promise](future-promise.md) — set a value asynchronously and retrieve it from another thread
- [Deduction Guides](deduction-guides.md) — write explicit deduction guides when the compiler cannot derive class template arguments from constructors alone

## Design Decisions

How to choose between multiple approaches when C++ gives you more than one way to do the same thing.

- [Initialization Syntax](initialization.md) — when to use `=`, `()`, or `{}` to initialize a variable
- [Type Casting](type-casting.md) — when to use `static_cast`, `reinterpret_cast`, or a C-style cast
- [Polymorphic Casting](polymorphic-casting.md) — when to use `static_cast`, `dynamic_cast`, or `dynamic_pointer_cast` on class hierarchies
- [Copy vs Move](copy-vs-move.md) — when to copy an object and when to move it, including RVO and copy elision
- [Parameter Passing](parameter-passing.md) — when to pass by value, by const reference, or by pointer
- [Operator Forms](operator-forms.md) — when to define an operator as a member method, free function, or hidden friend
- [Floating-Point Comparison](floating-point-comparison.md) — when to use `==`, absolute epsilon, or combined tolerance to compare floating-point values
