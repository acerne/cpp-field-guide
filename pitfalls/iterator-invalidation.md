[← C++ Field Guide](../README.md)

# Iterator Invalidation

Mutating a container can silently invalidate existing iterators, pointers, and references into it. Using an invalidated iterator is undefined behavior — the program may crash, silently corrupt data, or appear to work correctly.

```cpp
std::vector<int> v = {1, 2, 3};
auto it = v.begin();
v.push_back(4);   // may reallocate — `it` is now invalid
std::cout << *it; // UB
```

The rules differ by container:

| Container | Invalidating operations |
|---|---|
| `std::vector` | Any insertion that causes reallocation; `erase` invalidates from the erased element to `end` |
| `std::deque` | All insertions at front/back invalidate iterators; insertions in the middle invalidate all |
| `std::list`, `std::map`, `std::set` | Only iterators to the erased element are invalidated; all others remain valid |
| `std::unordered_map` | Rehashing (triggered by load factor) invalidates all iterators |

Pre-reserve capacity with `v.reserve(n)` to prevent reallocation when the final size is known.

> [!WARNING]
> A particularly common mistake is erasing elements mid-iteration — even a single `erase` inside a range-based `for` loop invalidates its internal iterator, producing undefined behavior. There is no visible iterator to reason about, so the bug is easy to miss.

---

Example: `push_back` while holding an iterator/reference
```cpp
#include <cstdio>
#include <vector>

int main()
{
    std::vector<int> v;
    v.reserve(3);  // capacity = 3, no reallocation for first 3 pushes

    v.push_back(10);
    v.push_back(20);
    v.push_back(30);

    auto it = v.begin() + 1;  // Iterator to second item
    std::printf("before push_back: *it = %d\n", *it);

    int &ref = v[1];  // Reference to second item
    std::printf("before push_back: ref = %d\n", ref);

    v.push_back(40);  // capacity exceeded — reallocates the vector.

    // Iterator and reference to item are now invalid. Accessing them is undefined behavior.
    // Next two lines may print garbage or crash.
    std::printf("after  push_back: *it = %d  (UB)\n", *it);
    std::printf("after  push_back: ref = %d  (UB)\n", ref);
}
```
Output (typical, but behavior is undefined):
```
before push_back: *it = 20
before push_back: ref = 20
after  push_back: *it = 0  (UB)
after  push_back: ref = 0  (UB)
```
[Try it!](https://godbolt.org/z/fafvK81bz)

---

Example: erase while iterating

Erasing inside a range-based `for` loop invalidates its internal iterator:

```cpp
std::vector<int> v = {1, 2, 3, 4, 5};
for (auto x : v)
    if (x % 2 == 0)
        v.erase(std::find(v.begin(), v.end(), x));  // UB
```

A manual increment after `erase` is also wrong — it skips the element that shifted into the erased position:

```cpp
for (auto it = v.begin(); it != v.end(); ++it)   // bug: ++it skips an element after erase
    if (*it % 2 == 0)
        v.erase(it);  // UB on next iteration for std::vector
```

The correct idiom uses `erase`'s return value, which is the iterator to the next valid element:

```cpp
#include <cstdio>
#include <vector>

int main()
{
    std::vector<int> v = {1, 2, 3, 4, 5};

    for (auto it = v.begin(); it != v.end(); ) {
        if (*it % 2 == 0)
            it = v.erase(it);
        else
            ++it;
    }

    for (int x : v)
        std::printf("%d ", x);
}
```
Output:
```
1 3 5
```

This works uniformly across `std::vector`, `std::list`, `std::map`, and `std::set`. For node-based containers, `erase` only invalidates the erased element, so all remaining iterators (including the returned one) stay valid.

In C++20, prefer `std::erase_if` when removing all elements that match a predicate:

```cpp
std::erase_if(v, [](int x) { return x % 2 == 0; });
```
