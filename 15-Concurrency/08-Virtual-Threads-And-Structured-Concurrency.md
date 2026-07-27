# Virtual Threads & Structured Concurrency

## Learning Objectives

- Understand precisely how Virtual Threads differ from platform (OS) threads
- Understand why Virtual Threads make the simple, blocking thread-per-task style scalable again
- Use Structured Concurrency to manage groups of related concurrent tasks safely
- Fully close the loop opened in Module 14, Topic 3

## Prerequisites

Module 14 Topic 3 (the C10K problem, Virtual Threads preview), [01 — Threads Fundamentals](01-Threads-Fundamentals.md), [05 — Executors & Thread Pools](05-Executors-And-Thread-Pools.md)

## Motivation

This is the payoff topic — both for this module and for the promise made all the way back in Module 14, Topic 3. Virtual Threads (finalized in Java 21) are one of the most significant changes to how Java concurrency is written in the language's history, and they directly, completely resolve the C10K-style scalability problem, using an approach genuinely different from both classic threads (Topic 1) and raw NIO (Module 14).

## Recap: The Problem (From Module 14, Topic 3)

**Platform threads** (the `Thread` objects you've used throughout this module until now) map **one-to-one** to actual **operating system threads**. Each one requires a real OS-level thread (with a real Stack allocation, typically ~1MB by default, and real OS scheduler overhead). This is precisely why the thread-per-connection model hits a genuine wall around thousands of concurrent connections (the C10K problem) — and why, historically, achieving true massive scalability required either raw NIO's complex event-loop style (Module 14, Topic 3) or reactive/async frameworks built on similar principles.

## Virtual Threads — A Fundamentally Different Kind of Thread

```java
Thread virtualThread = Thread.ofVirtual().start(() -> {
    System.out.println("Running on: " + Thread.currentThread());
});

// Or, via an executor specifically designed for virtual threads:
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 100_000; i++) {   // 100,000 tasks -- genuinely, comfortably feasible!
        executor.submit(() -> {
            Thread.sleep(Duration.ofSeconds(1));   // BLOCKING code -- exactly Topic 1's simple style!
            return processTask();
        });
    }
}   // try-with-resources (Module 12, Topic 4) -- ExecutorService implements AutoCloseable since Java 19
```

**A Virtual Thread is a lightweight thread managed entirely by the JVM, NOT mapped one-to-one to an OS thread.** Instead, the JVM runs **many** Virtual Threads on top of a much smaller pool of actual OS ("platform") threads — called **"carrier threads"** — automatically and transparently **unmounting** a Virtual Thread from its carrier whenever it blocks (on I/O, `sleep()`, lock acquisition, etc.), freeing that carrier thread to run a **different** Virtual Thread in the meantime, and **remounting** the original Virtual Thread onto a (possibly different) carrier thread once it's ready to resume.

```
                  A SMALL POOL OF REAL OS ("CARRIER") THREADS
                  ┌──────────┐  ┌──────────┐  ┌──────────┐
                  │ Carrier 1  │  │ Carrier 2  │  │ Carrier 3  │      (maybe just a handful,
                  └────┬─────┘  └────┬─────┘  └────┬─────┘       often ~ number of CPU cores)
                       │              │              │
              ┌────────┴────┐  ┌─────┴──────┐  ┌────┴───────┐
              ▼             ▼  ▼            ▼  ▼            ▼
        VirtualThread1  VirtualThread2  VirtualThread3  ... VirtualThread100000

  When VirtualThread1 BLOCKS (e.g., Thread.sleep, waiting on I/O):
     -> it's TRANSPARENTLY UNMOUNTED from Carrier 1
     -> Carrier 1 is now FREE to run a DIFFERENT waiting Virtual Thread
     -> when VirtualThread1's block condition resolves, it's REMOUNTED
        onto (possibly a DIFFERENT) available carrier thread, and resumes
```

**This is the complete, mechanical answer to the C10K problem, delivered in a genuinely different way than Module 14's raw NIO approach**: instead of you manually writing non-blocking, event-loop-style code (Module 14, Topic 3's `Selector` pattern) to avoid tying up a scarce OS thread while waiting, **the JVM does this transparently for you**, while you write **simple, ordinary, blocking-style code** — exactly Topic 1's straightforward thread-per-task style, now genuinely scalable to hundreds of thousands of concurrent tasks.

