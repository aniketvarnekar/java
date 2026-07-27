# Race Conditions & the Java Memory Model

## Learning Objectives

- Understand precisely why `count++` is not atomic, with a bytecode-level trace
- Understand the visibility problem — a genuinely different, separate hazard from race conditions
- Understand the Java Memory Model's `happens-before` relationship at a working level
- Use `volatile` correctly, and know exactly what it does and does not guarantee

## Prerequisites

[01 — Threads Fundamentals](01-Threads-Fundamentals.md), Module 01 Topic 5 (bytecode), Module 02 Topic 4 (JIT — reordering optimizations)

## Motivation

This is the conceptual heart of the entire module. Every remaining topic — `synchronized`, atomics, concurrent collections, even Virtual Threads — exists specifically to solve the two hazards this topic explains precisely: **race conditions** and **visibility problems**. Without understanding *why* these happen at a mechanical level, every subsequent tool feels like arbitrary ceremony rather than a targeted solution.

## Hazard 1: Race Conditions — `count++` Is Not One Operation

```java
class Counter {
    int count = 0;
    void increment() {
        count++;
    }
}
```

**This looks like a single, atomic operation. It is genuinely NOT.** Recall Module 01, Topic 5: `count++` compiles to **multiple, separate bytecode instructions**:

```
getfield count      // 1. READ the current value of count
iconst_1              // 2. push the constant 1
iadd                    // 3. ADD them together
putfield count            // 4. WRITE the new value back to count
```

**If two threads both call `increment()` "simultaneously," their individual steps can interleave** — and this interleaving can genuinely lose an update:

```
 Initial: count = 0

 Thread A: getfield count  -> reads 0
                                             Thread B: getfield count -> reads 0  (ALSO reads 0!)
 Thread A: iadd (0+1=1)
                                             Thread B: iadd (0+1=1)
 Thread A: putfield count=1
                                             Thread B: putfield count=1   <- OVERWRITES A's update!

 FINAL RESULT: count = 1   (should have been 2 -- ONE increment was SILENTLY LOST!)
```

**This is a race condition**: the program's correctness depends on the precise, unpredictable **timing** of thread interleaving — sometimes it produces the correct answer (`2`), sometimes it silently produces a wrong one (`1`), and **which happens is not deterministic**, making race conditions notoriously difficult to reproduce, debug, and test for (a test run that "passes" ten times in a row proves nothing — the wrong interleaving might simply not have occurred during those ten runs).

## Hazard 2: The Visibility Problem — A Genuinely Different, Separate Hazard

**Even if an operation genuinely IS atomic (a single, indivisible step), a second, entirely separate hazard exists**: one thread's write to shared memory is **not guaranteed to be visible** to another thread **at all**, or **promptly**.

```java
class Flag {
    boolean running = true;
    void doWork() {
        while (running) {   // ⚠️ might loop FOREVER, even after another thread sets running = false!
            // ... work ...
        }
    }
    void stop() {
        running = false;
    }
}
```

