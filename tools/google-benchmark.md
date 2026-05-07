[← Tools](README.md)

# Google Benchmark

> Microbenchmarking library for measuring C++ code performance with statistical output and parameterization support.

[Documentation](https://github.com/google/benchmark)

## Setup

Add Google Benchmark as a CMake dependency via FetchContent and link against `benchmark::benchmark_main`.

```cmake
include(FetchContent)
FetchContent_Declare(
    benchmark
    GIT_REPOSITORY https://github.com/google/benchmark.git
    GIT_TAG        v1.8.3
)
set(BENCHMARK_ENABLE_TESTING OFF)
FetchContent_MakeAvailable(benchmark)

add_executable(my_bench bench_foo.cpp)
target_link_libraries(my_bench benchmark::benchmark_main)
```

Build in **Release** mode — benchmark numbers from a Debug build are not meaningful.

---

## Usage

Benchmarks follow a state-loop pattern; only code inside the `for (auto _ : state)` loop is measured.

### Basic benchmark

```cpp
#include <benchmark/benchmark.h>

static void BM_Sort(benchmark::State& state) {
    std::vector<int> v(1000);
    std::iota(v.begin(), v.end(), 0);
    std::mt19937 rng{std::random_device{}()};
    for (auto _ : state) {
        std::shuffle(v.begin(), v.end(), rng);
        std::sort(v.begin(), v.end());
    }
}
BENCHMARK(BM_Sort);
```

Code inside the loop is measured; setup outside it is not. The framework runs the loop as many times as needed to collect a stable estimate.

### Preventing dead-code elimination

The optimizer may remove computations whose result is never observed. Wrap the result:

```cpp
for (auto _ : state) {
    auto result = compute(x);
    benchmark::DoNotOptimize(result);  // result is treated as an observable output
}
```

`benchmark::ClobberMemory()` forces all pending memory writes to be visible — useful after in-place operations:

```cpp
for (auto _ : state) {
    std::fill(v.begin(), v.end(), 0);
    benchmark::ClobberMemory();
}
```

### Parameterized benchmarks

```cpp
static void BM_Sort(benchmark::State& state) {
    std::vector<int> v(state.range(0));
    for (auto _ : state) { /* ... */ }
}
BENCHMARK(BM_Sort)->Arg(64)->Arg(512)->Arg(4096);
BENCHMARK(BM_Sort)->Range(8, 8 << 10);  // powers of two: 8, 16, ..., 8192
```

`state.range(0)` reads the first argument. Multiple arguments use `state.range(1)`, etc.

### Reading output

```
------------------------------------------------------------
Benchmark                  Time          CPU   Iterations
------------------------------------------------------------
BM_Sort/64                123 ns        122 ns     5721034
BM_Sort/512              1034 ns       1030 ns      678432
BM_Sort/4096             9821 ns       9800 ns       71203
```

- **Time**: wall-clock time per iteration
- **CPU**: CPU time per iteration, excluding time the process was descheduled
- **Iterations**: loop count; higher means a more stable estimate

---

## Running Benchmarks

Run the benchmark executable directly; flags control filtering, repetition, time units, and output format.

Run all benchmarks:

```bash
./my_bench
```

Run a specific benchmark by name (regex):

```bash
./my_bench --benchmark_filter=BM_Sort
```

Run with repeated measurements for stable results:

```bash
./my_bench --benchmark_repetitions=10
```

Change time unit:

```bash
./my_bench --benchmark_time_unit=us   # ns (default), us, ms, s
```

Output as JSON for scripted comparison:

```bash
./my_bench --benchmark_format=json --benchmark_out=results.json
```

Compare two result files using the comparison tool from the benchmark repo:

```bash
python3 tools/compare.py benchmarks results_before.json results_after.json
```