## Why Virtual Threads Are So Lightweight

Virtual Thread Stacks are **not** fixed ~1MB allocations the way platform thread Stacks are — they start small and **grow/shrink dynamically** as needed, stored on the **Heap** (Module 02, Topic 3) rather than requiring a dedicated, fixed-size OS-level Stack allocation. **This is precisely why you can have hundreds of thousands of Virtual Threads where you could only ever have a few thousand platform threads** — the per-thread memory cost is dramatically lower, and there's no OS-level scheduling overhead per Virtual Thread at all (only the much smaller number of carrier threads are actually scheduled by the OS).

## Virtual Threads vs. Platform Threads — Full Comparison

| | Platform Thread | Virtual Thread |
|---|---|---|
| Maps to | One OS thread, always | Many share a small pool of OS "carrier" threads |
| Stack | Fixed size (~1MB default), OS-allocated | Small, dynamically resizable, Heap-allocated |
| Creation cost | Real, measurable | Extremely cheap — creating thousands is routine |
| Practical scalability ceiling | Thousands (the C10K problem, Module 14) | Hundreds of thousands to millions |
| Blocking behavior | Blocks the underlying OS thread entirely | Transparently unmounts, freeing the carrier thread |
| Appropriate for | CPU-bound work, or a small, fixed number of long-lived threads | High-volume, typically I/O-bound, short-to-medium-lived tasks (exactly server-request-handling's classic shape) |

**Important, honest nuance**: Virtual Threads are specifically optimized for workloads dominated by **blocking I/O** (network calls, database queries, file access) — precisely the shape of most typical server request-handling code. **For CPU-bound work**, Virtual Threads offer no particular advantage over platform threads (there's no "blocking" moment to unmount during, since the thread is continuously, actively computing) — Module 22 revisits this distinction with concrete performance guidance.

## The Complete Resolution of Module 14's Story

Recall Module 14, Topic 3's honest trade-off table (simple blocking code vs. scalable but complex NIO event loops). **Virtual Threads genuinely deliver both simultaneously**:

```
 MODULE 14's OLD TRADE-OFF:                    VIRTUAL THREADS' RESOLUTION:

 Simple blocking code    -->  doesn't scale     Simple blocking code   -->  DOES scale
 (platform threads)           (C10K problem)     (Virtual Threads)          (JVM handles it internally)

 Scales well             -->  complex to write   [You just write simple code --
 (raw NIO Selector)           (event loops)        the JVM does the equivalent of
                                                     Selector-style scheduling FOR you,
                                                     transparently, underneath]
```

**This is the single most important practical takeaway from this entire module**: for the overwhelming majority of new, I/O-bound concurrent Java code (web servers, database-heavy services), Virtual Threads let you write straightforward, easy-to-read, easy-to-debug blocking-style code (Topic 1's original style) while achieving the scalability that historically required Module 14's much more complex NIO approach.

## Structured Concurrency (Java 21+, Preview/Evolving) — Managing Groups of Related Tasks

A closely related, complementary feature: **Structured Concurrency** enforces that a group of related concurrent subtasks are treated as a **single unit of work** — they're all started together, and the parent task **cannot proceed** until all of them have either completed or been properly, deliberately handled (including cancellation), preventing subtasks from silently "leaking" beyond their logical parent scope:

```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Supplier<String> user = scope.fork(() -> fetchUser());        // forked SUBTASKS,
    Supplier<Integer> orders = scope.fork(() -> fetchOrderCount()); // running concurrently

    scope.join();             // waits for BOTH subtasks (or a failure from either)
    scope.throwIfFailed();      // if EITHER subtask failed, propagate that failure NOW,
                                   // cleanly, rather than silently continuing with partial data

    System.out.println(user.get() + " has " + orders.get() + " orders");
}   // the SCOPE guarantees no subtask can outlive this block -- genuinely structured, not "fire and forget"
```

**Why does this matter, precisely?** Recall Topic 6's `CompletableFuture.thenCombine` — it combines two futures' results, but doesn't inherently guarantee **failure** in one automatically, cleanly cancels the other, or prevents subtasks from silently continuing to run in the background beyond their logical parent's own lifetime. **Structured Concurrency provides exactly this discipline**: a clear, enforced parent-child relationship between a task and its concurrent subtasks — directly analogous to how proper block-scoping (Module 04) prevents a variable from "leaking" outside the block it's declared in, now applied to **concurrent task lifetimes** instead of variable scopes.

## Real-World Analogy

Think of platform threads like **dedicated, full-time employees, each requiring their own office** — expensive to hire in large numbers, but each is a genuinely separate, real physical presence. Think of Virtual Threads like a **highly efficient hotdesking system** — thousands of "workers" (Virtual Threads) share a much smaller pool of actual desks (carrier threads), automatically stepping away from their desk the moment they're waiting on something (a phone call, a document to print) so someone else can use that desk in the meantime, then reclaiming any available desk the instant they're ready to continue — from each worker's own perspective, they experience simple, uninterrupted, sequential work, even though the actual physical desk they're using changes transparently behind the scenes. Structured Concurrency is like a **project manager who refuses to close out a project until every single subtask they delegated has genuinely finished or been explicitly cancelled** — no subtask is ever allowed to keep running silently, unaccounted for, after the project itself is supposedly "done."

## Advantages

- Virtual Threads let simple, readable, blocking-style code scale to hundreds of thousands of concurrent tasks, resolving Module 14's C10K problem without requiring complex event-loop-style code.
- Existing code, tools, and debuggers largely work with Virtual Threads the same way they do with platform threads — a genuinely significant adoption advantage over the fundamentally different, harder-to-debug reactive/async programming styles.
- Structured Concurrency prevents concurrent subtasks from silently outliving their logical parent scope, a real, meaningful safety improvement.

## Disadvantages / Trade-offs

- Virtual Threads offer no particular benefit for genuinely CPU-bound work — the advantage is specifically for blocking, I/O-bound workloads.
- Some existing libraries/patterns relying on `ThreadLocal` or certain forms of `synchronized` usage can interact with Virtual Threads in ways that reduce (though don't eliminate) their scalability benefit — a real, evolving area of the ecosystem's adaptation, worth being aware of for production use.
- Structured Concurrency's API was still evolving/in preview as of this course's writing — check current JDK documentation for its exact finalized form.

## Best Practices

- Default to Virtual Threads (`Executors.newVirtualThreadPerTaskExecutor()`) for new, I/O-bound, high-concurrency server-style code in Java 21+.
- Continue using platform threads (via `newFixedThreadPool`, Topic 5) for genuinely CPU-bound work, where Virtual Threads offer no advantage.
- Use Structured Concurrency when managing groups of related concurrent subtasks that should logically succeed or fail together.

## Common Mistakes

- Assuming Virtual Threads speed up CPU-bound computation — they don't; their benefit is specifically for blocking/I/O-bound workloads.
- Continuing to hand-write complex NIO `Selector` event-loop code for new projects, unaware Virtual Threads now offer a simpler path to the same scalability for most typical use cases.
- Using `CompletableFuture`-style "fire and forget" task submission where Structured Concurrency's stronger, enforced parent-child task lifetime guarantee would be more appropriate and safer.

## Interview Questions

1. **Q: What is a Virtual Thread, and how does it differ from a platform thread?**
   A: A Virtual Thread is a lightweight thread managed entirely by the JVM, not mapped one-to-one to an OS thread — many Virtual Threads share a small pool of actual OS "carrier" threads, with the JVM transparently unmounting a Virtual Thread from its carrier when it blocks (freeing the carrier for other work) and remounting it once ready to resume. Platform threads always map one-to-one to a real, comparatively expensive OS thread.

2. **Q: How do Virtual Threads resolve the C10K problem without requiring the complex, event-loop-style code raw NIO needed?**
   A: They let you write simple, ordinary, blocking-style code (the readable style from Topic 1), while the JVM internally performs the equivalent of non-blocking scheduling (unmounting/remounting Virtual Threads from carrier threads as they block/resume) — achieving NIO-level scalability without requiring you to manually write NIO-style event loops.

3. **Q: For what kind of workload do Virtual Threads offer no particular advantage?**
   A: Genuinely CPU-bound work — since Virtual Threads' benefit comes specifically from transparently freeing a carrier thread while a Virtual Thread is blocked (waiting on I/O, sleep, locks), and CPU-bound work has no such blocking moments to take advantage of.

4. **Q: What problem does Structured Concurrency solve that `CompletableFuture` alone doesn't guarantee?**
   A: It enforces that a group of related concurrent subtasks are treated as a single unit — the parent task cannot proceed until all subtasks have completed or been handled (including coordinated cancellation on failure), preventing subtasks from silently continuing to run beyond their logical parent's own scope/lifetime.

## Summary

- **Virtual Threads** (Java 21) are lightweight, JVM-managed threads, many sharing a small pool of real OS "carrier" threads — the JVM transparently unmounts a blocked Virtual Thread, freeing its carrier for other work, and remounts it upon resumption.
- This lets simple, readable, blocking-style code (Topic 1's original style) scale to hundreds of thousands of concurrent I/O-bound tasks, fully resolving Module 14's C10K problem without requiring hand-written NIO event-loop complexity.
- Virtual Threads specifically benefit blocking/I/O-bound workloads, not CPU-bound computation.
- **Structured Concurrency** enforces a clean parent-child relationship for groups of related concurrent subtasks, preventing them from silently outliving their logical scope.

## Module-Wide Quick Revision

- Threads share their process's Heap but have private Stacks (Module 02); prefer `Runnable` over extending `Thread`; always `start()`, never call `run()` directly (Topic 1).
- Race conditions (multi-step operations interleaving) and visibility problems (writes not promptly observed) are two genuinely separate hazards; `volatile` fixes only visibility, not atomicity (Topic 2).
- `synchronized` fixes both hazards via intrinsic locks; `wait()` must always be in a `while` loop; `ReentrantLock` offers more flexibility at the cost of manual unlocking; consistent lock ordering prevents deadlock (Topic 3).
- Atomic classes achieve lock-free thread safety via CAS, typically outperforming locking under low-to-moderate contention (Topic 4).
- `ExecutorService`/thread pools reuse threads instead of creating one per task, directly addressing C10K-style overhead; always `shutdown()` (Topic 5).
- `CompletableFuture` enables composable, non-blocking async chains (`thenApply`/`thenCompose`/`thenCombine`) with built-in exception handling (Topic 6).
- `ConcurrentHashMap`'s fine-grained locking and atomic compound operations outperform `synchronizedMap`; `CopyOnWriteArrayList` suits read-heavy data; `BlockingQueue` implements producer-consumer correctly (Topic 7).
- Virtual Threads let simple blocking code scale to huge concurrency, resolving the C10K problem; Structured Concurrency enforces safe subtask lifetimes (this topic).

## Common Pitfalls (Module-Wide)

- Calling `run()` instead of `start()`.
- Assuming `volatile` makes compound operations atomic.
- Calling `wait()` inside `if` instead of `while`.
- Forgetting `unlock()` in `finally` with `ReentrantLock`.
- Creating a raw thread per task instead of using a pool.
- Forgetting `ExecutorService.shutdown()`.
- Using `synchronizedMap`/check-then-act instead of `ConcurrentHashMap`'s atomic methods.
- Assuming Virtual Threads help CPU-bound work.

## Mini Quiz (Module-Wide)

1. Why is `count++` not atomic?
2. What does `volatile` guarantee, and what does it NOT guarantee?
3. Why must `wait()` be called in a `while` loop?
4. What is Compare-And-Swap?
5. Why does `ConcurrentHashMap` outperform `synchronizedMap`?
6. How do Virtual Threads achieve massive scalability with simple blocking code?

*(Answers are derivable from Topics 2, 2, 3, 4, 7, and this topic, respectively.)*

---

**Previous:** [07 — Concurrent Collections](07-Concurrent-Collections.md) · **Next:** [09 — Module Summary, Interview Questions & Exercises](09-Module-Summary-Exercises.md)
