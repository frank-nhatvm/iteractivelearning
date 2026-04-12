# Module 06 — `java.util.concurrent` Locks

**Level:** Intermediate  
**Goal:** Learn the more powerful and flexible lock types provided by the `java.util.concurrent` package.

---

## Theory

### 6.1 Why `synchronized` Is Sometimes Not Enough

`synchronized` limitations:
- **No timeout:** If a thread can't acquire the lock, it blocks forever.
- **No interruptibility:** A blocked thread cannot be interrupted while waiting for a lock.
- **No fairness control:** No guarantee that the longest-waiting thread gets the lock next.
- **No separate read/write semantics:** All access is exclusive even for read-only operations.

`java.util.concurrent.locks` solves all of these.

---

### 6.2 `ReentrantLock`

The most common replacement for `synchronized`.

```java
import java.util.concurrent.locks.ReentrantLock;

private final ReentrantLock lock = new ReentrantLock();

public void increment() {
    lock.lock();
    try {
        counter++;
    } finally {
        lock.unlock(); // ALWAYS unlock in finally — critical!
    }
}
```

**Key methods:**
```java
lock.lock();                         // acquire (blocks if unavailable)
lock.unlock();                       // release
lock.tryLock();                      // acquire immediately or return false
lock.tryLock(2, TimeUnit.SECONDS);   // try for up to 2s, then give up
lock.lockInterruptibly();            // can be interrupted while waiting
new ReentrantLock(true);             // fair mode — longest waiter gets lock first
```

**Reentrant:** A thread that already holds the lock can acquire it again (useful for recursive methods).

---

### 6.3 `Condition` — Replacement for `wait/notify`

`Condition` objects let you associate multiple wait queues with one lock — more expressive than `wait/notifyAll`.

```java
private final ReentrantLock lock = new ReentrantLock();
private final Condition notEmpty = lock.newCondition();
private final Condition notFull  = lock.newCondition();

// Producer
lock.lock();
try {
    while (buffer.isFull()) notFull.await();   // wait until not full
    buffer.add(item);
    notEmpty.signal();                          // signal a consumer
} finally { lock.unlock(); }

// Consumer
lock.lock();
try {
    while (buffer.isEmpty()) notEmpty.await(); // wait until not empty
    item = buffer.take();
    notFull.signal();                           // signal a producer
} finally { lock.unlock(); }
```

Advantage: `signal()` can wake up only consumers (not producers), avoiding unnecessary wakeups.

---

### 6.4 `ReadWriteLock`

Allows **multiple concurrent readers** or **one exclusive writer**.

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

private final ReadWriteLock rwLock = new ReentrantReadWriteLock();

public String read(String key) {
    rwLock.readLock().lock();
    try {
        return cache.get(key);          // many threads can read simultaneously
    } finally {
        rwLock.readLock().unlock();
    }
}

public void write(String key, String value) {
    rwLock.writeLock().lock();
    try {
        cache.put(key, value);          // exclusive access for writes
    } finally {
        rwLock.writeLock().unlock();
    }
}
```

**Best for:** Read-heavy data structures (caches, configuration stores).

---

## Sample

**`BankAccount` with `ReentrantLock` and timeout:**
```java
public class BankAccount {
    private double balance;
    private final ReentrantLock lock = new ReentrantLock();

    public boolean withdraw(double amount) {
        try {
            if (!lock.tryLock(1, TimeUnit.SECONDS)) {
                System.out.println("Could not acquire lock, aborting.");
                return false;
            }
            try {
                if (balance < amount) return false;
                balance -= amount;
                return true;
            } finally {
                lock.unlock();
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return false;
        }
    }
}
```

---

## Homework

Implement a thread-safe **`LRUCache<K, V>`** using `ReadWriteLock`:
- `get(K key)` — can be called by many threads simultaneously (use read lock).
- `put(K key, V value)` — exclusive access (use write lock).
- Evict the least-recently-used entry when capacity is exceeded.
- Write a test with 5 reader threads and 2 writer threads running concurrently for 5 seconds.

**Bonus:** Add an `evictionCount` metric using `AtomicInteger` (preview of Module 07).

---

**Previous:** [Module 05 — Synchronization](../module-05-synchronization/module-05.md)  
**Next:** [Module 07 — Atomic Variables](../module-07-atomic-variables/module-07.md)
