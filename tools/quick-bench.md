[← C++ Field Guide](../README.md)

# Quick-Bench

> Browser-based C++ microbenchmark tool built on Google Benchmark; compares the relative performance of code variants without local setup.

[quick-bench.com](https://quick-bench.com)

---

## Writing Benchmarks

Each benchmark is a function registered with `BENCHMARK`. The body runs the measured code inside the `state` loop:

```cpp
static void BM_StringCopy(benchmark::State& state) {
    std::string src = "hello world";
    for (auto _ : state) {
        std::string copy = src;
        benchmark::DoNotOptimize(copy);
    }
}
BENCHMARK(BM_StringCopy);
```

`benchmark::DoNotOptimize(expr)` prevents the optimizer from eliminating an expression whose result is otherwise unused. `benchmark::ClobberMemory()` forces pending writes to be visible to subsequent code.

---

## Comparing Variants

Define multiple `BENCHMARK`-registered functions in the same editor to compare them. Quick-Bench runs all of them and plots their wall-clock time as a bar chart, normalized to the slowest variant.

---

## Compiler and Flags

Select the compiler (GCC or Clang), C++ standard, and optimization level from the dropdowns above the editor. Always benchmark at `-O2` or `-O3` — measuring at `-O0` reflects the absence of optimization, not the relative cost of the actual code.

---

## Sharing

The **Share** button produces a stable link encoding the benchmark source, compiler, and flags.
