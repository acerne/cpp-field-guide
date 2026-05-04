[← Idioms](README.md)

# Immediately Invoked Lambda

> Initialize a `const` variable with complex logic in place — the lambda provides the initialization body, and `()` runs it immediately.

---

## The Problem

`const` variables must be initialized in a single expression. Complex initialization that requires conditionals, loops, or error handling forces a choice: drop `const`, or extract a named helper function far from the call site.

```cpp
int value;  // cannot be const — assigned below
switch (mode) {
    case Mode::A: value = compute_a(); break;
    case Mode::B: value = compute_b(); break;
    default:      value = 0;
}
// value is mutable but never reassigned — intent is lost
```

---

## The Pattern

Wrap the initialization logic in a lambda and invoke it immediately with `()`. The result is a single `const`-initialized expression with no helper function needed.

_Before:_
```cpp
int value;
switch (mode) {
    case Mode::A: value = compute_a(); break;
    case Mode::B: value = compute_b(); break;
    default:      value = 0;
}
```

_After:_
```cpp
const int value = [&] {
    switch (mode) {
        case Mode::A: return compute_a();
        case Mode::B: return compute_b();
        default:      return 0;
    }
}();
```

The compiler inlines trivial lambdas; there is no runtime overhead from the lambda wrapper.

---

## Common Uses

**Initializing a `const` member in a constructor initializer list:**

```cpp
struct Config {
    const std::string path;

    Config(const Options& opts)
        : path([&] {
            if (opts.override_path) {
                return opts.override_path.value();
            }
            return default_path(opts.base_dir);
          }())
    {}
};
```

**Initializing from a `try`/`catch` block:**

```cpp
const auto data = [&] {
    try {
        return load_file(path);
    } catch (const std::ios_base::failure&) {
        return default_data();
    }
}();
```

---

## Pros and Cons

**Pros**
- Enables `const` on variables that require non-trivial initialization.
- Keeps initialization logic inline, at the point of use — no need for a distant named helper.
- The compiler inlines the call; no runtime overhead.

**Cons**
- Unfamiliar to readers who have not seen the pattern; the trailing `()` can be easy to miss.
- A long or deeply nested lambda body is hard to read — at that point, a named helper is clearer.
