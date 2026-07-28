# `Collectors`

## Learning Objectives

- Use `Collectors.toList`/`toSet`/`toMap` correctly, including merge-function conflict handling
- Use `Collectors.groupingBy` and `partitioningBy` for real-world aggregation tasks
- Use `Collectors.joining` and summary-statistics collectors

## Prerequisites

[03 — Terminal Operations & `reduce`](03-terminal-operations-and-reduce.md), Module 10 (Collections)

## Motivation

`collect(...)` is the terminal operation you'll reach for most often in real code — and `java.util.stream.Collectors` is the utility class providing nearly every collecting strategy you'll need. This topic is the practical payoff of the entire module: turning a stream pipeline into an actual, usable `List`, `Map`, `String`, or aggregated statistic.

## `toList`/`toSet` — The Basics

```java
List<String> list = names.stream().filter(n -> n.length() > 3).collect(Collectors.toList());
Set<String> set = names.stream().collect(Collectors.toSet());

// Java 16+ shortcut, equivalent to Collectors.toUnmodifiableList():
List<String> list2 = names.stream().filter(n -> n.length() > 3).toList();
```

**`Collectors.toSet()` uses `equals()`/`hashCode()` for deduplication** — exactly Module 10, Topic 3's `HashSet` mechanics, and exactly Topic 2's `distinct()` mechanics, all the same underlying contract (Module 07, Topic 3), applied consistently throughout the Collections and Streams APIs.

## `toMap` — Building a `Map` From a Stream

```java
Map<String, Integer> nameToLength = names.stream()
    .collect(Collectors.toMap(
        name -> name,           // KEY function
        String::length             // VALUE function
    ));
```

**A genuinely important, real gotcha**: `toMap` throws `IllegalStateException` if the **key function** produces duplicate keys for two different elements — `Map` (Module 10, Topic 4) cannot have duplicate keys, and `toMap` refuses to silently pick a "winner" without your explicit direction:

```java
// If TWO people share the same name, this THROWS:
Map<String, Integer> broken = people.stream()
    .collect(Collectors.toMap(Person::getName, Person::getAge));   // ⚠️ IllegalStateException
                                                                       // if any two names collide!

// The FIX: supply a MERGE function, resolving conflicts explicitly:
Map<String, Integer> fixed = people.stream()
    .collect(Collectors.toMap(
        Person::getName,
        Person::getAge,
        (age1, age2) -> age1        // on conflict, KEEP the first one encountered (your explicit choice)
    ));
```

**This directly connects to Module 10, Topic 4's `HashMap.merge(...)` method** — `toMap`'s optional third argument plays exactly the same conflict-resolution role.

## `groupingBy` — The Most Powerful, Most Commonly Used Collector

```java
List<Person> people = List.of(
    new Person("Aniket", "Engineering"),
    new Person("Priya", "Engineering"),
    new Person("Rahul", "Sales")
);

Map<String, List<Person>> byDepartment = people.stream()
    .collect(Collectors.groupingBy(Person::getDepartment));
// {"Engineering": [Aniket, Priya], "Sales": [Rahul]}
```

**`groupingBy` is the stream-based equivalent of a SQL `GROUP BY` clause** — it partitions the stream's elements into a `Map`, keyed by whatever the classifier function (a `Function<T, K>`, Module 17, Topic 4) returns, with each key mapping to a `List` of every matching element.

### `groupingBy` With a Downstream Collector — Composing Collectors

```java
Map<String, Long> countByDepartment = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDepartment,
        Collectors.counting()        // a DOWNSTREAM collector -- instead of a List, COUNT each group
    ));
// {"Engineering": 2, "Sales": 1}

Map<String, List<String>> namesByDepartment = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDepartment,
        Collectors.mapping(Person::getName, Collectors.toList())   // TRANSFORM each group's elements
    ));
// {"Engineering": ["Aniket", "Priya"], "Sales": ["Rahul"]}
```

**This is a genuinely powerful, composable pattern**: `groupingBy`'s second argument accepts **any other `Collector`**, letting you nest arbitrarily sophisticated aggregation logic — group by one thing, then within each group, count, average, collect names only, or even group by a **second** criterion (a nested `groupingBy`) — directly mirroring Module 17, Topic 4's functional composition philosophy, applied specifically to collecting strategies.

## `partitioningBy` — A Specialized, Binary `groupingBy`

```java
Map<Boolean, List<Person>> partitioned = people.stream()
    .collect(Collectors.partitioningBy(p -> p.getDepartment().equals("Engineering")));
// {false: [Rahul], true: [Aniket, Priya]}
```

**`partitioningBy` is specifically for a `Predicate`-based (Module 17, Topic 4) binary split** — always producing a `Map<Boolean, List<T>>` with **exactly two keys** (`true`/`false`), guaranteed present even if one group is empty (unlike `groupingBy`, which only includes keys that actually occur). Use `partitioningBy` specifically when your grouping criterion is genuinely binary; use `groupingBy` for anything with more than two possible groups.

## `joining` — Building a `String`

```java
String joined = names.stream().collect(Collectors.joining(", "));           // "Aniket, Priya, Rahul"
String joined2 = names.stream().collect(Collectors.joining(", ", "[", "]"));   // "[Aniket, Priya, Rahul]"
```

