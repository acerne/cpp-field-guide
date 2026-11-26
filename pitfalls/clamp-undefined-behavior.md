[← C++ Field Guide](../README.md)

# std::clamp hi < lo Is Undefined Behavior

Standard function `std::clamp` contains no validity check to assert that `hi >= lo`. Cases where `lo` is greater than `hi` are **undefined behavior**.

```cpp
x = std::clamp(x, 5, 1); // UB
```

Not an issue when limiting to compile-time constants, but can be an issue if bounds are obtained at runtime. If invalid bounds are possible, the should be pre-validated before `std::clamp`.

```cpp
int clamp(int value, int lo, int hi) {
    if (hi < lo) {
        throw; // Handle it
    }
    return std::clamp(value, lo, hi);
}
```