# StringBuilder & StringBuffer

## Learning Objectives

- Explain precisely why repeated String concatenation is expensive, with a memory-level trace
- Use `StringBuilder` correctly and idiomatically
- Know the difference between `StringBuilder` and `StringBuffer`, and when (rarely) the latter is needed
- Compare `String` vs. `StringBuilder` vs. `StringBuffer` completely

## Prerequisites

[01 — String Immutability](01-String-Immutability.md), Module 06 Topic 3 (`this` — the fluent-chaining pattern `StringBuilder` uses)

## Motivation

Topic 1 flagged this cost directly: since `String` is immutable, every "modification" produces a new object. This topic shows exactly why that becomes a real, measurable performance problem in loops — and introduces the mutable alternative purpose-built to solve it.

## Problem Statement

```java
String result = "";
for (int i = 0; i < 5; i++) {
    result = result + i;      // looks innocent...
}
System.out.println(result);    // "01234"
```

This "looks" like simple, cheap string building — but recall Topic 1: **`String` is immutable**, so `result + i` on every iteration creates a **brand new** `String` object, discarding the previous one entirely:

```
 Iteration 0: result = "" + "0"       -> NEW String "0"           (old "" discarded)
 Iteration 1: result = "0" + "1"        -> NEW String "01"          (old "0" discarded)
 Iteration 2: result = "01" + "2"         -> NEW String "012"         (old "01" discarded)
 Iteration 3: result = "012" + "3"          -> NEW String "0123"        (old "012" discarded)
 Iteration 4: result = "0123" + "4"           -> NEW String "01234"       (old "0123" discarded)

 TOTAL: 5 NEW String objects created, 4 of them IMMEDIATELY discarded and eligible for GC!
```

**For small loops like this, the cost is negligible** — but for a loop running thousands or millions of times (building a large report, processing a big file line by line, constructing a large JSON payload), this pattern creates a genuinely large number of short-lived, discarded objects, adding real, measurable Garbage Collector pressure (Module 02, Topic 4) and wasted CPU time copying character data repeatedly.

## The Solution: `StringBuilder` — A Mutable Alternative

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 5; i++) {
    sb.append(i);      // MUTATES the SAME internal buffer -- no new object per iteration!
}
String result = sb.toString();    // convert to a String ONCE, at the very end
System.out.println(result);         // "01234"
```

`StringBuilder` maintains an internal, **resizable character array (a buffer)** that it genuinely mutates in place — `append()` adds characters directly into this buffer (growing it if needed), rather than creating an entirely new object each time. Only when you're finished do you call `.toString()` **once**, to produce the final, immutable `String` result.

```
 StringBuilder's internal buffer (mutated IN PLACE, not recreated):

 after append(0): [0, _, _, _, _, _, _, _]     (buffer has extra capacity, to reduce future resizing)
 after append(1): [0, 1, _, _, _, _, _, _]
 after append(2): [0, 1, 2, _, _, _, _, _]
 after append(3): [0, 1, 2, 3, _, _, _, _]
 after append(4): [0, 1, 2, 3, 4, _, _, _]

 ONLY ONE underlying buffer exists throughout the entire loop -- MUTATED directly each time,
 not recreated -- this is the fundamental performance difference from String concatenation
```

## `StringBuilder`'s API — Fluent, Chainable (Recall Module 06, Topic 3)

```java
StringBuilder sb = new StringBuilder();
sb.append("Hello")
  .append(", ")
  .append("World")
  .append("!");            // each append() returns 'this' -- FLUENT CHAINING (Module 06, Topic 3)

System.out.println(sb.toString());   // "Hello, World!"

