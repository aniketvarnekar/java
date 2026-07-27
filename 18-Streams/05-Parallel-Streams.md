# Parallel Streams

## Learning Objectives

- Understand exactly how `parallelStream()` executes work, including the `ForkJoinPool` it uses
- Understand precisely when parallel streams genuinely help, and when they genuinely hurt
- Know the correctness requirements (statelessness, non-interference) parallel streams demand

## Prerequisites

All prior topics in this module, Module 15 (Concurrency), especially Topic 5 (Executors/thread pools)

## Motivation

Parallel streams are simultaneously one of the most appealing-sounding and most commonly misused features in modern Java — "just add `.parallel()` and it's faster!" is genuinely, often false. This closing topic gives you the honest, complete picture: the real mechanism, and the real, specific conditions under which parallelism actually pays off.

## How to Create a Parallel Stream

```java
List<Integer> result = numbers.parallelStream()      // from a Collection, directly
    .map(n -> expensiveComputation(n))
    .collect(Collectors.toList());

List<Integer> result2 = numbers.stream()                // or convert an existing sequential stream
    .parallel()
    .map(n -> expensiveComputation(n))
    .collect(Collectors.toList());
```

**Syntactically, this looks almost trivially easy** — precisely why it's so commonly reached for without fully understanding the real trade-offs involved.

## The Mechanism: `ForkJoinPool` and the Fork/Join Framework

**Parallel streams don't create new threads per element.** Instead, they use a shared, JVM-wide thread pool called the **common `ForkJoinPool`** (recall Module 15, Topic 5's `ExecutorService`/thread-pool concepts — this is a specialized variant, sized by default to match your machine's available CPU cores).

**The underlying strategy is "fork/join" — divide and conquer**: the source data is recursively **split** into smaller chunks (**fork**), each chunk processed independently (potentially by a different pool thread, potentially further split again), and the sub-results are recursively **combined** back together (**join**) into the final result.

```
                       [entire dataset]
                              │
                        FORK (split)
                    ┌─────────┴─────────┐
              [first half]         [second half]
                    │                     │
              FORK again              FORK again
            ┌───────┴──────┐       ┌──────┴──────┐
        [chunk 1]      [chunk 2]  [chunk 3]   [chunk 4]
            │               │          │            │
        (processed,     (processed,  (processed,  (processed,
         possibly on      possibly on  possibly on  possibly on
         different        different    different    different
         pool threads)    pool threads) pool threads) pool threads)
            │               │          │            │
            └───────┬──────┘       └──────┬──────┘
                  JOIN (combine)        JOIN (combine)
                       └─────────┬─────────┘
                            JOIN (final combine)
                                 │
                          [final result]
```

**This recursive fork/join structure is precisely why some intermediate/terminal operation combinations parallelize far better than others** — operations that combine cleanly and independently (like `sum`, or a stateless `map`/`filter`) fork/join efficiently; operations with inherent sequential dependencies (like `sorted` on certain data shapes, or anything depending on encounter order) parallelize less effectively, or not at all.

## When Parallel Streams Genuinely Help

**Parallel streams pay off specifically when ALL of these conditions hold, together:**

