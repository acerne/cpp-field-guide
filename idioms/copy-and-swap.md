[← Idioms](README.md)

# Copy-and-Swap

> Implement copy assignment by copying into a local temporary then swapping, achieving exception safety and self-assignment correctness without special-casing either.

---

## The Problem

A naive copy assignment operator must handle two subtle cases: self-assignment (`a = a`) and exception safety. Implementing both correctly is easy to get wrong.

```cpp
Widget& operator=(const Widget& rhs) {
    delete[] data_;             // (1) destroy our data
    data_ = new int[rhs.size_]; // (2) if rhs is *this, source is already destroyed
    std::copy(rhs.data_, rhs.data_ + rhs.size_, data_);
    size_ = rhs.size_;
    return *this;
}
```

If `new` throws after `delete[]`, `data_` is a dangling pointer. If `rhs` is `*this`, the delete destroys the source before the copy.

---

## The Pattern

Take the right-hand side by value — the copy constructor handles the copy and any exception. Then swap internals with `*this` and let the old state destroy with the temporary at scope exit.

```cpp
class Widget {
public:
    Widget(const Widget& rhs);        // copy ctor
    Widget(Widget&& rhs) noexcept;    // move ctor

    Widget& operator=(Widget rhs) noexcept {  // by value — copy or move on entry
        swap(*this, rhs);
        return *this;
    }

    friend void swap(Widget& a, Widget& b) noexcept {
        using std::swap;
        swap(a.data_, b.data_);
        swap(a.size_, b.size_);
    }

private:
    int* data_;
    std::size_t size_;
};
```

**Self-assignment:** `rhs` is an independent copy, so swapping with `*this` leaves both sides valid.

**Exception safety:** if the copy constructor throws before `operator=` starts, `*this` is untouched — strong guarantee.

**Move assignment:** when called with an rvalue, the move constructor runs instead of the copy constructor, and `swap` transfers ownership cheaply.

---

## Pros and Cons

**Pros**
- One `operator=` covers both copy and move assignment.
- Strong exception-safety guarantee — `*this` is unchanged if the copy throws.
- Self-assignment is trivially correct with no branch needed.

**Cons**
- Always makes a full copy even when an in-place update would be cheaper (e.g., reusing an existing allocation of the same size).
- Requires a `noexcept` `swap`; a throwing swap weakens the exception-safety guarantee.
- The by-value parameter surprises readers unfamiliar with the idiom.
