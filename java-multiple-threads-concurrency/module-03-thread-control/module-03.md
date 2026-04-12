# Module 03 — Thread Control

**Level:** Basics  
**Goal:** Learn how to control thread execution flow — pausing, waiting, interrupting, and managing thread types.

---

## Theory

### 3.1 `sleep()` — Pause a Thread
```java
Thread.sleep(2000); // pauses current thread for 2 seconds
```
- Puts the current thread into **TIMED_WAITING** state.
- Does **not** release any locks the thread holds.
- Throws `InterruptedException` — always handle it.

---

### 3.2 `join()` — Wait for a Thread to Finish
```java
Thread t = new Thread(() -> doWork());
t.start();
t.join(); // main thread blocks here until t finishes
System.out.println("t is done");
```
- Useful to coordinate: "don't continue until these threads are all done."
- Can also timeout: `t.join(3000)` — wait at most 3 seconds.

---

### 3.3 `interrupt()` — Signal a Thread to Stop
```java
thread.interrupt(); // sets the interrupt flag on that thread
```
- If the target thread is in `sleep()` or `wait()`, it throws `InterruptedException` immediately.
- If it's running normally, only sets a flag — the thread must check it:
```java
while (!Thread.currentThread().isInterrupted()) {
    doWork();
}
```
- **Never swallow `InterruptedException` silently.** Either re-throw it or restore the flag:
```java
} catch (InterruptedException e) {
    Thread.currentThread().interrupt(); // restore the flag
}
```

---

### 3.4 Daemon Threads vs. User Threads

| | User Thread | Daemon Thread |
|--|-------------|---------------|
| JVM exits when | All user threads finish | (doesn't block JVM exit) |
| Use for | Core work | Background tasks (logging, GC helpers) |
| How to set | default | `t.setDaemon(true)` before `start()` |

> If only daemon threads are running, the JVM shuts down even if they haven't finished.

---

### 3.5 Thread Naming and Priority
```java
t.setName("payment-processor");    // helpful for debugging thread dumps
t.setPriority(Thread.MAX_PRIORITY); // hint to the scheduler — not guaranteed
```
- Always name your threads in production code — makes logs and thread dumps readable.
- **Priority is unreliable** — the OS scheduler is free to ignore it. Never use priority for correctness.

---

### 3.6 `Thread.currentThread()`
Returns a reference to the thread that is currently executing. Useful for logging and self-interruption checks.

---

## Sample

**Main thread waits for 3 workers, then prints a summary.**

```java
public class JoinDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread[] workers = new Thread[3];

        for (int i = 0; i < 3; i++) {
            final int id = i + 1;
            workers[i] = new Thread(() -> {
                System.out.println("Worker " + id + " starting...");
                try { Thread.sleep(1000 * id); } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                System.out.println("Worker " + id + " done.");
            }, "worker-" + id);
            workers[i].start();
        }

        for (Thread w : workers) {
            w.join(); // wait for each one
        }

        System.out.println("All workers finished. Main continues.");
    }
}
```

---

## Homework

Implement a **timeout mechanism** for a slow task:

1. Start a worker thread that simulates a long task (e.g., `sleep(10_000)`).
2. The main thread waits **at most 3 seconds**.
3. If the worker hasn't finished in 3 seconds, interrupt it and print `"Task timed out, cancelling..."`.
4. The worker must respond to the interruption gracefully and print `"Worker cancelled."`.

**Bonus:** Wrap this logic in a reusable `runWithTimeout(Runnable task, long timeoutMs)` method.

---

**Previous:** [Module 02 — Creating & Starting Threads](../module-02-creating-threads/module-02.md)  
**Next:** [Module 04 — The Problem of Shared State](../module-04-shared-state/module-04.md)
