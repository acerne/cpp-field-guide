[← Idioms](README.md)

# Type Erasure

> Hide the concrete type of an object behind a uniform interface, without requiring the object to inherit from a common base.

---

## The Problem

Lambdas, function pointers, and functors all share the same call signature but have unrelated types. You cannot store them in a single container or pass them through a non-template API.

```cpp
auto double_it = [](int x) { return x * 2; };
auto add_one   = [](int x) { return x + 1; };

// decltype(double_it) != decltype(add_one) — no common type to use here
std::vector<???> transforms;
```

The same problem arises with any set of types that share behaviour but share no base class.

---

## Standard Library Type Erasure

`std::function` is the canonical example. It accepts any callable with a matching signature and hides its concrete type:

```cpp
std::vector<std::function<int(int)>> transforms = {
    [](int x) { return x * 2; },
    [](int x) { return x + 1; },
    &some_free_function,
};

for (auto& f : transforms) {
    result = f(result);
}
```

`std::any` does the same for arbitrary values (without a fixed interface); `std::variant` is a closed alternative when the set of types is known at compile time.

---

## Manual Type Erasure

When no standard type fits, implement erasure with the *concept + model* pattern: an internal abstract base (`Concept`) holds the interface; a template wrapper (`Model<T>`) adapts any conforming type.

```cpp
struct Shape {
    template<typename T>
    Shape(T obj) : impl_(std::make_unique<Model<T>>(std::move(obj))) {}

    void draw() const { impl_->draw(); }

private:
    struct Concept {
        virtual void draw() const = 0;
        virtual ~Concept() = default;
    };

    template<typename T>
    struct Model : Concept {
        explicit Model(T t) : data_(std::move(t)) {}
        void draw() const override { data_.draw(); }
        T data_;
    };

    std::unique_ptr<Concept> impl_;
};
```

`Circle` and `Square` need no knowledge of `Shape` — the coupling is one-directional:

```cpp
struct Circle { void draw() const { /* ... */ } };
struct Square { void draw() const { /* ... */ } };

std::vector<Shape> shapes;
shapes.emplace_back(Circle{});
shapes.emplace_back(Square{});

for (const auto& s : shapes) {
    s.draw();
}
```

The vtable is hidden inside `Shape`. Callers see value semantics; the concrete type is gone.

---

## Pros and Cons

**Pros**
- Concrete types are fully decoupled from the erasing type — non-intrusive, no shared base required.
- Supports heterogeneous collections with value semantics.
- The interface contract is defined by the consumer, not imposed on the erased types.

**Cons**
- Manual implementation is verbose; the concept + model pattern requires significant boilerplate.
- Each erased value typically heap-allocates, hurting cache locality for collections.
- The original type is unrecoverable without additional machinery (cf. `std::any_cast`).
