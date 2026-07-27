# Executors & Thread Pools

## Learning Objectives

- Understand precisely why creating raw threads directly doesn't scale, connecting to Module 14's C10K discussion
- Use `ExecutorService` and the standard thread pool factory methods correctly
- Use `Future` to retrieve results from asynchronous work
- Choose the right thread pool type for a given workload

## Prerequisites

[01 — Threads Fundamentals](01-Threads-Fundamentals.md), Module 14 Topic 3 (the C10K problem)

## Motivation

Recall Module 14, Topic 3's precise cost breakdown: each thread requires real memory (a Stack) and real OS scheduling overhead. Creating a brand-new `Thread` for every single unit of work — as Topic 1's basic examples did — repeats exactly the C10K-style problem Module 14 explained in depth. This topic covers Java's standard, idiomatic solution: **reusing** a managed pool of threads instead.

## The Problem with `new Thread()` Per Task

```java
for (int i = 0; i < 10000; i++) {
    new Thread(() -> processTask(i)).start();   // ⚠️ creates 10,000 SEPARATE threads!
}
```

**This directly repeats Module 14, Topic 3's C10K memory/scheduling problem** — 10,000 threads, each with their own Stack allocation and OS scheduling overhead, created and destroyed constantly, for what might be very short-lived individual tasks. **Thread creation itself also has real, measurable overhead** (allocating the Stack, registering with the OS scheduler) — paying this cost freshly for every single small task is wasteful when the same handful of threads could simply be **reused** across many tasks over time.

## `ExecutorService` — A Managed Pool of Reusable Threads

```java
import java.util.concurrent.*;

ExecutorService executor = Executors.newFixedThreadPool(4);   // a pool of exactly 4 reusable threads

for (int i = 0; i < 10000; i++) {
    final int taskId = i;
    executor.submit(() -> processTask(taskId));   // submits WORK -- the pool REUSES its 4 threads
}                                                     // across all 10,000 tasks, queuing as needed

executor.shutdown();   // signals: "no more new tasks will be submitted; finish what's queued, then stop"
```

**This is the standard, idiomatic pattern for concurrent task execution in Java** — `ExecutorService` maintains a fixed (or dynamically-sized) pool of **reusable** worker threads, and a **queue** of pending tasks. Submitting a task doesn't create a new thread — it adds work to the queue, which the pool's existing threads pick up and execute as they become free, **reusing** each thread across many tasks over its lifetime rather than creating a fresh thread per task.

```
 executor.submit() × 10,000 tasks

           ┌─────────── TASK QUEUE ───────────┐
           │  task1, task2, task3, ... task10000 │
           └───────────┬──────────────────────┘
                          │  worker threads pull tasks as they become free
        ┌────────┬────────┼────────┬────────┐
        ▼        ▼          ▼        ▼
   Thread 1  Thread 2   Thread 3  Thread 4     <- ONLY 4 threads total, REUSED across ALL 10,000 tasks
```

## Standard Thread Pool Factory Methods

```java
Executors.newFixedThreadPool(n);       // exactly n threads, always -- good for CPU-bound work with a
                                          // known, appropriate parallelism level (often ~ number of CPU cores)

Executors.newCachedThreadPool();          // creates threads AS NEEDED, reuses idle ones, and REMOVES
                                             // threads that stay idle too long -- good for many short-lived,
                                             // I/O-bound tasks, but can grow unboundedly under heavy load!

Executors.newSingleThreadExecutor();        // exactly ONE thread -- guarantees tasks run SEQUENTIALLY,
                                               // one at a time, in submission order (useful when you
                                               // specifically need serialized execution, not parallelism)

Executors.newScheduledThreadPool(n);          // supports DELAYED and REPEATING task execution
```

