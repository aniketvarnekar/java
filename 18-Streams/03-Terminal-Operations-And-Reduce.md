# Terminal Operations & `reduce`

## Learning Objectives

- Use `forEach`, `count`, matching operations, and `min`/`max`/`findFirst` correctly
- Understand `reduce` precisely, including its identity value and accumulator function
- Understand short-circuiting terminal operations and why they can stop a pipeline early

## Prerequisites

[02 — Intermediate Operations](02-Intermediate-Operations.md)

## Motivation

Recall Topic 1: nothing in a stream pipeline actually runs until a terminal operation triggers it. This topic covers the full range of terminal operations — the ones that actually produce a final result (or side effect) from the pipeline you've built.

## `forEach` — A Terminal Operation for Side Effects

```java
names.stream()
    .filter(n -> n.length() > 3)
    .forEach(System.out::println);   // Consumer<T> (Module 17, Topic 4) -- a SIDE EFFECT, no return value
```

**`forEach` is the terminal-operation equivalent of Module 04's for-each loop** — but note it returns `void`; it's meant for side effects (printing, adding to an external collection), not for producing a new value. **If you find yourself using `forEach` to build up a result** (e.g., manually adding to a `List` inside the lambda), **that's almost always a sign `collect` (Topic 4) is the more idiomatic tool** — mutating external state from within a stream pipeline is a recognized anti-pattern, working against the stream model's intent.

## `count`, `min`, `max`, `findFirst`/`findAny`

```java
long total = names.stream().filter(n -> n.length() > 3).count();

Optional<String> longest = names.stream()
    .max(Comparator.comparing(String::length));   // returns Optional<T> (Module 17, Topic 5!) --
                                                       // since the stream MIGHT be empty

Optional<String> first = names.stream().filter(n -> n.startsWith("P")).findFirst();
```

**Notice `max`/`min`/`findFirst`/`findAny` all return `Optional<T>`** — a direct, concrete, everyday application of Module 17, Topic 5's `Optional` — since a stream (or the filtered result within it) might genuinely be empty, and these operations honestly communicate that possibility through their return type, rather than risking a `null` or throwing on an empty stream.

## Matching Operations — `anyMatch`/`allMatch`/`noneMatch`

```java
boolean hasLongName = names.stream().anyMatch(n -> n.length() > 5);    // true if AT LEAST ONE matches
boolean allLong = names.stream().allMatch(n -> n.length() > 3);          // true if EVERY element matches
boolean noneEmpty = names.stream().noneMatch(String::isEmpty);             // true if NO element matches
```

**These are all `Predicate`-based (Module 17, Topic 4) and, importantly, `short-circuiting`**: `anyMatch` stops processing the **instant** it finds a match (no need to check the rest); `allMatch` stops the **instant** it finds a non-match; `noneMatch` stops the **instant** it finds a match. This is Topic 1's laziness/short-circuiting benefit made concrete — for a very large stream, these operations can potentially finish almost immediately, without ever touching most of the data, exactly mirroring Module 03, Topic 5's `&&`/`||` short-circuit evaluation, applied at the stream level.

## `reduce` — The Most General, Most Powerful Terminal Operation

**This is the deepest, most important terminal operation in this topic.** `reduce` combines every element of a stream into a **single** result, by repeatedly applying a combining function:

```java
int sum = Stream.of(1, 2, 3, 4, 5)
    .reduce(0, (accumulator, element) -> accumulator + element);   // 15

// or, using a method reference:
int sum2 = Stream.of(1, 2, 3, 4, 5).reduce(0, Integer::sum);
```

