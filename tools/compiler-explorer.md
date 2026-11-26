[← C++ Field Guide](../README.md)

# Compiler Explorer

> Browser-based tool that compiles C++ and displays annotated assembly output; supports multiple compilers and side-by-side comparisons.

[godbolt.org](https://godbolt.org)

---

## Compilers

Compiler Explorer supports GCC, Clang, and MSVC across many versions. Multiple compiler panes can be open simultaneously for comparing output between compiler families or optimization levels.

Select the compiler and flags in the dropdown at the top of each pane. Useful flag pairs to compare:

| Flags | Effect |
|-------|--------|
| `-O0` | No optimization — closest to source |
| `-O2` | Typical release build |
| `-O3` | Aggressive optimization |
| `-Os` | Optimize for size |

The assembly pane color-codes each instruction to the source line that produced it, making it straightforward to trace what the compiler did with a specific expression.

---

## Execution

The **Execute the code** button runs the compiled binary and shows stdout. Useful when assembly output alone is not enough to verify behavior.

---

## Libraries

The **Add library** button makes many popular C++ libraries available (Boost, {fmt}, range-v3, Abseil, and others) without any local setup.

---

## Sharing

The **Share** button produces a stable short link encoding the full editor state: source, compiler, flags, and libraries. Use these links in bug reports, documentation, or questions instead of pasting raw code.
