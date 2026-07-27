# Module 15 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-Module-Overview.md) has been covered:

- [x] **Threads Fundamentals** — process vs. thread (extending Module 02's memory model), `Runnable` vs. `Thread`, `start()` vs. `run()`, the complete thread lifecycle, `join()`
- [x] **Race Conditions & the Java Memory Model** — the precise bytecode-level cause of `count++`'s non-atomicity, the separate visibility hazard, `happens-before`, `volatile`'s precise (and limited) guarantee
- [x] **Synchronization & Locks** — `synchronized`'s intrinsic lock mechanism (finally explaining Module 07's `wait`/`notify` preview), the `while`-loop `wait()` rule, `ReentrantLock`'s extra capabilities, deadlock and its prevention
- [x] **Atomic Classes & CAS** — lock-free thread safety via Compare-And-Swap, and its genuine trade-offs against locking
- [x] **Executors & Thread Pools** — why raw thread-per-task doesn't scale (Module 14's C10K, revisited), `ExecutorService`, pool types, `Future`
- [x] **`CompletableFuture` & Async Programming** — composable async chains, `thenApply`/`thenCompose`/`thenCombine`, exception handling
- [x] **Concurrent Collections** — `ConcurrentHashMap`'s fine-grained locking and atomic compound operations (delivering on Module 10's preview), `CopyOnWriteArrayList`, `BlockingQueue`
- [x] **Virtual Threads & Structured Concurrency** — the complete, mechanical resolution of Module 14's C10K story, and safe concurrent subtask management

## Practical Connections

- **Every Spring Boot web application handling concurrent HTTP requests** relies on exactly this module's thread pool concepts (Topic 5) — and, in Spring Boot 3+/Java 21+, increasingly on Virtual Threads (Topic 8) for dramatically improved request-handling scalability with simple, familiar blocking-style controller code.
- **Database connection pools** (HikariCP and similar) are themselves a specific application of Topic 5's thread-pool-style resource reuse principle, applied to connections instead of threads.
- **Microservices calling multiple downstream services in parallel** are a textbook `CompletableFuture`/Structured Concurrency use case (Topics 6, 8) — fetch from three services concurrently, combine results, handle partial failures cleanly.
- **In-memory caches** in real production services are built on `ConcurrentHashMap`'s `computeIfAbsent` pattern (Topic 7) constantly.
- **Kafka consumers, background job processors** — the entire "worker pool consuming from a queue" pattern is `ExecutorService` + `BlockingQueue` (Topics 5, 7), applied directly.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| Race condition vs. visibility problem | Race condition: multi-step operation interleaves across threads, losing updates. Visibility problem: a write isn't promptly observed by another thread — a separate hazard, only the latter fixed by `volatile` alone. |
| `synchronized` vs `ReentrantLock` | `synchronized`: automatic, exception-safe release, simpler. `ReentrantLock`: more flexible (`tryLock`, fairness, conditions), requires manual `finally`-block unlocking. |
| Locking vs. CAS/atomics | Locking: pessimistic, blocks other threads, no wasted retries. CAS: optimistic, never blocks, but can waste CPU on retries under high contention. |
| `Future` vs `CompletableFuture` | `Future`: can only be blocked on via `.get()`, no chaining. `CompletableFuture`: composable, chainable, non-blocking callbacks. |
| Platform thread vs Virtual Thread | Platform: one-to-one with a real, ~1MB-stack OS thread. Virtual: lightweight, JVM-managed, many share a small pool of carrier threads, transparently unmounted while blocked. |

## Consolidated Interview Questions (Module 15)

1. What's the difference between a process and a thread?
2. What actually happens if you call `thread.run()` instead of `thread.start()`?
3. Why is `count++` not thread-safe, even though it looks like one operation?
4. Does `volatile` make compound operations like `count++` thread-safe?
5. What does `synchronized` do mechanically, and why must `wait()` be in a `while` loop?
6. What can `ReentrantLock` do that `synchronized` cannot?
7. What causes deadlock, and how is it prevented?
8. How does `AtomicInteger` achieve thread safety without locking?
9. Why is creating a new `Thread` per task considered poor practice at scale?
10. What limitation of `Future` does `CompletableFuture` solve?
11. How does `ConcurrentHashMap` outperform `Collections.synchronizedMap`?
12. What is a Virtual Thread, and how does it resolve the C10K problem?
13. For what kind of workload do Virtual Threads offer no advantage?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** From memory, explain the difference between a race condition and a visibility problem, and which Java tools fix each.
2. **Hands-on:** Reproduce Topic 2's `count++` race condition with two threads incrementing a shared, non-synchronized counter one million times each — observe a final count less than 2,000,000. Fix it three ways: `synchronized`, `ReentrantLock`, and `AtomicInteger`.
3. **Hands-on:** Implement a producer-consumer pattern two ways: manually with `wait()`/`notifyAll()` (Topic 3), then with `BlockingQueue` (Topic 7) — compare code length and clarity.
4. **Hands-on:** Submit 100,000 short, I/O-simulating tasks (`Thread.sleep`) using `Executors.newVirtualThreadPerTaskExecutor()`, and observe that this completes without issue, unlike attempting the same with 100,000 raw platform threads.
5. **Conceptual:** Explain, referencing Module 14's C10K discussion, why Virtual Threads let you write simple blocking code that still scales — what is the JVM doing transparently underneath?
6. **Synthesis:** Design a small service method that fetches data from two independent "APIs" (simulated with `Thread.sleep` + a return value) concurrently using `CompletableFuture.thenCombine`, then rewrite it using Structured Concurrency, comparing the guarantees each approach provides.

## What's Next

Module 15 completed Java's concurrency story, from the fundamental hazards through modern Virtual Threads — genuinely one of the most consequential topics in professional Java development. **Module 16 — JVM Internals** now returns to the JVM itself, going deeper than Module 02: bytecode in full detail, garbage collection algorithms (Serial, Parallel, G1, ZGC), the Java Memory Model's full formal treatment, and Reflection — completing your understanding of what's actually happening beneath every line of Java code you write.

---

**Previous:** [08 — Virtual Threads & Structured Concurrency](08-Virtual-Threads-And-Structured-Concurrency.md) · **Module Overview:** [00 — Module Overview](00-Module-Overview.md)

**Type "Continue" to begin Module 16 — JVM Internals.**
