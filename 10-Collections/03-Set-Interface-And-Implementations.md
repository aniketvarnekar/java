# The `Set` Interface & Implementations

## Learning Objectives

- Use `HashSet`, `LinkedHashSet`, and `TreeSet` correctly
- Understand precisely why `Set` implementations depend entirely on Module 07's `equals()`/`hashCode()` contract
- Choose the right `Set` implementation based on ordering needs

## Prerequisites

Module 07 Topic 3 (`equals()`/`hashCode()` — this topic is that lesson's direct, practical payoff), [02 — The `List` Interface & Implementations](02-List-Interface-And-Implementations.md)

## Motivation

This topic is where Module 07's `equals()`/`hashCode()` contract stops being an abstract rule and becomes something you rely on every single time you use a `Set`. If you understood *why* that contract exists, `HashSet`'s behavior here will feel completely natural — if you didn't, this is where the consequences become concrete and visible.

## The `Set` Interface

```java
Set<String> tags = new HashSet<>();
tags.add("java");
tags.add("backend");
tags.add("java");        // DUPLICATE -- silently ignored, NOT added again
System.out.println(tags.size());   // 2, not 3
```

`Set<E>` extends `Collection<E>` **without adding any new methods** — its entire distinguishing behavior is a **contractual guarantee**: no two elements in a `Set` are ever `equals()`-equal to each other. This is enforced entirely through **how** `add()` behaves (silently rejecting duplicates), not through a different method signature.

## `HashSet` — Backed by a `HashMap` Internally

```java
Set<String> set = new HashSet<>();
```

**A genuinely interesting implementation fact**: `HashSet` is internally implemented as a thin wrapper around a `HashMap<E, Object>` — every element you add becomes a **key** in that internal map, with a single, shared dummy placeholder value. This is precisely why `HashSet`'s performance characteristics and internal mechanics (Topic 4 covers `HashMap`'s bucket structure in full depth) are identical to `HashMap`'s key-handling behavior.

**This is EXACTLY why `HashSet` requires correct `equals()`/`hashCode()` on its elements** — recall Module 07, Topic 3's full explanation: a hash-based structure uses `hashCode()` to find the right "bucket," then `equals()` to confirm an exact match within that bucket. If you add a custom object to a `HashSet` without correctly overriding both methods together, you get **exactly** Module 07, Topic 3's demonstrated bug:

```java
class Point {
    int x, y;
    Point(int x, int y) { this.x = x; this.y = y; }
    // NO equals()/hashCode() override -- uses Object's identity-based defaults
}

Set<Point> points = new HashSet<>();
points.add(new Point(1, 2));
points.add(new Point(1, 2));            // "duplicate" content, but DIFFERENT object identity
System.out.println(points.size());        // 2 !! -- both were added, since equals() (identity-based,
                                              // Object's default) says they're DIFFERENT objects,
                                              // even though their content is logically identical
```

