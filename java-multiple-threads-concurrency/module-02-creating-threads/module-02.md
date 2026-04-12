# Module 02 — Creating & Starting Threads

**Level:** Basics  
**Goal:** Learn all the ways to create threads in Java and understand the thread lifecycle.

---

## Theory

### 2.1 Three Ways to Create a Thread

#### Option A: Extend `Thread`
```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Running in: " + Thread.currentThread().getName());
    }
}

MyThread t = new MyThread();
t.start();
```

#### Option B: Implement `Runnable`
```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Running in: " + Thread.currentThread().getName());
    }
}

Thread t = new Thread(new MyRunnable());
t.start();
```

> Preferred over extending `Thread` — keeps the task logic separate from the thread mechanism.

#### Option C: Lambda (Java 8+)
```java
Thread t = new Thread(() -> {
    System.out.println("Running in: " + Thread.currentThread().getName());
});
t.start();
```

> The cleanest and most common approach for simple tasks today.

---

### 2.2 Thread Lifecycle

```
NEW ──► RUNNABLE ──► (running on CPU)
              │
              ├──► BLOCKED      (waiting for a lock)
              ├──► WAITING      (waiting indefinitely: wait(), join())
              ├──► TIMED_WAITING (waiting with timeout: sleep(ms))
              │
              └──► TERMINATED   (run() has finished)
```

- A thread is **NEW** after construction but before `start()`.
- A thread is **RUNNABLE** after `start()` — it may or may not be on the CPU at any given moment.
- A thread is **TERMINATED** when `run()` returns or throws an uncaught exception.

---

### 2.3 `start()` vs. `run()` — The Critical Difference

| Call | What happens |
|------|-------------|
| `t.start()` | Spawns a **new thread** and calls `run()` on it |
| `t.run()` | Calls `run()` directly on the **current thread** — no new thread is created |

**This is one of the most common beginner mistakes.** Calling `t.run()` gives you no concurrency at all.

---

## Sample

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

**Sample output** (order varies each run — that's the point):
```
Main thread done launching.
Hello from Thread 3 [worker-3]
Hello from Thread 1 [worker-1]
Hello from Thread 5 [worker-5]
Hello from Thread 2 [worker-2]
Hello from Thread 4 [worker-4]
```

---

## Homework

Write a program that:
1. Launches **5 threads**, each assigned an ID from 1 to 5.
2. Each thread sleeps for a **random duration** between 100ms and 1000ms.
3. When a thread wakes up, it prints: `"Thread X finished in Yms"`.
4. After all threads finish, the main thread prints each thread's finishing order (1st, 2nd, 3rd...).

**Hint:** Think about how the main thread can know when all worker threads are done. (You'll learn a clean tool for this in Module 3 — try to figure it out yourself first.)

---

**Previous:** [Module 01 — Why Concurrency?](../module-01-why-concurrency/module-01.md)  
**Next:** [Module 03 — Thread Control](../module-03-thread-control/module-03.md)
