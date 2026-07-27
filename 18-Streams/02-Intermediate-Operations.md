# Intermediate Operations

## Learning Objectives

- Use `map`, `filter`, `sorted`, `distinct`, `limit`/`skip` correctly
- Understand `flatMap` precisely, and why it's genuinely different from `map`
- Recognize which Module 17 functional interface powers each operation

## Prerequisites

[01 — Stream Fundamentals](01-Stream-Fundamentals.md), Module 17 (especially Topic 4's `Function`/`Predicate`)

## Motivation

This topic is where Module 17's functional interfaces become concretely, constantly useful — every intermediate operation covered here takes exactly one of `Function`, `Predicate`, or `Comparator` as its argument, directly reusing vocabulary you already have.

## `filter` — Keep Elements Matching a `Predicate`

```java
List<String> names = List.of("Aniket", "Priya", "Bo", "Rahul");

List<String> longNames = names.stream()
    .filter(name -> name.length() > 3)     // Predicate<String> (Module 17, Topic 4)
    .collect(Collectors.toList());
// ["Aniket", "Priya", "Rahul"]
```

`filter` takes a `Predicate<T>` and produces a new stream containing only the elements for which it returns `true` — directly analogous to a `for` loop with an `if` (Module 04) wrapped around a `continue`, but expressed declaratively.

## `map` — Transform Each Element via a `Function`

```java
List<Integer> lengths = names.stream()
    .map(String::length)     // Function<String, Integer> (Module 17, Topic 3's method reference)
    .collect(Collectors.toList());
// [6, 5, 2, 5]
```

`map` takes a `Function<T, R>` and produces a new stream where **every** element has been transformed — the stream's **element type can change** (here, `Stream<String>` becomes `Stream<Integer>`), unlike `filter`, which only ever removes elements without changing their type.

## `sorted` — Order Elements

```java
List<String> sortedNames = names.stream()
    .sorted()                                     // natural order (Comparable, Module 10, Topic 7)
    .collect(Collectors.toList());

List<String> sortedByLength = names.stream()
    .sorted(Comparator.comparing(String::length))    // custom order (Comparator, Module 10, Topic 7)
    .collect(Collectors.toList());
```

**This directly reuses Module 10, Topic 7's `Comparable`/`Comparator` vocabulary** — `sorted()` (no argument) uses natural ordering; `sorted(Comparator)` uses a supplied comparison strategy, exactly the same distinction taught there, now applied within a stream pipeline.

## `distinct` — Remove Duplicates

```java
Stream.of(1, 2, 2, 3, 3, 3).distinct();   // 1, 2, 3
```

**`distinct` uses `equals()` (Module 07, Topic 3) to determine duplicates** — for custom objects, this means `distinct()`'s correctness depends entirely on a correctly-implemented `equals()`/`hashCode()` (exactly Module 07, Topic 3's contract, and Module 10, Topic 3's `HashSet` dependency, applied here identically).

## `limit`/`skip` — Bound the Stream

```java
Stream.of(1, 2, 3, 4, 5).limit(3);    // 1, 2, 3
Stream.of(1, 2, 3, 4, 5).skip(2);       // 3, 4, 5
Stream.of(1, 2, 3, 4, 5).skip(1).limit(2);   // 2, 3 -- classic PAGINATION pattern
```

**`limit`/`skip` are exactly what makes Topic 1's `Stream.iterate(...)`-style infinite streams safely usable** — bounding them to a finite, processable size before any terminal operation runs.

## `flatMap` — The Genuinely Different One

**This is the intermediate operation most learners find confusing on first encounter** — directly analogous to Module 15, Topic 6's `CompletableFuture.thenCompose` vs. `thenApply` distinction:

```java
List<List<String>> nestedLists = List.of(
    List.of("a", "b"),
    List.of("c", "d", "e")
);

// map alone produces a STREAM OF STREAMS -- awkward, rarely what you actually want:
Stream<Stream<String>> awkward = nestedLists.stream().map(list -> list.stream());

// flatMap FLATTENS the result into ONE single-level stream:
List<String> flat = nestedLists.stream()
    .flatMap(list -> list.stream())    // Function<List<String>, Stream<String>>
    .collect(Collectors.toList());
// ["a", "b", "c", "d", "e"]  -- ONE flat list, not a list of lists!
```

**Why does `flatMap` exist as a distinct operation from `map`?** When your mapping function itself produces **another stream** (or, commonly, a `List`/`Collection` you convert to a stream) for **each** input element, plain `map` would leave you with an awkward, nested `Stream<Stream<T>>` — genuinely unwieldy to work with directly. **`flatMap` automatically flattens** this one level, merging every inner stream's elements into a single, unified outer stream — exactly the same "avoid awkward nesting" motivation behind `CompletableFuture.thenCompose` (Module 15, Topic 6) and `Optional`'s design (Module 17, Topic 5), applied here to streams specifically.

```java
// A genuinely common, real use case: getting every word across many sentences
List<String> sentences = List.of("Hello world", "Java is great");
List<String> words = sentences.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
    .collect(Collectors.toList());
// ["Hello", "world", "Java", "is", "great"]
```

## Chaining Multiple Intermediate Operations

```java
List<String> result = List.of("Aniket", "Priya", "Bo", "Rahul", "Priya")
    .stream()
    .filter(name -> name.length() > 3)      // "Aniket", "Priya", "Rahul", "Priya"
    .distinct()                                // "Aniket", "Priya", "Rahul"
    .map(String::toUpperCase)                   // "ANIKET", "PRIYA", "RAHUL"
    .sorted()                                     // "ANIKET", "PRIYA", "RAHUL" (already sorted here)
    .collect(Collectors.toList());
```

**Each operation processes the stream produced by the previous one, in the exact order written** — this reads almost like a specification of the transformation, directly communicating intent far more clearly than an equivalent nested-loop-with-conditionals implementation would.

## Real-World Analogy

Think of `filter` like a **quality-control inspector removing defective items from a conveyor belt**, without changing the surviving items at all. Think of `map` like a **painting station**, transforming every single item that passes through (same number of items, different appearance/type). Think of `flatMap` like **unpacking crates that each contain several smaller items, placing all the individual items directly onto the main belt** rather than leaving the crates themselves (a "stream of crates," each holding its own inner stream) on the belt — you want the actual contents flowing individually, not nested boxes.

## Advantages

- Each operation is small, focused, and independently composable — directly mirroring Module 17, Topic 4's functional composition philosophy.
- `flatMap` elegantly solves the "nested collections" problem that would otherwise require manual, explicit loop-within-a-loop code.
- Chained pipelines read close to a plain-English specification of the transformation.

## Disadvantages / Trade-offs

- `flatMap`'s purpose and mechanics are genuinely non-obvious on first encounter — a real learning curve, even with `thenCompose`/`Optional` as prior analogies from Module 15/17.
- `distinct()`'s correctness silently depends on well-implemented `equals()`/`hashCode()` for custom object types — an easy detail to overlook.

## Best Practices

- Reach for `flatMap` specifically when your mapping function itself produces a stream/collection per element — recognizing this "would produce nested streams" signal is the key trigger.
- Ensure custom classes used with `distinct()` (or `Collectors.toSet()`, Topic 4) have correctly implemented `equals()`/`hashCode()` (Module 07, Topic 3).
- Order operations deliberately — filtering before an expensive `map` transformation, for instance, avoids wasted work on elements that would be filtered out anyway.

## Common Mistakes

- Using `map` where `flatMap` was needed, ending up with an awkward, hard-to-use `Stream<Stream<T>>`.
- Forgetting `distinct()`'s reliance on `equals()`/`hashCode()`, producing unexpected duplicate retention for custom objects with default (identity-based) equality.
- Chaining `sorted()` before an expensive `filter`, doing unnecessary sorting work on elements that will be discarded anyway (a minor but real, avoidable inefficiency).

## Interview Questions

1. **Q: What's the difference between `map` and `flatMap`?**
   A: `map` transforms each element into exactly one new element (the stream's size stays the same, only element type/value changes). `flatMap` is used when the mapping function itself produces a stream (or collection) per input element, automatically flattening those inner streams into one single, unified outer stream — avoiding an awkward nested `Stream<Stream<T>>`.

2. **Q: What functional interface does `filter` take, and what about `map`?**
   A: `filter` takes a `Predicate<T>` (Module 17, Topic 4), keeping elements for which it returns true. `map` takes a `Function<T, R>`, transforming every element, potentially changing the stream's element type.

3. **Q: What does `distinct()` rely on to determine duplicates?**
   A: `equals()` (and, implicitly, correct `hashCode()` for efficient internal implementation) — Module 07, Topic 3's equals/hashCode contract applies directly; custom objects need it correctly implemented for `distinct()` to behave as expected.

## Summary

- **`filter`** (takes a `Predicate`) keeps matching elements; **`map`** (takes a `Function`) transforms every element, possibly changing type.
- **`sorted()`**/**`sorted(Comparator)`** reuse Module 10, Topic 7's ordering vocabulary directly.
- **`distinct()`** relies on `equals()`/`hashCode()` (Module 07, Topic 3); **`limit`/`skip`** bound a stream, essential for infinite streams (Topic 1).
- **`flatMap`** flattens a per-element stream-producing mapping into one unified stream — the stream-specific instance of the same "avoid awkward nesting" pattern seen in `CompletableFuture.thenCompose` (Module 15) and `Optional` (Module 17).

## Exercises

1. Given a `List<String>` of sentences, use `flatMap` to produce a single flat `List<String>` of every individual word across all sentences.
2. Write a pipeline that filters a `List<Integer>` to only even numbers, sorts them descending, and limits to the top 3.
3. Explain, precisely, why `map(list -> list.stream())` produces a `Stream<Stream<T>>`, and why `flatMap` avoids this.

---

**Previous:** [01 — Stream Fundamentals](01-Stream-Fundamentals.md) · **Next:** [03 — Terminal Operations & `reduce`](03-Terminal-Operations-And-Reduce.md)