**Choosing the right pool type is a real, practical decision**: `newFixedThreadPool` for predictable, bounded parallelism (especially CPU-bound work, where more threads than CPU cores generally doesn't help — Module 22 covers this trade-off in more depth); `newCachedThreadPool` for many short, I/O-bound tasks (though its **unbounded growth** under sustained heavy load is a genuine, real production risk worth being aware of); `newSingleThreadExecutor` when tasks must run one at a time, in order.

## `Future` — Retrieving a Result From Asynchronous Work

```java
ExecutorService executor = Executors.newFixedThreadPool(2);

Future<Integer> future = executor.submit(() -> {
    Thread.sleep(1000);
    return 42;              // this task RETURNS a value (a Callable, not a plain Runnable)
});

System.out.println("Doing other work while the task runs...");

Integer result = future.get();   // BLOCKS until the task completes, THEN returns its result
System.out.println("Result: " + result);
```

**`submit(...)` returns a `Future<T>`** — a handle representing a computation that **may not have finished yet**. `future.get()` blocks the calling thread until the result is actually available (conceptually similar to Topic 1's `join()`, but specifically for retrieving a **return value**, not just waiting for completion). **`Future.get()` also has an overload accepting a timeout** (`future.get(5, TimeUnit.SECONDS)`), throwing `TimeoutException` if the result isn't ready in time — a genuinely important, real safety mechanism to avoid waiting forever on a stuck task.

## Always `shutdown()` Your Executor

```java
ExecutorService executor = Executors.newFixedThreadPool(4);
try {
    // ... submit tasks ...
} finally {
    executor.shutdown();   // without this, the pool's threads keep the JVM ALIVE indefinitely!
}
```

**A genuinely important, real practical detail**: an `ExecutorService`'s worker threads are, by default, **non-daemon** threads (a distinction touched on in Module 07, Topic 5's GC discussion — the JVM won't exit while non-daemon threads are still alive) — **forgetting to call `shutdown()` means your program may never actually terminate**, even after all its real work is done, since the pool's idle worker threads remain alive, waiting for more tasks that will never come.

## Real-World Analogy

Think of `new Thread()` per task like **hiring and training a brand-new employee for every single customer who walks in, then firing them the moment that one customer leaves** — technically works, but the hiring/training/firing overhead, repeated constantly, is wasteful compared to simply having a **standing team of employees** (a thread pool) who handle customers one after another, reused continuously throughout the workday. `Future` is like a **claim ticket** you're handed when dropping off dry cleaning — you can go do other things, and come back later to redeem the ticket (`get()`) for your finished item, waiting only if it genuinely isn't ready yet.

## Advantages

- Reusing threads across many tasks eliminates the repeated creation/destruction overhead of a new-thread-per-task approach, directly addressing Module 14, Topic 3's C10K-style memory/scheduling concerns.
- `Future` provides a clean, standard way to retrieve results from asynchronous work, with built-in timeout support.
- Different pool types (fixed, cached, single, scheduled) match different real workload shapes.

## Disadvantages / Trade-offs

- `newCachedThreadPool`'s unbounded growth is a genuine, real production risk under sustained heavy load — it can create an unbounded number of threads if tasks arrive faster than they complete.
- Forgetting `shutdown()` is a common, real bug that silently prevents JVM exit.
- Choosing an inappropriate pool size for CPU-bound work (too many threads competing for limited CPU cores) can actually hurt performance rather than help it.

## Best Practices

- Always call `shutdown()` (or use try-with-resources with Java 19+'s `AutoCloseable` `ExecutorService`, a modern convenience) when done with an executor.
- Choose `newFixedThreadPool` with a size appropriate to available CPU cores for CPU-bound work; be cautious with `newCachedThreadPool` under genuinely high, sustained load.
- Use `Future.get(timeout, unit)` rather than the no-timeout overload, when waiting forever on a potentially-stuck task would be unacceptable.

## Common Mistakes

- Creating a raw `new Thread()` for every task instead of using a reusable pool, repeating Module 14's C10K-style overhead.
- Forgetting to call `shutdown()`, leaving the JVM unable to exit.
- Using `newCachedThreadPool` under sustained heavy load without understanding its unbounded thread-growth risk.
- Calling `future.get()` with no timeout on a task that could genuinely hang forever.

## Interview Questions

1. **Q: Why is creating a new `Thread` for every single task considered poor practice at scale?**
   A: Thread creation has real, measurable overhead (Stack allocation, OS scheduling registration), and having many threads simultaneously alive repeats the C10K-style memory/scheduling cost problem (Module 14, Topic 3) — reusing a managed pool of threads across many tasks avoids this repeated overhead entirely.

2. **Q: What does `ExecutorService.submit(...)` return, and what does calling `.get()` on it do?**
   A: It returns a `Future<T>`, a handle to a computation that may not have completed yet. Calling `.get()` blocks the calling thread until the result is available, then returns it — with an overloaded, timeout-accepting variant to avoid waiting indefinitely.

3. **Q: What happens if you forget to call `shutdown()` on an `ExecutorService`?**
   A: Its worker threads (non-daemon by default) remain alive indefinitely, waiting for more tasks — this can prevent the JVM from ever exiting, even after all actual application work has completed.

## Summary

- Creating a raw `new Thread()` per task repeats Module 14's C10K-style overhead; `ExecutorService` maintains a reusable pool of worker threads plus a task queue, avoiding repeated thread creation cost.
- Standard pool types: `newFixedThreadPool` (bounded, predictable), `newCachedThreadPool` (grows as needed, but can grow unboundedly), `newSingleThreadExecutor` (sequential execution), `newScheduledThreadPool` (delayed/repeating tasks).
- **`Future<T>`** represents a possibly-not-yet-complete asynchronous computation's result; `.get()` blocks until available (optionally with a timeout).
- Always call `shutdown()` — forgetting it can prevent the JVM from exiting.

## Exercises

1. Rewrite the 10,000-task loop from this topic's opening example using `Executors.newFixedThreadPool(4)`, submitting all tasks and calling `shutdown()` at the end.
2. Write code submitting a `Callable<Integer>` task to an executor, retrieving its result via `Future.get()` with a 2-second timeout, and handling the `TimeoutException` case.
3. Explain, referencing Module 14, Topic 3's C10K discussion, why reusing a pool of threads is more scalable than creating one thread per task.
4. Explain the specific risk of using `newCachedThreadPool` under sustained, very heavy load, and what pool type would be a safer choice instead.

---

**Previous:** [04 — Atomic Classes & CAS](04-Atomic-Classes-And-CAS.md) · **Next:** [06 — `CompletableFuture` & Async Programming](06-CompletableFuture-And-Async-Programming.md)
