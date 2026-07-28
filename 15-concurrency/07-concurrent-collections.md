# Concurrent Collections

## Learning Objectives

- Understand `ConcurrentHashMap`'s internals precisely, and why it outperforms `Collections.synchronizedMap`
- Understand `CopyOnWriteArrayList`'s trade-offs and appropriate use cases
- Use `BlockingQueue` for producer-consumer coordination without manual `wait`/`notify`

## Prerequisites

Module 10 Topic 8 (`Collections.synchronizedMap` preview), [03 — Synchronization & Locks](03-synchronization-and-locks.md)

## Motivation

Module 10, Topic 8 promised this module would deliver `ConcurrentHashMap`'s full mechanics — that promise is fulfilled here. This topic completes your Collections Framework knowledge with the specific, purpose-built tools for concurrent access, each solving a genuinely different concurrent-access pattern.

## `ConcurrentHashMap` — Fine-Grained Locking, Not One Big Lock

Recall Module 10, Topic 8's preview: `Collections.synchronizedMap(new HashMap<>())` wraps **every single method call** in synchronization on **one single lock** — meaning even two threads accessing **completely unrelated** keys must still wait for each other, since they're both contending for that one shared lock.

**`ConcurrentHashMap` takes a fundamentally different, far more scalable approach**: instead of one lock guarding the entire map, it uses **fine-grained locking** — historically, locking small segments/buckets of the underlying hash table independently (in modern JDK versions, refined even further, down to per-bin synchronization combined with CAS operations, Topic 4, for many operations). **The practical result: two threads accessing different parts of the map can proceed genuinely concurrently**, without contending for the same lock at all — only threads that happen to touch the **same specific bucket** need to coordinate.

```
 synchronizedMap (ONE lock for the ENTIRE map):        ConcurrentHashMap (fine-grained locking):

 Thread A: put("apple", 1)   -- needs THE lock          Thread A: put("apple", 1)    -- locks only
 Thread B: put("zebra", 2)   -- WAITS for A's lock!                                      apple's bucket
                                                          Thread B: put("zebra", 2)     -- locks only
                                                                                            zebra's bucket
                                                                                            (DIFFERENT bucket --
                                                                                             proceeds CONCURRENTLY,
                                                                                             no waiting at all!)
```

**This directly, concretely answers Module 10, Topic 8's "why is `synchronizedMap` largely superseded" question** — `ConcurrentHashMap` provides genuinely better real-world concurrent throughput, precisely because it doesn't force unrelated operations to serialize behind one single lock.

## Atomic Compound Operations — Solving the Check-Then-Act Problem

Recall Module 10, Topic 8's other criticism of `synchronizedMap`: individual method calls are synchronized, but **multi-step, compound operations** (check-if-present, then-add-if-not) are **not** automatically safe as a unit:

```java
// UNSAFE, even on a synchronizedMap -- two separate method calls, a race condition between them:
if (!map.containsKey(key)) {   // Thread A checks: not present...
                                  // ⚠️ Thread B could ALSO check here, ALSO see "not present"!
    map.put(key, computeValue());   // BOTH threads now put -- one overwrites the other's work,
}                                     // or worse, the "expensive compute" happens TWICE, wastefully
```

**`ConcurrentHashMap` provides genuinely atomic compound operations specifically to solve this**:

```java
map.putIfAbsent(key, computeValue());       // ATOMIC: check-and-put, as ONE indivisible operation

map.computeIfAbsent(key, k -> computeValue());   // ATOMIC, and only calls computeValue() if
                                                     // genuinely needed (LAZY evaluation) -- the
                                                     // standard, idiomatic pattern for "get or
                                                     // compute-and-cache" logic

map.merge(key, 1, Integer::sum);                     // ATOMIC: "add 1, or set to 1 if absent" --
                                                        // the standard idiom for concurrent counting/
                                                        // frequency-map use cases
```