**The two arguments, precisely:**
- **Identity** (`0`): the starting value, AND the result if the stream is genuinely empty.
- **Accumulator** (a `BinaryOperator<T>`, Module 17, Topic 4's two-argument function family): takes the running total so far and the next element, returns the new running total.

```
 reduce(0, (acc, x) -> acc + x)  over [1, 2, 3, 4, 5]:

 acc=0, x=1  ->  acc becomes 1
 acc=1, x=2  ->  acc becomes 3
 acc=3, x=3  ->  acc becomes 6
 acc=6, x=4  ->  acc becomes 10
 acc=10, x=5 ->  acc becomes 15    <- FINAL RESULT
```

**This is exactly the general pattern behind `sum`, `count`, `max`, `min`** — all of them are conceptually specific, convenient shortcuts for particular common `reduce` operations, provided directly on the Stream API because they're used so frequently that a dedicated, more readable method is worth having.

```java
// Without a dedicated identity value -- returns Optional<T> (again, Module 17, Topic 5!),
// since there's no meaningful "default" starting value to fall back to for an empty stream:
Optional<Integer> max = Stream.of(3, 1, 4, 1, 5).reduce((a, b) -> a > b ? a : b);
```

## `collect` — Previewed Here, Full Depth Topic 4

```java
List<String> result = names.stream()
    .filter(n -> n.length() > 3)
    .collect(Collectors.toList());   // the MOST commonly used terminal operation of all
```

**`collect` is genuinely the terminal operation you'll use most often in real code** — it deserves, and gets, a fully dedicated topic (Topic 4) covering the rich `Collectors` utility class.

## Real-World Analogy

Think of `forEach` like **handing every item on a conveyor belt to a worker who does something with each one individually** (prints it, logs it) but hands nothing back. Think of matching operations (`anyMatch`, etc.) like a **quality inspector who can stop the ENTIRE belt the instant they spot (or fail to spot) what they're looking for**, rather than needing to inspect every single remaining item once the answer is already certain. Think of `reduce` like a **single worker standing at the end of the belt with a running tally sheet**, updating one cumulative total (or combined result) as each item passes by, until the very last item, at which point the tally sheet holds the final, single answer.

## Advantages

- Matching operations and other short-circuiting terminals leverage laziness (Topic 1) for genuine, automatic efficiency on large streams.
- `reduce` provides a single, general, well-understood pattern underlying many common aggregation operations, unifying `sum`/`max`/`min`/`count`-style logic conceptually.
- Returning `Optional<T>` from operations that might legitimately find nothing (`max`, `findFirst`) directly, honestly communicates that possibility through the type system (Module 17, Topic 5).

## Disadvantages / Trade-offs

- Using `forEach` to mutate external state (rather than for genuine side effects like printing) works against the stream model's intent and is a recognized anti-pattern — `collect` (Topic 4) is almost always the better choice for building a result.
- `reduce`, while powerful and general, can be genuinely less readable than a dedicated method (`sum()`, `count()`) for the simple, common cases those dedicated methods already cover well.

## Best Practices

- Use dedicated terminal operations (`count`, `sum` via `IntStream`, `max`/`min`) for their specific common cases; reserve general `reduce` for genuinely custom aggregation logic those dedicated methods don't cover.
- Never use `forEach` to accumulate a result into an external, mutable collection — use `collect` (Topic 4) instead.
- Take advantage of short-circuiting matching operations (`anyMatch`/`allMatch`/`noneMatch`) instead of manually collecting a full result just to check a single boolean condition afterward.

## Common Mistakes

- Using `forEach` with a side-effecting lambda that mutates an external list, instead of the idiomatic `collect`.
- Forgetting that `max`/`min`/`findFirst`/`reduce` (without an identity) return `Optional<T>`, and calling `.get()` without checking presence (Module 17, Topic 5's anti-pattern, directly relevant here).
- Not leveraging short-circuiting matching operations, instead manually collecting a full list just to check `!list.isEmpty()` afterward.

## Interview Questions

1. **Q: Why does `forEach` return `void`, and when is using it considered an anti-pattern?**
   A: It's designed for side effects (printing, logging), not for producing values — using it to mutate an external collection to "build a result" works against the stream model's declarative intent; `collect` (Topic 4) is the idiomatic tool for actually producing a result.

2. **Q: What are `reduce`'s two arguments, and what does each represent?**
   A: An identity value (the starting point, and the result for an empty stream) and an accumulator function (a `BinaryOperator<T>`, taking the running total and the next element, returning the new running total) — applied repeatedly across every element to combine them into a single final result.

3. **Q: Why are `anyMatch`/`allMatch`/`noneMatch` described as "short-circuiting," and why does this matter?**
   A: They stop processing the instant the final answer is already determined (a match found for `anyMatch`, a non-match found for `allMatch`, etc.), rather than needing to examine every remaining element — directly leveraging Topic 1's laziness for genuine, automatic performance benefit on large streams.

## Summary

- **`forEach`** performs a side effect per element, returning nothing — not for building results.
- **`count`/`min`/`max`/`findFirst`/`findAny`** are common, dedicated terminal operations; several return `Optional<T>` (Module 17, Topic 5) since the stream might be empty.
- **`anyMatch`/`allMatch`/`noneMatch`** are `Predicate`-based and short-circuiting, stopping as soon as the final answer is determined.
- **`reduce(identity, accumulator)`** is the general pattern underlying `sum`/`max`/`min`/`count`-style aggregation, combining every element into a single final result.

## Exercises

1. Write a `reduce` that computes the product of all elements in a `Stream<Integer>`, and predict its result for `[1, 2, 3, 4]`.
2. Explain why `Stream.of(3, 1, 4).max(Comparator.naturalOrder())` returns `Optional<Integer>` instead of a plain `Integer`.
3. Write a pipeline using `anyMatch` to check whether any element in a large stream exceeds a threshold, and explain why this can be more efficient than collecting the full filtered list first and then checking `!list.isEmpty()`.

---

**Previous:** [02 — Intermediate Operations](02-Intermediate-Operations.md) · **Next:** [04 — `Collectors`](04-Collectors.md)
