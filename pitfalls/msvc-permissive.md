[← C++ Field Guide](../README.md)

# MSVC `/permissive-`

> MSVC defaults to non-conformant mode; always enable `/permissive-` to compile standard-compliant C++.

---

## The naming trap

The trailing `-` is MSVC's negation suffix. The flag reads as "not-permissive" — stricter, not more lenient. Without it, MSVC runs in permissive mode by default, silently accepting non-standard code.

Documentation: https://learn.microsoft.com/en-us/cpp/build/reference/permissive-standards-conformance

---

## How to enable

### Command line

```
cl /permissive- source.cpp
```

### CMake

```cmake
target_compile_options(my_target PRIVATE /permissive-)
```

### Visual Studio

**Project Properties → C/C++ → Language → Conformance mode → Yes (`/permissive-`)**

New projects in Visual Studio 2019 and later default to conformance mode. Older project files do not and must be updated manually.

