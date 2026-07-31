[← C++ Field Guide](../README.md)

# Temporary Lifetime Extension

Binding a `const` lvalue reference or an rvalue reference directly to a temporary extends the temporary's lifetime to match the reference. But this extension is shallow — it does not propagate through function calls, member access, or struct members.

```cpp
const std::string& r = std::string("hello");  // OK: lifetime extended to scope of r
const char& c = std::string("hi")[0];         // UB: temporary destroyed; c dangles
```

In the second line, `operator[]` returns a reference *into* the temporary string. The reference `c` binds to that inner reference, not to the string itself, so the string is destroyed immediately.

Common failure patterns:

```cpp
// Through a member access
const int& r = std::vector<int>{1, 2, 3}.front();  // dangling: vector destroyed

// Through a function call
const std::string& f() { return std::string("x"); }  // always dangling
const std::string& r = f();                           // UB

// In a struct member (constructor init list)
struct S { const std::string& ref; };
S s{std::string("hello")};  // ref dangles after the constructor returns
```

The rule is simple: lifetime extension only fires when a reference binds *directly* to the temporary, in the same expression, without indirection.

---

Example: element reference into a temporary container
```cpp
#include <cstdio>
#include <vector>

int main()
{
    // Correct: reference extends lifetime of the int temporary
    const int& a = 42;
    std::printf("a = %d\n", a);  // OK

    // UB: front() returns a reference into the vector, not the vector itself;
    // the vector is destroyed at the semicolon, leaving `b` dangling.
    const int& b = std::vector<int>{10, 20, 30}.front();
    std::printf("b = %d (UB, non-deterministic)\n", b);
}
```
Output (typical, but behavior is undefined):
```
a = 42
b = 56492 (UB, non-deterministic)
```
[Try it!](https://godbolt.org/z/Esfzxz7ec)

With ASAN or in a release build the second line may produce garbage, because the vector's heap allocation is released before `b` is read.
