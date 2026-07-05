[← Idioms](README.md)

# RAII

> Bind resource lifetime to object lifetime — the constructor acquires, the destructor releases, guaranteeing cleanup on every exit path.

---

## The Problem

When a resource requires a paired acquire/release, every exit path must call the release manually. Early returns and exceptions make this easy to get wrong.

```cpp
void process(const char* path) {
    FILE* f = std::fopen(path, "r");
    if (!f) {
        return;
    }
    if (!validate(f)) {
        std::fclose(f);   // must remember on every exit
        return;
    }
    doWork(f); // can it throw?
    std::fclose(f);
}
```

Each new exit path (early return, exception) is another place to forget the release.

---

## RAII

Wrap the resource in an object that owns it. Acquisition happens in the constructor; release happens in the destructor, automatically, on every exit.

_Before:_
```cpp
void process(const char* path) {
    FILE* f = std::fopen(path, "r");
    if (!f) {
        return;
    }
    if (!validate(f)) {
        std::fclose(f);
        return;
    }
    doWork(f);
    std::fclose(f);
}
```

_After:_
```cpp
void process(const char* path) {
    std::ifstream f(path);
    if (!f) {
        return;
    }
    if (!validate(f)) {
        return;
    }
    doWork(f);
}   // closed automatically, regardless of exit path
```

The resource is *owned* by the wrapper. It cannot be leaked — scope exit is the only release path.

---

## Standard Library RAII Types

The standard library covers the most common resources:

| Resource | RAII type |
|---|---|
| Heap memory | `std::unique_ptr`, `std::shared_ptr` |
| File | `std::ifstream`, `std::ofstream`, `std::fstream` |
| Mutex | `std::lock_guard`, `std::unique_lock`, `std::scoped_lock` |

Prefer these over manual management. A raw `new`/`delete` or `fopen`/`fclose` pair is a code smell.

---

## Custom RAII Wrappers

For resources without a standard wrapper, write a small owning type. The key rules: acquire in the constructor, release in the destructor, disable copying.

```cpp
class FileHandle {
public:
    explicit FileHandle(const char* path)
      : f_(std::fopen(path, "r"))
    {}
    ~FileHandle() {
        if (f_) {
            std::fclose(f_);
        }
    }

    FileHandle(const FileHandle&)            = delete;
    FileHandle& operator=(const FileHandle&) = delete;

    explicit operator bool() const { return f_ != nullptr; }
    FILE* get() const { return f_; }

private:
    FILE* f_;
};
```

```cpp
void process(const char* path) {
    FileHandle f(path);
    if (!f) {
        return;
    }
    // use f.get() ...
}   // fclose called automatically
```

Move semantics (`= default` move constructor/assignment) can be added if ownership transfer is needed.

---

## Pros and Cons

**Pros**
- Cleanup is guaranteed on every exit path, including exceptions — correctness is structural.
- No release call to forget; adding a new exit point costs nothing.
- Composes naturally: multiple resources in one function each clean up independently.

**Cons**
- Non-standard resources require writing a custom owning wrapper class.
- `unique_ptr`-based wrappers heap-allocate, adding indirection compared to raw handles.
- Move semantics require careful attention to avoid double-release when wrappers are moved.
