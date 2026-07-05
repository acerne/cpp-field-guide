[← Idioms](README.md)

# Non-Virtual Interface

> Keep the public interface non-virtual and delegate to private virtual functions — the base class controls the call protocol while derived classes customize only the behavior.

---

## The Problem

When a public member function is virtual, derived classes override the entire function body. Any pre/post-conditions, logging, or invariant checks the base class adds can be silently bypassed.

```cpp
struct Logger {
    virtual void log(std::string_view msg) {
        std::cout << timestamp() << " " << msg << "\n";
    }
};

struct SilentLogger : Logger {
    void log(std::string_view msg) override {
        file_ << msg << "\n";  // timestamp always skipped
    }
    std::ofstream file_;
};
```

The base class intended `timestamp()` to always run; the override skips it with no warning.

---

## The Pattern

Make the public function non-virtual. Have it call a private virtual hook that derived classes override.

```cpp
struct Logger {
    void log(std::string_view msg) {         // public, non-virtual
        doLog(formatMessage(msg));           // formatMessage always runs
    }

private:
    virtual void doLog(std::string_view msg) {  // private, virtual
        std::cout << msg << "\n";
    }

    std::string formatMessage(std::string_view msg) const {
        return std::string(timestamp()) + " " + std::string(msg);
    }
};

struct FileLogger : Logger {
private:
    void doLog(std::string_view msg) override {
        file_ << msg << "\n";
    }
    std::ofstream file_;
};
```

`FileLogger` cannot skip `formatMessage` — the base class controls when it runs.

---

## Pure Virtual NVI

NVI works equally well for abstract interfaces. The public function states the contract; the pure virtual hook defines what derived classes must supply:

```cpp
struct Serializer {
    std::string serialize(const Data& d) {
        validate(d);              // always enforced, cannot be bypassed
        return doSerialize(d);
    }

private:
    virtual std::string doSerialize(const Data& d) = 0;

    void validate(const Data& d) { /* ... */ }
};
```

---

## Pros and Cons

**Pros**
- Pre/post-conditions and invariants in the public function cannot be bypassed by derived classes.
- The public interface and override interface are decoupled — the base can change the call sequence without touching derived classes.
- The `doX` naming convention signals to readers that the function is an implementation hook, not part of the external API.

**Cons**
- Adds one extra indirection: public calls private calls override.
- Derived classes cannot call `Base::doLog()` directly (it is private), which is intentional but occasionally limiting.
- Readers unfamiliar with NVI may find a public function delegating to a private virtual unexpected.
