[← Idioms](README.md)

# Pimpl

> Move private members into a forward-declared implementation struct, so the header exposes no private details and implementation changes do not trigger recompilation of consumers.

---

## The Problem

Private members must appear in the class definition, so they live in the header. Any translation unit that includes the header must recompile whenever a private member changes — even though clients cannot access them. Private `#include`s also propagate transitively to every consumer.

```cpp
// widget.h — every consumer must include <vector> and <string>
#include <vector>
#include <string>

class Widget {
public:
    void doWork();
private:
    std::vector<int> data_;
    std::string name_;
    void helper();
};
```

Changing `data_` from `std::vector<int>` to `std::deque<int>` recompiles every file that includes `widget.h`.

---

## The Pattern

Forward-declare a nested `Impl` struct in the header and hold a `unique_ptr` to it. Define `Impl` only in the `.cpp` file.

*widget.h:*
```cpp
#include <memory>

class Widget {
public:
    Widget();
    ~Widget();
    Widget(Widget&&) noexcept;
    Widget& operator=(Widget&&) noexcept;

    void doWork();

private:
    struct Impl;
    std::unique_ptr<Impl> impl_;
};
```

*widget.cpp:*
```cpp
#include "widget.h"
#include <vector>
#include <string>

struct Widget::Impl {
    std::vector<int> data;
    std::string name;
    void helper() { /* ... */ }
};

Widget::Widget() : impl_(std::make_unique<Impl>()) {}
Widget::~Widget() = default;                         // Impl must be complete here
Widget::Widget(Widget&&) noexcept = default;
Widget& Widget::operator=(Widget&&) noexcept = default;

void Widget::doWork() {
    impl_->helper();
}
```

`~Widget` must be defined in the `.cpp` where `Impl` is complete — `unique_ptr`'s destructor calls `delete` on an incomplete type if it is defined in the header.

---

## Copyability

`unique_ptr` is move-only, so copy operations are deleted by default. To make `Widget` copyable, write explicit copy constructor and assignment in the `.cpp`:

```cpp
Widget::Widget(const Widget& rhs)
    : impl_(std::make_unique<Impl>(*rhs.impl_)) {}

Widget& Widget::operator=(const Widget& rhs) {
    *impl_ = *rhs.impl_;
    return *this;
}
```

---

## Pros and Cons

**Pros**
- Implementation changes only recompile `widget.cpp` — consumers are insulated.
- Private `#include`s stay out of the header, reducing transitive include dependencies.
- The public interface (vtable layout, member offsets) is stable across implementation changes, preserving ABI for shared libraries.

**Cons**
- Every member access goes through an extra pointer dereference and a heap allocation per `Widget`.
- Copyability requires explicit deep-copy operations in the `.cpp`.
- More boilerplate: destructor, move operations, and `Impl` accessor forwarding must all be written explicitly.
