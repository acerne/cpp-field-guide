[← C++ Field Guide](../README.md)

# Scope Guard

> Execute cleanup code unconditionally when a scope exits — regardless of when or how the resource was acquired.

---

## The Pattern

A scope guard differs from [RAII](raii.md) in one key way: the resource is not necessarily acquired at object construction. The guard's only job is to ensure a cleanup action runs when the enclosing scope exits, whether normally, via early return, or via exception.

The standard library covers the most common case with `std::lock_guard` — the mutex already exists, the guard just ensures it gets unlocked:

_Before:_
```cpp
void update(std::mutex& m, Data& d) {
    m.lock();
    if (!d.isReady()) {
        m.unlock();   // cleanup on every exit path
        return;
    }
    d.value = compute(); // can it throw?
    m.unlock();
}
```

_After:_
```cpp
void update(std::mutex& m, Data& d) {
    std::lock_guard<std::mutex> lock(m);
    if (!d.isReady()) {
        return;
    }
    d.value = compute();
}   // unlocked automatically, regardless of exit path
```

---

## Generic Scope Guard

For paired C API calls or any cleanup without a dedicated stdlib type, a `finally` helper captures an arbitrary cleanup action:

```cpp
template<typename F>
class ScopeGuard {
public:
    explicit ScopeGuard(F f) : f_(std::move(f)) {}
    ~ScopeGuard() { f_(); }
    ScopeGuard(const ScopeGuard&) = delete;
    ScopeGuard& operator=(const ScopeGuard&) = delete;
private:
    F f_;
};

template<typename F>
ScopeGuard<F> finally(F f) { return ScopeGuard<F>(std::move(f)); }
```

```cpp
void process(Context* ctx) {
    beginBatch(ctx);
    auto guard = finally([ctx] { endBatch(ctx); });

    for (auto& item : items) {
        processItem(item, ctx);   // may throw or return early
    }
}   // endBatch called automatically
```

---

## Scope Guard vs RAII Wrapper

Prefer a full RAII wrapper (or a standard type like `unique_ptr` / `fstream`) when the resource has clear ownership semantics. Reach for a scope guard when:

- The resource is acquired by someone else and passed in (mutex, connection, context).
- The cleanup is a one-off that does not warrant its own class.
- The paired API does not model ownership (`beginBatch`/`endBatch`, `push`/`pop`).

---

## Pros and Cons

**Pros**
- Guarantees cleanup on all exit paths without writing a new class.
- Captures cleanup inline, at the acquisition site, keeping cause and effect together.
- Works for any paired calls that do not model ownership.

**Cons**
- Less reusable than a named RAII type — the cleanup lambda is ad hoc and not composable.
- The cleanup action always fires; cancellation requires adding a `dismiss()` mechanism.
- Capturing variables by reference requires care to avoid dangling references.
