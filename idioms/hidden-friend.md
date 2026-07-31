[← C++ Field Guide](../README.md)

# Hidden Friend

> Define non-member functions as `friend` inside the class body so they are only reachable via argument-dependent lookup, not by unqualified or qualified lookup.

---

## The Idiom

A `friend` function defined *inside* a class body is injected into the enclosing namespace but is invisible to ordinary name lookup. It can only be found when the class type appears as a function argument and ADL kicks in.

```cpp
class Token {
public:
    explicit Token(std::string s) : value_(std::move(s)) {}

    friend bool operator==(const Token& lhs, const Token& rhs) {
        return lhs.value_ == rhs.value_;
    }

private:
    std::string value_;
};
```

`operator==` is defined inside `Token` but it is a free function in the enclosing namespace, accessible to private members, and only findable through ADL:

```cpp
Token a{"hello"}, b{"world"};
a == b;              // OK — ADL finds operator==
operator==(a, b);    // error — not reachable by qualified lookup
```

---

## Why Hidden Matters

Defining the same operator as an ordinary free function leaves it visible to unqualified lookup everywhere. That lets it participate in overload resolution even when called with arguments of unrelated types, producing unexpected implicit conversions.

_Free function (not hidden):_
```cpp
bool operator==(const Token& lhs, const Token& rhs);  // visible everywhere

struct Wrapper { operator Token() const; };
Wrapper w; Token t{"x"};
w == t;   // compiles — implicit conversion found via unqualified lookup
```

_Hidden friend:_
```cpp
class Token {
    friend bool operator==(const Token& lhs, const Token& rhs) { ... }
};

Wrapper w; Token t{"x"};
w == t;   // error — ADL does not find operator== unless an argument is Token
```

Hiding the function removes it from consideration in contexts where neither argument is of the associated type. Overload sets stay smaller and error messages are cleaner.

---

## Operators

Operators that operate symmetrically on the class type are the primary use case. Defining them as hidden friends gives them access to private members without the asymmetry of member operators (member `operator==` lets implicit conversions happen on the right-hand side but not the left-hand side).

### Comparison operators

```cpp
class Timestamp {
public:
    explicit Timestamp(long ms) : ms_(ms) {}

    friend bool operator==(const Timestamp& lhs, const Timestamp& rhs) {
        return lhs.ms_ == rhs.ms_;
    }
    friend bool operator< (const Timestamp& lhs, const Timestamp& rhs) {
        return lhs.ms_ < rhs.ms_;
    }

private:
    long ms_;
};
```

In C++20, a single hidden-friend `operator<=>` generates all six comparisons:

```cpp
class Timestamp {
public:
    explicit Timestamp(long ms) : ms_(ms) {}

    friend auto operator<=>(const Timestamp& lhs, const Timestamp& rhs) = default;

private:
    long ms_;
};
```

### Stream insertion

`operator<<` cannot be a member (the left-hand side is `std::ostream`, not the class). As a hidden friend it stays coupled to the class while keeping the ADL-only restriction:

```cpp
class Vec2 {
public:
    float x, y;

    friend std::ostream& operator<<(std::ostream& os, const Vec2& v) {
        return os << '(' << v.x << ", " << v.y << ')';
    }
};
```

```cpp
Vec2 p{1.0f, 2.0f};
std::cout << p;   // prints: (1, 2)
```

### `swap`

Defining `swap` as a hidden friend lets `std::swap` be specialized for the class via ADL. Code that calls `swap(a, b)` in a template with `using std::swap;` will find it automatically:

```cpp
class Buffer {
public:
    Buffer() = default;

    friend void swap(Buffer& lhs, Buffer& rhs) noexcept {
        using std::swap;
        swap(lhs.data_, rhs.data_);
        swap(lhs.size_, rhs.size_);
    }

private:
    std::byte* data_ = nullptr;
    std::size_t size_ = 0;
};
```

```cpp
// In generic code:
using std::swap;
swap(buf1, buf2);   // ADL finds the hidden-friend swap; no copy needed
```

---

## Combining With CRTP

A CRTP mixin can inject hidden friends into its derived class, generating operators once for any type that opts in:

```cpp
template<typename Derived>
struct Printable {
    friend std::ostream& operator<<(std::ostream& os, const Derived& obj) {
        obj.print(os);
        return os;
    }
};

class Color : public Printable<Color> {
public:
    Color(uint8_t r, uint8_t g, uint8_t b) : r_(r), g_(g), b_(b) {}
    void print(std::ostream& os) const {
        os << "rgb(" << +r_ << ',' << +g_ << ',' << +b_ << ')';
    }

private:
    uint8_t r_, g_, b_;
};
```

```cpp
Color c{255, 128, 0};
std::cout << c;   // rgb(255,128,0)
```

---

## Pros and Cons

**Pros**
- Access to private members without a member function, keeping operator symmetry.
- ADL-only visibility prevents unwanted implicit conversions and keeps overload sets small.
- Definition lives next to the class, making it easy to maintain and read.
- Works with CRTP to generate operators in a mixin without touching derived-class code.

**Cons**
- Not reachable by qualified lookup — debugging with explicit `ClassName::operator==` calls is not possible; the function has no qualified name.
- ADL rules can still surprise when multiple associated namespaces are involved.
- Overly long friend definitions inside the class body harm readability; keep bodies short or forward to a private helper.
