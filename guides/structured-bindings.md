[← Guides](README.md)

# Structured Bindings

_C++17_

> Bind names to the members of a struct, pair, tuple, or array in a single declaration.

---

## Syntax

A structured binding declares one name per element. The qualifier controls whether the binding copies, references, or const-references the source.

```cpp
auto  [x, y] = point;          // copy
auto& [x, y] = point;          // reference — writes go back to point
const auto& [x, y] = point;    // read-only reference
```

The number of names must match the number of elements exactly.

See [structured bindings on cppreference](https://en.cppreference.com/w/cpp/language/structured_binding).

---

## Structs and Arrays

Any struct or class with only public data members can be decomposed. Bindings are assigned in declaration order.

```cpp
struct Point { int x; int y; };

Point p{3, 7};
auto [x, y] = p;   // x == 3, y == 7
```

Plain arrays work the same way:

```cpp
int arr[3] = {1, 2, 3};
auto [a, b, c] = arr;
```

---

## Pairs and Tuples

`std::pair` and `std::tuple` — and any type that provides `std::tuple_size` and `get<N>` — are supported.

```cpp
std::pair<std::string, int> entry{"apple", 5};
auto& [name, count] = entry;   // name == "apple", count == 5
```

```cpp
auto t = std::make_tuple(1, 3.14, std::string{"hi"});
auto [i, d, s] = t;
```

---

## In Range-Based For Loops

The most common use is iterating over maps and ranges of pairs, avoiding `.first` / `.second`.

_Before:_
```cpp
for (const auto& kv : scores) {
    std::cout << kv.first << ": " << kv.second << "\n";
}
```

_After:_
```cpp
for (const auto& [name, score] : scores) {
    std::cout << name << ": " << score << "\n";
}
```

Use `auto&` to modify values in place:

```cpp
for (auto& [key, value] : scores) {
    value *= 2;
}
```