**This is directly analogous to Module 08, Topic 3's `String.join(...)`**, but usable as a stream terminal operation, letting you filter/transform/sort elements first, then join the final result — combining Topics 2–3's pipeline power with Module 08's string-joining convenience in one fluent chain.

## Summary Statistics

```java
IntSummaryStatistics stats = people.stream()
    .collect(Collectors.summarizingInt(Person::getAge));

stats.getAverage();   // average age
stats.getMax();          // oldest
stats.getMin();            // youngest
stats.getSum();              // total
stats.getCount();              // how many people
```

**A single collector call producing an entire bundle of common aggregate statistics at once** — genuinely convenient compared to computing each one via separate `reduce`/`min`/`max`/`count` terminal operations, which would require **multiple, separate passes** over the stream (recall Topic 1: a stream is single-use — you'd need a fresh stream for each separate terminal operation otherwise).

## Real-World Analogy

Think of `toList`/`toSet`/`toMap` like **different container types waiting at the end of the assembly line** — a simple box (`List`), a box that automatically rejects duplicates (`Set`), or a filing cabinet requiring a unique label per drawer (`Map`, with `toMap`'s merge function as the explicit instruction for what to do if two items claim the same label). `groupingBy` is like a **sorting station that routes items into different labeled bins based on some property**, with `partitioningBy` being the specific, simpler case of only ever having exactly two bins ("yes" and "no"). Summary statistics are like a **single inspection station that, in one pass, simultaneously records the count, total, average, minimum, and maximum of everything that passed through**, rather than needing separate inspection stations for each individual measurement.

## Advantages

- `Collectors` provides a rich, composable toolkit covering nearly every common aggregation need directly, without hand-written accumulation logic.
- `groupingBy` with downstream collectors enables genuinely sophisticated, SQL-`GROUP BY`-like aggregation in a single, readable, composable expression.
- Summary statistics compute multiple aggregates in a single pass, avoiding the need for several separate stream traversals.

## Disadvantages / Trade-offs

- `toMap`'s duplicate-key exception, while a genuinely correct safety behavior, is a real, common source of confusion for developers who don't anticipate it and forget the merge-function overload.
- Deeply nested `groupingBy`-with-downstream-collectors expressions can become genuinely hard to read if composed too deeply without care.

## Best Practices

- Always supply a merge function to `Collectors.toMap` when key collisions are even remotely possible, rather than discovering the `IllegalStateException` in production.
- Use `partitioningBy` specifically for genuinely binary groupings; use `groupingBy` otherwise.
- Use summary-statistics collectors instead of multiple separate `reduce`/`count`/`max` calls when you need several aggregates from the same data — one pass instead of several (recalling Topic 1's single-use stream constraint).

## Common Mistakes

- Using `Collectors.toMap` without a merge function on data where key collisions are possible, encountering an unexpected `IllegalStateException`.
- Confusing `groupingBy` (any number of resulting groups) with `partitioningBy` (always exactly two, `true`/`false`).
- Computing multiple separate aggregates (count, sum, average) via multiple separate stream traversals instead of a single summary-statistics collector.

## Interview Questions

1. **Q: Why does `Collectors.toMap` throw an exception on duplicate keys by default, and how do you handle it?**
   A: Because a `Map` cannot have duplicate keys (Module 10, Topic 4), and `toMap` refuses to silently choose a winner without explicit direction. Supplying a third argument — a merge function (`BinaryOperator<V>`) — resolves the conflict explicitly, exactly analogous to `HashMap.merge(...)`.

2. **Q: What's the difference between `groupingBy` and `partitioningBy`?**
   A: `groupingBy` partitions elements into any number of groups based on a classifier function's result. `partitioningBy` is a specialized version for a `Predicate`-based binary split, always producing exactly two keys (`true`/`false`), even if one group is empty.

3. **Q: What does a downstream collector let you do with `groupingBy`?**
   A: Compose additional aggregation logic within each group — instead of collecting each group into a plain `List`, you can count elements, transform them, compute statistics, or even nest a second `groupingBy`, all within one composed, readable expression.

## Summary

- **`toList`/`toSet`** collect into basic collections; **`toMap`** requires care around duplicate keys, resolved via an optional merge function.
- **`groupingBy`** partitions a stream into a `Map` keyed by a classifier function, optionally composed with a downstream collector for sophisticated per-group aggregation; **`partitioningBy`** is its specialized binary (`Predicate`-based) form.
- **`joining`** builds a `String`, directly analogous to Module 08's `String.join`; **summary-statistics collectors** compute multiple aggregates in one single pass.

## Exercises

1. Given a `List<Person>` with potential duplicate names, write a `Collectors.toMap` call with a merge function keeping the person with the higher age on conflict.
2. Write a `groupingBy` collector grouping a `List<Order>` by customer, with a downstream `Collectors.summingDouble(Order::getTotal)` computing each customer's total spend.
3. Rewrite a `partitioningBy` example as an equivalent `groupingBy` call, and explain the key difference in the resulting `Map`'s guarantees.

---

**Previous:** [03 — Terminal Operations & `reduce`](03-terminal-operations-and-reduce.md) · **Next:** [05 — Parallel Streams](05-parallel-streams.md)
