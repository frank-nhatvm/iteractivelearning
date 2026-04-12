# Module 12 — Fork/Join Framework

**Level:** Advanced  
**Goal:** Use the Fork/Join framework to efficiently parallelize divide-and-conquer algorithms.

---

## Theory

### 12.1 The Divide-and-Conquer Pattern

```
Problem(data)
  ├── if small enough: solve directly
  └── else:
        left  = fork(Problem(left half))
        right = fork(Problem(right half))
        return merge(left.join(), right.join())
```

This naturally maps to parallel execution if the halves are independent — which is exactly what Fork/Join is designed for.

---

### 12.2 `ForkJoinPool`

A specialized thread pool designed for recursive, divide-and-conquer tasks.

```java
ForkJoinPool pool = new ForkJoinPool();                    // uses all available CPUs
ForkJoinPool pool = new ForkJoinPool(4);                   // fixed 4 threads
ForkJoinPool pool = ForkJoinPool.commonPool();             // shared JVM-wide pool
```

---

### 12.3 `RecursiveTask<V>` — Divide and Return a Result

```java
class SumTask extends RecursiveTask<Long> {
    private final int[] array;
    private final int start, end;
    private static final int THRESHOLD = 1000;

    @Override
    protected Long compute() {
        int length = end - start;
        if (length <= THRESHOLD) {
            // base case: compute directly
            long sum = 0;
            for (int i = start; i < end; i++) sum += array[i];
            return sum;
        }
        // recursive case: split in two
        int mid = start + length / 2;
        SumTask leftTask  = new SumTask(array, start, mid);
        SumTask rightTask = new SumTask(array, mid, end);
        leftTask.fork();                     // submit left to pool asynchronously
        long rightResult = rightTask.compute(); // compute right on current thread
        long leftResult  = leftTask.join();  // wait for left result
        return leftResult + rightResult;
    }
}

// Usage:
long total = ForkJoinPool.commonPool().invoke(new SumTask(array, 0, array.length));
```

> **Pattern:** `fork()` the left half, `compute()` the right half directly, then `join()` the left. This avoids creating an unnecessary thread.

---

### 12.4 `RecursiveAction` — Divide Without Returning a Result

Use when the task modifies data in-place rather than returning a value.

```java
class ParallelFill extends RecursiveAction {
    @Override
    protected void compute() {
        if (length <= THRESHOLD) {
            Arrays.fill(array, start, end, value);
            return;
        }
        int mid = start + length / 2;
        invokeAll(new ParallelFill(array, start, mid, value),
                  new ParallelFill(array, mid, end, value));
    }
}
```

`invokeAll()` forks both tasks and waits for both — a convenient shorthand.

---

### 12.5 Work-Stealing Algorithm

Each thread in the pool has its own **deque** of tasks:
- Threads push/pop from their own deque's front.
- Idle threads **steal** from the back of other threads' deques.

This means threads with more work are rarely interrupted (they pop from front), while idle threads can pick up tasks smoothly (steal from back). **Result:** very high CPU utilization.

---

### 12.6 Fork/Join vs. `ExecutorService`

| | Fork/Join | ExecutorService |
|--|-----------|----------------|
| Task type | Recursive, divide-and-conquer | Independent tasks |
| Work stealing | Yes | No |
| Overhead | Low for many small tasks | Higher per-task overhead |
| Use for | Parallelizing algorithms on data | I/O-bound or independent work units |

---

### 12.7 `parallelStream()` Uses Fork/Join

```java
long sum = IntStream.range(0, 1_000_000)
    .parallel()          // uses ForkJoinPool.commonPool() internally
    .sum();
```

Understanding Fork/Join helps you reason about `parallelStream()` performance.

---

## Sample

**Parallel sum of a large array:**

```java
public class ParallelSum extends RecursiveTask<Long> {
    private static final int THRESHOLD = 10_000;
    private final long[] data;
    private final int start, end;

    public ParallelSum(long[] data, int start, int end) {
        this.data = data; this.start = start; this.end = end;
    }

    @Override
    protected Long compute() {
        if (end - start <= THRESHOLD) {
            long sum = 0;
            for (int i = start; i < end; i++) sum += data[i];
            return sum;
        }
        int mid = (start + end) / 2;
        ParallelSum left  = new ParallelSum(data, start, mid);
        ParallelSum right = new ParallelSum(data, mid, end);
        left.fork();
        return right.compute() + left.join();
    }

    public static void main(String[] args) {
        long[] data = LongStream.range(0, 10_000_000).toArray();
        long result = ForkJoinPool.commonPool().invoke(new ParallelSum(data, 0, data.length));
        System.out.println("Sum: " + result);
    }
}
```

---

## Homework

Implement **parallel quicksort** using `ForkJoinPool`:

1. Write a `QuickSortTask extends RecursiveAction` that partitions an array and recursively sorts subarrays in parallel.
2. Use a threshold (e.g. arrays smaller than 10,000 elements) to switch to `Arrays.sort()` for the base case.
3. **Benchmark:** Sort an array of 10 million random integers using:
   - Your Fork/Join quicksort
   - `Arrays.sort()` (sequential)
   - `Arrays.parallelSort()` (built-in parallel sort — also Fork/Join based)
4. Record and compare all three timings. Explain the results.

---

**Previous:** [Module 11 — `CompletableFuture` & Async Programming](../module-11-completable-future/module-11.md)  
**Next:** [Module 13 — Java Memory Model (Deep Dive)](../module-13-java-memory-model/module-13.md)
