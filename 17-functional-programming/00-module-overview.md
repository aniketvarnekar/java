# Module 17 — Functional Programming

## Module Goal

Module 01, Topic 2's version timeline flagged Java 8 (2014) as "the biggest philosophical shift in Java's history." This module delivers on that claim in full: lambda expressions (now grounded in Module 16's `invokedynamic` mechanism), functional interfaces, method references, and the standard `java.util.function` toolkit — the foundation everything in Module 18 (Streams) is built on.

## Topics Covered in This Module

1. **[Functional Interfaces](01-functional-interfaces.md)** — the concept that makes lambdas possible at all, and `@FunctionalInterface`.
2. **[Lambda Expressions](02-lambda-expressions.md)** — syntax, type inference, and closures over effectively-final variables.
3. **[Method References](03-method-references.md)** — the four kinds, and when they're clearer than an equivalent lambda.
4. **[Built-In Functional Interfaces](04-built-in-functional-interfaces.md)** — `Function`, `Supplier`, `Consumer`, `Predicate`, and composing them.
5. **[`Optional`](05-optional.md)** — Java 8's answer to `null`-related bugs, correct usage, and real anti-patterns.
6. **[Module Summary, Interview Questions & Exercises](06-module-summary-exercises.md)** — consolidated recap, quiz, and practice problems.

## Prerequisites

- Module 16 (JVM Internals), especially Topic 6 (`invokedynamic` — the mechanism enabling lambdas).
- Module 06 (Classes), especially Topic 6 (Nested Classes — anonymous classes, lambdas' direct predecessor).
- Module 05 (OOP), especially Topic 6 (Interfaces — `default`/`static` methods, essential to this module's toolkit).
- Module 03 (Java Basics), Topic 7 ("effectively final" preview).

## How to Study This Module

Topic 1 is the conceptual key that unlocks everything else — understanding *why* a lambda can only exist where a functional interface is expected (not "anywhere," despite how flexible lambda syntax feels) prevents a lot of confusion in Topics 2–4. Topic 5 (`Optional`) is a smaller, more self-contained topic, but a genuinely important one for writing idiomatic modern Java.

---

**Previous module:** [16 — JVM Internals](../16-jvm-internals/00-module-overview.md) · **Next:** [01 — Functional Interfaces](01-functional-interfaces.md)
