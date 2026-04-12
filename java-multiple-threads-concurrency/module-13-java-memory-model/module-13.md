# Module 13 — Java Memory Model (Deep Dive)

**Level:** Advanced  
**Goal:** Understand the formal rules the JVM and CPU follow for memory operations, and apply them to write correctly synchronized code.

---

## Theory

### 13.1 Why the JVM Reorders Operations

Modern CPUs and the JIT compiler aggressively **reorder instructions** for performance:
- CPU out-of-order execution
- Compiler instruction reordering
- CPU cache write buffering

Within a single thread this is invisible — reordering maintains the illusion of sequential execution. But across threads, reordering can make your program behave in shocking ways.

---

### 13.2 The Full Happens-Before Rules

A **happens-before (HB)** edge between action A and action B guarantees: A's writes are visible to B.

| Rule | Happens-before edge |
|------|-------------------|
| **Program order** | Each action in a thread HB the next action in the same thread |
| **Monitor lock** | `unlock()` of a lock HB any subsequent `lock()` of the same lock |
| **Volatile** | Write to a `volatile` HB any subsequent read of the same variable |
| **Thread start** | `Thread.start()` HB any action in the started thread |
| **Thread join** | All actions in a thread HB `Thread.join()` returning |
| **Transitivity** | If A HB B and B HB C, then A HB C |
| **`final` fields** | All writes to `final` fields in a constructor HB the first read of the object reference by another thread (after the constructor returns) |

---

### 13.3 `final` Fields and Safe Publication

An object is **safely published** if its reference is made visible to other threads only after construction is complete.

```java
// SAFE — final guarantees visibility of fields after construction
class Config {
    final String host;
    final int port;
    Config(String h, int p) { this.host = h; this.port = p; }
}
```

```java
// UNSAFE — reference escapes before construction is complete
class Unsafe {
    int value;
    Unsafe() {
        register(this); // OTHER THREADS may see value == 0 even if set next line
        value = 42;
    }
}
```

**Safe publication patterns:**
- Initialize in a `static` field initializer.
- Use `final` fields.
- Use `volatile` for the reference.
- Use synchronization before sharing the reference.

---

### 13.4 Double-Checked Locking — Broken and Fixed

**The broken version (pre-Java 5):**
```java
// BROKEN — do NOT use
private static Singleton instance;

public static Singleton getInstance() {
    if (instance == null) {                    // check 1 (no lock)
        synchronized (Singleton.class) {
            if (instance == null) {            // check 2 (with lock)
                instance = new Singleton();    // PROBLEM: partially constructed object
            }                                  // may be visible to check 1 in another thread
        }
    }
    return instance;
}
```

`instance = new Singleton()` is three operations: allocate memory, initialize object, assign reference. The JVM can reorder steps 2 and 3 — a thread can see a non-null but uninitialized object.

**The correct version:**
```java
// CORRECT — volatile prevents reordering
private static volatile Singleton instance;

public static Singleton getInstance() {
    if (instance == null) {
        synchronized (Singleton.class) {
            if (instance == null) {
                instance = new Singleton(); // volatile write — no reordering
            }
        }
    }
    return instance;
}
```

---

### 13.5 Initialization-On-Demand Holder (Best Pattern)

```java
public class Singleton {
    private Singleton() {}

    private static class Holder {
        static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE; // class loaded lazily on first call, guaranteed safe
    }
}
```

- No `synchronized` needed.
- No `volatile` needed.
- Lazy initialization guaranteed — `Holder` class is loaded only when `getInstance()` is called.
- Thread-safety guaranteed by the class loading mechanism (Java Language Specification).

---

## Sample

**Broken DCL vs. fixed DCL vs. Holder demonstrated in comments:**

```java
// 1. BROKEN — possible to return partially initialized instance
private static Connection broken;
public static Connection getBroken() { /* see above broken pattern */ }

// 2. FIXED with volatile
private static volatile Connection fixed;
public static Connection getFixed() { /* see above volatile DCL */ }

// 3. BEST — Holder idiom
public static Connection getBest() { return ConnectionHolder.INSTANCE; }
private static class ConnectionHolder {
    static final Connection INSTANCE = new Connection("jdbc://...");
}
```

---

## Homework

1. **Review** your code from Modules 5–9. For each piece of shared mutable state you used, ask:
   - Is there a happens-before relationship ensuring visibility?
   - Is there any possible unsafe publication?
   - Are `final` fields used where possible?

2. **Fix** the broken DCL example below:
```java
class Config {
    private static Config instance;
    private Map<String, String> settings;

    private Config() {
        settings = loadFromFile(); // I/O operation
    }

    public static Config get() {
        if (instance == null) {
            synchronized (Config.class) {
                if (instance == null) {
                    instance = new Config();
                }
            }
        }
        return instance;
    }
}
```
   - Identify the bug.
   - Provide two correct implementations: one using `volatile` DCL and one using the Holder idiom.
   - Explain why the Holder idiom is preferable here.

---

**Previous:** [Module 12 — Fork/Join Framework](../module-12-fork-join/module-12.md)  
**Next:** [Module 14 — Real-World Patterns & Pitfalls](../module-14-real-world-patterns/module-14.md)
