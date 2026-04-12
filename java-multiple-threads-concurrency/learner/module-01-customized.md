# Module 01 — Why Concurrency? *(Customized)*

**Learner:** Beginner · Code-first · Web backend focus · ~30 min sessions

---

## Section 1.1 — Process vs. Thread

### Code first

Here's a simple Java program. When you run it, Java starts **one process** — and inside that process, there is already **one thread** running your `main` method:

```java
public class SingleThreaded {
    public static void main(String[] args) {
        System.out.println("Thread name: " + Thread.currentThread().getName());
        // prints: Thread name: main
    }
}
```

That `main` thread is what executes every line of code, top to bottom, one at a time.

### What that means

| Term | What it is |
|------|-----------|
| **Process** | Your whole Java program — its own memory, its own resources |
| **Thread** | A single line of execution *inside* the process |
| **Multiple threads** | Multiple lines of execution running inside the same process, **sharing the same memory** |

The key word is **sharing** — that shared memory is what makes concurrency powerful *and* dangerous (you'll see why in Module 04).

---

**Check:** Does this make sense so far? Type "ready" to continue to the next section, or ask any question.

---

## Section 1.2 — Why Single-Threaded Programs Hit Limits

### Code first — a web backend example

Imagine a Spring-style endpoint that handles a checkout request:

```java
// This all runs on ONE thread, sequentially
public CheckoutResult checkout(Order order) {
    CartResult   cart      = cartService.validate(order);      // 500ms — DB call
    InventoryResult stock  = inventoryService.check(order);    // 600ms — DB call
    PaymentResult  payment = paymentService.charge(order);     // 800ms — external API

    return new CheckoutResult(cart, stock, payment);
    // Total wait time: 500 + 600 + 800 = ~1900ms per request
}
```

The thread is **idle** (waiting) most of that time — it's not doing any computation, just waiting for responses from other services.

This is called an **I/O-bound** bottleneck.

### Two types of bottlenecks

| Type | What it means | Web backend example |
|------|--------------|---------------------|
| **I/O-bound** | Thread sits idle waiting for network/DB/disk | Calling 3 microservices sequentially |
| **CPU-bound** | Thread is maxed out computing | Generating a PDF, resizing an image |

For web backends, **I/O-bound** is by far the most common situation.

---

**Check:** Ready to continue to Section 1.3?

---

## Section 1.3 — Concurrent vs. Parallel

### The distinction (with code context)

**Concurrent** = multiple tasks are *in progress at the same time* — they may take turns on one CPU core.  
**Parallel** = multiple tasks *literally run at the same instant* on different CPU cores.

In Java, you don't usually control which one happens — the OS and JVM decide. But here's the mental model:

```
Concurrent (1 core, interleaved):
Core 1:  [Task A]--[Task B]--[Task A]--[Task B]--[Task A done]--[Task B done]

Parallel (2 cores, simultaneous):
Core 1:  [Task A]--------------------------[Task A done]
Core 2:  [Task B]--------------------------[Task B done]
```

For your web backend work: most servers today have multiple cores, so **both** happen.

---

**Check:** Ready to continue?

---

## Section 1.4 — The OS Controls Scheduling (Not You)

This is short but important to internalize early.

The OS gives each thread a tiny slice of CPU time, then **pauses it** and gives time to another thread — even in the middle of a line of logic. You have **zero control** over when this happens.

```
Thread A running: balance = balance + 100;
   ← OS pauses Thread A RIGHT HERE, switches to Thread B
Thread B running: balance = balance - 50;
   ← OS resumes Thread A
Thread A continues: (but "balance" was already changed by Thread B!)
```

This is the root cause of most concurrency bugs. You don't need to understand it fully yet — just hold onto this mental image. We'll come back to it in Module 04.

---

**Check:** Ready for the final section?

---

## Section 1.5 — Java's Threading Model (Overview)

Keep this simple for now:

- Every Java thread corresponds to a real **OS thread** (pre-Java 21).
- The JVM delegates actual scheduling to the OS — it doesn't manage this itself.
- Java 21 introduced **Virtual Threads** — much lighter, don't map 1:1 to OS threads. We'll cover this in Module 14 (the capstone).

For now: **1 Java thread = 1 OS thread = 1 lane of execution**.

---

## Module 01 Summary

| Concept | One-liner |
|---------|-----------|
| Thread | A single lane of execution inside your Java program |
| Process | The whole program — one process can have many threads |
| I/O-bound | Thread is idle waiting — concurrency helps a lot here |
| CPU-bound | Thread is computing — parallelism helps here |
| Concurrent | Multiple tasks in progress (may interleave) |
| Parallel | Multiple tasks running at the exact same instant |
| OS scheduling | You don't control it — this causes concurrency bugs |

---

## Homework

Think about a web backend you've worked on (or one you can imagine), and answer:

1. Describe **2 real endpoints** where single-threaded sequential calls are a bottleneck.
2. For each: is it I/O-bound or CPU-bound? What are the sequential steps? How would threads help?

> Example: *"A `/checkout` endpoint calls cartService, inventoryService, and paymentService one by one. Each is an I/O-bound DB/API call. Running them concurrently would cut response time from ~1900ms to ~800ms (the slowest one)."*

No need to write code yet — just reasoning.
