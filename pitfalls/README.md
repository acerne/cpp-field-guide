[← C++ Field Guide](../README.md)

# Pitfalls

> A collection of C++ traps organized by topic — each entry links to a dedicated file.

## Floating Point

- [Floating Point Comparison](floating-point-comparison.md) — equality with `==` fails due to rounding error
- [Floating Point Associativity](floating-point-associativity.md) — `(a+b)+c` ≠ `a+(b+c)` for floats
- [Float Addition Precision Loss](float-addition-precision-loss.md) — small values vanish when added to large ones
- [std::round vs Truncation](round-vs-truncation.md) — rounding functions differ for negative numbers
- [std::min / std::fmin and NaN](nan-min-fmin.md) — `std::min` result depends on argument order when NaN is present
- [NaN in Sorted Containers](nan-sorted-containers.md) — NaN violates strict weak ordering, corrupting ordered containers

## Integer

- [Signed Integer Overflow](signed-integer-overflow.md) — UB, not wrap-around; compilers exploit this
- [INT_MIN Negation](int-min-negation.md) — negating `INT_MIN` is undefined behavior
- [Integer Division Truncation](integer-division.md) — division truncates toward zero, not toward −∞
- [Signed / Unsigned Comparison](signed-unsigned-comparison.md) — `-1 < 1u` is `false`
- [Integer Promotion in Arithmetic](integer-promotion.md) — `char`/`uint8_t` arithmetic promotes to `int`; `~uint8_t(0xFF)` is `-256`, not `0`

## Lifetime

- [Uninitialized Variables Are Undefined Behavior](uninitialized-variables.md) — reading an uninitialized local is UB, not just a garbage value
- [Iterator Invalidation](iterator-invalidation.md) — `push_back` may reallocate, silently invalidating all existing iterators
- [Temporary Lifetime Extension](temporary-lifetime-extension.md) — binding `const&` to a temporary extends lifetime only when direct; member access or function returns still dangle

## Inheritance

- [Virtual Functions in Constructors and Destructors](virtual-in-constructors.md) — virtual calls during construction dispatch to the current class, bypassing overrides
- [Passing this to External Code During Construction](this-escape-constructor.md) — registry or callback receives a partially-constructed object; virtual dispatch and dynamic_cast both misbehave
- [delete via Non-Virtual Base Pointer](delete-non-virtual-base.md) — UB if the base lacks a virtual destructor; derived destructor never called

## Standard Library

- [std::vector\<bool\> Is Not a Container](vector-bool.md) — bit-packing specialization returns a proxy reference, breaking generic code and `auto`
- [Unchecked Access Is Undefined Behavior](unchecked-access.md) — `operator[]` and `operator*` are UB without a precondition check; `at()` and `value()` throw instead

## Operators

- [Non-const Member Operator](non-const-member-operator.md) — member comparison operators without `const` silently break for const objects and const references

## Compiler

- [MSVC /permissive-](msvc-permissive.md) — MSVC is non-conformant by default; the conformance flag name is counterintuitive
