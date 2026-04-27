[← Pitfalls](README.md)

# Signed Integer Overflow

Signed integer overflow is **undefined behavior**. The compiler is free to assume it never happens and optimize accordingly — the result is not guaranteed to wrap around.

```cpp
int x = INT_MAX;
x + 1;  // UB — not necessarily INT_MIN
```

A common optimizer exploit: the compiler may assume `i + 1 > i` is always true for `int i`, eliminating overflow checks entirely:
```cpp
// intended: loop until overflow
for (int i = 0; i >= 0; ++i) { /* ... */ }
// compiler proves i >= 0 is always true (overflow is UB) → infinite loop
```

Unsigned overflow is **defined** — it wraps modulo 2^N. Use unsigned types when wrapping arithmetic is intentional.

---

Example: Signed integer overflow and compiler optimization
```cpp
#include <climits>
#include <cstdio>

int main()
{
    int x = INT_MAX;

    std::printf("x   = %d\n", x);
    std::printf("x+1 = %d\n", x + 1);
    std::printf("x+1 < x : %s\n", x + 1 < x ? "true" : "false");

    return 0;
}
```
Output:
```
x   = 2147483647
x+1 = -2147483648
x+1 < x : false
```
[Try it!](https://godbolt.org/z/rxEEGeaYn)

The compiler proves `x + 1 < x` is never true for signed `int` — overflow is UB, so it assumes it cannot happen and folds the check to `false`.
