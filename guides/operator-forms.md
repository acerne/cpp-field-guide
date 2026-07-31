[← C++ Field Guide](../README.md)

# Operator Forms

> Prefer hidden friends for symmetric operators — they give private access, symmetric overload resolution, and ADL-only visibility.

---

## Member Method

Defines the operator as a member function with an implicit `this` as the left-hand operand.

```cpp
struct Foo {
    int value;
    bool operator==(const Foo& rhs) const { return value == rhs.value; }
};
```

The left-hand side must be `Foo` — no implicit conversions on the left. The right-hand side accepts any type convertible to `const Foo&`.

```cpp
Foo s{5};
s == Foo{5};   // OK
s == 5;          // OK if Foo(int) exists — implicit conversion on the right only
5 == s;          // error — int has no operator==
```

This asymmetry causes subtle bugs in generic code that assumes commutativity. It also requires `const` on the member or the operator silently breaks for const-qualified callers — see [Non-const Member Operator](../pitfalls/non-const-member-operator.md).

Member functions are the right choice for operators that inherently act on `*this`: `operator[]`, `operator()`, `operator->`, compound assignment (`operator+=`), and increment/decrement.

---

## Free Function

Defines the operator outside the class as an ordinary non-member function.

```cpp
struct Foo {
    int value;
};

bool operator==(const Foo& lhs, const Foo& rhs) {
    return lhs.value == rhs.value;
}
```

Both operands are treated symmetrically. However, the function is visible to all unqualified name lookups in the enclosing namespace, so it participates in overload resolution even at unrelated call sites — unexpected implicit conversions become possible.

```cpp
struct Wrapper { operator Foo() const; };
Wrapper w; Foo s{5};
w == s;   // compiles — unqualified lookup finds operator==, then converts Wrapper
```

Access to private members also requires a `friend` declaration, coupling the out-of-class definition back to the class.

---

## Hidden Friend

Defines the operator as a `friend` inside the class body. The function is a non-member in the enclosing namespace but is only reachable via ADL.

```cpp
struct Foo {
    int value;
    friend bool operator==(const Foo& lhs, const Foo& rhs) {
        return lhs.value == rhs.value;
    }
};
```

- Symmetric: both operands are explicit parameters — no left/right asymmetry.
- Private access: the definition is inside the class body.
- ADL-only: the function is invisible unless at least one argument is of type `Foo`.

```cpp
Foo a{3}, b{5};
a == b;              // OK — ADL finds the hidden friend
operator==(a, b);    // error — not reachable by qualified lookup
```

See [Hidden Friend](../idioms/hidden-friend.md) for comparison operators, `swap`, `operator<<`, and CRTP mixins.

---

## Comparison

Each form makes different trade-offs across four properties.

| | Member method | Free function | Hidden friend |
|---|---|---|---|
| Accesses private members | yes | no | yes |
| Symmetric | no | yes | yes |
| Implicit conversions | right side only | both sides | both sides |
| Name lookup | member call | unqualified | ADL-only |

---

## Prefer

Use **hidden friends** for symmetric binary operators (`==`, `!=`, `<`, `<=>`, `<<`).  
Use **member functions** for operators that act on the object itself (`[]`, `()`, `->`, `+=`, `++`).
