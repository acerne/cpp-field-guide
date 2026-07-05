[← Guides](README.md)

# Deduction Guides

_C++17_

> Write explicit deduction guides when the compiler cannot derive class template arguments from constructors alone.

---

## Context

C++17 CTAD generates implicit deduction guides from each constructor, enabling construction without explicit template arguments:

```cpp
std::pair p{1, 2.0};      // std::pair<int, double>
std::lock_guard lk{mtx};  // std::lock_guard<std::mutex>
```

Implicit guides cover straightforward cases. When the constructor parameters don't correspond directly to the class template parameters, explicit guides are required.

---

## Specialization-Only Templates

When a class has no defined primary template, there are no constructors to derive implicit guides from. The guides also select which specialization to instantiate.

```cpp
template <typename T>
struct Handle;  // primary undefined — only specializations exist

template <typename T>
struct Handle<T*> {
    T* raw;
    Handle(T* p) : raw{p} {}
};

template <typename T>
struct Handle<std::shared_ptr<T>> {
    std::shared_ptr<T> ptr;
    Handle(std::shared_ptr<T> p) : ptr{std::move(p)} {}
};

template <typename T> Handle(T*)                 -> Handle<T*>;
template <typename T> Handle(std::shared_ptr<T>) -> Handle<std::shared_ptr<T>>;

int x = 42;
Handle h1{&x};                        // Handle<int*>
Handle h2{std::make_shared<int>(42)}; // Handle<std::shared_ptr<int>>
```

---

## Constructor Type Mismatch

When a constructor takes a type that is related to but distinct from the class template parameter, implicit deduction leaves `T` undetermined.

```cpp
template <typename T>
class Stack {
    std::vector<T> data_;
public:
    template <typename Iter>
    Stack(Iter first, Iter last) : data_(first, last) {}
};

// T is not deducible from Iter — guide maps iterator type to element type:
template <typename Iter>
Stack(Iter, Iter) -> Stack<typename std::iterator_traits<Iter>::value_type>;

std::vector<int> src{1, 2, 3};
Stack s{src.begin(), src.end()};  // Stack<int>
```