**These atomic compound methods are precisely why `ConcurrentHashMap` is generally the correct default choice for any concurrently-accessed map**, not just `synchronizedMap`'s finer locking granularity alone — the API itself is purpose-designed for safe, correct compound operations that a wrapped `HashMap` simply cannot offer without additional, manual external synchronization.

## `CopyOnWriteArrayList` — Optimized for Read-Heavy, Rarely-Modified Data

```java
List<String> list = new CopyOnWriteArrayList<>();
```

**A genuinely different, specialized strategy**: every **write** operation (`add`, `remove`, `set`) creates an **entirely new copy** of the underlying array (recall Module 09, Topic 4's `Arrays.copyOf`-style reallocation, applied here deliberately, on every single write) — while **reads** require **zero locking or coordination at all**, since they always operate on a stable, immutable snapshot of the array as it existed at read time.

**Why is this a genuinely good trade-off for specific workloads?** For collections that are **read constantly but modified rarely** (a list of event listeners, a configuration list loaded once and read by many threads repeatedly), paying a real cost on the (rare) writes in exchange for **completely free, lock-free, contention-free reads** is an excellent trade. **For write-heavy workloads, this is a genuinely poor choice** — copying the entire underlying array on every single write is expensive, and would badly hurt performance if writes are frequent.

## `BlockingQueue` — Producer-Consumer Without Manual `wait`/`notify`

Recall Topic 3's manually-implemented producer-consumer `Buffer` example, using explicit `wait()`/`notify()`. **`BlockingQueue` provides this exact coordination pattern, built in, correctly, and far more simply**:

```java
BlockingQueue<String> queue = new LinkedBlockingQueue<>(10);   // capacity-bounded queue

// Producer thread:
queue.put("task");   // BLOCKS automatically if the queue is FULL, until room is available

// Consumer thread:
String task = queue.take();   // BLOCKS automatically if the queue is EMPTY, until an item arrives
```

**`put()`/`take()` handle all of Topic 3's manual `wait`/`notifyAll`/`while`-loop-condition-recheck complexity internally, correctly, for you.** This is precisely the standard, idiomatic tool for producer-consumer coordination in real Java code — you should essentially never need to hand-write the manual `wait`/`notify` pattern from Topic 3 in application code; `BlockingQueue` (or an even higher-level `Executor`, Topic 5) is almost always the better, safer choice. (Topic 3's manual pattern remains valuable to understand precisely *because* it reveals the mechanism `BlockingQueue` correctly implements on your behalf.)

## Real-World Analogy

Think of `synchronizedMap` like a **single-file line at one cash register for an entire large store** — even customers buying completely unrelated items must all wait behind each other in the same line. Think of `ConcurrentHashMap` like a **store with many independent checkout aisles**, where customers only wait behind others buying items that happen to route through the exact same specific aisle — most of the time, genuinely parallel, uncontended checkout. Think of `CopyOnWriteArrayList` like a **community bulletin board that gets entirely reprinted and reposted fresh every time someone adds a new notice**, but anyone can freely read the currently-posted version at any moment, from any distance, with zero risk of interfering with anyone else reading it, precisely because it's a stable, standalone printed copy. Think of `BlockingQueue` like a **restaurant's order-ticket rail** — cooks (consumers) simply wait at the rail until a new ticket appears, and servers (producers) simply clip a new ticket on whenever they have one — neither side needs to manually coordinate timing with the other at all.

## Advantages

- `ConcurrentHashMap`'s fine-grained locking provides dramatically better concurrent throughput than a single-lock-wrapped `HashMap`, plus genuinely atomic compound operations solving the check-then-act problem.
- `CopyOnWriteArrayList` provides completely lock-free, contention-free reads for read-heavy, rarely-modified data.
- `BlockingQueue` correctly implements producer-consumer coordination, eliminating the need to hand-write error-prone `wait`/`notify` logic.

