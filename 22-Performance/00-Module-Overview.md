# Module 22 — Performance

## Module Goal

This module is the practical, hands-on payoff of nearly everything the course has built toward: Module 02's Execution Engine, Module 16's GC algorithms, Module 15's concurrency model, and the JIT warm-up behavior mentioned since Module 01. Here, that knowledge becomes **actionable** — how to actually tune a JVM, how to measure performance correctly (and why naive measurement is usually wrong), and the specific, common code-level pitfalls worth knowing and avoiding.

## Topics Covered in This Module

1. **[JVM Tuning: Heap & GC Selection](01-JVM-Tuning-Heap-And-GC-Selection.md)** — heap sizing flags, and choosing/configuring a garbage collector, directly applying Module 16, Topic 2.
2. **[Profiling & Benchmarking](02-Profiling-And-Benchmarking.md)** — why naive `System.nanoTime()` benchmarking is usually wrong (Module 02's JIT warm-up), and the tools that measure correctly.
3. **[Common Performance Pitfalls & Optimization](03-Common-Performance-Pitfalls-And-Optimization.md)** — autoboxing, string concatenation, collection sizing, escape analysis, and a final, practical revisit of GraalVM Native Image's trade-offs.
4. **[Module Summary, Interview Questions & Exercises](04-Module-Summary-Exercises.md)** — consolidated recap, quiz, and practice problems.

## Prerequisites

- Module 02 (JVM), especially Topic 4 (Execution Engine — JIT warm-up).
- Module 16 (JVM Internals), especially Topic 2 (GC Algorithms).
- Module 03 (Java Basics), Topic 6 (autoboxing) and Module 08 (Strings), Topic 4 (`StringBuilder`).

## How to Study This Module

This module deliberately synthesizes rather than introduces brand-new concepts — nearly every topic here is "here's how to actually *apply*, in practice, something you already understand the mechanism of." Topic 2's warning about naive benchmarking is the single most important lesson to internalize — it's a genuine, common, real source of misleading performance conclusions, even among experienced developers.

---

**Previous module:** [21 — Modules](../21-Modules/00-Module-Overview.md) · **Next:** [01 — JVM Tuning: Heap & GC Selection](01-JVM-Tuning-Heap-And-GC-Selection.md)
