[← Pitfalls](README.md)

# Virtual Functions in Constructors and Destructors

During construction and destruction, virtual function calls do not dispatch to the most-derived override — they dispatch to the class being constructed or destroyed at that moment. The dynamic type of the object is not yet (or no longer) the fully-derived type.

```cpp
struct Base {
    Base() { init(); }           // calls Base::init, not Derived::init
    virtual void init() { std::puts("Base::init"); }
};

struct Derived : Base {
    void init() override { std::puts("Derived::init"); }
};

Derived d;  // prints "Base::init"
```

This applies symmetrically to destructors: by the time `~Base` runs, the `Derived` sub-object has already been destroyed, so `Base::~Base` calling a virtual function dispatches to `Base`'s version.

The reason is safety: the derived members do not exist yet during `Base::Base()`, so dispatching to `Derived::init` could access uninitialized data.

The fix is to not call virtual functions from constructors or destructors. Instead, require callers to invoke the initialization step explicitly, or use a factory function:

```cpp
struct Base {
    static std::unique_ptr<Base> make();
    virtual void init() = 0;
};

auto obj = Base::make();
obj->init();  // now fully constructed — dispatches correctly
```

---

Example: virtual call during construction goes to the base
```cpp
#include <cstdio>

struct Logger {
    Logger()          { log("Logger constructed"); }
    virtual ~Logger() { log("Logger destroyed"); }

    virtual void log(const char* msg) { std::printf("[BASE] %s\n", msg); }
};

struct FileLogger : Logger {
    void log(const char* msg) override { std::printf("[file] %s\n", msg); }
};

int main()
{
    FileLogger fl;  // constructs Logger first, then FileLogger
    fl.log("manual call");

    return 0;
}
```
Output:
```
[BASE] Logger constructed
[file] manual call
[BASE] Logger destroyed
```
[Try it!](https://godbolt.org/z/r593hTorz)