[← C++ Field Guide](../README.md)

# Passkey

> Restrict a public constructor or function to specific callers by requiring a private token type that only those callers can construct.

---

## The Problem

`friend` declarations grant full access to all private members. When only one operation — construction, a factory call — needs to be restricted, `friend` is wider than necessary and makes the access grant invisible at the call site.

```cpp
class Session {
    explicit Session(int id) : id_(id) {}
    friend class ConnectionManager;  // grants access to all private members
    int id_;
};
```

Making the constructor public is worse — it grants access to everyone.

---

## The Pattern

Introduce a nested `Key` type. The protected function is public but requires a `Key` argument. Only classes that are `friend` of `Key` — not of the outer class — can construct a `Key` and therefore call the function.

```cpp
class Session {
public:
    class Key {
        friend class ConnectionManager;  // only ConnectionManager can create a Key
        Key() = default;
    };

    explicit Session(Key, int id) : id_(id) {}  // public, but Key is the gatekeeper

private:
    int id_;
};

class ConnectionManager {
public:
    Session open(int id) {
        return Session(Session::Key{}, id);  // Key{} is accessible here
    }
};

// Elsewhere:
Session s(Session::Key{}, 42);  // error: Key() is private
```

The `friend` declaration lives on `Key`, not on `Session`, so `ConnectionManager` gains no access to `Session`'s private members beyond what the public constructor exposes.

---

## Reusable Template Form

A single `Passkey<T>` template eliminates the per-class boilerplate:

```cpp
template<typename T>
class Passkey {
    friend T;
    Passkey() = default;
};

class Session {
public:
    explicit Session(Passkey<ConnectionManager>, int id) : id_(id) {}
private:
    int id_;
};

class ConnectionManager {
public:
    Session open(int id) {
        return Session(Passkey<ConnectionManager>{}, id);
    }
};
```

The template can be defined once in a shared header and reused across the codebase.

---

## Pros and Cons

**Pros**
- Access is surgical: only the named class can construct the `Key`, and all other private members of `Session` remain private.
- The access constraint is visible in the function signature — no hidden `friend` grant on the outer class.
- The template form is reusable without per-class boilerplate.

**Cons**
- Adds a dummy `Key` parameter to the constructor or function signature.
- Still requires a `friend` declaration — it is scoped to `Key` only, but is not eliminated.
- Less familiar than `friend class X`; readers need to trace the pattern to understand the access model.
