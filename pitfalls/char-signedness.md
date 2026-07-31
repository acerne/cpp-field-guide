[← C++ Field Guide](../README.md)

# `char` Signedness

> Plain `char` is a distinct type from both `signed char` and `unsigned char`, and whether it behaves as signed or unsigned is implementation-defined.

Most x86/x64 compilers make plain `char` signed; ARM and PowerPC toolchains commonly default it to unsigned. Code that assumes one or the other silently changes behavior when ported or recompiled for a different target.

```cpp
char raw = static_cast<char>(0x80);
int widened = raw;   // sign-extends only if plain char is signed
```

On a platform where `char` is signed, `widened` is `-128`. Where `char` is unsigned, it's `128`. Same source, same value, different result — because the standard leaves the signedness of `char` up to the implementation ([`std::is_signed`](https://en.cppreference.com/w/cpp/types/is_signed)).

---

## Use fixed-width integer types for byte data

When the intent is "a byte" or "an 8-bit integer," use `std::uint8_t` or `std::int8_t` instead of `char`. Their signedness is fixed by the type itself, not by the compiler.

```cpp
std::uint8_t raw = 0x80;
int widened = raw;   // always 128, regardless of platform
```

## Use `char` only for text, and spell out signedness when it matters

The only legitimate use of plain `char` is to hold a text character, since that's the type string literals, `std::string`, and the iostream/string APIs are built around. Even then, if code inspects the numeric value — comparisons, arithmetic, passing to functions like [`std::isalpha`](https://en.cppreference.com/w/cpp/string/byte/isalpha) that require a non-negative value representable as `unsigned char` — cast or declare explicitly as `signed char` or `unsigned char` rather than relying on the platform default.

```cpp
char c = getchar_from_somewhere();
if (std::isalpha(static_cast<unsigned char>(c)))  // avoid UB for negative char values
{
    // ...
}
```

