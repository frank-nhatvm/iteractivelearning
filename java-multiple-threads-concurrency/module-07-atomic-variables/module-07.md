# Module 07 — Atomic Variables

**Level:** Intermediate  
**Goal:** Use lock-free atomic operations for simple concurrent state management.

---

## Theory

### 7.1 The Problem with `synchronized` for Simple Operations

For a single counter, `synchronized` adds overhead:
- Thread blocking and unblocking
- Context switches
- Lock acquisition/release cost

For simple operations on a single variable, Java offers **atomic classes** — lock-free and faster under low-to-moderate contention.

---

### 7.2 Compare-And-Swap (CAS)

Atomic classes are built on a hardware instruction: **Compare-And-Swap (CAS)**.

```
CAS(variable, expectedValue, newValue):
  if variable == expectedValue:
      variable = newValue   // success
      return true
  else:
      return false          // someone changed it first, try again
```

This is done atomically by the CPU — no locks needed. Java calls this via `Unsafe` internally, but you use it through clean API classes.

---

### 7.3 The Atomic Classes

#### `AtomicInteger` / `AtomicLong`
```java
import java.util.concurrent.atomic.AtomicInteger;

AtomicInteger counter = new AtomicInteger(0);

counter.incrementAndGet();       // ++counter (atomic)
counter.getAndIncrement();       // counter++ (atomic)
counter.addAndGet(5);            // counter += 5 (atomic)
counter.get();                   // read current value
counter.set(100);                // set directly (not CAS, just volatile write)
counter.compareAndSet(100, 200); // if current==100, set to 200; returns true/false
```

#### `AtomicBoolean`
```java
AtomicBoolean initialized = new AtomicBoolean(false);

// Safe one-time initialization guard:
if (initialized.compareAndSet(false, true)) {
    // only one thread will enter here
    init();
}
```

#### `AtomicReference<T>`
```java
AtomicReference<String> ref = new AtomicReference<>("initial");
ref.compareAndSet("initial", "updated"); // atomic reference swap
```

---

### 7.4 When to Use Atomics vs. `synchronized`

| Scenario | Use |
|----------|-----|
| Single variable, simple ops (increment, CAS) | `AtomicInteger` / `AtomicLong` |
| Multiple variables must change together | `synchronized` or `Lock` |
| Read-heavy, write-rare shared data | `ReadWriteLock` |
| Complex business logic needing exclusion | `synchronized` or `Lock` |

---

### 7.5 The ABA Problem (Awareness)

A CAS on value A might succeed even if the value was changed A → B → A by another thread. For most counters this doesn't matter, but for reference types (e.g., linked list nodes) it can cause bugs. Solution: `AtomicStampedReference` (adds a version stamp).

---

## Sample

**Fixing the buggy counter with `AtomicInteger`:**
```java
public class AtomicCounter {
    private final AtomicInteger counter = new AtomicInteger(0);

    public void increment() {
        counter.incrementAndGet(); // no synchronized needed
    }

    public int get() {
        return counter.get();
    }

    public static void main(String[] args) throws InterruptedException {
        AtomicCounter c = new AtomicCounter();
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 100_000; i++) c.increment();
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 100_000; i++) c.increment();
        });
        t1.start(); t2.start();
        t1.join();  t2.join();
        System.out.println("Final: " + c.get()); // always 200,000
    }
}
```

---

## Homework

Implement a thread-safe **rate limiter** using `AtomicInteger`:

1. The limiter allows at most **100 requests per second** across all threads.
2. `boolean tryAcquire()` — returns `true` if the request is allowed, `false` if the rate limit is exceeded.
3. The counter resets every second.
4. Write a test that simulates 10 threads each trying to make 20 requests/second and verify:
   - No more than 100 requests/second are ever allowed.
   - The limiter correctly resets each second.

**Hint:** Use `AtomicInteger` for the counter and `AtomicLong` for the last-reset timestamp. Use `compareAndSet` for the reset to avoid a race condition on the reset itself.

---

**Previous:** [Module 06 — `java.util.concurrent` Locks](../module-06-concurrent-locks/module-06.md)  
**Next:** [Module 08 — The Executor Framework](../module-08-executor-framework/module-08.md)
