# Module 04 — The Problem of Shared State

**Level:** Core Concept  
**Goal:** Understand why sharing data between threads is dangerous and recognize the root causes of concurrency bugs.

---

## Theory

### 4.1 Race Condition
A **race condition** occurs when two or more threads access shared data simultaneously and the outcome depends on the unpredictable order of execution.

**Classic example — incrementing a counter:**
```java
counter++; // looks like one operation, but it's THREE:
           // 1. READ counter from memory
           // 2. ADD 1 to the value
           // 3. WRITE the result back to memory
```
If two threads each read `counter = 5`, both compute `6`, and both write `6` — the counter ends up at `6` instead of `7`. One increment is lost.

---

### 4.2 Memory Visibility Problem
Each thread may cache variables in its own CPU register or L1/L2 cache. A write by Thread A may **not be visible** to Thread B until much later (or never, without proper synchronization).

```java
boolean running = true; // Thread A sets this to false

// Thread B — may loop forever because it sees a cached "true"
while (running) {
    doWork();
}
```

---

### 4.3 The Java Memory Model (JMM)
The JMM defines the rules under which memory writes by one thread become visible to another.

- Each thread has its own **working memory** (cache).
- The **main memory** is shared.
- Without synchronization, there is **no guarantee** when (or if) a thread's write reaches main memory.

---

### 4.4 The Happens-Before Relationship
A **happens-before** relationship between two operations means: if A happens-before B, then A's effects are guaranteed to be visible to B.

Key sources of happens-before (details in Module 13):
- A `synchronized` block's release happens-before any subsequent acquisition of the same lock.
- A `volatile` write happens-before any subsequent read of the same variable.
- `Thread.start()` happens-before any action in the started thread.
- `Thread.join()` happens-before the caller returning from `join()`.

Without happens-before, there are **no visibility guarantees**.

---

### 4.5 The Three Classic Concurrency Problems

| Problem | Description | Example |
|---------|-------------|---------|
| **Deadlock** | Thread A holds lock 1, waits for lock 2. Thread B holds lock 2, waits for lock 1. Both wait forever. | Two threads each waiting for the other's database table lock. |
| **Livelock** | Threads keep responding to each other but make no progress. | Two people in a corridor both stepping aside for each other, back and forth endlessly. |
| **Starvation** | A thread never gets CPU time because other threads always get priority. | A low-priority thread that never runs because high-priority threads monopolize the CPU. |

---

## Sample

**Buggy counter — spot the problem:**

```java
public class BuggyCounter {
    private static int counter = 0;

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 100_000; i++) counter++;
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 100_000; i++) counter++;
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Final counter: " + counter);
        // Expected: 200,000
        // Actual:   somewhere between 100,000 and 200,000 — unpredictable
    }
}
```

Run this program multiple times. **The result will differ each time** — that is the race condition in action.

---

## Homework

1. **Run** the buggy counter example above on your machine.
2. Run it **at least 5 times** and record each output.
3. Answer these questions:
   - What is the lowest value you observed? What is the highest?
   - Why is the result always at least 100,000? *(Think carefully about the worst case.)*
   - Why does the result change between runs?
   - Is this bug reproducible in a debugger? Why or why not?

> **Discussion point:** This class of bug is particularly dangerous because it often doesn't appear in testing (debuggers slow threads down and can accidentally "fix" races) but shows up in production under high load.

---

**Previous:** [Module 03 — Thread Control](../module-03-thread-control/module-03.md)  
**Next:** [Module 05 — Synchronization: `synchronized` & `volatile`](../module-05-synchronization/module-05.md)
