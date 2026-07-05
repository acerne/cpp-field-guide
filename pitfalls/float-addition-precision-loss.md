[← Pitfalls](README.md)

# Float Addition Precision Loss

When adding two floats of very different magnitudes, the smaller value may be completely lost. The hardware aligns exponents before adding mantissas — the smaller mantissa is shifted right until exponents match, dropping its significant bits.

```cpp
double big   = 1e16;
double small = 1.0;
big + small == big;  // true — 1.0 is below the precision of 1e16
```

Accumulating many small values into a large sum hits the same problem. Use [Kahan summation](https://en.wikipedia.org/wiki/Kahan_summation_algorithm) or sort values by magnitude (smallest first) when precision matters.

---

Example: Addition order matters — accumulate small values first
```cpp
#include <cstdio>
#include <vector>
#include <numeric>

int main()
{
    // List of values, ordered large-first
    std::vector values{ 1e16, 1., 1., 1., 1., 1., 1., 1., 1., 1., 1. };

    double large_first_sum = std::accumulate(values.begin(), values.end(), 0.0);
    double small_first_sum = std::accumulate(values.rbegin(), values.rend(), 0.0);

    std::printf("large_first_sum = %.1f\n", large_first_sum);
    std::printf("small_first_sum = %.1f\n", small_first_sum);
    std::printf("difference      = %.1f\n", small_first_sum - large_first_sum);

    return 0;
}
```
Output:
```
large_first_sum = 10000000000000000.0
small_first_sum = 10000000000000010.0
difference      = 10.0
```

[Try it!](https://godbolt.org/z/f5n9ejW8h)
