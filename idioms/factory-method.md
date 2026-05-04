[← Idioms](README.md)

# Factory Method

> A static member function that validates inputs and controls how the resulting object is owned and stored.

---

## The Problem

Constructors have no return type, so the only way to signal a validation failure is to throw. That forces callers to use exceptions even when failure is routine and expected.

```cpp
class Fraction {
public:
    Fraction(int num, int den) : num_(num), den_(den) {
        if (den == 0) throw std::invalid_argument("zero denominator");
    }
    // ...
private:
    int num_, den_;
};

Fraction f(3, 0);   // throws — caller must catch to handle a predictable case
```

Exceptions also cannot be used in `noexcept` code or in environments where they are disabled.

---

## Factory Method

A `static` member function validates before delegating to a private constructor. The private constructor ensures an invalid object cannot be built directly.

```cpp
class Fraction {
public:
    static std::optional<Fraction> create(int num, int den) {
        if (den == 0) return std::nullopt;
        return Fraction(num, den);
    }

    int numerator()   const { return num_; }
    int denominator() const { return den_; }

private:
    Fraction(int num, int den) : num_(num), den_(den) {}

    int num_;
    int den_;
};
```

```cpp
auto f = Fraction::create(3, 4);   // std::optional<Fraction>
if (!f) {
    // handle invalid input without exceptions
}
```

Because the constructor is private, `Fraction(1, 0)` is a compile error — an invalid `Fraction` cannot exist.

---

## Member Invariants

Because the constructor is only reachable through the factory, every successfully constructed object is guaranteed to hold valid members. Methods can rely on this without rechecking.

```cpp
class Scaler {
public:
    static std::optional<Scaler> create(double factor) {
        if (factor == 0.0) return std::nullopt;
        return Scaler(factor);
    }

    double apply(double value) const {
        return value / mFactor;   // no zero-check needed — guaranteed by create()
    }

private:
    explicit Scaler(double factor) : mFactor(factor) {}

    double mFactor;   // non-zero for the entire lifetime of Scaler
};
```

Any method that would overwrite `mFactor` must validate the new value first, to preserve the same guarantee.

---

## Return Types

The return type serves double duty: it signals whether construction succeeded and it communicates how the object is stored and owned. The right choice depends on what the caller needs to do with the result.

### `std::optional`

Returns the object by value on success, `std::nullopt` on failure. Use when the failure reason is obvious from context.

```cpp
static std::optional<Fraction> create(int num, int den) {
    if (den == 0) return std::nullopt;
    return Fraction(num, den);
}
```

### `std::expected`

Like `std::optional`, but the error case carries a value explaining *why* construction failed. Available in C++23.

```cpp
enum class FractionError { ZeroDenominator };

static std::expected<Fraction, FractionError> create(int num, int den) {
    if (den == 0) return std::unexpected(FractionError::ZeroDenominator);
    return Fraction(num, den);
}
```

```cpp
auto result = Fraction::create(1, 0);
if (!result) {
    switch (result.error()) {
        case FractionError::ZeroDenominator: /* ... */ break;
    }
}
```

### `std::unique_ptr`

Returns a heap-allocated object with unique ownership transferred to the caller. `nullptr` signals failure. This is the natural choice for polymorphic types where the factory may return different derived classes.

```cpp
static std::unique_ptr<Fraction> create(int num, int den) {
    if (den == 0) return nullptr;
    return std::unique_ptr<Fraction>(new Fraction(num, den));
}
```

`std::make_unique` cannot be used here because it calls the constructor from outside the class, bypassing `private` access. The factory function itself is a member, so `new Fraction(...)` works.

The return type alone tells the caller: this object lives on the heap and you own it exclusively.

### `std::shared_ptr`

Returns a heap-allocated object with shared ownership. Use when the factory retains its own reference — for example, in a cache or object registry — or when the caller needs to share ownership with other parties.

```cpp
static std::shared_ptr<Fraction> create(int num, int den) {
    if (den == 0) return nullptr;
    return std::shared_ptr<Fraction>(new Fraction(num, den));
}
```

Prefer `std::unique_ptr` when sharing is not required — it is cheaper and communicates clearer ownership. A `unique_ptr` can always be implicitly converted to `shared_ptr` at the call site if needed.

---

## Pros and Cons

**Pros**
- Validation failure is expressed in the return type — no exceptions required.
- A private constructor makes an invalid object unrepresentable; the type's invariants are enforced by construction.
- Members are guaranteed valid for the object's entire lifetime — methods need no defensive checks (see Member Invariants).
- The return type explicitly communicates ownership: by-value, unique, or shared.
- Named factory functions (e.g., `Fraction::fromDouble`, `Fraction::fromString`) can express intent better than overloaded constructors.

**Cons**
- `std::make_unique` and `std::make_shared` cannot be used with a private constructor; `new` must be called directly inside the factory.
- `nullptr` as a failure signal for pointer-returning factories is ambiguous; prefer `std::optional<std::unique_ptr<T>>` when callers cannot rely on documentation alone.
- Adds a layer of indirection that can surprise callers who expect direct construction.
