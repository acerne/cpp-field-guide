[← C++ Field Guide](../README.md)

# GoogleTest

> Google's C++ testing framework — xUnit-style tests with a rich assertion library, fixtures, parameterized cases, and matchers.

[Documentation](https://google.github.io/googletest/)

## Setup

Add GoogleTest as a CMake dependency via FetchContent and link the test executable against `GTest::gtest_main`.

```cmake
include(FetchContent)
FetchContent_Declare(
    googletest
    URL https://github.com/google/googletest/archive/refs/tags/v1.14.0.tar.gz
)
FetchContent_MakeAvailable(googletest)

add_executable(my_tests test_foo.cpp)
target_link_libraries(my_tests GTest::gtest_main)
```

`GTest::gtest_main` provides `main()`. Link against `GTest::gtest` and supply your own `main()` if you need custom global setup.

---

## Usage

Tests are defined with `TEST` or `TEST_F` macros and use `EXPECT_*` or `ASSERT_*` assertion families.

### Basic tests

```cpp
#include <gtest/gtest.h>

TEST(MathTest, AddsTwoNumbers) {
    EXPECT_EQ(2 + 2, 4);
}
```

`EXPECT_*` records a failure and continues. `ASSERT_*` stops the current test immediately.

Common assertions:

```cpp
EXPECT_EQ(a, b);           // a == b
EXPECT_NE(a, b);           // a != b
EXPECT_LT(a, b);           // a < b
EXPECT_TRUE(expr);
EXPECT_FALSE(expr);
EXPECT_NEAR(a, b, delta);  // |a - b| <= delta
EXPECT_THROW(expr, ExType);
EXPECT_NO_THROW(expr);
```

### Fixtures

Share setup and teardown across tests in a suite:

```cpp
class QueueTest : public ::testing::Test {
protected:
    void SetUp() override { q_.push(1); q_.push(2); }
    std::queue<int> q_;
};

TEST_F(QueueTest, PopReducesSize) {
    q_.pop();
    EXPECT_EQ(q_.size(), 1u);
}
```

`SetUp` and `TearDown` run before and after each `TEST_F` independently.

### Matchers

`EXPECT_THAT` pairs with matchers from `<gmock/gmock.h>`:

```cpp
#include <gmock/gmock.h>

EXPECT_THAT(vec, ::testing::ElementsAre(1, 2, 3));
EXPECT_THAT(str, ::testing::HasSubstr("foo"));
EXPECT_THAT(val, ::testing::AllOf(::testing::Gt(0), ::testing::Lt(100)));
```

### Parameterized tests

```cpp
class EvenTest : public ::testing::TestWithParam<int> {};

TEST_P(EvenTest, IsEven) {
    EXPECT_EQ(GetParam() % 2, 0);
}

INSTANTIATE_TEST_SUITE_P(EvenNumbers, EvenTest,
    ::testing::Values(2, 4, 6, 8));
```

### Death tests

```cpp
EXPECT_DEATH(foo(-1), ".*negative.*");  // process must abort; message matches regex
```

---

## Running Tests

Build the test executable and run it directly; flags control filtering, listing, and repetition.

Run all tests:

```bash
./my_tests
```

Run tests matching a filter (`*` wildcard, `Suite.Test` format):

```bash
./my_tests --gtest_filter=MathTest.*
./my_tests --gtest_filter=*Add*
```

List all tests without running:

```bash
./my_tests --gtest_list_tests
```

Repeat tests — useful for detecting flakiness:

```bash
./my_tests --gtest_repeat=20 --gtest_filter=FlakyTest.*
```
