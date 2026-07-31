[← C++ Field Guide](../README.md)

# Non-const Member Operator

> A member comparison operator without `const` compiles for non-const callers but silently breaks for const-qualified objects and anywhere the standard library stores or accepts const references.

A member `operator==` must be marked `const` to be callable on const objects. Forgotten `const` is easy to overlook — the operator works fine for non-const objects and only fails when a const-qualified instance is involved.

The error points to the call site and says "discards qualifiers", making it non-obvious that the fix is at the definition, not the call.

The same applies to every comparison and inspection operator: `operator<`, `operator!=`, `operator>`, `operator<=`, `operator>=`, `operator<=>`. Any member function that only reads `*this` must be `const`.

Alternatively, a better fix is to define comparison operators as hidden friends. See [Operator Forms](../guides/operator-forms.md).

> [!note]
> Some compilers can warn about operators missing const.

---

Example 1: Compile error comparing const objects
```cpp
#include <cstdio>

struct Foo {
    int value;
    bool operator==(const Foo &rhs) { // missing const
        return value == rhs.value;
    }
};

int main()
{
    Foo a{1}, b{2};
    std::printf("a == b: %s\n", a == b ? "true" : "false"); // Ok
    
    const Foo c{3}, d{4};
    std::printf("c == d: %s\n", c == d ? "true" : "false"); // Error

    return 0;
}
```
Compiler:
```
/app/main.cpp:16:38: error: passing 'const Foo' as 'this' argument discards qualifiers [-fpermissive]
   16 |     std::printf("c == d: %s\n", c == d ? "true" : "false"); // Error
      |  
/app/main.cpp:5:10: note:   in call to 'bool Foo::operator==(const Foo&)'
    5 |     bool operator==(const Foo &rhs) { // missing const
      |          ^~~~~~~~
```

[Try it!](https://godbolt.org/z/a4cTGnc1d)

This issue can cause compile errors in the codebase later, once someone uses operator on const objects for the first time.

---

Example 2: Deleted operator use compiles because of implicit conversion
```cpp
#include <cstdio>
#include <array>

class Vector3 {
public:
    Vector3(double x, double y, double z) 
      : mData{x,y,z} 
    {}

    bool operator==(const Vector3 &) = delete; // Forgotten const

    // Implicit conversion to data pointer
    operator double *() {
        return mData.data(); 
    }
    operator double const *() const {
        return mData.data();
    }

    // Functions that intend to replace equality operator
    bool equals(const Vector3 &rhs) const;
    bool equalsApprox(const Vector3 &rhs, double epsilon) const;

private:
    std::array<double, 3> mData;
};

int main()
{
    const Vector3 a{1,2,3};
    // Issue #1: operator== is supposed to be deleted, this compiles fine.
    std::printf("a == a: %s\n", a == a ? "true" : "false");

    const Vector3 b{a};
    // Issue #2: b is copy of a, but comparison says not equals. Comparing pointers here!
    std::printf("a == b: %s\n", a == b ? "true" : "false");

    return 0;
}
```
Output:
```
a == a: true
a == b: false
```
[Try it!](https://godbolt.org/z/83Ebsh4Mf)

This issue is more **severe**. It causes the use of deleted comparison operator to compile and worse: it defaults to a completely different comparison - comparing two pointers via `operator==(double const *, double const *)`, as they both get implicitly converted. 
