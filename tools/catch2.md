[← Tools](README.md)

# Catch2

> Expressive C++ testing framework with natural-language test names, hierarchical sections, and BDD-style macros. An alternative to GoogleTest.

[Documentation](https://github.com/catchorg/Catch2/blob/devel/docs/Readme.md)

## Setup

Add Catch2 as a CMake dependency via FetchContent and link against `Catch2::Catch2WithMain`.

```cmake
include(FetchContent)
FetchContent_Declare(
    Catch2
    GIT_REPOSITORY https://github.com/catchorg/Catch2.git
    GIT_TAG        v3.5.0
)
FetchContent_MakeAvailable(Catch2)

add_executable(my_tests test_foo.cpp)
target_link_libraries(my_tests Catch2::Catch2WithMain)
```

`Catch2WithMain` provides `main()`. Use `Catch2::Catch2` and supply your own if you need custom startup.

---

## Usage

Tests use `TEST_CASE` with a descriptive name and optional filter tags; `REQUIRE` stops on failure, `CHECK` continues.

### Basic tests

```cpp
#include <catch2/catch_test_macros.hpp>

TEST_CASE("vector can be sized and resized", "[vector]") {
    std::vector<int> v{1, 2, 3};
    REQUIRE(v.size() == 3);
    CHECK(v.capacity() >= 3);
}
```

`REQUIRE` stops the test on failure. `CHECK` records a failure and continues — mirrors `ASSERT_*` vs `EXPECT_*` in GoogleTest.

Tags in `"[brackets]"` are used for filtering at runtime; a test case can have multiple.

### Sections

Each `SECTION` re-runs the test case from the top, giving each section a clean starting state:

```cpp
TEST_CASE("vector operations", "[vector]") {
    std::vector<int> v{1, 2, 3};   // constructed fresh for each section

    SECTION("push_back increases size") {
        v.push_back(4);
        REQUIRE(v.size() == 4);
    }

    SECTION("pop_back decreases size") {
        v.pop_back();
        REQUIRE(v.size() == 2);
    }
}
```

### BDD style

```cpp
#include <catch2/catch_test_macros.hpp>

SCENARIO("vector grows as elements are added", "[vector]") {
    GIVEN("an empty vector") {
        std::vector<int> v;
        WHEN("an element is pushed") {
            v.push_back(1);
            THEN("size is 1") { REQUIRE(v.size() == 1); }
        }
    }
}
```

`SCENARIO`/`GIVEN`/`WHEN`/`THEN` are aliases for `TEST_CASE`/`SECTION` — identical semantics, different labels.

### Matchers

```cpp
#include <catch2/matchers/catch_matchers_string.hpp>
#include <catch2/matchers/catch_matchers_vector.hpp>

using namespace Catch::Matchers;

REQUIRE_THAT(str, ContainsSubstring("foo"));
REQUIRE_THAT(vec, Contains(42));
REQUIRE_THAT(val, WithinAbs(3.14, 0.01));
```

---

## Running Tests

Run the test executable directly; filter by name substring or tag, or export results for CI reporting.

Run all tests:

```bash
./my_tests
```

Run by test name (substring match):

```bash
./my_tests "vector can be sized"
```

Run by tag:

```bash
./my_tests [vector]
./my_tests "[vector][!slow]"   # tag vector AND NOT slow
```

List all tests and tags without running:

```bash
./my_tests --list-tests
./my_tests --list-tags
```

Verbose output:

```bash
./my_tests -v
```

Output as JUnit XML for CI reporting:

```bash
./my_tests -r junit -o results.xml
```
