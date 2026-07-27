# The `Collections` Utility Class & Concurrent Collections Preview

## Learning Objectives

- Use `java.util.Collections`'s most important static methods correctly
- Understand unmodifiable/immutable collection wrappers and why they're valuable
- Understand, at a preview level, why `synchronizedList`/`synchronizedMap` are largely superseded by `java.util.concurrent`

## Prerequisites

All prior topics in this module

## Motivation

Just as `java.util.Arrays` (Module 09, Topic 3) provides utility operations for raw arrays, `java.util.Collections` (note the plural — a genuinely easy name to confuse with the singular `Collection` interface from Topic 1) provides utility operations for the Collections Framework as a whole. This closing topic also previews Module 15's concurrent collections, so you know what exists before you need it.

## `Collections.sort()`, `min()`, `max()`

```java
List<Integer> nums = new ArrayList<>(List.of(5, 2, 8, 1));
Collections.sort(nums);                    // natural order (Comparable, Topic 7) -- [1, 2, 5, 8]
Collections.sort(nums, Comparator.reverseOrder());   // custom order (Comparator, Topic 7) -- [8, 5, 2, 1]
Collections.max(nums);                        // 8
Collections.min(nums);                          // 1
```

**Modern note**: since Java 8, `List` itself has an instance method `list.sort(comparator)` (used throughout Topic 7's examples) — functionally equivalent to `Collections.sort(list, comparator)`, and generally preferred in new code for its more direct, fluent syntax. `Collections.sort(list)` remains common in older code and is still perfectly valid.

## Unmodifiable (Read-Only) Views

```java
List<String> mutable = new ArrayList<>(List.of("a", "b", "c"));
List<String> readOnly = Collections.unmodifiableList(mutable);

readOnly.add("d");      // throws UnsupportedOperationException !!
mutable.add("d");         // LEGAL -- and this change IS VISIBLE through 'readOnly' too!
```

**Critical nuance**: `Collections.unmodifiableList(...)` (and its `unmodifiableSet`/`unmodifiableMap` siblings) returns a **view** — a wrapper that **forbids modification through itself**, but does **not** create an independent copy, and does **not** protect against the **original** collection being modified elsewhere (which **does** show through the read-only view, exactly like Module 07, Topic 4's shallow-copy lesson). **This is precisely analogous to Module 09, Topic 4's `Arrays.asList()` quirk** — a wrapper providing a restricted view of the same underlying data, not a genuinely independent, fully protected copy.

**Why is this genuinely useful, despite the nuance?** It's a common, idiomatic way for a class to expose a field's contents to external callers **without** letting them mutate that internal state directly — directly applying Module 05, Topic 2's encapsulation principle:

```java
class Team {
    private final List<String> members = new ArrayList<>();

    public List<String> getMembers() {
        return Collections.unmodifiableList(members);   // callers can READ, but never mutate,
    }                                                       // Team's own internal list directly

    public void addMember(String name) {
        members.add(name);   // the ONLY sanctioned way to modify the list -- through Team's own method
    }
}
```

## Immutable Factory Methods (Java 9+) — A Simpler, More Common Modern Alternative

```java
List<String> immutable = List.of("a", "b", "c");     // genuinely immutable, NOT just a restricted view
Set<String> immutableSet = Set.of("x", "y", "z");
Map<String, Integer> immutableMap = Map.of("a", 1, "b", 2);

immutable.add("d");   // throws UnsupportedOperationException
```

**These Java 9+ static factory methods (`List.of`, `Set.of`, `Map.of`) are generally preferred over `Collections.unmodifiableList(...)` in modern code** — they create genuinely independent, fully immutable collections directly (not a view wrapping a separately-mutable original), and are more concise. `Collections.unmodifiableXxx` remains relevant specifically when you need a **read-only view of an existing, already-populated mutable collection** (as in the `Team` example above) — the two tools solve subtly different problems.

## `Collections.synchronizedList()`, `synchronizedMap()` — And Why They're Largely Superseded

```java
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
```

This wraps a collection so that **every individual method call** is internally `synchronized` (Module 15 covers this mechanism in full depth) — conceptually similar to `Vector`'s/`Hashtable`'s blanket, per-method synchronization (Module 10, Topics 2 and 4), just applied as a wrapper around an arbitrary existing collection instead of being built into a specific class.

**Why is this "largely superseded"?** Wrapping every individual method call in synchronization protects against low-level data corruption, but does **not** make **compound, multi-step operations** (like "check if a key exists, then add it if not" — two separate calls) atomic/safe as a whole — a genuine, real correctness gap under real concurrent use, requiring extra manual synchronization around such sequences regardless. **Modern Java strongly prefers the purpose-built concurrent collections in `java.util.concurrent`** (full depth: Module 15) — most notably **`ConcurrentHashMap`**, which provides much better real concurrent performance (via fine-grained internal locking strategies, rather than one single lock guarding the entire structure) **and** provides genuinely atomic compound operations (like `computeIfAbsent`, `putIfAbsent`) specifically designed for safe concurrent use, which `synchronizedMap` alone does not.

```java
// PREVIEW ONLY -- full depth Module 15:
Map<String, Integer> concurrent = new ConcurrentHashMap<>();
```

**This module deliberately stops at a preview here** — genuine thread-safety, the Java Memory Model, and `java.util.concurrent`'s full toolkit are substantial enough to warrant their own dedicated module (15), building on Module 02's JVM Stack-per-thread model and everything about synchronization this course has flagged along the way.

## Other Useful `Collections` Methods

```java
Collections.reverse(list);        // reverses IN PLACE
Collections.shuffle(list);          // randomizes order IN PLACE
Collections.frequency(list, x);       // counts occurrences of x in list
Collections.emptyList();                // a genuinely immutable, shared empty List instance
Collections.nCopies(5, "x");              // an immutable list: ["x", "x", "x", "x", "x"]
```

## Real-World Analogy

Think of `Collections.unmodifiableList` like **handing someone a museum display case they can look through but not open** — the actual artifacts inside can still be rearranged by museum staff (the original collection) through a back door the visitor doesn't have access to, and any such rearrangement is immediately visible through the display glass too, since it's the same actual artifacts, just viewed through a restricted-access pane. `List.of(...)` (Java 9+), by contrast, is like **casting a genuinely permanent bronze sculpture** — there is no back door at all; nothing about it can ever be rearranged by anyone, from any angle, period.

## Advantages

- `Collections` provides essential, well-tested cross-cutting operations (`sort`, `min`/`max`, unmodifiable wrappers, `shuffle`, etc.) that complement the individual interfaces' own methods.
- Unmodifiable views and immutable factory methods provide real, practical encapsulation tools (Module 05, Topic 2) for safely exposing internal collection state.
- Knowing `ConcurrentHashMap` and friends exist now (even before Module 15's full depth) prevents reaching for the older, less effective `synchronizedXxx` wrappers by default out of unfamiliarity.

## Disadvantages / Trade-offs

- `Collections.unmodifiableXxx`'s "view, not copy" behavior is a genuine, real trap if not precisely understood — mirroring Module 09, Topic 4's `Arrays.asList()` lesson exactly.
- `synchronizedXxx` wrappers provide weaker, less complete concurrency guarantees than purpose-built `java.util.concurrent` collections, despite superficially "looking" thread-safe.

## Best Practices

- Prefer `List.of`/`Set.of`/`Map.of` (Java 9+) for genuinely immutable collections created fresh; use `Collections.unmodifiableXxx` specifically for exposing a read-only view of an existing, separately-mutable collection.
- Prefer `list.sort(comparator)` (instance method) over `Collections.sort(list, comparator)` in new code, for more direct, fluent syntax — both remain valid.
- Prefer `java.util.concurrent` collections (`ConcurrentHashMap`, etc. — Module 15) over `Collections.synchronizedXxx` wrappers for genuine multi-threaded use, once you reach that module.

## Common Mistakes

- Assuming `Collections.unmodifiableList(original)` protects against `original` itself being modified elsewhere — it doesn't; changes to the original remain fully visible through the "unmodifiable" view.
- Using `synchronizedMap` and assuming compound operations (like check-then-act) are automatically safe — they aren't, without additional manual synchronization.
- Confusing `Collections` (the utility class) with `Collection` (the root interface, Topic 1) — genuinely easy to mix up by name alone.

## Interview Questions

1. **Q: Does `Collections.unmodifiableList(list)` create an independent, fully protected copy?**
   A: No — it returns a view wrapping the original list, forbidding modification *through the view itself*, but changes made directly to the original list remain fully visible through the view. For a genuinely independent, immutable copy, use `List.copyOf(list)` or construct via `List.of(...)` (Java 9+).

2. **Q: Why are `Collections.synchronizedMap`/`synchronizedList` considered largely superseded in modern Java?**
   A: They synchronize each individual method call, but don't make multi-step compound operations (like check-then-add) atomic as a whole, requiring extra manual synchronization anyway. Purpose-built `java.util.concurrent` collections like `ConcurrentHashMap` (Module 15) provide better real concurrent performance and genuinely atomic compound operations designed specifically for safe concurrent use.

## Summary

- `java.util.Collections` (plural, distinct from the `Collection` interface) provides utility operations: `sort`, `min`/`max`, `reverse`, `shuffle`, unmodifiable view wrappers, and more.
- `Collections.unmodifiableXxx` returns a restricted **view**, not an independent copy — changes to the original remain visible through it; `List.of`/`Set.of`/`Map.of` (Java 9+) create genuinely independent immutable collections instead.
- `Collections.synchronizedXxx` wrappers provide only per-method synchronization, not atomic compound operations — modern concurrent code should prefer `java.util.concurrent` collections (full depth: Module 15).

## Module-Wide Quick Revision

- The Collections Framework has two root hierarchies: `Iterable → Collection → (List/Set/Queue)`, and the separate `Map` (Topic 1).
- `ArrayList` (array-backed, O(1) random access) vs `LinkedList` (node-chain, O(1) at-ends operations, poor cache locality) — default to `ArrayList` (Topic 2).
- `Set` guarantees no duplicates; `HashSet`/`LinkedHashSet` depend on `equals()`/`hashCode()` (Module 07); `TreeSet` depends on `compareTo`/`Comparator` (Topic 3).
- `HashMap`'s bucket algorithm: hashCode → bucket index, equals() distinguishes collisions within a bucket; load factor 0.75 triggers resize; Java 8 treeifies long chains (Topic 4).
- `Queue`/`Deque` provide FIFO/double-ended access with dual exception/special-value method pairs; `PriorityQueue` retrieves by priority, not insertion order; prefer `ArrayDeque` over `LinkedList` (Topic 5).
- The for-each loop is sugar over `Iterator`; fail-fast `ConcurrentModificationException` protects against unsafe direct modification during iteration — use `Iterator.remove()`/`removeIf()` instead (Topic 6).
- `Comparable` defines one natural order on the class itself; `Comparator` defines unlimited external, pluggable orderings, with modern chaining via `comparing`/`thenComparing`/`reversed` (Topic 7).
- `Collections` utility class complements the framework; prefer `List.of`/etc. for immutability and `java.util.concurrent` for genuine thread safety (this topic).

## Common Pitfalls (Module-Wide)

- Assuming `Map` is a `Collection`.
- Assuming `LinkedList` is generally faster for insertions without accounting for O(n) position-finding cost.
- Storing custom objects in `HashSet`/`HashMap` without correctly implementing `equals()`/`hashCode()` together.
- Modifying a collection directly during a for-each loop over it.
- Assuming `Collections.unmodifiableList` creates an independent copy.

## Mini Quiz (Module-Wide)

1. Why isn't `Map` a `Collection`?
2. Why is `ArrayList.get(i)` O(1) while `LinkedList.get(i)` is O(n)?
3. What two things does `HashMap.put` use `hashCode()` and `equals()` for, respectively?
4. What causes `ConcurrentModificationException`, precisely?
5. What's the difference between `Comparable` and `Comparator`?

*(Answers are derivable from Topics 1, 2, 4, 6, and 7 respectively.)*

---

**Previous:** [07 — `Comparable` & `Comparator`](07-Comparable-And-Comparator.md) · **Next:** [09 — Module Summary, Interview Questions & Exercises](09-Module-Summary-Exercises.md)
