[← C++ Field Guide](../README.md)

# Standard Behavior Categories

> The standard sorts non-portable or unsafe code into distinct categories with different guarantees — knowing which one applies tells you whether a program is broken, merely non-portable, or fine.

---

## Undefined Behavior

The standard imposes no requirements at all: a compiler may assume undefined behavior never happens and optimize accordingly, so the result is not "garbage value" or "crash" but genuinely anything, including code before the triggering line behaving unexpectedly. Signed overflow, out-of-bounds access, and reading an uninitialized variable are all undefined behavior — see [Signed Integer Overflow](../pitfalls/signed-integer-overflow.md), [Unchecked Access Is Undefined Behavior](../pitfalls/unchecked-access.md), and [Uninitialized Variables Are Undefined Behavior](../pitfalls/uninitialized-variables.md).

[Read more](https://en.cppreference.com/w/cpp/language/ub)

Juniors tend to read "it worked when I ran it" as proof of correctness. Undefined behavior often does work, until a different compiler, optimization level, or surrounding code changes what the compiler assumes is reachable.

---

## Implementation-Defined Behavior

The result varies between implementations, but each conforming implementation must pick one consistent behavior and document it. Whether plain `char` is signed or unsigned, and the width of `int`, are both implementation-defined — see [char Signedness](../pitfalls/char-signedness.md).

```cpp
constexpr bool char_is_signed = (CHAR_MIN < 0);  // implementation-defined
```

This is not a bug on any single platform — the behavior is stable and documented there. It becomes a bug when code silently assumes one implementation's choice and ships to another.

---

## Unspecified Behavior

Like implementation-defined behavior, the result is one of a set of valid outcomes chosen by the implementation — but the implementation isn't required to document which one, and it may even differ between calls in the same program. Order of evaluation of function arguments is a common example:

```cpp
f(g(), h());   // order of the calls to g() and h() is unspecified
```

Both call orders are standard-conforming. Code that depends on one of them is portable only by accident.
