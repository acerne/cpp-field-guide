[← C++ Field Guide](../README.md)

# Early Return

> Use guard clauses that return immediately on failure, keeping the happy path at the top indentation level.

---

## The Problem

When a function must validate several preconditions before doing real work, the naive approach nests each check inside the previous one. The actual work ends up buried several levels deep, and the failure paths are scattered at the closing braces of each `if`.

```cpp
bool process(const Request& req) {
    if (req.isValid()) {
        if (req.hasPermission()) {
            if (req.hasData()) {
                auto result = doWork(req.data());
                return result.ok();
            }
        }
    }
    return false;
}
```

Adding more preconditions pushes the happy path further right. The reader must track matching braces to understand which `return false` belongs to which condition.

---

## Early Return

Invert each guard and return immediately on failure. The happy path stays at the top indentation level and reads top-to-bottom without interruption.

_Before:_
```cpp
bool process(const Request& req) {
    if (req.isValid()) {
        if (req.hasPermission()) {
            if (req.hasData()) {
                auto result = doWork(req.data());
                return result.ok();
            }
        }
    }
    return false;
}
```

_After:_
```cpp
bool process(const Request& req) {
    if (!req.isValid()) {
        return false;
    }
    if (!req.hasPermission()) {
        return false;
    }
    if (!req.hasData()) {
        return false;
    }

    auto result = doWork(req.data());
    return result.ok();
}
```

Each guard is now a one-liner. Adding a new precondition costs exactly one line and does not affect indentation of anything below it.

---

## With Logging

The readability difference becomes especially clear when log calls are added. In the nested version, each log message is separated from its corresponding `return` by braces and indentation levels:

```cpp
bool process(const Request& req) {
    if (req.isValid()) {
        if (req.hasPermission()) {
            if (req.hasData()) {
                log("processing");
                auto result = doWork(req.data());
                if (!result.ok())
                {
                    log("work failed");
                    return false;
                }
                log("done");
                return true;
            }
            else {
                log("no data");
                return false;
            }
        }
        else {
            log("no permission");
            return false;
        }
    }
    else {
        log("invalid");
        return false;
    }
}
```

With early return, each guard, its log, and its `return` are collocated on one line. The sequence of log messages the caller will see mirrors the top-to-bottom structure of the code:

```cpp
bool process(const Request& req) {
    if (!req.isValid()) {
        log("invalid");
        return false;
    }
    if (!req.hasPermission()) {
        log("no permission");
        return false;
    }
    if (!req.hasData()) {
        log("no data");
        return false;
    }

    log("processing");
    auto result = doWork(req.data());
    if (!result.ok()) {
        log("work failed");
        return false;
    }
    log("done");
    return true;
}
```

---

## Pros and Cons

**Pros**
- Reduces nesting; the happy path reads top-to-bottom at a single indentation level.
- Each guard and its error handling are collocated — easy to scan and modify independently.
- New preconditions cost one line and do not shift indentation of anything below (smaller git diff).

**Cons**
- Multiple exit points make manual cleanup error-prone; pair with [RAII](raii.md) or [scope guards](scope-guard.md).
- Some coding standards (MISRA, safety-critical guidelines) require a single exit point per function.