**Why could this genuinely loop forever?** Modern CPUs have multiple **layers of caching** (Module 02's Execution Engine touches on JIT optimization; this is a related, hardware-level concern) — a thread reading `running` repeatedly in a tight loop might have that value cached in a CPU register or core-local cache, **never re-reading it from main, shared memory at all**, and therefore **never observing** another thread's write to it. **Additionally**, the JIT compiler (Module 02, Topic 4) is permitted to apply optimizations — including **reordering** instructions — that are perfectly safe for **single-threaded** correctness, but can produce genuinely surprising results when **multiple** threads are involved, since the compiler's reasoning about "safe reordering" doesn't automatically account for what other threads might be doing concurrently.

**This is a fundamentally different problem from the race condition above** — it's not about *interleaving* of multi-step operations; it's about whether a write is ever **observed** by another thread at all.

## The Java Memory Model (JMM) — The Formal Rulebook

> The **Java Memory Model** is the JVM specification's formal definition of exactly when one thread's writes to shared memory are **guaranteed** to be visible to another thread, and what reorderings of operations are (and aren't) permitted.

**The central concept: `happens-before`.** The JMM guarantees that if action A **happens-before** action B, then A's effects (including memory writes) are **guaranteed visible** to B. Critically, **without** an established `happens-before` relationship, the JVM/compiler/CPU are free to reorder, cache, or delay operations in ways that can produce genuinely surprising, non-intuitive results across threads — this is precisely the cause of the `Flag` example's potential infinite loop.

**Several actions establish a `happens-before` relationship** (a partial list, the ones most relevant at this stage):
- Every action within a single thread happens-before every subsequent action in **that same thread** (ordinary, expected sequential ordering, within one thread).
- **Unlocking** a `synchronized` block/method happens-before a subsequent **locking** of that **same** lock by another thread (Topic 3 — this is precisely *why* `synchronized` fixes both hazards simultaneously, not just the race-condition one).
- Writing to a **`volatile`** field happens-before a subsequent read of that **same** field by another thread (explained fully below).
- A thread calling `Thread.start()` happens-before any action in the started thread.
- Any action in a thread happens-before another thread successfully returning from `join()` on it.

## `volatile` — Guaranteeing Visibility (But NOT Atomicity!)

```java
class Flag {
    volatile boolean running = true;   // 'volatile' guarantees VISIBILITY
    void doWork() {
        while (running) {   // now CORRECTLY sees another thread's write, promptly
            // ... work ...
        }
    }
    void stop() {
        running = false;
    }
}
```

**`volatile` fixes the visibility problem specifically**: it establishes a `happens-before` relationship for that specific field — every write is guaranteed visible to subsequent reads by other threads, and the compiler is forbidden from certain reorderings/caching optimizations for that field specifically.

**Critical, commonly-tested nuance: `volatile` does NOT make compound operations atomic.**

```java
class Counter {
    volatile int count = 0;
    void increment() {
        count++;   // ⚠️ STILL NOT ATOMIC, even with volatile! The race condition from Hazard 1 REMAINS!
    }
}
```

**`volatile` solves the visibility problem, not the race-condition problem** — these are the two genuinely separate hazards this topic opened with, and `volatile` only addresses one of them. `count++` is still read-modify-write, still multiple separate steps, still capable of losing updates under interleaving — `volatile` merely guarantees that whatever value **is** read is the most recently written one, and that a completed write is promptly visible; it says nothing about protecting a multi-step read-modify-write sequence from interleaving with another thread's own multi-step sequence.

## Summary Table: The Two Hazards

| | Race Condition | Visibility Problem |
|---|---|---|
| Root cause | A multi-step operation (like `count++`) gets interleaved between threads | A thread doesn't (promptly, or ever) observe another thread's write |
| Fixed by `volatile` alone? | **No** | **Yes** |
| Fixed by `synchronized` alone? | **Yes** | **Yes** (it establishes happens-before too) |
| Symptom | Lost updates, inconsistent final values | Stale reads, infinite loops on a "stuck" flag |

**This table is the precise, mechanical reason `synchronized` (Topic 3) is often reached for by default**: a single tool that addresses **both** hazards simultaneously, whereas `volatile` addresses only one of the two.

## Real-World Analogy

Think of a **race condition** like **two people simultaneously reading the same whiteboard's balance ("$100"), each independently calculating "$100 + $10 = $110," and each writing "$110" back** — even though $20 total was actually deposited, the board only ever shows $110, because the second write completely overwrote the first, silently losing one deposit's effect. Think of the **visibility problem** like **one person writing an update on a sticky note that's stuck to the back of their own personal notebook**, rather than the shared whiteboard everyone else is actually looking at — from everyone else's perspective, nothing changed at all, even though the update genuinely happened, just somewhere no one else is currently looking.

## Advantages of Understanding This Precisely

- Distinguishing race conditions from visibility problems lets you diagnose real concurrency bugs correctly, rather than reaching for `volatile` (or `synchronized`) by guesswork.
- Understanding `happens-before` gives you a genuine, formal framework for reasoning about concurrent correctness, rather than relying on intuition (which is frequently and famously wrong for concurrent code).

## Disadvantages / Trade-offs

- This is genuinely one of the harder conceptual topics in all of Core Java — race conditions and visibility problems are often non-deterministic and can be difficult to reproduce reliably even once understood.
- `volatile`'s narrow guarantee (visibility only, not atomicity) is a common, real source of subtle bugs when developers assume it solves more than it actually does.

## Best Practices

- Never assume a compound operation (`count++`, "check-then-act" patterns) is atomic just because it looks like one line of code.
- Use `volatile` specifically and only for simple flag-style fields where visibility alone is the actual concern (no multi-step read-modify-write involved).
- When in doubt about whether a shared, mutable operation is safe across threads, default to `synchronized` (Topic 3) or a higher-level concurrent tool (Topics 4, 7), rather than guessing.

## Common Mistakes

- Assuming `volatile` makes `count++`-style compound operations thread-safe — it doesn't; it only guarantees visibility, not atomicity.
- Writing concurrent code without any coordination mechanism at all, assuming "it probably won't happen" for a race condition — non-deterministic bugs frequently pass casual testing while remaining genuinely present.
- Confusing "the code looks like one line" with "the operation is atomic" — bytecode-level step count (Module 01, Topic 5) is what actually matters.

## Interview Questions

1. **Q: Why is `count++` not thread-safe, even though it looks like a single operation?**
   A: It compiles to multiple separate bytecode instructions (read, add, write) — if two threads interleave these steps, one thread's update can be silently overwritten by another's, based on the same stale read value, losing an increment without any exception or error.

2. **Q: What's the difference between a race condition and a visibility problem?**
   A: A race condition arises from interleaving of a multi-step operation across threads, causing lost or inconsistent updates. A visibility problem arises when one thread's write to shared memory isn't promptly (or ever) observed by another thread, due to caching/reordering — a genuinely separate hazard, not about interleaving at all.

3. **Q: Does `volatile` make a field's compound operations (like `count++`) thread-safe?**
   A: No — `volatile` guarantees visibility (a write is promptly visible to subsequent reads by other threads) but does not guarantee atomicity for multi-step operations; `count++` on a `volatile` field remains a genuine race condition.

4. **Q: What does `happens-before` mean in the Java Memory Model?**
   A: A formal guarantee that if action A happens-before action B, A's effects (including memory writes) are guaranteed visible to B. Without an established happens-before relationship between threads, the JVM/compiler/CPU are free to reorder or delay visibility of operations in ways that can produce surprising cross-thread results.

## Summary

- **Race conditions**: multi-step operations (like `count++`, actually 4 bytecode instructions) can interleave across threads, silently losing updates — a timing-dependent, non-deterministic bug.
- **Visibility problems**: a genuinely separate hazard — a thread's write to shared memory may not be promptly (or ever) visible to another thread, due to CPU caching and JIT reordering optimizations.
- The **Java Memory Model**'s `happens-before` relationship formally defines when cross-thread visibility is guaranteed.
- **`volatile`** guarantees visibility for a specific field, but does **not** make compound operations atomic — it solves only one of the two hazards.

## Exercises

1. Trace, step by step (in this topic's interleaving-diagram style), a scenario where two threads calling `increment()` on a non-volatile, non-synchronized `Counter` produce a final count of `1` instead of the expected `2`.
2. Explain, precisely, why `volatile` fixes the `Flag`/`running` infinite-loop example but would NOT fix the `Counter`/`count++` race condition — reference the specific hazard each addresses.
3. Explain what `happens-before` means, and name two specific actions that establish a happens-before relationship between threads.

---

**Previous:** [01 — Threads Fundamentals](01-Threads-Fundamentals.md) · **Next:** [03 — Synchronization & Locks](03-Synchronization-And-Locks.md)
