[← Pitfalls](README.md)

# NaN in Sorted Containers

NaN violates strict weak ordering: `!(NaN < x) && !(x < NaN)` for any `x`, which the standard interprets as NaN being equivalent to all other values. This breaks the internal invariants of ordered containers and algorithms.

```cpp
std::set<double> s = {1.0, 2.0, 3.0};
s.insert(std::numeric_limits<double>::quiet_NaN());
s.count(1.0);  // may return 0, 1, loop forever, or crash
```

The same applies to `std::map`, `std::sort`, `std::binary_search`, and anything relying on `operator<`.

Guard against NaN at input boundaries with `std::isnan`. If NaN must be stored, use a container that does not require ordering (`std::unordered_set` with a custom hash), or define a total order that maps NaN to a canonical position.

---

Example: NaN inserted first absorbs all subsequent `std::set` insertions
```cpp
#include <cmath>
#include <cstdio>
#include <limits>
#include <set>

int main()
{
    const double nan = std::numeric_limits<double>::quiet_NaN();
    std::set<double> s;
    s.insert(nan);
    s.insert(1.0); // Never inserted
    s.insert(2.0); // Never inserted
    s.insert(3.0); // Never inserted

    std::printf("size:       %zu\n", s.size());
    std::printf("count(9.0): %zu\n", s.count(9.0));
    std::printf("count(nan): %zu\n", s.count(nan));
}
```
Output:
```
size:       1
count(9.0): 1
count(nan): 1
```
[Try it!](https://godbolt.org/z/oavvsrzPM)

NaN is equivalent to every value under `operator<`, so subsequent inserts are treated as duplicates and dropped. `count()` for any key returns 1 — the NaN node matches every lookup.

---

Example: NaN in a sorted range causes `std::binary_search` to return false positives
```cpp
#include <algorithm>
#include <cmath>
#include <cstdio>
#include <limits>
#include <vector>

int main()
{
    const double nan = std::numeric_limits<double>::quiet_NaN();
    std::vector<double> v = {1.0, 2.0, 3.0, 4.0, nan};

    std::printf("binary_search(2.0): %s\n",
        std::binary_search(v.begin(), v.end(), 2.0) ? "found" : "not found");
    std::printf("binary_search(5.0): %s\n",
        std::binary_search(v.begin(), v.end(), 5.0) ? "found" : "not found");
}
```
Output:
```
binary_search(2.0): found
binary_search(5.0): found
```
[Try it!](https://godbolt.org/z/915oeo9Tz)

`5.0` is not in the array. `std::binary_search` internally calls `std::lower_bound`; NaN's broken ordering steers `lower_bound` to the NaN element, which then compares as equivalent to any searched value.
