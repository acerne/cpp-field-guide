[← Idioms](README.md)

# Tag Dispatch

> Select a function overload at compile time by passing a lightweight tag type derived from a trait, with no runtime cost.

---

## The Pattern

A tag is an empty struct whose *type* carries the selection information. The dispatcher constructs a tag from a trait and passes it as a dummy argument; overload resolution picks the right implementation.

```cpp
struct slow_path_tag {};
struct fast_path_tag {};

template<typename Iter>
void advance_impl(Iter& it, int n, slow_path_tag) {
    while (n--) { ++it; }
}

template<typename Iter>
void advance_impl(Iter& it, int n, fast_path_tag) {
    it += n;
}

template<typename Iter>
void advance(Iter& it, int n) {
    using Tag = std::conditional_t<
        std::is_base_of_v<std::random_access_iterator_tag,
                          typename std::iterator_traits<Iter>::iterator_category>,
        fast_path_tag,
        slow_path_tag>;
    advance_impl(it, n, Tag{});
}
```

The standard library uses exactly this mechanism internally for `std::advance`, `std::distance`, and algorithm overloads.

---

## vs `if constexpr`

C++17 `if constexpr` can replace tag dispatch for binary or linear branches:

```cpp
template<typename Iter>
void advance(Iter& it, int n) {
    if constexpr (std::is_base_of_v<std::random_access_iterator_tag,
                                    typename std::iterator_traits<Iter>::iterator_category>) {
        it += n;
    } else {
        while (n--) { ++it; }
    }
}
```

Prefer `if constexpr` for self-contained functions with a fixed set of branches. Tag dispatch is better when:

- Third parties need to add new tags without modifying existing code (open for extension).
- The number of specialisations is large enough that a single function body would be hard to read.
- The tag hierarchy carries semantic meaning (iterator category tags form an inheritance tree that encodes capability).

---

## Pros and Cons

**Pros**
- Open for extension: new tags slot in without modifying existing dispatch code.
- Zero runtime cost; selection is purely compile-time overload resolution.
- Tag hierarchies encode capability relationships in a way both the compiler and reader can follow.

**Cons**
- More verbose than `if constexpr` for simple two-branch cases.
- Tag arguments appear in function signatures, complicating overload sets and documentation.
- Mismatch errors surface as template instantiation failures, which are often hard to read.
