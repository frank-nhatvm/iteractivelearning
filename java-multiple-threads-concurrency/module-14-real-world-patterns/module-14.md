# Module 14 — Real-World Patterns & Pitfalls (Capstone)

**Level:** Capstone  
**Goal:** Apply everything learned to real-world patterns, learn to diagnose production concurrency bugs, and build a complete concurrent system.

---

## Theory

### 14.1 Classic Concurrency Patterns

#### Producer-Consumer
A queue buffers work between producers and consumers, decoupling their speeds.
```
Producers → [BlockingQueue] → Consumers
```
Use: Message queues, pipeline stages, background job processing.

#### Thread-per-Request
Spawn one thread per incoming request (classic web server model). Each request is isolated.
- **Pros:** Simple logic, easy to reason about.
- **Cons:** Doesn't scale to 10,000+ concurrent requests.
- **Modern alternative:** Virtual Threads (Java 21) bring this back at scale.

#### Worker Pool (Thread Pool)
Fixed number of threads pull tasks from a shared queue. Bounded resource usage.
```
[Task Queue] → Worker 1
             → Worker 2
             → Worker 3
```
Use: Any server with bounded concurrency (the `ExecutorService` pattern from Module 8).

#### Pipeline
Output of one stage feeds into the next, each stage running concurrently.
```
Stage 1 → [Queue] → Stage 2 → [Queue] → Stage 3
```
Use: Data processing pipelines, ETL systems, streaming applications.

---

### 14.2 Diagnosing Deadlocks with `jstack`

```bash
# Get the PID of your running Java process
jps -l

# Take a thread dump
jstack <pid> > threaddump.txt
```

Look for in the dump:
```
Found one Java-level deadlock:
=============================
"Thread-A":
  waiting to lock monitor 0x... (object 0x..., a com.example.Account),
  which is held by "Thread-B"
"Thread-B":
  waiting to lock monitor 0x... (object 0x..., a com.example.Account),
  which is held by "Thread-A"
```

**Preventing deadlocks:**
- Always acquire multiple locks in a **consistent global order**.
- Use `tryLock()` with timeout and back off if you can't acquire all locks.
- Minimize the scope of synchronized blocks.
- Prefer higher-level structures (`BlockingQueue`, `ConcurrentHashMap`) over manual locking.

---

### 14.3 Detecting Race Conditions

**Tools:**
- **ThreadSanitizer** (TSan) — available in some JVMs for race detection.
- **Java PathFinder** — model checker for Java concurrency.
- **IntelliJ IDEA** — highlights `@GuardedBy` annotation violations with the Thread Safety plugin.
- **Load testing** — races often only appear under high concurrency; a dedicated load test can surface them.

**Code review checklist:**
- [ ] Every shared mutable field is guarded by a lock or is `volatile`/`AtomicXxx`.
- [ ] Compound check-then-act operations are atomic.
- [ ] No object escapes its constructor.
- [ ] Lock ordering is consistent.
- [ ] Every `lock()` has a corresponding `unlock()` in a `finally` block.
- [ ] `wait()` is always in a `while` loop.

---

### 14.4 Performance Considerations

| Issue | Symptom | Fix |
|-------|---------|-----|
| **Lock contention** | Threads mostly waiting, low throughput | Use finer-grained locks, `ConcurrentHashMap`, atomics |
| **False sharing** | CPU cache thrashing on adjacent fields | Pad fields to cache line boundaries (`@Contended`, Java 8+) |
| **Thread overhead** | Too many threads, high context switching | Use thread pools; Java 21 virtual threads |
| **Lock convoy** | One slow thread holding a lock blocks all others | Minimize work inside locks; use read-write locks |

---

### 14.5 Virtual Threads (Java 21 Preview)

```java
// Create a virtual thread (Java 21)
Thread.ofVirtual().start(() -> handleRequest(request));

// Virtual thread executor:
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
```

- Virtual threads are **JVM-managed**, extremely lightweight (~1KB stack vs. ~512KB for platform threads).
- Millions can exist simultaneously.
- Blocking calls (I/O, `sleep`, `synchronized`) automatically yield the underlying OS thread.
- Bring back the simple **thread-per-request** model at massive scale.
- Not covered in depth here — but knowing they exist shapes architecture decisions.

---

## Sample

**Reading a thread dump from a deadlocked application:**

```
Found one Java-level deadlock:
"worker-thread-2" waiting for lock held by "worker-thread-1"
"worker-thread-1" waiting for lock held by "worker-thread-2"

worker-thread-1 holds: AccountLock(id=101)  wants: AccountLock(id=102)
worker-thread-2 holds: AccountLock(id=102)  wants: AccountLock(id=101)
```

**Root cause:** Two threads transferring between accounts in opposite order.  
**Fix:** Always lock the account with the lower ID first — enforces a consistent lock ordering.

```java
void transfer(Account from, Account to, double amount) {
    Account first  = from.id < to.id ? from : to;
    Account second = from.id < to.id ? to : from;
    synchronized (first) {
        synchronized (second) {
            from.debit(amount);
            to.credit(amount);
        }
    }
}
```

---

## Capstone Project

Build a **concurrent task execution engine** with the following requirements:

### Features
1. **Thread-safe task queue** — supports priority ordering (higher-priority tasks run first).
2. **Configurable worker pool** — number of threads set at creation time.
3. **Task submission** — `submit(Runnable task, int priority)` returns a `Future<Void>`.
4. **Graceful shutdown** — `shutdown()` stops accepting new tasks, waits for queued tasks to finish.
5. **Basic metrics** (thread-safe):
   - Total tasks submitted
   - Total tasks completed
   - Total tasks failed
   - Average execution time (ms)

### Technical Constraints
- Do not use `ExecutorService` internally — implement the worker threads and queue yourself.
- Use `PriorityBlockingQueue` for the task queue.
- Use `AtomicLong` / `AtomicInteger` for metrics.
- Workers must handle task exceptions gracefully (log and continue — don't die).

### Testing
- Submit 1,000 tasks with random priorities and sleep durations (10–100ms).
- Verify all tasks complete.
- Verify metrics are accurate.
- Test graceful shutdown — no tasks lost.

### Bonus
- Add a `cancel(Future)` method.
- Add task timeout — if a task hasn't started within N seconds, cancel it.
- Add VisualVM or JConsole monitoring by exposing metrics via JMX.

---

## Review: Concurrency Tools Cheat Sheet

| Tool | Use When |
|------|----------|
| `synchronized` | Simple mutual exclusion on one object |
| `volatile` | Single-variable visibility flag |
| `ReentrantLock` | Need timeout, fairness, or interruptible locking |
| `ReadWriteLock` | Read-heavy data with occasional writes |
| `AtomicInteger` | Single-counter increment/CAS without locks |
| `ExecutorService` | Pooled execution of independent tasks |
| `CompletableFuture` | Composing async operations non-blocking |
| `BlockingQueue` | Producer-consumer decoupling |
| `ConcurrentHashMap` | Thread-safe map with atomic compound ops |
| `CountDownLatch` | Wait for N one-time events |
| `CyclicBarrier` | Reusable multi-thread checkpoint |
| `Semaphore` | Limit concurrent access to a resource |
| `ForkJoinPool` | Recursive divide-and-conquer parallelism |

---

**Previous:** [Module 13 — Java Memory Model](../module-13-java-memory-model/module-13.md)  
**Back to:** [Course Outline](../outline.md)
