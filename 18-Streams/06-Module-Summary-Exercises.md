# Module 18 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-Module-Overview.md) has been covered:

- [x] **Stream Fundamentals** — the stream-is-not-a-data-structure model, the source/intermediate/terminal pipeline, laziness, and single-use semantics
- [x] **Intermediate Operations** — `filter`/`map`/`sorted`/`distinct`/`limit`/`skip`, and `flatMap`'s genuinely distinct nested-flattening purpose
- [x] **Terminal Operations & `reduce`** — `forEach`, matching operations' short-circuiting behavior, and `reduce`'s general identity/accumulator pattern
- [x] **`Collectors`** — `toList`/`toSet`/`toMap` (including the duplicate-key gotcha), `groupingBy`/`partitioningBy`, `joining`, and summary statistics
- [x] **Parallel Streams** — the fork/join mechanism, the honest conditions under which parallelism helps vs. hurts, and the statelessness requirement

## Practical Connections

- **Every Spring Data repository query result processed with `.stream().filter(...).map(...).collect(...)`** is direct, everyday application of this entire module.
- **REST API response transformation** (converting entity objects to DTOs) is a textbook `map` + `collect(toList())` use case.
- **Reporting/analytics code** (grouping transactions by category, computing totals) is a direct, common `groupingBy` + downstream-collector application.
- **Batch data processing pipelines** are where parallel streams (Topic 5) genuinely can help — large datasets, CPU-bound transformation work — though always with the honest, measured caution this module emphasized.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| `Stream` vs `Collection` | A `Collection` stores data; a `Stream` describes a one-time-use computation pipeline over a data source. |
| `map` vs `flatMap` | `map` transforms each element 1-to-1; `flatMap` flattens a per-element stream-producing mapping into one unified stream. |
| `forEach` vs `collect` | `forEach` is for side effects (returns void); `collect` is for building a result — never mutate external state inside `forEach`. |
| `groupingBy` vs `partitioningBy` | `groupingBy`: any number of groups, based on a classifier. `partitioningBy`: always exactly two groups (`true`/`false`), based on a `Predicate`. |
| Sequential vs parallel streams | Sequential: one thread, predictable order. Parallel: fork/join across the shared common `ForkJoinPool`, genuinely faster only for large, CPU-bound, efficiently-splittable, stateless workloads. |

## Consolidated Interview Questions (Module 18)

1. Is a `Stream` a data structure?
2. Why don't intermediate operations execute immediately when called?
3. Can a `Stream` be reused after its terminal operation runs?
4. What's the difference between `map` and `flatMap`?
5. Why is using `forEach` to mutate an external collection considered an anti-pattern?
6. What are `reduce`'s two arguments?
7. Why are `anyMatch`/`allMatch`/`noneMatch` short-circuiting?
8. Why does `Collectors.toMap` throw on duplicate keys by default?
9. What's the difference between `groupingBy` and `partitioningBy`?
10. How do parallel streams actually execute work internally?
11. Why might a parallel stream be slower than a sequential one?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** From memory, draw the source → intermediate → terminal pipeline model, marking which parts are lazy and which trigger execution.
2. **Hands-on:** Given a `List<Order>` (with `customer`, `product`, `amount` fields), write a pipeline that filters orders over $100, groups them by customer, and sums each customer's total using `groupingBy` + a downstream collector.
3. **Hands-on:** Use `flatMap` to flatten a `List<List<Integer>>` into a single sorted, deduplicated `List<Integer>`.
4. **Hands-on:** Write a `Collectors.toMap` call on data with potential duplicate keys, first observing the exception, then fixing it with a merge function.
5. **Conceptual:** Explain why a parallel stream operating on a small `List<Integer>` (10 elements) with a cheap `map` transformation is likely to be slower than the sequential equivalent.
6. **Synthesis:** Design a complete pipeline processing a `List<Employee>`: filter active employees, group by department, and within each group, compute both the count and average salary using summary statistics or composed collectors — explain each step's role.

## What's Next

Module 18 completed the modern, functional-style data processing toolkit — lambdas (Module 17) and streams (this module) together represent the biggest philosophical shift in Java's history, and you now have full command of both. **Module 19 — Networking** shifts to a different, foundational practical skill: sockets, the `HttpClient` API (Module 01, Topic 3's "Distributed" feature, made concrete), and building networked Java applications.

---

**Previous:** [05 — Parallel Streams](05-Parallel-Streams.md) · **Module Overview:** [00 — Module Overview](00-Module-Overview.md)

**Type "Continue" to begin Module 19 — Networking.**
