# Module 15 — Concurrency

## Module Goal

This module has been previewed constantly throughout the course — `Vector`/`Hashtable`'s obsolete synchronization (Module 10), `StringBuffer` vs `StringBuilder` (Module 08), `wait`/`notify` living on `Object` (Module 07), `ConcurrentHashMap` (Module 10, Topic 8), and NIO's entire C10K/Selector story (Module 14, Topic 3). This is the module where every one of those threads (pun intended) gets pulled together into a complete, coherent understanding: how threads actually work, why concurrent code is genuinely hard to get right, the tools Java provides to make it tractable, and — closing the loop from Module 14 — Java 21's Virtual Threads and Structured Concurrency in full depth.

## Topics Covered in This Module

1. **[Threads Fundamentals](01-threads-fundamentals.md)** — process vs. thread, creating and starting threads, the thread lifecycle.
2. **[Race Conditions & the Java Memory Model](02-race-conditions-and-the-java-memory-model.md)** — why concurrent code breaks in ways sequential code never does, and the JMM's `happens-before` model.
3. **[Synchronization & Locks](03-synchronization-and-locks.md)** — `synchronized`, intrinsic locks/monitors, `wait`/`notify` (finally, in full), and `Lock`/`ReentrantLock`.
4. **[Atomic Classes & CAS](04-atomic-classes-and-cas.md)** — lock-free concurrency via compare-and-swap.
5. **[Executors & Thread Pools](05-executors-and-thread-pools.md)** — `ExecutorService`, thread pools, and why you should almost never call `new Thread()` directly.
6. **[`CompletableFuture` & Async Programming](06-completablefuture-and-async-programming.md)** — composing asynchronous work.
7. **[Concurrent Collections](07-concurrent-collections.md)** — `ConcurrentHashMap` in full depth, `CopyOnWriteArrayList`, `BlockingQueue`.
8. **[Virtual Threads & Structured Concurrency](08-virtual-threads-and-structured-concurrency.md)** — Java 21's landmark concurrency model, delivering fully on Module 14's preview.
9. **[Module Summary, Interview Questions & Exercises](09-module-summary-exercises.md)** — consolidated recap, quiz, and practice problems.

## Prerequisites

- Module 02 (JVM), especially Topic 3 (per-thread JVM Stack) and Topic 4 (Execution Engine).
- Module 07 (Objects), Topic 1 (`wait`/`notify` preview).
- Module 10 (Collections), Topic 8 (`ConcurrentHashMap` preview).
- Module 14 (NIO), Topic 3 (the C10K problem and Virtual Threads preview).

## How to Study This Module

Topic 2 is the conceptual core of this entire module — until you understand *why* naive concurrent code breaks (not just "use synchronized"), every subsequent tool will feel like arbitrary ceremony. Topics 3–4 give you the classical toolkit; Topics 5–7 give you the practical, everyday tools real applications actually use; Topic 8 is the payoff — modern Java's answer to making concurrent programming dramatically simpler.

---

**Previous module:** [14 — NIO](../14-nio/00-module-overview.md) · **Next:** [01 — Threads Fundamentals](01-threads-fundamentals.md)
