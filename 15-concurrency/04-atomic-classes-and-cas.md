# Atomic Classes & CAS

## Learning Objectives

- Use `AtomicInteger` and related atomic classes correctly
- Understand Compare-And-Swap (CAS) precisely, at the hardware-instruction level
- Understand why lock-free approaches can outperform locking under contention, and their real trade-offs

## Prerequisites

[03 — Synchronization & Locks](03-synchronization-and-locks.md)

## Motivation

Locking (Topic 3) genuinely solves both hazards from Topic 2 — but at a real cost: threads waiting for a lock make zero progress. This topic covers Java's alternative approach for simple cases: achieving thread safety **without locking at all**, using a hardware-level primitive.

## `AtomicInteger` and Friends — Solving `count++` Without Locking

```java
import java.util.concurrent.atomic.AtomicInteger;

class Counter {
    private final AtomicInteger count = new AtomicInteger(0);

    void increment() {
        count.incrementAndGet();   // GENUINELY atomic -- no synchronized needed at all!
    }

    int getCount() {
        return count.get();
    }
}
```

**This directly solves Topic 2's exact `count++` race condition**, without any `synchronized` block or explicit `Lock` — `incrementAndGet()` is guaranteed atomic as a **single, indivisible** operation, no matter how many threads call it concurrently. The `java.util.concurrent.atomic` package provides `AtomicInteger`, `AtomicLong`, `AtomicBoolean`, `AtomicReference<T>` (for arbitrary object references), each offering atomic read-modify-write operations (`incrementAndGet`, `getAndIncrement`, `compareAndSet`, `updateAndGet`, and more) without locking.

## How This Actually Works — Compare-And-Swap (CAS)

**This is the deepest, most important mechanism in this topic.** Atomic classes achieve thread safety using a special **hardware CPU instruction** (available on essentially all modern processors) called **Compare-And-Swap (CAS)**:

> **CAS(location, expectedValue, newValue)**: atomically checks whether `location`'s current value equals `expectedValue`; if so, updates it to `newValue` and reports success — **all as a single, indivisible hardware operation, guaranteed by the CPU itself, no software-level locking involved at all.** If the current value does **not** match `expectedValue` (meaning some other thread changed it in the meantime), the operation reports **failure**, changing nothing.

```java
// Conceptually, incrementAndGet() does something like this internally:
int oldValue, newValue;
do {
    oldValue = count.get();               // 1. READ the current value
    newValue = oldValue + 1;                // 2. COMPUTE the new value
} while (!compareAndSet(oldValue, newValue));   // 3. Try to atomically apply it --
                                                     //    ONLY succeeds if nothing else
                                                     //    changed 'count' since step 1!
                                                     //    if it FAILS, LOOP and retry
```

**This is called an "optimistic," lock-free approach**: rather than **preventing** other threads from touching the value at all (locking's pessimistic approach — "block everyone else until I'm done"), CAS **optimistically assumes no conflict will occur**, attempts the update, and **retries** if it turns out another thread beat it to the update in the meantime. **No thread is ever blocked waiting** — every thread is always either making progress or immediately retrying, never sitting idle waiting for a lock to be released.

```
 LOCKING approach:                          CAS approach:

 Thread A: acquires lock                     Thread A: reads value, computes new value,
 Thread B: BLOCKED, waiting...                          attempts CAS -- SUCCEEDS (no conflict)
 Thread A: does work, releases lock          Thread B: reads value, computes new value,
 Thread B: (finally) acquires lock,                     attempts CAS -- FAILS (A already changed it!)
           does its own work                            RETRIES: reads the NEW value, recomputes,
                                                          attempts CAS again -- SUCCEEDS this time
 Thread B was COMPLETELY IDLE while         Thread B was ALWAYS actively computing/retrying,
 waiting for the lock                        NEVER blocked/idle
```

## Why CAS Can Outperform Locking Under Certain Conditions

**Under low-to-moderate contention** (threads rarely actually collide on the same operation at the same instant), CAS-based atomics are typically **significantly faster** than locking — there's no OS-level thread-blocking/waking overhead at all (which is genuinely expensive — Module 02's Execution Engine touches on context-switching costs), and most CAS attempts simply **succeed on the first try**, with retries being comparatively rare.

**Under very high contention** (many threads constantly fighting over the exact same value), CAS-based approaches can actually perform **worse** than locking — many threads end up retrying repeatedly (a phenomenon sometimes called "livelock-adjacent" busy-spinning), each burning real CPU cycles on failed attempts, rather than simply waiting quietly (as a blocked, locked thread does, consuming essentially no CPU while waiting).

