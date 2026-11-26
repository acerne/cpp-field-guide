[← C++ Field Guide](../README.md)

# Integer Promotion in Arithmetic

Arithmetic operations on types narrower than `int` — `char`, `unsigned char`, `short`, `uint8_t`, etc. — silently promote both operands to `int` (or `unsigned int`) before the operation. The result has type `int`, not the original narrow type. This produces surprising values when the promoted result is then used in comparisons or bitwise operations.

```cpp
uint8_t a = 0xFF;
auto b = ~a;        // ~(int)0xFF = 0xFFFFFF00 — b is int, value is -256
uint8_t c = ~a;     // truncates to 0x00 = 0
```

`~a` looks like it should produce `0x00` (all bits flipped on a byte), but promotion widens `a` to `int` first, so the NOT applies to a 32-bit value and the result is `0xFFFFFF00`.

Comparisons can also surprise:

```cpp
auto diff2 = uint8_t(50) - uint8_t(100);  // int -50, not uint8_t 206
if (diff2 < 0)  // true: unsigned arithmetic gives negative int
```

Because `uint8_t` is promoted to `int` before subtraction, the result is a signed `int` and can be negative — even though `uint8_t` itself cannot.

When truncating back to `uint8_t` matters, cast explicitly:

```cpp
uint8_t result = static_cast<uint8_t>(~a);  // 0xFF as intended
```

---

Example: bitwise NOT on a byte yields a negative int
```cpp
#include <cstdio>
#include <cstdint>

int main()
{
    uint8_t a = 0xFF;

    auto b = ~a;                  // promoted to int before ~
    uint8_t c = ~a;               // truncated back to 8 bits

    std::printf("~uint8_t(0xFF) as auto:    %d  (type is int)\n", b);
    std::printf("~uint8_t(0xFF) as uint8_t: %u\n", static_cast<unsigned int>(c));

    uint8_t x = 50, y = 100;
    auto diff = x - y;            // int -50
    std::printf("uint8_t(50) - uint8_t(100) = %d  (not 206)\n", diff);
}
```
Output:
```
~uint8_t(0xFF) as auto:    -256  (type is int)
~uint8_t(0xFF) as uint8_t: 0
uint8_t(50) - uint8_t(100) = -50  (not 206)
```
[Try it!](https://godbolt.org/z/jKnjjY7x8)

The `-256` result (0xFFFFFF00 interpreted as signed int) and the `-50` subtraction (not the 206 wrap-around expected from 8-bit arithmetic) both stem from promotion before the operation.
