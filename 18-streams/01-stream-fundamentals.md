# Stream Fundamentals

## Learning Objectives

- Understand precisely what a `Stream` is, and what it is NOT
- Understand the source → intermediate → terminal pipeline model
- Understand laziness, and why intermediate operations don't execute until a terminal operation runs

## Prerequisites

Module 17 (Functional Programming), Module 10 (Collections)

## Motivation

The single most important thing to get right before writing any stream code is the mental model — a `Stream` is fundamentally different from a `Collection` (Module 10), despite superficially similar-looking method chains. Getting this model right up front prevents a whole category of confusing "why didn't this run" and "why can't I reuse this stream" questions.

## What a `Stream` Is — And Is Not

> A **`Stream<T>`** represents a **sequence of elements to be processed through a pipeline of operations** — it is **not** a data structure that stores elements (unlike `List`/`Set`, Module 10), and it does **not** hold any data itself.

```java
List<String> names = List.of("Aniket", "Priya", "Rahul");

Stream<String> stream = names.stream();   // creates a Stream FROM the List
```

**A `Stream` is best understood as a description of a computation over a data source, not a container.** The `List` (Module 10) is the actual data structure holding the elements; the `Stream` is a **pipeline** you build **on top of** that data, describing what should happen to each element as it flows through.

## The Pipeline Model: Source → Intermediate Operations → Terminal Operation

```
    SOURCE               INTERMEDIATE OPERATIONS              TERMINAL OPERATION
 (a Collection,        (map, filter, sorted, ... --          (collect, forEach,
  array, etc.)          each returns a NEW Stream,             reduce, count, ...
                          LAZY, not yet executed)                ACTUALLY RUNS the pipeline)

 names.stream()  .filter(n -> n.length() > 5)  .map(String::toUpperCase)  .collect(toList())
      │                    │                            │                        │
      ▼                    ▼                            ▼                        ▼
  Stream<String>      Stream<String>              Stream<String>           List<String>
  (the SOURCE)         (a NEW stream,               (a NEW stream,          (the FINAL
                         still describing              still describing        RESULT --
                         the pipeline, not              the pipeline, NOT       the pipeline
                         yet run)                        yet run)                ACTUALLY RUNS
                                                                                    HERE)
```

**Every intermediate operation** (`filter`, `map`, `sorted`, and more — Topic 2) **returns a brand-new `Stream`**, describing an additional processing step, **without actually processing anything yet.** **Only the terminal operation** (`collect`, `forEach`, `reduce`, and more — Topic 3) **actually triggers execution** of the entire pipeline, from source through every intermediate step, all at once.

## Laziness — Intermediate Operations Don't Run Until Triggered

```java
Stream<String> pipeline = names.stream()
    .filter(n -> {
        System.out.println("Filtering: " + n);   // this WON'T print yet!
        return n.length() > 5;
    });

System.out.println("Pipeline built, but nothing has printed above this line yet...");

List<String> result = pipeline.collect(Collectors.toList());   // ONLY NOW does the filter
                                                                    // logic actually execute!
```

**Output:**
```
Pipeline built, but nothing has printed above this line yet...
Filtering: Aniket
Filtering: Priya
Filtering: Rahul
```

**This is a genuinely important, real, and often surprising behavior**: building the pipeline (calling `.filter(...)`) does **not** execute the filter's lambda body at all — it only happens once `.collect(...)` (a terminal operation) is called, at which point the **entire pipeline runs, element by element**, from source through every intermediate step, in a single pass.

**Why does this matter, concretely?** Laziness enables genuine, real optimizations the JVM/Stream implementation can apply — most importantly, **short-circuiting**: if a terminal operation only needs the **first** matching element (like `findFirst()`), the pipeline can stop processing entirely once found, rather than needlessly running every intermediate operation across the **entire** source first. A naive loop-based equivalent achieves this too, of course — but a stream pipeline achieves it **automatically**, as a structural consequence of laziness, without you needing to manually write early-exit logic.

## Streams Are Single-Use

```java
Stream<String> stream = names.stream();
stream.forEach(System.out::println);   // consumes the stream
stream.forEach(System.out::println);   // ⚠️ throws IllegalStateException: stream has already been operated upon or closed
```

**A `Stream` can be traversed (via its terminal operation) exactly once.** This is a direct, structural consequence of what a stream actually *is* — a one-time, single-pass description of a computation, not a reusable data structure. If you need to run the same processing logic again, you must create a **new** stream from the original source (`names.stream()` again) — the underlying `List` remains fully reusable (Module 10); it's specifically the `Stream` **pipeline object itself** that's single-use.