1. **The dataset is genuinely large** (thousands to millions of elements — for small datasets, the fork/join coordination overhead itself dwarfs any parallelism benefit).
2. **The per-element work is genuinely CPU-intensive** (real computation, not I/O — recall Module 15, Topic 8's Virtual Threads discussion: I/O-bound work benefits far more from *that* mechanism than from CPU-parallelism-focused fork/join).
3. **The underlying data source splits efficiently** (arrays and `ArrayList` — Module 09's contiguous, indexable structure — split very efficiently; `LinkedList`, Module 10, Topic 2's node-chain structure, splits poorly, since finding a "midpoint" requires traversal rather than direct index arithmetic).
4. **The operations are genuinely stateless and non-interfering** (see the correctness requirements below).

## The Real, Honest Costs — Why "Just Add `.parallel()`" Is Often Wrong

- **Coordination overhead**: forking, tracking, and joining sub-tasks has real cost — for small-to-moderate datasets or cheap per-element work, this overhead frequently **exceeds** any parallelism benefit, making the parallel version genuinely **slower** than the sequential one.
- **Shared thread pool contention**: the common `ForkJoinPool` is **shared JVM-wide** — if your application already uses it elsewhere (including, notably, `CompletableFuture`'s default async methods, Module 15, Topic 6, which use the *same* common pool by default), parallel streams can contend with **unrelated** work for the same limited thread pool, causing unexpected slowdowns elsewhere in your application, not just in the stream itself.
- **Correctness requirements**: parallel execution genuinely requires your lambdas to be **stateless** and **non-interfering** — recall Module 15's entire race-condition discussion (Topic 2): a `map`/`filter` lambda that mutates shared, external state is now running **concurrently, across multiple threads**, reintroducing exactly the race conditions Module 15 taught you to avoid, silently, unless you're deliberately careful.

```java
// ⚠️ A genuine, real bug -- mutating shared state from within a PARALLEL stream:
List<Integer> results = new ArrayList<>();   // NOT thread-safe (Module 10, Topic 2)!
numbers.parallelStream().forEach(n -> results.add(n * 2));   // RACE CONDITION -- Module 15's
                                                                 // entire hazard discussion, reintroduced!
```

## Real-World Analogy

Think of a sequential stream like **one chef preparing an entire meal alone, start to finish** — simple to coordinate, no risk of two chefs bumping into each other, but limited by that one chef's own speed. Think of a parallel stream like **hiring a whole team of chefs and splitting the meal's preparation among them** — genuinely faster **if** the meal is large and complex enough to be worth splitting up, **and** the kitchen (data structure) is actually organized in a way that splits cleanly (a big, open, well-organized kitchen — an array — vs. one narrow galley kitchen only one person can move through at a time — a linked list). But if the meal is trivially small, or the chefs constantly need to reach into the **same** shared bowl at the same time without coordination (shared, mutable state), you'll get chefs bumping into each other and a worse outcome than just letting one chef handle it alone.

## Advantages

- Genuine, real performance improvement for large datasets with substantial per-element CPU-bound work and efficiently-splittable sources.
- Uses the JVM's shared, already-managed `ForkJoinPool`, requiring no manual thread pool setup (Module 15, Topic 5) from the developer.

## Disadvantages / Trade-offs

- Genuinely, often **slower** than sequential streams for small datasets or cheap per-element work, due to real fork/join coordination overhead.
- Shares the JVM-wide common `ForkJoinPool` with other parallel work (including `CompletableFuture`'s default async methods), risking unexpected contention across unrelated parts of an application.
- Requires genuinely stateless, non-interfering lambda logic — mutating shared external state reintroduces Module 15's race conditions, often silently.

## Best Practices

- Measure before reaching for `.parallel()`/`.parallelStream()` — never assume it's faster without verifying for your specific, actual workload and data size.
- Prefer parallel streams specifically for large datasets with genuinely CPU-intensive per-element work, backed by an efficiently-splittable source (arrays, `ArrayList`).
- Ensure lambdas used within a parallel stream are genuinely stateless and never mutate shared, external state — use `collect` (Topic 4), never manual external accumulation via `forEach`.

## Common Mistakes

- Adding `.parallel()` reflexively "for performance," without measuring, on small datasets where it's actually slower.
- Using a `LinkedList` (Module 10, Topic 2) as a parallel stream source, missing out on the efficient splitting arrays/`ArrayList` provide.
- Mutating shared, external state from within a parallel stream's lambda, silently reintroducing Module 15's race conditions.

## Interview Questions

1. **Q: How do parallel streams actually execute work internally?**
   A: Via the fork/join framework, using the JVM's shared common `ForkJoinPool` — the data source is recursively split ("forked") into smaller chunks, each processed independently (potentially concurrently across pool threads), with sub-results recursively combined ("joined") back into the final result.

2. **Q: Why might `.parallelStream()` actually be slower than `.stream()` for a given task?**
   A: Fork/join coordination has real overhead; for small datasets or cheap per-element work, this overhead can exceed any parallelism benefit. Additionally, contention on the shared common `ForkJoinPool` (used by other parallel work, including `CompletableFuture`'s default async methods) can cause unexpected slowdowns.

3. **Q: What correctness requirement do parallel stream lambdas have that sequential stream lambdas don't strictly need to worry about?**
   A: Statelessness and non-interference — since operations may run concurrently across multiple threads, a lambda mutating shared external state reintroduces genuine race conditions (Module 15), which sequential (single-threaded) stream execution wouldn't expose.

## Summary

- Parallel streams use the fork/join framework and the JVM's shared common `ForkJoinPool` to recursively split, process, and recombine data.
- They genuinely help specifically for large datasets with substantial CPU-bound per-element work and efficiently-splittable sources (arrays/`ArrayList`) — and can genuinely hurt for small datasets, cheap per-element work, poorly-splittable sources (`LinkedList`), or when contending with other users of the shared common pool.
- Parallel stream lambdas must be stateless and non-interfering — mutating shared external state reintroduces Module 15's race conditions.
- **Always measure before parallelizing** — it is not a free, automatic performance win.

## Module-Wide Quick Revision

- A `Stream` describes a computation pipeline, not a data structure; it's lazy (nothing runs until a terminal operation) and single-use (Topic 1).
- `filter`/`map`/`sorted`/`distinct`/`limit`/`skip` are the core intermediate operations; `flatMap` flattens per-element stream-producing mappings (Topic 2).
- `forEach` is for side effects; matching operations short-circuit; `reduce(identity, accumulator)` is the general aggregation pattern (Topic 3).
- `Collectors.toList`/`toMap`/`groupingBy`/`partitioningBy`/`joining`/summary statistics cover nearly every real-world aggregation need (Topic 4).
- Parallel streams use fork/join over the shared common `ForkJoinPool`; genuinely help only for large, CPU-bound, efficiently-splittable workloads with stateless lambdas — always measure (this topic).

## Common Pitfalls (Module-Wide)

- Assuming intermediate operations execute eagerly.
- Reusing a consumed stream.
- Using `map` where `flatMap` was needed.
- Forgetting `toMap`'s duplicate-key exception.
- Reflexively parallelizing without measuring, or mutating shared state inside a parallel stream.

## Mini Quiz (Module-Wide)

1. When does an intermediate operation actually execute?
2. What's the difference between `map` and `flatMap`?
3. What does `reduce`'s identity value represent?
4. What happens if `Collectors.toMap` encounters duplicate keys with no merge function?
5. Why can a parallel stream be slower than a sequential one?

*(Answers are derivable from Topics 1, 2, 3, 4, and this topic, respectively.)*

---

**Previous:** [04 — `Collectors`](04-Collectors.md) · **Next:** [06 — Module Summary, Interview Questions & Exercises](06-Module-Summary-Exercises.md)
