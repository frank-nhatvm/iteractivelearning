# Module 08 — The Executor Framework

**Level:** Intermediate → Advanced  
**Goal:** Stop managing threads manually. Use thread pools to safely run concurrent tasks at scale.

---

## Theory

### 8.1 Why Manual Thread Creation Is Dangerous

```java
// BAD — creating a new thread per request
for (Request r : incomingRequests) {
    new Thread(() -> handle(r)).start(); // 10,000 requests = 10,000 threads = crash
}
```

Problems:
- Thread creation is expensive (memory, OS resources).
- Uncontrolled growth → `OutOfMemoryError`.
- No lifecycle management, no easy way to shut down.

**Solution:** Thread pools — create a fixed set of threads and reuse them.

---

### 8.2 `ExecutorService` — The Core Interface

```java
import java.util.concurrent.*;

ExecutorService executor = Executors.newFixedThreadPool(4);

executor.execute(() -> doWork());        // fire-and-forget (Runnable)
Future<String> result = executor.submit(() -> compute()); // returns a Future (Callable)

executor.shutdown();                     // no new tasks, finish existing ones
executor.shutdownNow();                  // try to cancel running tasks
executor.awaitTermination(10, TimeUnit.SECONDS); // wait for clean shutdown
```

---

### 8.3 Thread Pool Types

| Factory Method | Behavior | Use When |
|---------------|----------|----------|
| `newFixedThreadPool(n)` | Always exactly n threads | Predictable, bounded concurrency |
| `newCachedThreadPool()` | Grows/shrinks, idle threads expire after 60s | Many short-lived tasks |
| `newSingleThreadExecutor()` | Exactly 1 thread, tasks queue up | Serialize access to a resource |
| `newScheduledThreadPool(n)` | Supports delay and periodic execution | Scheduled jobs, polling |

> For production code, prefer constructing `ThreadPoolExecutor` directly for full control over queue size, rejection policy, and thread naming.

---

### 8.4 `Callable` vs. `Runnable`

| | `Runnable` | `Callable<V>` |
|--|-----------|--------------|
| Returns value | No (`void`) | Yes (`V`) |
| Throws checked exceptions | No | Yes |
| Submit with | `execute()` or `submit()` | `submit()` |

```java
Callable<Integer> task = () -> expensiveComputation();
Future<Integer> future = executor.submit(task);
```

---

### 8.5 `Future<T>` — Handling Results

```java
Future<String> future = executor.submit(() -> fetchFromDB());

// ... other work ...

String result = future.get();              // blocks until done
String result = future.get(3, SECONDS);   // blocks at most 3s, then TimeoutException
boolean done = future.isDone();            // non-blocking check
future.cancel(true);                       // attempt to cancel
```

---

### 8.6 Proper Shutdown Pattern

```java
executor.shutdown(); // politely stop accepting new tasks
try {
    if (!executor.awaitTermination(30, TimeUnit.SECONDS)) {
        executor.shutdownNow(); // force if still running after 30s
    }
} catch (InterruptedException e) {
    executor.shutdownNow();
    Thread.currentThread().interrupt();
}
```

---

## Sample

**Concurrent web scraper — fetch 20 URLs with a pool of 5 threads:**

```java
public class WebScraper {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        List<String> urls = List.of(/* 20 URLs */);
        ExecutorService pool = Executors.newFixedThreadPool(5);

        List<Future<String>> futures = new ArrayList<>();
        for (String url : urls) {
            futures.add(pool.submit(() -> fetch(url)));
        }

        for (Future<String> f : futures) {
            System.out.println("Result: " + f.get()); // blocks per-result
        }

        pool.shutdown();
    }

    static String fetch(String url) {
        // simulate HTTP call
        Thread.sleep(500);
        return "Content of " + url;
    }
}
```

---

## Homework

Build a **parallel image processor**:

1. Given a list of 50 mock "image paths" (Strings), simulate a resize operation on each (sleep 100ms to mimic work).
2. Use a `newFixedThreadPool(8)` to process them concurrently.
3. Collect all results via `Future<String>` and log them in the order they were submitted (not necessarily the order they finished).
4. Ensure clean shutdown even if some tasks fail.
5. **Bonus:** Add a timeout — if any single task takes more than 500ms, cancel it and log a warning.

---

**Previous:** [Module 07 — Atomic Variables](../module-07-atomic-variables/module-07.md)  
**Next:** [Module 09 — Concurrent Collections](../module-09-concurrent-collections/module-09.md)
