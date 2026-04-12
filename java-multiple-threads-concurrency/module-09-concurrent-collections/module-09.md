# Module 09 — Concurrent Collections

**Level:** Intermediate  
**Goal:** Replace unsafe standard collections with thread-safe alternatives designed for concurrent access.

---

## Theory

### 9.1 Why Standard Collections Are Not Thread-Safe

```java
Map<String, Integer> map = new HashMap<>();
List<String> list = new ArrayList<>();
```

These have **no synchronization**. If two threads call `map.put()` simultaneously, you can get:
- Lost updates
- Infinite loops (in older Java HashMap implementations during resize)
- `ConcurrentModificationException` when iterating while another thread modifies

**Wrong fix:** `Collections.synchronizedMap(map)` — wraps every method in `synchronized`, but compound operations (check-then-act) are still not atomic and iteration still needs external synchronization.

---

### 9.2 `ConcurrentHashMap`

The production-grade thread-safe map.

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

map.put("key", 1);
map.get("key");

// Atomic compound operations:
map.putIfAbsent("key", 1);                  // add only if key absent
map.computeIfAbsent("key", k -> compute()); // compute value if absent
map.compute("key", (k, v) -> v == null ? 1 : v + 1); // atomic read-modify-write
map.merge("key", 1, Integer::sum);          // atomic accumulator
```

- Uses **segment-level locking** (Java 7) / **node-level CAS** (Java 8+) — much less contention than a single lock.
- Iterators are **weakly consistent** — they won't throw `ConcurrentModificationException` but may not reflect mid-iteration writes.

---

### 9.3 `CopyOnWriteArrayList`

```java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
list.add("item");
```

- On every **write**, creates a full copy of the underlying array.
- **Reads** are lock-free because the array reference is never mutated.
- **Use when:** reads are very frequent, writes are rare (e.g., event listener lists).
- **Avoid when:** the collection is large and written to frequently — copying cost is high.

---

### 9.4 `BlockingQueue` — The Concurrency Workhorse

A queue where:
- `put(item)` **blocks** if the queue is full (until space is available).
- `take()` **blocks** if the queue is empty (until an item arrives).

This makes it perfect for **producer-consumer** patterns without manual `wait/notify`.

```java
BlockingQueue<Task> queue = new LinkedBlockingQueue<>(100); // capacity 100

// Producer thread
queue.put(task);          // blocks if full
queue.offer(task);        // returns false if full (non-blocking)
queue.offer(task, 1, SECONDS); // waits up to 1s

// Consumer thread
Task t = queue.take();    // blocks if empty
Task t = queue.poll();    // returns null if empty (non-blocking)
Task t = queue.poll(1, SECONDS); // waits up to 1s
```

**Implementations:**

| Class | Behavior |
|-------|----------|
| `LinkedBlockingQueue` | Optionally bounded, linked nodes |
| `ArrayBlockingQueue` | Bounded, array-backed, optional fairness |
| `PriorityBlockingQueue` | Unbounded, elements ordered by priority |
| `SynchronousQueue` | No capacity — each put must wait for a take |
| `DelayQueue` | Elements can only be taken after a delay expires |

---

## Sample

**Task queue system: producers and consumer threads:**

```java
public class TaskQueue {
    private static final BlockingQueue<String> queue = new ArrayBlockingQueue<>(10);

    public static void main(String[] args) {
        // 2 producers
        for (int i = 0; i < 2; i++) {
            final int id = i;
            new Thread(() -> {
                for (int j = 0; j < 10; j++) {
                    try {
                        String task = "task-" + id + "-" + j;
                        queue.put(task);
                        System.out.println("Produced: " + task);
                    } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
                }
            }).start();
        }

        // 3 consumers
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                while (true) {
                    try {
                        String task = queue.take();
                        System.out.println("Consumed: " + task + " by " + Thread.currentThread().getName());
                        Thread.sleep(50);
                    } catch (InterruptedException e) { Thread.currentThread().interrupt(); break; }
                }
            }).start();
        }
    }
}
```

---

## Homework

Implement a simple **in-memory message broker**:

1. Multiple **publisher threads** push String messages to a topic.
2. Multiple **subscriber threads** consume messages from the same topic.
3. Use `BlockingQueue` as the backbone.
4. Requirements:
   - Messages must not be lost.
   - Each message is consumed by exactly one subscriber (competing consumers model).
   - The broker supports **graceful shutdown**: when `stop()` is called, publishers stop producing and consumers drain the remaining messages before exiting.
5. Test with 3 publishers (50 messages each) and 5 subscribers.

---

**Previous:** [Module 08 — The Executor Framework](../module-08-executor-framework/module-08.md)  
**Next:** [Module 10 — Synchronization Utilities](../module-10-sync-utilities/module-10.md)
