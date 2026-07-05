[← Pitfalls](README.md)

# Unchecked Access Is Undefined Behavior

The standard library provides two accessor styles for many types: a checked form that throws on invalid access, and an unchecked form that is undefined behavior if its precondition is violated. Using the unchecked form without first meeting that precondition is UB — potentially silent in release builds and exploitable by the optimizer.

---

## `std::vector`

`operator[]` is UB if the index is out of range. `at()` performs the same access but throws `std::out_of_range` instead.

```cpp
std::vector<int> v = {1, 2, 3};
v[5];      // UB
v.at(5);   // throws std::out_of_range
```

Once the precondition is verified, use the unchecked form — calling `at()` after a size check performs the check twice:

```cpp
// UB if i is out of range
process(v[i]);

// Pre-check verifies index
if (i >= 0 && i < v.size()) {
    process(v.at(i));  // redundant check
    process(v[i]);  // no checks
}

// For loop limits index
for(int i = 0; i < v.size(); ++i) {
    process(v.at(i));  // redundant check
    process(v[i]);  // no checks
}
```

---


## `std::optional` and `std::expected`

`operator*` is UB on an empty `optional` (or on an `expected` holding an error). `value()` throws `std::bad_optional_access` (or `std::bad_expected_access<E>`) instead.

```cpp
std::optional<int> opt = maybe_get();
*opt;          // UB if opt is empty
opt.value();   // throws std::bad_optional_access if empty
```

Check first with `has_value()` or an `if` test, then dereference with `operator*`:

```cpp
// UB if opt is empty
process(*opt);

// redundant check
if (opt.has_value()) {
    process(opt.value());
}

// correct
if (opt.has_value()) {
    process(*opt);
}
```