sb.insert(5, " there");    // insert at a specific index -- "Hello there, World!"
sb.delete(0, 6);              // delete a range -- " there, World!"
sb.reverse();                    // reverses the ENTIRE buffer in place
sb.length();                       // current length of the buffer's content
```

This is **exactly** the same "return `this` for chaining" pattern taught in Module 06, Topic 3 — `StringBuilder` is the canonical, real-world standard library example of that pattern in action.

## Why the Modern Java Compiler Already Uses `StringBuilder` For You (Sometimes)

A genuinely important nuance: **`javac` automatically optimizes simple, single-expression String concatenation** (like `"a" + b + "c"` written as one expression) by internally rewriting it to use `StringBuilder` for you, behind the scenes — you never see this, but it's real, and it means simple, single-line concatenation is **not** actually as wasteful as the naive "every `+` creates a new object" mental model might suggest, *for a single expression*.

**The performance problem specifically arises in LOOPS** — `result = result + i;` **inside a loop** creates a **brand-new `StringBuilder` on every single iteration** (since each iteration is a separate statement/expression the compiler optimizes independently), defeating the purpose entirely. **Manually using one single `StringBuilder` across the entire loop** (as shown in the fix above) avoids this repeated-creation problem, which the compiler cannot automatically detect and fix on your behalf across multiple iterations.

```java
// Compiler-optimized automatically -- fine, a single expression:
String greeting = "Hello, " + name + "!";

