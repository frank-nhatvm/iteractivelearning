# Module 02 — Creating & Starting Threads *(Customized)*

**Learner:** Beginner · Code-first · Web backend focus · ~30 min sessions

---

## Section 2.1 — Three Ways to Create a Thread

### Code first

In web backend code, a "task" is usually business logic (call DB, call remote service, transform data). In Java, that task can run inside a `thread` using these 3 approaches.

#### Option A: Extend `Thread`

```java
class CheckoutThread extends Thread {
    @Override
    public void run() {
        System.out.println("Running checkout task on: " + Thread.currentThread().getName());
    }
}

CheckoutThread t = new CheckoutThread();
t.start();
```

#### Option B: Implement `Runnable`

```java
class InventoryTask implements Runnable {
    @Override
    public void run() {
        System.out.println("Checking inventory on: " + Thread.currentThread().getName());
    }
}

Thread t = new Thread(new InventoryTask());
t.start();
```

#### Option C: Lambda (Java 8+)

```java
Thread t = new Thread(() -> {
    System.out.println("Calling payment gateway on: " + Thread.currentThread().getName());
});
t.start();
```

### What to use in real projects

- Use `Runnable` or lambda most of the time.
- Avoid extending `Thread` unless you have a very specific reason.
- Why: extending `Thread` couples task logic with thread mechanism.
- In backend services, we usually model "work" separately from "where/how it runs".

### Quick backend mapping

| Approach | Mental model in backend |
|---------|--------------------------|
| Extend `Thread` | "My task is a thread class" (tightly coupled) |
| Implement `Runnable` | "My task is a unit of work" (clean separation) |
| Lambda | "Inline quick task" (most concise) |

---

**Check:** Any questions on this before we continue? You can type `/explain <topic>` for deeper detail.

---

## Section 2.2 — Thread Lifecycle

### Code first mental model

When you create and run a `thread`, it moves through states.

```java
Thread t = new Thread(() -> {
    try {
        Thread.sleep(300);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
});

System.out.println(t.getState()); // NEW
t.start();
System.out.println(t.getState()); // usually RUNNABLE (timing-dependent)
```

### Lifecycle states

```
NEW -> RUNNABLE -> (running on CPU)
          |
          +-> BLOCKED       (waiting for a lock)
          +-> WAITING       (waiting indefinitely: wait(), join())
          +-> TIMED_WAITING (sleep(ms), wait(timeout), join(timeout))
          +-> TERMINATED    (run() finished)
```

### Practical backend meaning

- `RUNNABLE`: request task is ready to run.
- `BLOCKED`: waiting to enter a `synchronized` block.
- `TIMED_WAITING`: sleeping/retrying or waiting with timeout.
- `TERMINATED`: task completed (success/failure handled by your code).

---

**Check:** Does this state model make sense in your backend context?

---

## Section 2.3 — `start()` vs `run()` (Most Important in This Module)

### Code first

```java
Thread t = new Thread(() -> {
    System.out.println("Current thread: " + Thread.currentThread().getName());
});

// Correct: creates a new thread
t.start();

// Wrong if you expect concurrency: just a normal method call on current thread
// t.run();
```

### Key difference

| Call | Behavior |
|------|----------|
| `t.start()` | Creates a new `thread`, then that thread executes `run()` |
| `t.run()` | Executes `run()` on the current `thread` (no concurrency) |

### Why this bug appears in backend code

Developers write thread code but accidentally call `run()`. Then requests still behave sequentially, and they think "threads are slow" while the real issue is no new thread was created.

---

**Check:** Want a quick mini-example that prints thread names to prove `start()` vs `run()` visually?

---

## Module 02 Code Exercise

Create `HelloThreads` and run it.

```java
public class HelloThreads {
    public static void main(String[] args) {
        for (int i = 1; i <= 5; i++) {
            final int id = i;
            Thread t = new Thread(() -> {
                System.out.println("Hello from Thread " + id
                    + " [" + Thread.currentThread().getName() + "]");
            });
            t.setName("worker-" + id);
            t.start();
        }
        System.out.println("Main thread done launching.");
    }
}
```

Expected behavior: output order is non-deterministic between runs.

---

## Homework

Write a program that:

1. Launches 5 `thread` workers (ID 1..5).
2. Each worker sleeps random 100ms..1000ms.
3. When done, worker prints: `Thread X finished in Yms`.
4. Main thread prints finish order (1st, 2nd, 3rd, ...).

For now, do this with basic tools from Modules 1-2 only. In Module 3 you will learn cleaner control utilities.
