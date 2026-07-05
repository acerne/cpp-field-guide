[← Pitfalls](README.md)

# Signed / Unsigned Comparison

When a signed and unsigned integer are compared, the signed value is implicitly converted to unsigned. Negative values wrap around to large positive numbers.

```cpp
-1 < 1u;  // false — -1 becomes 4294967295
```

Common form of the bug — iterating backwards over a container:
```cpp
for (int i = v.size() - 1; i >= 0; --i) { /* ... */ }
// v.size() returns size_t (unsigned); if v is empty, v.size() - 1 wraps to SIZE_MAX
```

---

Example: Signed-to-unsigned conversion makes -1 greater than 1
```cpp
#include <cstdio>

int main()
{
    std::printf("uint(-1) = %u\n",  static_cast<unsigned int>(-1));
    std::printf("signed/unsigned:\n");
    std::printf("  -1 < 1u : %s\n", (-1 < 1u)          ? "true" : "false");
    std::printf("  1u > -1 : %s\n", (1u > -1)          ? "true" : "false");
    std::printf("signed only:\n");
    std::printf("  -1 <  1  : %s\n", (-1 < 1)     ? "true" : "false");
    std::printf("   1 > -1 : %s\n", (1 > -1)     ? "true" : "false");

    return 0;
}
```
Output:
```
uint(-1) = 4294967295

signed/unsigned:
  -1 < 1u : false
  1u > -1 : false

signed only:
  -1 <  1 : true
   1 > -1 : true
```
[Try it!](https://godbolt.org/z/8YYYTW7Wr)