// NOT automatically optimized across iterations -- a NEW StringBuilder gets created EVERY loop pass:
String result = "";
for (int i = 0; i < 1000; i++) {
    result = result + i;   // ⚠️ creates a new StringBuilder(), appends, calls toString() -- EVERY iteration
}
// Manual, single StringBuilder used across ALL iterations -- the CORRECT, efficient approach:
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);            // reuses the SAME buffer across ALL 1000 iterations
}
String result2 = sb.toString();
```

## `StringBuilder` vs. `StringBuffer`

`StringBuffer` is `StringBuilder`'s **older**, near-identical twin — same API, same purpose (a mutable character buffer) — with exactly **one** difference:

> **`StringBuffer`'s methods are `synchronized`** (Module 15 will cover synchronization's full mechanics) — meaning it's safe to share a single `StringBuffer` instance across multiple threads without external coordination. **`StringBuilder`'s methods are NOT synchronized**, making it faster (no synchronization overhead) but unsafe to share across threads without your own external coordination.

**Historical context:** `StringBuffer` came first (Java 1.0). `StringBuilder` was added later (Java 5) specifically because, in the **overwhelming majority of real use cases**, a `StringBuilder`/`StringBuffer` instance is used entirely locally within a single method, on a single thread, never shared — meaning `StringBuffer`'s synchronization overhead was being paid **constantly, for no actual benefit**, in the vast majority of real usage. `StringBuilder` was introduced as the correct default for that common case, with `StringBuffer` remaining available specifically for the rarer case of genuine, deliberate cross-thread sharing.

**Modern guidance: always use `StringBuilder`, unless you have a specific, verified need to share a single mutable string-buffer instance across multiple threads** — in which case, prefer higher-level `java.util.concurrent` tools (Module 15) over `StringBuffer` in most modern designs anyway.

## Full Three-Way Comparison

| | `String` | `StringBuilder` | `StringBuffer` |
|---|---|---|---|
| Mutable? | No (Topic 1) | Yes | Yes |
| Thread-safe? | Yes (inherently, since immutable) | No | Yes (synchronized methods) |
| Performance | Fast for single/few operations; poor for many repeated modifications in a loop | Fast — no synchronization overhead | Slower than `StringBuilder` — synchronization overhead |
| Introduced | Java 1.0 | Java 5 | Java 1.0 |
| When to use | Fixed or rarely-changing text; most general-purpose use | Building/modifying text incrementally, single-threaded (the common case) | Building/modifying text incrementally, genuinely shared across threads (rare) |

## Real-World Analogy

Think of `String` concatenation in a loop like **rewriting an entire letter from scratch on a brand-new sheet of paper every single time you want to add one more sentence** — wasteful, since you're re-copying everything you already wrote, over and over. `StringBuilder` is like **writing directly onto one single, growing scroll** — each new sentence is simply added onto the end of the same physical document, with zero need to re-copy anything already written. `StringBuffer` is the same scroll, but with a **"only one person may write at a time" lock** attached — safe for multiple writers, but adding real overhead that's wasted if, as is usually the case, only one person (thread) is ever writing to it anyway.

## Advantages

- `StringBuilder` eliminates the repeated-object-creation cost of loop-based String concatenation entirely.
- Fluent, chainable API (Module 06, Topic 3's pattern) makes building complex strings readable.
- The compiler already handles simple, single-expression concatenation efficiently — you only need to reach for `StringBuilder` manually for genuinely iterative/loop-based building.

## Disadvantages / Trade-offs

- `StringBuilder` is mutable and **not** thread-safe — sharing one instance across threads without external synchronization is a genuine, real bug risk (Module 15).
- Slightly more verbose than simple `+` concatenation for trivial, one-off cases — not worth reaching for `StringBuilder` when the compiler's automatic optimization already handles a single expression well.

## Best Practices

- Use `StringBuilder` (not repeated `+=`/`+` concatenation) for building strings incrementally inside loops.
- Default to `StringBuilder` over `StringBuffer` unless you have a specific, verified multi-threaded sharing need.
- Trust the compiler's automatic optimization for simple, single-expression concatenation — don't manually convert every `+` to `StringBuilder` out of habit; that's unnecessary noise for cases the compiler already handles well.

## Common Mistakes

- Building strings with `+=` inside loops, unaware of the repeated-object-creation cost this incurs at scale.
- Using `StringBuffer` by default/habit "for safety," paying unnecessary synchronization overhead in the overwhelming majority of cases where the instance is never actually shared across threads.
- Forgetting to call `.toString()` at the end when a genuine `String` (not a `StringBuilder`) is needed by other code (e.g., a method with a `String` return type or parameter).

## Interview Questions

1. **Q: Why is repeated String concatenation inside a loop inefficient, and what should be used instead?**
   A: Because `String` is immutable, each `+`/`+=` inside a loop creates a brand-new `String` object, discarding the previous one — for a loop running many times, this creates significant numbers of short-lived, discarded objects, adding real GC pressure. `StringBuilder`, used once across the entire loop, mutates a single internal buffer in place instead, avoiding this repeated-creation cost entirely.

2. **Q: What's the difference between `StringBuilder` and `StringBuffer`?**
   A: They have an identical API and purpose (a mutable character buffer), but `StringBuffer`'s methods are `synchronized` (thread-safe, with overhead), while `StringBuilder`'s are not (faster, but not safe to share across threads without external coordination). `StringBuilder` (Java 5+) is the recommended default for the common single-threaded case; `StringBuffer` remains for genuine cross-thread sharing needs.

3. **Q: Does `String greeting = "Hello, " + name + "!";` (a single expression, not in a loop) suffer the same performance problem as loop-based concatenation?**
   A: No — the compiler automatically rewrites simple, single-expression concatenation to use `StringBuilder` internally. The performance problem specifically arises in loops, where each iteration's concatenation is a separate statement, causing a new `StringBuilder` (and discarded intermediate `String`) to be created on every single pass, which the compiler cannot automatically consolidate across iterations.

## Summary

- Because `String` is immutable, repeated concatenation (especially in loops) creates many short-lived, discarded objects — a real, measurable performance cost at scale.
- **`StringBuilder`** provides a mutable, resizable character buffer, letting you build/modify text in place, converting to a final `String` only once, via `.toString()`.
- **`StringBuffer`** is functionally identical but `synchronized` (thread-safe, with overhead) — `StringBuilder` is the correct modern default unless genuine cross-thread sharing is needed.
- The compiler already optimizes simple, single-expression concatenation automatically — manual `StringBuilder` use matters specifically for loop-based/iterative string building.

## Exercises

1. Rewrite this loop to use `StringBuilder` instead of repeated concatenation, and explain the performance difference: `String csv = ""; for (String item : items) { csv += item + ","; }`
2. Explain, step by step (using the buffer diagram style from this topic), what happens internally when you call `.append()` three times on the same `StringBuilder` instance.
3. Explain precisely when `StringBuffer` would be the more appropriate choice over `StringBuilder`, and why that scenario is comparatively rare in typical application code.

---

**Previous:** [03 — String Methods & API](03-String-Methods-and-API.md) · **Next:** [05 — String Formatting & Text Blocks](05-String-Formatting-And-Text-Blocks.md)
