[← Pitfalls](README.md)

# `std::vector<bool>` Is Not a Container

`std::vector<bool>` is a space-optimized specialization that packs bits instead of storing one `bool` per byte. Its `operator[]` and iterators return a **proxy reference**, not a `bool&`. This breaks generic code, `auto`, address-taking, and range-based `for` with references.

```cpp
std::vector<bool> v = {true, false, true};

bool& b = v[0];   // error: no bool& to bind to
auto  x = v[0];   // x is a proxy object, not a bool
bool* p = &v[0];  // error: cannot take address of proxy
```

The proxy behaves like `bool` for reads and assignments, but silently differs in any context that requires a real `bool` lvalue:

```cpp
void flip(bool& b) { b = !b; }
flip(v[0]);  // error: proxy does not convert to bool&
```

Generic code that works for `vector<int>` silently fails or errors for `vector<bool>`:

```cpp
template<typename T>
void zero_first(std::vector<T>& v) {
    T& ref = v[0];  // compiles for int, fails for bool
}
```

**Alternatives:**
- `std::vector<uint8_t>` — stores one byte per element, no proxy.
- `std::deque<bool>` — not specialized; returns real `bool&`.
- `std::bitset<N>` — fixed-size bit array with a similar proxy but documented explicitly.

**Thread safety:** `vector<bool>` is not safe for concurrent writes to separate elements. Bits are packed into words; writing element `i` requires a read-modify-write on the underlying word. Two threads writing to different elements that share the same word have a data race. All other `vector<T>` specializations guarantee that concurrent writes to different elements are safe, because each element occupies a distinct memory location.

```cpp
std::vector<bool> v(64, false);

std::thread t1([&]{ v[0] = true; });
std::thread t2([&]{ v[1] = true; });  // data race: v[0] and v[1] share a word
t1.join(); t2.join();

std::vector<int> vi(64, 0);
std::thread t3([&]{ vi[0] = 1; });
std::thread t4([&]{ vi[1] = 1; });  // safe: different memory locations
t3.join(); t4.join();
```

---

Example: `auto` capturing a proxy causes surprising lifetime behavior
```cpp
#include <cstdio>
#include <vector>

template <typename T>
void test_flip(std::vector<T> vec){
    // if int, x will hold value
    // if bool, x will hold proxy to value
    auto x = vec[0];
    std::printf(" x before flip: %d\n", bool(x));
    // Modifying the source vector
    vec[0] = T(vec[0] == 0); 
    // x will reflect the change for bool vector
    std::printf(" x after flip:  %d\n", bool(x));
}

int main()
{
    std::printf("std::vector<int>:\n");
    test_flip(std::vector<int>{1, 0, 1});

    std::printf("\n");
    std::printf("std::vector<bool>:\n");
    test_flip(std::vector<bool>{true, false, true});

    return 0;
}
```
Output:
```
std::vector<int>:
 x before flip: 1
 x after flip:  1

std::vector<bool>:
 x before flip: 1
 x after flip:  0
```
[Try it!](https://godbolt.org/z/azsza6nqv)