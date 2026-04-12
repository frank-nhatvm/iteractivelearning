# Module 01 — Why Concurrency?

**Level:** Foundation  
**Goal:** Understand what concurrency is, why it matters, and how Java fits into the picture.

---

## Theory

### 1.1 Process vs. Thread
- A **process** is an independent program running in its own memory space.
- A **thread** is a lightweight unit of execution *within* a process; threads share the same memory.
- A single Java program runs as one process but can contain many threads.

### 1.2 Why Single-Threaded Programs Hit Limits
- **CPU-bound tasks:** Heavy computation (e.g., image processing, sorting) – one thread maxes out one CPU core.
- **I/O-bound tasks:** File reads, network calls, DB queries – the thread sits idle waiting for a response.
- With multiple threads, one thread can do work while another waits, using CPU time more efficiently.

### 1.3 Parallel vs. Concurrent
- **Concurrent:** Multiple tasks are *in progress* at the same time (may interleave on one core).
- **Parallel:** Multiple tasks run *simultaneously* on multiple CPU cores.
- Java threads can be both concurrent and parallel depending on the hardware.

### 1.4 How the OS Schedules Threads
- The OS assigns CPU time slices to threads; threads can be paused and resumed at any moment.
- You have no direct control over scheduling — this is the root cause of many concurrency bugs.

### 1.5 The Java Threading Model Overview
- Each Java thread maps to a native OS thread (pre-Java 21).
- The JVM manages thread creation, scheduling delegation, and garbage collection interactions.
- Java 21 introduced **Virtual Threads** (Project Loom) for lightweight concurrency — covered in Module 14.

---

## Sample

**Scenario:** A program needs to download 5 files from the internet.

**Single-threaded (slow):**
```
Download file 1... (wait 2s)
Download file 2... (wait 2s)
Download file 3... (wait 2s)
Total: ~10 seconds
```

**Multi-threaded (fast):**
```
Thread 1: Download file 1 ─┐
Thread 2: Download file 2   ├─ all run at the same time
Thread 3: Download file 3 ─┘
Total: ~2 seconds
```

The improvement comes from overlapping I/O wait time across threads.

---

## Homework

Think about your own projects or daily-use applications and answer the following:

1. Describe **3 real-world scenarios** (in your own projects or apps you use) where multithreading could improve performance.
2. For each scenario, explain:
   - Is this a **CPU-bound** or **I/O-bound** problem?
   - What would the single-threaded bottleneck be?
   - How would multiple threads help?

> Example answer for scenario 1:  
> *"An e-commerce checkout that simultaneously validates the cart, checks inventory, and charges the card — each is an independent I/O call to a different service. Single-threaded, they run sequentially (~1.5s each = 4.5s total). With threads, all 3 run in parallel (~1.5s total)."*

---

**Next:** [Module 02 — Creating & Starting Threads](../module-02-creating-threads/module-02.md)