## Real-World Analogy

Think of locking like a **single-key conference room** — only the person holding the key can enter; everyone else simply waits patiently outside, doing nothing, until the key is returned. Think of CAS like **everyone attempting to sign a shared sign-up sheet simultaneously, optimistically** — you write your name assuming the next empty line is still empty, but if you glance back and discover someone else already filled that exact line while you were writing, you simply cross out your attempt and retry on the next available line — no one is ever forced to stand around doing nothing, but if the sheet is extremely popular and crowded (very high contention), a lot of people end up repeatedly scratching out and retrying, which itself becomes wasteful.

## Advantages

- Genuinely lock-free — no thread is ever blocked waiting, eliminating both the OS-level context-switching cost of blocking and any deadlock risk entirely (there's no lock to deadlock on).
- Typically faster than locking under low-to-moderate contention, a common real-world scenario.
- Simple, focused API (`AtomicInteger`, etc.) for the common case of a single, independently-updated value.

## Disadvantages / Trade-offs

- Under very high contention, repeated CAS retries can waste more CPU than a simple, quiet lock-wait would.
- Atomic classes handle **one single value's** atomicity well, but don't directly help coordinate **multiple, related** values that need to change together consistently — that generally still requires locking (Topic 3) or a higher-level concurrent structure (Topic 7).

## Best Practices

- Use `AtomicInteger`/`AtomicLong`/`AtomicBoolean`/`AtomicReference` for simple, single-value counters/flags/references updated concurrently — a common, everyday, low-friction thread-safety solution.
- Reach for locking (Topic 3) when multiple related values must be updated together, consistently, as a unit — atomics alone don't coordinate across multiple fields.
- Don't assume CAS-based atomics are always strictly faster than locking — under genuinely high contention, this assumption can be wrong; measure for your specific workload if it matters.

## Common Mistakes

- Assuming atomic classes solve every concurrency problem — they only provide atomicity for a single value's own operations, not coordination across multiple related fields.
- Using `synchronized` for a simple single-counter increment when a much lighter-weight `AtomicInteger` would suffice.
- Assuming lock-free is unconditionally "better" than locking — it's a genuine trade-off, favorable specifically under low-to-moderate contention.

## Interview Questions

1. **Q: How does `AtomicInteger.incrementAndGet()` achieve thread safety without using `synchronized`?**
   A: It uses the CPU's Compare-And-Swap (CAS) hardware instruction — atomically checking whether the value is still what was last read, and if so, updating it; if another thread changed it in the meantime, the operation fails and retries with the new current value, in a loop, until it succeeds. No software-level locking or thread-blocking is involved at all.

2. **Q: What is Compare-And-Swap (CAS), precisely?**
   A: A hardware-level atomic instruction that checks whether a memory location's current value equals an expected value, and if so, updates it to a new value — all as one indivisible operation. If the current value doesn't match (another thread changed it since it was last read), the operation fails, reporting that the caller should retry.

3. **Q: When might locking actually outperform a CAS-based atomic approach?**
   A: Under very high contention, where many threads constantly collide on the same value — CAS-based retries burn CPU cycles repeatedly failing and retrying, whereas a blocked, lock-waiting thread consumes essentially no CPU while quietly waiting its turn.

## Summary

- **Atomic classes** (`AtomicInteger`, etc.) provide genuinely atomic operations without locking, directly solving Topic 2's race-condition hazard for single values.
- They work via **Compare-And-Swap (CAS)**, a hardware-level instruction that atomically checks-and-updates a value, retrying (in a loop) if another thread changed it first — an "optimistic," lock-free approach.
- CAS typically outperforms locking under low-to-moderate contention (no blocking overhead); locking can outperform CAS under very high contention (avoiding wasteful repeated retries).
- Atomics handle single-value atomicity well, but don't coordinate multiple related fields — that still requires locking or higher-level concurrent tools (Topic 7).

## Exercises

1. Rewrite the `Counter` class from Topics 2–3 using `AtomicInteger` instead of `synchronized`, and explain why this achieves the same thread-safety guarantee for `count++` without any explicit locking.
2. Explain, in your own words, the CAS retry loop's internal logic — what happens when the compare-and-swap attempt fails?
3. Explain a concrete scenario where a single `AtomicInteger` would NOT be sufficient to guarantee correctness — specifically, one where two related fields must be updated together consistently — and explain why locking would be needed instead.

---

**Previous:** [03 — Synchronization & Locks](03-synchronization-and-locks.md) · **Next:** [05 — Executors & Thread Pools](05-executors-and-thread-pools.md)
