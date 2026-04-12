# Module 05 — Synchronization: `synchronized` & `volatile`

**Level:** Core Tool  
**Goal:** Learn Java's built-in tools for fixing race conditions and memory visibility problems.

---

## Theory

### 5.1 The `synchronized` Keyword

Every Java object has an **intrinsic lock** (also called a monitor). `synchronized` ensures only one thread can execute the guarded code at a time.

#### Synchronized method:
```java
public synchronized void increment() {
    counter++; // only one thread can be here at a time
}
```
Locks on `this` (the current object instance).

#### Synchronized block (preferred — finer control):
```java
public void increment() {
    synchronized (this) {
        counter++;
    }
}
```
You can also lock on any object:
```java
private final Object lock = new Object();

public void increment() {
    synchronized (lock) {
        counter++;
    }
}
```

#### Static synchronized method:
```java
public static synchronized void increment() { ... }
```
Locks on the **Class object** — shared across all instances.

---

### 5.2 How Locking Works
- When Thread A enters a `synchronized` block, it **acquires** the lock.
- If Thread B tries to enter a block guarded by the **same lock**, it **blocks** (goes to BLOCKED state).
- When Thread A exits the block, it **releases** the lock — allowing Thread B to proceed.
- `synchronized` also guarantees **memory visibility**: all writes before the release are visible to the next thread that acquires the lock.

---

### 5.3 `volatile` — Visibility Without Atomicity

`volatile` guarantees that reads/writes go directly to main memory — no CPU caching.

```java
private volatile boolean running = true;

// Thread A
running = false; // immediately visible to all threads

// Thread B
while (running) { doWork(); } // will see the updated false
```

**`volatile` does NOT make compound operations atomic:**
```java
volatile int counter = 0;
counter++; // STILL a race condition — read-modify-write is not atomic
```

| | `synchronized` | `volatile` |
|--|---------------|-----------|
| Mutual exclusion | YES | NO |
| Memory visibility | YES | YES |
| Performance | Slower (blocking) | Faster (no blocking) |
| Use when | Multiple operations need to be atomic | Single-variable visibility flag |

---

### 5.4 `wait()`, `notify()`, `notifyAll()`

Used for **thread coordination** (one thread signals another). Must be called inside a `synchronized` block on the same lock object.

```java
// Consumer thread
synchronized (lock) {
    while (queue.isEmpty()) {
        lock.wait(); // releases lock and waits
    }
    process(queue.poll());
}

// Producer thread
synchronized (lock) {
    queue.add(item);
    lock.notifyAll(); // wake up all waiting threads
}
```

- `wait()` releases the lock and suspends the thread.
- `notify()` wakes up one waiting thread (arbitrary).
- `notifyAll()` wakes up all waiting threads.
- Always use `while` (not `if`) before `wait()` — spurious wakeups are possible.

---

## Sample

**Fixed counter:**
```java
public class FixedCounter {
    private int counter = 0;

    public synchronized void increment() {
        counter++;
    }

    public synchronized int get() {
        return counter;
    }
}
```

**Simple producer-consumer with `wait/notifyAll`:**
```java
public class ProducerConsumer {
    private final List<Integer> buffer = new ArrayList<>();
    private final int CAPACITY = 5;

    public synchronized void produce(int value) throws InterruptedException {
        while (buffer.size() == CAPACITY) {
            wait(); // buffer full, wait
        }
        buffer.add(value);
        System.out.println("Produced: " + value);
        notifyAll();
    }

    public synchronized int consume() throws InterruptedException {
        while (buffer.isEmpty()) {
            wait(); // buffer empty, wait
        }
        int value = buffer.remove(0);
        System.out.println("Consumed: " + value);
        notifyAll();
        return value;
    }
}
```

---

## Homework

Implement a thread-safe **`BankAccount`** class with:
- `deposit(double amount)`
- `withdraw(double amount)` — must reject if balance would go negative
- `getBalance()`

Requirements:
1. No race conditions — multiple threads can call deposit/withdraw concurrently.
2. No overdraft is ever possible, even under heavy concurrent load.
3. Write a test that launches 10 threads: 5 depositing, 5 withdrawing simultaneously, and verify the final balance is always correct.

---

**Previous:** [Module 04 — The Problem of Shared State](../module-04-shared-state/module-04.md)  
**Next:** [Module 06 — `java.util.concurrent` Locks](../module-06-concurrent-locks/module-06.md)
