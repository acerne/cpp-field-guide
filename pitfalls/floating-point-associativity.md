[← Pitfalls](README.md)

# Floating Point Associativity

Floating point addition is not associative. Rearranging the parentheses changes the result because each operation rounds independently.

```cpp
float a = 1e8f, b = -1e8f, c = 1.0f;
(a + b) + c == a + (b + c);  // false
```

Compilers must respect parentheses by default. Flags like `-ffast-math` / `-fassociative-math` allow reordering for speed at the cost of reproducibility.

---

Example: Same operands, different grouping, different result
```cpp
#include <cstdio>

int main()
{
    float a = 1e8f, b = -1e8f, c = 1.0f;

    float left  = (a + b) + c;
    float right = a + (b + c);

    std::printf("(a + b) + c = %.1f\n", left);
    std::printf("a + (b + c) = %.1f\n", right);

    return 0;
}
```
Output:
```
(a + b) + c = 1.0
a + (b + c) = 0.0
```
[Try it!](https://godbolt.org/z/scf5xYoq7)
