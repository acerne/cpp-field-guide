[← Pitfalls](README.md)

# `delete` via Non-Virtual Base Pointer

Deleting a derived object through a base-class pointer is **undefined behavior** if the base class does not have a `virtual` destructor. The derived destructor is never called, resources may leak, and the behavior of the `delete` expression itself is implementation-defined.

```cpp
struct Base {
    ~Base() { std::puts("~Base"); }   // not virtual
};

struct Derived : Base {
    ~Derived() { std::puts("~Derived"); }
};

Base* p = new Derived;
delete p;  // UB — ~Derived never called
```

The standard is explicit: if the static type of the pointer differs from the dynamic type of the object, the base's destructor must be `virtual`, or the behavior is undefined ([expr.delete]/3).

In practice, only `~Base` runs, leaking any resources owned by `Derived`. A sanitizer or debug runtime may report a mismatched `new`/`delete` pair.

**Rule:** Any class designed to be used polymorphically must declare its destructor `virtual`. Mark non-polymorphic base classes `final` or make the destructor `protected` (non-virtual) to prevent accidental deletion through the base pointer.

```cpp
struct Base {
    virtual ~Base() = default;  // safe for polymorphic delete
};
```

---

Example: destructor not called, resource leaked
```cpp
#include <cstdio>

struct Resource {
    Resource()  { std::puts("   Resource"); }
    ~Resource() { std::puts("   ~Resource"); }
};

struct Base {
    Base() { std::puts(" Base"); }
    ~Base() { std::puts(" ~Base"); }
};

struct Derived : Base {
    Resource r;
    Derived() { std::puts("  Derived"); }
    ~Derived() { std::puts("  ~Derived"); }
};

int main()
{
    std::puts("correct: delete through Derived*");
    Derived *d = new Derived;
    delete d;

    std::puts("incorrect: delete through Base*");
    Base* p = new Derived;
    delete p;  // UB: ~Derived and Resource::~Resource never called
}
```
Output:
```
correct: delete through Derived*
 Base
   Resource
  Derived
  ~Derived
   ~Resource
 ~Base
incorrect: delete through Base*
 Base
   Resource
  Derived
 ~Base
```
[Try it!](https://godbolt.org/z/hdzfeWr8d)

The resource is acquired but never released, and `~Derived` is skipped entirely.
