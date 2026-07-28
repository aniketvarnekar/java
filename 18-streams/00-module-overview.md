# Module 18 — Streams

## Module Goal

Module 17 gave you lambdas, method references, and `Function`/`Predicate`/`Consumer`/`Supplier`. This module puts every one of those tools to constant, direct use: the Stream API — Java 8's declarative, functional-style replacement for much of the explicit loop-writing you've done since Module 04. By the end, you'll be writing `list.stream().filter(...).map(...).collect(...)` fluently, and — just as importantly — you'll understand precisely when a stream pipeline is the right tool and when a plain loop remains clearer.

## Topics Covered in This Module

1. **[Stream Fundamentals](01-stream-fundamentals.md)** — what a stream actually is, the source/intermediate/terminal pipeline model, and laziness.
2. **[Intermediate Operations](02-intermediate-operations.md)** — `map`, `filter`, `sorted`, `distinct`, `limit`/`skip`, and `flatMap`.
3. **[Terminal Operations & `reduce`](03-terminal-operations-and-reduce.md)** — `forEach`, `collect`, `reduce`, `count`, matching operations, and `min`/`max`.
4. **[`Collectors`](04-collectors.md)** — `toList`, `toMap`, `groupingBy`, `joining`, `partitioningBy`, and summary statistics.
5. **[Parallel Streams](05-parallel-streams.md)** — how they actually work (the `ForkJoinPool`), and when they genuinely help vs. hurt.
6. **[Module Summary, Interview Questions & Exercises](06-module-summary-exercises.md)** — consolidated recap, quiz, and practice problems.

## Prerequisites

- Module 17 (Functional Programming) — this entire module is built on lambdas, method references, and `java.util.function`.
- Module 10 (Collections) — streams are almost always created from, and often collected back into, `List`/`Set`/`Map`.
- Module 15 (Concurrency), Topic 5 (Executors) — Topic 5 of this module directly references the `ForkJoinPool`.

## How to Study This Module

Topic 1's mental model — a stream pipeline is a **description** of a computation, not the computation itself, until a terminal operation runs it — is the single most important conceptual foundation for this entire module; get it right before moving to Topics 2–3's specific operations. Topic 5 (parallel streams) is deliberately last, and deliberately cautious — parallel streams are genuinely oversold in casual tutorials, and this module gives you the honest trade-offs.

---

**Previous module:** [17 — Functional Programming](../17-functional-programming/00-module-overview.md) · **Next:** [01 — Stream Fundamentals](01-stream-fundamentals.md)