## Disadvantages / Trade-offs

- `CopyOnWriteArrayList`'s per-write full-array-copy cost makes it a genuinely poor choice for write-heavy workloads.
- `ConcurrentHashMap`, while much better than `synchronizedMap`, still isn't free — fine-grained locking/CAS still carries real, if much smaller, overhead compared to an unsynchronized `HashMap` in a genuinely single-threaded context.

## Best Practices

- Default to `ConcurrentHashMap` (not `Collections.synchronizedMap`) for any map accessed by multiple threads.
- Use `computeIfAbsent`/`merge`/`putIfAbsent` for atomic compound operations, rather than manual check-then-act logic.
- Reserve `CopyOnWriteArrayList` specifically for read-heavy, rarely-modified collections; avoid it for write-heavy use cases.
- Use `BlockingQueue` for producer-consumer coordination instead of hand-writing `wait`/`notify` logic.

## Common Mistakes

- Using `Collections.synchronizedMap` instead of `ConcurrentHashMap` for genuinely concurrent access, missing out on both better performance and atomic compound operations.
- Using check-then-act logic (`if (!map.containsKey(...)) map.put(...)`) even on a `ConcurrentHashMap`, instead of using its atomic `putIfAbsent`/`computeIfAbsent`.
- Using `CopyOnWriteArrayList` for a frequently-modified collection, incurring repeated, expensive full-array copies.

## Interview Questions

1. **Q: How does `ConcurrentHashMap` achieve better concurrent performance than `Collections.synchronizedMap(new HashMap<>())`?**
   A: `synchronizedMap` wraps every method call in synchronization on one single lock, forcing even unrelated operations to serialize. `ConcurrentHashMap` uses fine-grained locking (historically per-segment/bucket, refined further in modern JDKs with CAS), letting threads accessing different parts of the map proceed genuinely concurrently.

2. **Q: Why does `ConcurrentHashMap` provide methods like `putIfAbsent`/`computeIfAbsent`/`merge`?**
   A: To provide genuinely atomic compound operations — a plain check-then-act sequence (`containsKey` then `put`) is not safe as a unit even on a synchronized map, since another thread could interleave between the two separate calls; these methods perform the check-and-update as one indivisible operation.

3. **Q: What trade-off does `CopyOnWriteArrayList` make, and when is it appropriate?**
   A: Every write creates an entirely new copy of the underlying array, making writes expensive, but reads require zero locking/coordination at all, operating on a stable snapshot. Appropriate for read-heavy, rarely-modified collections; a poor choice for write-heavy workloads.

## Summary

- **`ConcurrentHashMap`** uses fine-grained locking (not one map-wide lock) for much better concurrent throughput than `synchronizedMap`, plus atomic compound operations (`putIfAbsent`, `computeIfAbsent`, `merge`) solving the check-then-act problem.
- **`CopyOnWriteArrayList`** trades expensive, full-array-copy writes for completely lock-free reads — ideal for read-heavy, rarely-modified data.
- **`BlockingQueue`** (`put`/`take`) correctly implements producer-consumer coordination, eliminating the need for hand-written `wait`/`notify` logic (Topic 3).

## Exercises

1. Rewrite a check-then-act "get or compute-and-cache" pattern using `ConcurrentHashMap.computeIfAbsent`, and explain why the naive check-then-act version isn't safe even with a `synchronizedMap`.
2. Explain the trade-off `CopyOnWriteArrayList` makes, and construct one scenario where it's an excellent choice and one where it would be a poor choice.
3. Rewrite Topic 3's manual `wait`/`notifyAll`-based producer-consumer `Buffer` using `BlockingQueue` instead, and compare the code's length and clarity.

---

**Previous:** [06 — `CompletableFuture` & Async Programming](06-completablefuture-and-async-programming.md) · **Next:** [08 — Virtual Threads & Structured Concurrency](08-virtual-threads-and-structured-concurrency.md)
