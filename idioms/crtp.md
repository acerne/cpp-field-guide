[← Idioms](README.md)

# CRTP

> A derived class passes itself as a template argument to its base, letting the base call derived methods without virtual dispatch.

---

## The Pattern

In the Curiously Recurring Template Pattern (CRTP), the base class is a template parameterized on its own derived class. The base reaches derived behaviour through a `static_cast` — resolved at compile time, no vtable involved.

```cpp
template<typename Derived>
class Base {
public:
    void interface() {
        static_cast<Derived*>(this)->implementation();
    }
};

class Concrete : public Base<Concrete> {
public:
    void implementation() { /* ... */ }
};
```

---

## Mixins

The most common use of CRTP is adding functionality to derived classes without repeating it. The base provides generic operations built on a small interface the derived class supplies.

A `Comparable` mixin generates all six comparison operators from just `==` and `<`:

```cpp
template<typename Derived>
struct Comparable {
    bool operator!=(const Derived& rhs) const {
        return !(static_cast<const Derived&>(*this) == rhs);
    }
    bool operator> (const Derived& rhs) const {
        return rhs < static_cast<const Derived&>(*this);
    }
    bool operator<=(const Derived& rhs) const {
        return !(rhs < static_cast<const Derived&>(*this));
    }
    bool operator>=(const Derived& rhs) const {
        return !(static_cast<const Derived&>(*this) < rhs);
    }
};

struct Point : Comparable<Point> {
    int x, y;
    bool operator==(const Point& rhs) const { return x == rhs.x && y == rhs.y; }
    bool operator< (const Point& rhs) const { return x < rhs.x || (x == rhs.x && y < rhs.y); }
};
```

`Point` defines two operators; the mixin generates the other four. Multiple CRTP bases can be combined freely.

---

## Static Polymorphism

CRTP can replace virtual dispatch in hot paths where the concrete type is always known at the call site. The compiler can inline through the `static_cast`; a virtual call cannot be inlined unless devirtualized.

```cpp
// Virtual: runtime dispatch, vtable per object
struct Encoder {
    virtual void encode(std::span<const std::byte> data) = 0;
    virtual ~Encoder() = default;
};

// CRTP: compile-time dispatch, no vtable
template<typename Derived>
struct Encoder {
    void encode(std::span<const std::byte> data) {
        static_cast<Derived*>(this)->encodeImpl(data);
    }
};

struct XorEncoder : Encoder<XorEncoder> {
    void encodeImpl(std::span<const std::byte> data) { /* ... */ }
};
```

The trade-off: CRTP requires the concrete type to be known at compile time, so heterogeneous collections (e.g., `std::vector<Encoder*>`) are not possible without type erasure on top.

---

## Pros and Cons

**Pros**
- No vtable; the compiler can inline through `static_cast<Derived*>` calls in hot paths.
- Mixins compose freely — a class can inherit multiple CRTP bases without conflict.
- No per-object storage overhead compared to virtual dispatch.

**Cons**
- `Base<A>` and `Base<B>` are unrelated types; heterogeneous use requires type erasure on top.
- The self-referential template argument is unfamiliar, and errors are typically hard to read.
- A mismatched cast (`static_cast` to the wrong `Derived`) is silent undefined behavior.
