# Module 11 — `CompletableFuture` & Async Programming

**Level:** Advanced  
**Goal:** Compose and chain asynchronous operations without blocking threads, using `CompletableFuture`.

---

## Theory

### 11.1 The Problem with Plain `Future`

```java
Future<String> future = executor.submit(() -> fetchData());
String result = future.get(); // BLOCKS the calling thread until done
```

Problems:
- `get()` is blocking — the thread sits idle while waiting.
- No way to chain operations: "when this finishes, do that."
- No built-in exception handling or timeout composition.
- Can't combine multiple futures elegantly.

`CompletableFuture` solves all of this.

---

### 11.2 Creating a `CompletableFuture`

```java
// Run async, no return value
CompletableFuture<Void> cf = CompletableFuture.runAsync(() -> doWork());

// Run async, returns a value
CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> fetchData());

// With a custom executor (avoid using the default ForkJoinPool for blocking I/O):
CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> fetchData(), myExecutor);
```

---

### 11.3 Chaining Operations

```java
CompletableFuture.supplyAsync(() -> fetchUser(id))        // async: fetch user
    .thenApply(user -> user.getName())                     // transform result
    .thenApply(String::toUpperCase)                        // transform again
    .thenAccept(name -> System.out.println("User: " + name)); // consume result (returns Void)
```

| Method | Input | Returns | Use for |
|--------|-------|---------|---------|
| `thenApply(fn)` | result | new result | Transform (like `map`) |
| `thenAccept(fn)` | result | `Void` | Consume result, no return |
| `thenRun(fn)` | nothing | `Void` | Run action after, ignore result |
| `thenCompose(fn)` | result | `CompletableFuture<U>` | Chain another async call (like `flatMap`) |
| `thenCombine(cf, fn)` | both results | new result | Combine two independent futures |

```java
// thenCompose — async chain (avoid nested futures):
CompletableFuture.supplyAsync(() -> fetchUserId())
    .thenCompose(id -> fetchUserProfile(id)); // fetchUserProfile returns a CompletableFuture

// thenCombine — two independent async calls, combine results:
CompletableFuture<String> priceF   = CompletableFuture.supplyAsync(() -> getPrice());
CompletableFuture<Integer> stockF  = CompletableFuture.supplyAsync(() -> getStock());
priceF.thenCombine(stockF, (price, stock) -> "Price: " + price + ", Stock: " + stock);
```

---

### 11.4 Combining Multiple Futures

```java
// Wait for ALL to complete:
CompletableFuture.allOf(cf1, cf2, cf3).join();

// Collect all results after allOf:
List<CompletableFuture<String>> futures = List.of(cf1, cf2, cf3);
CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]))
    .thenApply(v -> futures.stream().map(CompletableFuture::join).collect(toList()));

// Return the result of whichever finishes FIRST:
CompletableFuture.anyOf(cf1, cf2, cf3).thenAccept(result -> use(result));
```

---

### 11.5 Exception Handling

```java
CompletableFuture.supplyAsync(() -> riskyFetch())
    .exceptionally(ex -> {
        log.error("Fetch failed", ex);
        return "default-value";         // fallback value
    });

// Handle both success and failure:
    .handle((result, ex) -> {
        if (ex != null) return "fallback";
        return result.toUpperCase();
    });
```

---

### 11.6 Custom Executor

The default `CompletableFuture.supplyAsync()` uses `ForkJoinPool.commonPool()`. For blocking I/O tasks, use a dedicated executor to avoid starving the common pool:

```java
ExecutorService ioPool = Executors.newFixedThreadPool(20);
CompletableFuture.supplyAsync(() -> blockingDbCall(), ioPool);
```

---

## Sample

**Call 3 APIs concurrently, combine results, non-blocking:**

```java
ExecutorService pool = Executors.newFixedThreadPool(10);

CompletableFuture<String> userF    = CompletableFuture.supplyAsync(() -> fetchUser(id), pool);
CompletableFuture<String> ordersF  = CompletableFuture.supplyAsync(() -> fetchOrders(id), pool);
CompletableFuture<String> reviewsF = CompletableFuture.supplyAsync(() -> fetchReviews(id), pool);

CompletableFuture.allOf(userF, ordersF, reviewsF)
    .thenApply(v -> buildResponse(userF.join(), ordersF.join(), reviewsF.join()))
    .exceptionally(ex -> errorResponse(ex))
    .thenAccept(response -> sendToClient(response));
```

Total time ≈ max(fetchUser, fetchOrders, fetchReviews). Not their sum.

---

## Homework

Build an **async product detail page loader**:

1. Given a `productId`, concurrently fetch (simulate with `Thread.sleep`):
   - Product info (300ms)
   - Customer reviews (500ms)
   - Inventory & stock status (200ms)
2. Combine all three into a single `ProductPageResponse` object.
3. If any one fetch fails, the page should still load with a partial response (degraded gracefully — failed section shows a placeholder).
4. The entire operation must complete within 1 second (add a `completeOnTimeout` fallback — Java 9+).
5. Write the code using `CompletableFuture` with **no blocking `get()` calls** in the main thread.

---

**Previous:** [Module 10 — Synchronization Utilities](../module-10-sync-utilities/module-10.md)  
**Next:** [Module 12 — Fork/Join Framework](../module-12-fork-join/module-12.md)
