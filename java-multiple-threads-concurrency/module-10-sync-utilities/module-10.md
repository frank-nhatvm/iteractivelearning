# Module 10 — Synchronization Utilities

**Level:** Advanced  
**Goal:** Use high-level coordination tools to synchronize groups of threads cleanly and efficiently.

---

## Theory

### 10.1 `CountDownLatch` — Wait for N Events

A one-shot gate: the latch starts at count N, each `countDown()` decrements it. Threads calling `await()` block until the count reaches zero.

```java
CountDownLatch latch = new CountDownLatch(3); // wait for 3 events

// Workers
new Thread(() -> {
    doWork();
    latch.countDown(); // signal: "I'm done"
}).start();

// Main thread or coordinator
latch.await();           // blocks until countDown() called 3 times
latch.await(5, SECONDS); // with timeout
```

**Not reusable.** Once it reaches zero it stays at zero.

**Use cases:**
- Wait for multiple services to start before accepting traffic.
- Wait for all parallel tasks to complete before aggregating results.

---

### 10.2 `CyclicBarrier` — Meet at a Checkpoint

All threads must reach the barrier before any of them continue. Unlike `CountDownLatch`, **it resets automatically** and can be reused in cycles/phases.

```java
CyclicBarrier barrier = new CyclicBarrier(3, () -> {
    System.out.println("All threads reached barrier — starting next phase");
});

// In each thread:
doPhaseOneWork();
barrier.await(); // block until all 3 threads arrive
doPhaseTwoWork();
barrier.await(); // block again for phase 2
```

**Use cases:**
- Parallel algorithms with multiple phases (e.g., parallel sort).
- Simulations where all agents must synchronize each time step.

---

### 10.3 `Semaphore` — Limit Concurrent Access

Controls access to a pool of resources. Initialized with N permits; each `acquire()` takes one; each `release()` returns one. Blocks when permits are exhausted.

```java
Semaphore semaphore = new Semaphore(10); // max 10 concurrent users

// Worker thread
semaphore.acquire();        // wait for a permit
try {
    useResource();
} finally {
    semaphore.release();    // return the permit
}

// Non-blocking try:
if (semaphore.tryAcquire()) { ... }
if (semaphore.tryAcquire(500, MILLISECONDS)) { ... }
```

**Use cases:**
- Database connection pools.
- Rate limiting per-thread (e.g., max 10 concurrent HTTP calls).
- Controlling access to any finite resource.

---

### 10.4 `Phaser` — Dynamic Multi-Phase Barrier

A more flexible version of `CyclicBarrier`: supports dynamic registration/deregistration of parties and variable phase counts.

```java
Phaser phaser = new Phaser(3); // 3 registered parties

// In each thread:
doWork();
phaser.arriveAndAwaitAdvance(); // like CyclicBarrier.await()

// Thread can deregister:
phaser.arriveAndDeregister();
```

**Use cases:** Long-running parallel tasks where the number of active workers changes over time.

---

### 10.5 `Exchanger` — Swap Data Between Two Threads

Two threads exchange objects at a meeting point.

```java
Exchanger<List<Integer>> exchanger = new Exchanger<>();

// Thread A (producer fills buffer, then swaps for empty one)
filledBuffer = exchanger.exchange(filledBuffer);

// Thread B (consumer processes filled buffer, provides empty one)
emptyBuffer = exchanger.exchange(processedBuffer);
```

**Use cases:** Double-buffering, pipeline stages passing work between exactly two threads.

---

## Sample

**Race start gate with `CountDownLatch` + DB pool with `Semaphore`:**

```java
// --- Race gate ---
CountDownLatch startGate = new CountDownLatch(1);
CountDownLatch endGate   = new CountDownLatch(5);

for (int i = 0; i < 5; i++) {
    new Thread(() -> {
        try {
            startGate.await();  // all racers wait at the gate
            run();
        } finally {
            endGate.countDown(); // signal finish
        }
    }).start();
}
startGate.countDown(); // "GO!" — all racers start simultaneously
endGate.await();       // wait for all to finish

// --- DB connection pool ---
Semaphore dbPool = new Semaphore(10, true); // 10 connections, fair

void queryDB() {
    dbPool.acquire();
    try { executeQuery(); }
    finally { dbPool.release(); }
}
```

---

## Homework

Implement **parallel merge sort** using `CyclicBarrier`:

1. Split an array of 1,000 integers into N equal chunks (N = number of worker threads, e.g. 4).
2. **Phase 1:** Each worker thread sorts its chunk independently. All threads wait at a `CyclicBarrier`.
3. **Phase 2:** Pairs of threads merge their sorted chunks. All wait at the barrier again.
4. Continue until the full array is sorted.
5. Verify the result is correctly sorted.

**Bonus:** Compare wall-clock time against `Arrays.sort()` on the same data.

---

**Previous:** [Module 09 — Concurrent Collections](../module-09-concurrent-collections/module-09.md)  
**Next:** [Module 11 — `CompletableFuture` & Async Programming](../module-11-completable-future/module-11.md)
