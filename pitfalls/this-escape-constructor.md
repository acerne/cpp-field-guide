[← Pitfalls](README.md)

# Passing `this` to External Code During Construction

When a constructor passes `this` to external code — a registry, event bus, or callback — that code receives a partially-constructed object. Virtual dispatch resolves to the base-class version, and `dynamic_cast` to any derived type returns `nullptr`.

---

The most common trigger is a self-registering base class:

```cpp
struct Base {
    Base() { Registry::add(this); }      // this escapes before Derived is constructed
    virtual void on_registered() { std::puts("Base::on_registered"); }
};

struct Derived : Base {
    void on_registered() override { std::puts("Derived::on_registered"); }
};
```

If `Registry::add` calls `on_registered()` synchronously, it always dispatches to `Base::on_registered` — `Derived`'s vtable is not yet installed.

---

Full example:

```cpp
#include <cstdio>

struct Base {
    Base();
    virtual void on_registered() { std::puts("Base::on_registered"); }
};

struct Derived : Base {
    void on_registered() override { std::puts("Derived::on_registered"); }
};

static void registry_add(Base* obj) { obj->on_registered(); }

Base::Base() { 
    // Derived does not exist yet. registry_add will call Base method,
    // even though we are making Derived object.
    registry_add(this); 
}

int main()
{
    Derived d;
    return 0;
}
```
Output:
```
Base::on_registered
```
[Try it!](https://godbolt.org/z/7f98oK839)

---

The same applies to `dynamic_cast`. The runtime type is still `Base` at that point, so a cast to `Derived*` returns `nullptr` and silently skips initialization:

```cpp
static void registry_add(Base* obj) {
    if (auto* d = dynamic_cast<Derived*>(obj))
        d->special_init();   // never reached when called from Base::Base()
}
```

**Fix:** do not pass `this` to external code from a constructor. Register after the object is fully constructed — either at the call site or through a factory:

```cpp
// call site
auto obj = std::make_unique<Derived>();
registry_add(obj.get());   // fully constructed — dispatches correctly

// factory: keeps registration automatic without exposing the raw constructor
struct Base {
    template<typename T, typename... Args>
    static std::unique_ptr<T> make(Args&&... args) {
        auto obj = std::make_unique<T>(std::forward<Args>(args)...);
        registry_add(obj.get());
        return obj;
    }
protected:
    Base() = default;
};
```