## Creating Streams

```java
List<String> list = List.of("a", "b", "c");
Stream<String> s1 = list.stream();               // from a Collection (most common)

Stream<String> s2 = Stream.of("a", "b", "c");        // directly from individual values

int[] arr = {1, 2, 3};
IntStream s3 = Arrays.stream(arr);                     // from an array (Module 09) --
                                                           // NOTE: IntStream, a primitive-specialized
                                                           // stream (directly analogous to Module 17,
                                                           // Topic 4's primitive functional interfaces --
                                                           // avoids autoboxing for numeric processing!)

Stream<Integer> s4 = Stream.iterate(1, n -> n * 2).limit(5);   // an INFINITE stream (1, 2, 4, 8, 16, ...),
                                                                   // MUST be bounded with limit() (Topic 2)
                                                                   // before any terminal operation, or it
                                                                   // would run forever
```

**`IntStream`/`LongStream`/`DoubleStream` are directly analogous to Module 17, Topic 4's primitive-specialized functional interfaces** — avoiding the real, measurable autoboxing overhead (Module 03, Topic 6) that a `Stream<Integer>` would otherwise incur across potentially millions of elements.

## Real-World Analogy

Think of a `Stream` like a **factory assembly line's blueprint**, not the factory floor itself. Describing the line ("items arrive here, get filtered here, get repainted here, get boxed here") doesn't move a single physical item — it's just a **plan**. Only when you actually **start the conveyor belt** (a terminal operation) do items genuinely flow through every station, in one continuous pass, from start to finish. And once that one production run finishes, the exact same blueprint instance can't be "restarted" from where it left off — you'd set up a fresh run (a new stream) from the original raw materials (the source collection) if you needed to process them again.

## Advantages

- The pipeline model, combined with laziness, enables genuine optimizations (short-circuiting) automatically, without manual early-exit logic.
- Declarative, chainable syntax (`filter`/`map`/`collect`) is often significantly more readable than the equivalent explicit loop, especially for multi-step transformations.
- The exact same API works uniformly across in-memory collections, arrays, and (Topic 5) parallel execution.

## Disadvantages / Trade-offs

- The mental model (stream ≠ data structure, laziness, single-use) genuinely differs from everyday `Collection` usage and takes real, deliberate learning to internalize correctly.
- Debugging a long, chained stream pipeline can be less immediately intuitive than stepping through an equivalent explicit loop, especially before you're fluent with the model.

## Best Practices

- Always remember a `Stream` is a one-time-use pipeline description, not a reusable container — create a fresh stream from the source collection each time you need to reprocess it.
- Bound infinite streams (`Stream.iterate`, `Stream.generate`) with `limit(...)` before any terminal operation.
- Use primitive-specialized streams (`IntStream`, etc.) for numeric processing at scale, avoiding unnecessary autoboxing.

## Common Mistakes

- Attempting to reuse a `Stream` object after a terminal operation has already consumed it.
- Assuming intermediate operations execute immediately as they're called, rather than only once a terminal operation triggers the entire pipeline.
- Forgetting to bound an infinite stream (`Stream.iterate`/`Stream.generate`) with `limit(...)`, causing the terminal operation to run forever.

## Interview Questions

1. **Q: Is a `Stream` a data structure?**
   A: No — it's a one-time-use pipeline describing a sequence of operations to perform over a data source (like a `Collection`, Module 10). It stores no elements itself; the underlying source does.

2. **Q: Why don't intermediate operations like `filter`/`map` execute immediately when called?**
   A: Streams are lazy — intermediate operations only build up a description of the pipeline; the entire pipeline actually executes, element by element, only when a terminal operation (like `collect`/`forEach`) is called, enabling optimizations like short-circuiting.

3. **Q: Can a `Stream` be reused after its terminal operation runs?**
   A: No — attempting to reuse a consumed stream throws `IllegalStateException`. A new stream must be created from the source collection to reprocess the same data.

## Summary

- A `Stream` describes a computation pipeline over a data source — it is not itself a data structure, and stores no elements.
- The pipeline model: **source** → any number of **intermediate operations** (lazy, each returning a new stream) → one **terminal operation** (actually triggers execution).
- Streams are **single-use** — once a terminal operation runs, the stream is consumed and cannot be reused.
- Primitive-specialized streams (`IntStream`, etc.) avoid autoboxing overhead, directly analogous to Module 17, Topic 4's primitive functional interfaces.