[← C++ Field Guide](../README.md)

# Uninitialized Variables Are Undefined Behavior

Reading an uninitialized local variable is **undefined behavior**, not merely "garbage value" arithmetic. The compiler is permitted to assume the read never happens and may eliminate branches, reorder code, or produce results that are impossible under any reading of the source.

```cpp
int x;
if (x > 0)          // UB: x is indeterminate
    launch_missile();
```

The optimizer may prove that the `if` body is unreachable (since UB makes any value of `x` equally valid) and remove it — or keep it, or do something else entirely. The result is not predictable across compilers, optimization levels, or builds.

Zero-initialization only happens automatically for:
- Non-local (namespace-scope, `static`) variables.
- Objects constructed with `{}` or `= {}`.
- POD members of a value-initialized aggregate.

Local variables of built-in type are **not** zero-initialized by default.

```cpp
int a;       // indeterminate
int b{};     // zero-initialized (OK)
int c = {};  // zero-initialized (OK)
```

Compilers warn (`-Wuninitialized`, enabled in `-Wall`) but cannot always detect it, especially through pointers or in complex control flow.

---

Example: compiler eliminating code due to uninitialized read
```cpp
#include <cstdio>

bool check(int* p, int n)
{
    int result;
    for (int i = 0; i < n; ++i)
        if (p[i] < 0)
            result = -1;
    return result == -1;  // UB if no negative element was found: result uninitialized
}

int main()
{
    int data[] = {1, 2, 3};
    std::printf("has_negative: %s\n", check(data, 3) ? "yes" : "no");

    int data2[] = {1, -1, 3};
    std::printf("has_negative: %s\n", check(data2, 3) ? "yes" : "no");
}
```
```
has_negative: yes
has_negative: yes
```
[Try it!](https://godbolt.org/z/W3YvTzv8b)

The compiler proves that reading result uninitialized is UB, and since UB "cannot happen" by assumption, it concludes the only reachable path is the one where result was assigned. This produces consistent "yes, yes" output.
