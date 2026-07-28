# Module 16 — JVM Internals

## Module Goal

Module 02 gave you the JVM's architecture; Module 15 gave you the Java Memory Model's practical, working rules. This module goes deeper into the mechanics that power everything else in this course: the actual bytecode instruction format (with hands-on `javap` use), the real garbage collection algorithms behind Module 02's "the GC reclaims unreachable objects," a more formal treatment of the JMM, and — genuinely powerful, if advanced tools — Reflection, Annotations, and Dynamic Proxies, which underpin nearly every framework you'll use professionally (Spring, Hibernate, JUnit).

## Topics Covered in This Module

1. **[Bytecode Deep Dive](01-bytecode-deep-dive.md)** — the `.class` file format, bytecode instruction categories, and hands-on `javap` disassembly.
2. **[Garbage Collection Algorithms](02-garbage-collection-algorithms.md)** — the generational hypothesis, and the real algorithms (Serial, Parallel, G1, ZGC, Shenandoah) behind Module 02's GC preview.
3. **[Java Memory Model Deep Dive](03-java-memory-model-deep-dive.md)** — reordering, memory barriers, and a more formal treatment building on Module 15's `happens-before`.
4. **[Reflection](04-reflection.md)** — inspecting and invoking code at runtime, and why frameworks depend on it.
5. **[Annotations & Dynamic Proxies](05-annotations-and-dynamic-proxies.md)** — writing custom annotations, and the proxy mechanism that makes Spring's `@Transactional`/`@Autowired`-style "magic" actually work.
6. **[Method Handles & Modern Internals](06-method-handles-and-modern-internals.md)** — `MethodHandle`, `invokedynamic`, and a brief look at what powers modern Java's performance.
7. **[Module Summary, Interview Questions & Exercises](07-module-summary-exercises.md)** — consolidated recap, quiz, and practice problems.

## Prerequisites

- Module 02 (JVM) — this module directly extends Topics 2 (Class Loading), 3 (Runtime Data Areas), and 4 (Execution Engine).
- Module 15 (Concurrency), especially Topic 2 (the JMM's `happens-before` model).
- Module 05 (OOP), especially Topic 6 (Interfaces — Dynamic Proxies build directly on interface implementation).

## How to Study This Module

Topics 1–3 deepen concepts you already have a working model of from Modules 02 and 15 — read them as "here's the mechanism behind what you already trust." Topics 4–5 (Reflection, Dynamic Proxies) are the most immediately practically relevant for understanding how real frameworks work — if you've ever wondered "how does Spring know to inject this dependency without me writing the wiring code," this is where that question gets answered.

---

**Previous module:** [15 — Concurrency](../15-concurrency/00-module-overview.md) · **Next:** [01 — Bytecode Deep Dive](01-bytecode-deep-dive.md)