**With correct `equals()`/`hashCode()` (Module 07, Topic 3's template), this works exactly as expected** — the second `add()` is correctly recognized as a duplicate and silently rejected.

**`HashSet` provides no ordering guarantee at all** — iterating it may produce elements in an order that appears essentially arbitrary (actually determined by each element's hash code and the internal bucket structure — Topic 4), and this order is **not guaranteed to be stable** across different runs or JDK versions.

## `LinkedHashSet` — `HashSet` Plus Insertion Order

```java
Set<String> set = new LinkedHashSet<>();
set.add("banana");
set.add("apple");
set.add("cherry");
System.out.println(set);   // [banana, apple, cherry]  -- PRESERVES insertion order!
```

`LinkedHashSet` extends `HashSet`, adding an internal **doubly-linked list** threading through all entries specifically to track and preserve **insertion order** — giving you `HashSet`'s fast, hash-based performance **plus** predictable iteration order, at the cost of a small amount of extra memory (for the linking structure) and bookkeeping overhead per operation, compared to plain `HashSet`.

## `TreeSet` — Sorted Order, Backed by a Red-Black Tree

```java
Set<Integer> set = new TreeSet<>();
set.add(5);
set.add(1);
set.add(3);
System.out.println(set);   // [1, 3, 5]  -- ALWAYS sorted, automatically, regardless of insertion order!
```

`TreeSet` maintains its elements in **sorted order** at all times, internally backed by a **self-balancing binary search tree** (specifically a Red-Black Tree — full algorithmic depth beyond this course's Core Java scope, but the *concept* — a balanced tree keeping operations efficient — matters). Because it needs to compare elements to determine sort order, **`TreeSet` requires its elements to be mutually comparable** — either they implement `Comparable` (Topic 7), or you supply a `Comparator` (also Topic 7) when constructing the `TreeSet`.

```java
Set<String> names = new TreeSet<>();   // String implements Comparable -- natural alphabetical order
names.add("Charlie");
names.add("Alice");
names.add("Bob");
System.out.println(names);   // [Alice, Bob, Charlie]
```

**A critical, easy-to-miss fact**: `TreeSet` (like `TreeMap`, Topic 4) determines "duplicate" status using its **comparison logic** (`compareTo`/`Comparator`), **not** `equals()`/`hashCode()` — an object considered "equal" by `compareTo` (returning `0`) is treated as a duplicate by `TreeSet`, **even if `equals()` would say otherwise**. This is a genuinely subtle, real interview trap, worth remembering precisely: `HashSet`/`LinkedHashSet` rely on `equals()`/`hashCode()`; `TreeSet` relies on `compareTo`/`Comparator` instead.

## The Three-Way Comparison

| | `HashSet` | `LinkedHashSet` | `TreeSet` |
|---|---|---|---|
| Internal structure | `HashMap` wrapper | `HashMap` wrapper + linked list | Red-Black Tree |
| Iteration order | Unspecified/arbitrary | **Insertion order** | **Sorted order** |
| `add`/`remove`/`contains` | **O(1)** average | **O(1)** average | **O(log n)** |
| Requires elements to be... | Correct `equals()`/`hashCode()` | Correct `equals()`/`hashCode()` | `Comparable` or a supplied `Comparator` |
| Duplicate detection basis | `equals()`/`hashCode()` | `equals()`/`hashCode()` | `compareTo`/`Comparator` (returns 0) |
| Allows `null`? | One `null` element | One `null` element | No (`NullPointerException` on `compareTo`) |

## When to Use Each

- **`HashSet`**: the default choice — fastest, when you don't care about iteration order at all (the overwhelmingly common case).
- **`LinkedHashSet`**: when you need `HashSet`'s performance **and** predictable, insertion-order iteration (e.g., processing unique items in the order they were first seen).
- **`TreeSet`**: when you need elements maintained in **sorted** order at all times, or need range-based operations (`headSet`, `tailSet`, `first`, `last` — additional `TreeSet`/`NavigableSet` methods beyond `Set`'s basic contract).

## Real-World Analogy

Think of `HashSet` like a **pile of uniquely-labeled poker chips thrown into a bin** — fast to check "do I already have a chip with this exact label" (hash lookup), but the bin's physical arrangement is essentially arbitrary. `LinkedHashSet` is the same bin, but chips are **also threaded on a string in the exact order they were dropped in** — same fast lookup, plus a predictable retrieval order. `TreeSet` is like a **fully alphabetized filing cabinet** — slightly slower to file a new item (must find its correct sorted position, Topic 7), but you can always retrieve everything in perfect order, and easily find "everything between M and R" (range queries).

## Advantages

- `Set` implementations provide the "no duplicates" guarantee entirely automatically, using either hashing (`HashSet`/`LinkedHashSet`) or comparison (`TreeSet`) — no manual duplicate-checking code ever needed.
- Three implementations covering the full practical spectrum: fastest-with-no-order-guarantee, fast-with-insertion-order, and slower-but-always-sorted.

## Disadvantages / Trade-offs

- `HashSet`'s lack of any ordering guarantee can be surprising if not anticipated — printing/iterating it may look "random" (though it's actually deterministic given the same hash codes and internal state, just not meaningfully predictable from a user's perspective).
- `TreeSet`'s O(log n) operations are genuinely slower than `HashSet`'s O(1) — only pay this cost when sorted order is actually needed.
- Getting `equals()`/`hashCode()` wrong on custom objects silently breaks `HashSet`/`LinkedHashSet` correctness (Module 07, Topic 3), while `TreeSet` has its own, separate correctness dependency on `compareTo`/`Comparator`.

## Best Practices

- Default to `HashSet` unless you specifically need insertion order (`LinkedHashSet`) or sorted order (`TreeSet`).
- Always correctly implement `equals()`/`hashCode()` (Module 07, Topic 3's template) for any custom class you intend to store in a `HashSet`/`LinkedHashSet`.
- Remember `TreeSet` uses `compareTo`/`Comparator` for duplicate detection, not `equals()`/`hashCode()` — ensure your comparison logic is consistent with your equality logic if both matter for your use case (Topic 7 discusses this "consistency with equals" concern precisely).

## Common Mistakes

- Storing custom objects in a `HashSet` without correctly overriding `equals()`/`hashCode()` together, resulting in silent, unexpected duplicates.
- Assuming `HashSet` iteration order is meaningful or stable — it's neither guaranteed nor something to rely on.
- Forgetting `TreeSet` requires `Comparable` elements (or a supplied `Comparator`) and throws `ClassCastException` if elements aren't mutually comparable.
- Assuming `TreeSet`'s duplicate detection uses `equals()` — it uses `compareTo`/`Comparator` instead, which can produce surprising results if the two are inconsistent.

## Interview Questions

1. **Q: What is `HashSet` internally backed by, and why does this matter?**
   A: A `HashMap`, with each element stored as a key mapped to a shared dummy value. This is exactly why `HashSet` inherits `HashMap`'s hash-bucket-based performance characteristics, and exactly why it depends entirely on correct `equals()`/`hashCode()` implementations on its elements (Module 07, Topic 3) for correctness.

2. **Q: What's the difference between `HashSet`, `LinkedHashSet`, and `TreeSet`?**
   A: `HashSet` offers no ordering guarantee with O(1) average operations. `LinkedHashSet` adds insertion-order iteration on top of `HashSet`'s performance. `TreeSet` maintains elements in sorted order at all times (O(log n) operations), requiring elements to be `Comparable` or given a `Comparator`.

3. **Q: Does `TreeSet` use `equals()` to detect duplicates?**
   A: No — it uses `compareTo()` (or a supplied `Comparator`) returning `0` to determine duplicate status, which is a genuinely different basis than `HashSet`/`LinkedHashSet`'s reliance on `equals()`/`hashCode()`. This can produce surprising results if an object's comparison logic isn't consistent with its equality logic.

## Summary

- `Set<E>` guarantees no duplicate elements, adding no new methods beyond `Collection` — the guarantee is behavioral, enforced by `add()`.
- **`HashSet`**: fastest (O(1) average), no ordering guarantee, internally a `HashMap` wrapper, depends entirely on correct `equals()`/`hashCode()`.
- **`LinkedHashSet`**: `HashSet`'s performance plus insertion-order iteration.
- **`TreeSet`**: always sorted (O(log n)), requires `Comparable`/`Comparator`, uses `compareTo` (not `equals()`) for duplicate detection.
- Default to `HashSet`; use `LinkedHashSet` for insertion order; use `TreeSet` for sorted order or range queries.

## Exercises

1. Write a `Point` class, store two logically-identical instances in a `HashSet` without overriding `equals()`/`hashCode()`, observe the resulting duplicate bug, then fix it using Module 07, Topic 3's correct template.
2. Given the same set of unique words added in a specific order, demonstrate the different iteration output of `HashSet`, `LinkedHashSet`, and `TreeSet` for the same input.
3. Explain precisely why `TreeSet`'s duplicate detection can disagree with `equals()`-based duplicate detection, and construct a small example where a `TreeSet` and a `HashSet` would treat the same two objects differently (one as duplicates, one as distinct).

---

**Previous:** [02 — The `List` Interface & Implementations](02-List-Interface-And-Implementations.md) · **Next:** [04 — The `Map` Interface & Implementations](04-Map-Interface-And-Implementations.md)
