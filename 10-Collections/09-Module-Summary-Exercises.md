# Module 10 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-Module-Overview.md) has been covered:

- [x] **Collections Framework Overview** — the complete `Iterable`/`Collection`/`Map` hierarchy, why `Map` deliberately isn't a `Collection`, `Iterable` as the for-each foundation
- [x] **`List` Interface & Implementations** — `ArrayList` vs. `LinkedList`'s genuinely different internal structures and performance trade-offs (including the crucial cache-locality reality check), `Vector`'s obsolescence
- [x] **`Set` Interface & Implementations** — `HashSet`/`LinkedHashSet`/`TreeSet`, `HashSet`'s total dependence on Module 07's `equals()`/`hashCode()` contract, `TreeSet`'s `compareTo`-based duplicate detection
- [x] **`Map` Interface & Implementations** — `HashMap`'s complete bucket algorithm (hash-spreading, bucket indexing, collision handling), load factor and resizing, Java 8 treeification, `LinkedHashMap`/`TreeMap`/`Hashtable`
- [x] **`Queue` & `Deque`** — dual exception/special-value method pairs, `Deque` as both queue and stack, `PriorityQueue`'s binary-heap-based priority ordering, `ArrayDeque` vs `LinkedList`
- [x] **Iterators & Iteration** — `Iterator` as the for-each mechanism, `Iterator.remove()`, the precise `ConcurrentModificationException`/fail-fast mechanism, `ListIterator`
- [x] **`Comparable` & `Comparator`** — natural vs. custom ordering, modern chaining (`comparing`/`thenComparing`/`reversed`), "consistent with equals"
- [x] **`Collections` Utility Class & Concurrent Preview** — `sort`/`unmodifiableXxx`/`List.of`, and why `synchronizedXxx` is superseded by `java.util.concurrent` (Module 15)

## Practical Connections

- **Every Spring Data repository method returning `List<Entity>`** is a direct, everyday application of this entire module — and correctly implementing `equals()`/`hashCode()` on entity classes (Module 07 + Topic 3/4 here) is a genuinely common real-world requirement for using them safely in `Set`s or as `Map` keys.
- **REST API response caching** frequently uses `ConcurrentHashMap` (Topic 8's preview, full depth Module 15) as an in-memory cache — understanding why `HashMap` alone is unsafe there is directly informed by this module.
- **JSON deserialization** (Jackson, Gson) produces `List`/`Map` structures constantly — understanding `HashMap`'s lack of ordering guarantee (Topic 4) explains why `LinkedHashMap` is often specifically configured for JSON libraries when key order matters (e.g., for reproducible output).
- **Sorting business objects for display** (a product catalog by price, a leaderboard by score) is a direct, constant real-world use of `Comparator` chaining (Topic 7).
- **Task queues and job schedulers** in real backend systems are textbook `PriorityQueue`/`Deque` (Topic 5) applications.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| `Collection` vs `Collections` | The root interface (singular) vs. the utility class (plural, Topic 8) — easy to confuse by name. |
| `ArrayList` vs `LinkedList` | Array-backed (O(1) random access, great cache locality) vs. node-chain-backed (O(1) at-ends operations, poor cache locality) — default to `ArrayList`. |
| `HashSet`/`HashMap` vs `TreeSet`/`TreeMap` | Hash-based (O(1) average, no order, depend on `equals`/`hashCode`) vs. tree-based (O(log n), sorted, depend on `compareTo`/`Comparator`). |
| `Iterator.remove()` vs `Collection.remove()` during iteration | The former is safe (coordinates with the active iteration); the latter triggers `ConcurrentModificationException`. |
| `Comparable` vs `Comparator` | One built-in natural order on the class itself vs. unlimited external, pluggable orderings. |
| `Collections.unmodifiableList` vs `List.of` | A restricted view of a still-mutable original (changes show through) vs. a genuinely independent immutable collection. |

## Consolidated Interview Questions (Module 10)

1. Why doesn't `Map` extend `Collection`?
2. What's the internal structural difference between `ArrayList` and `LinkedList`, and why does `ArrayList` usually win in practice despite `LinkedList`'s better theoretical Big-O for some operations?
3. Why does `HashSet` require correct `equals()`/`hashCode()` on its elements?
4. Explain, step by step, what happens internally when you call `hashMap.put(key, value)`.
5. What is HashMap's load factor, and what is Java 8's treeification optimization?
6. Does `TreeSet` use `equals()` for duplicate detection?
7. Why does `Queue` provide both `add`/`offer`, `remove`/`poll`, and `element`/`peek` method pairs?
8. How does `PriorityQueue` determine what `poll()` returns, and is its iteration order sorted?
9. What powers the enhanced for-each loop?
10. Why does modifying a list directly during a for-each loop over it throw `ConcurrentModificationException`?
11. What's the difference between `Comparable` and `Comparator`?
12. Does `Collections.unmodifiableList` create an independent copy?
13. Why are `Collections.synchronizedMap`/`synchronizedList` considered largely superseded?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** Redraw the full Collections Framework hierarchy from memory (Topic 1), then separately write out `HashMap.put`'s complete algorithm step by step (Topic 4).
2. **Hands-on:** Implement a `Book` class with correct `equals()`/`hashCode()` (Module 07) and `Comparable` (by title), then demonstrate storing it correctly in a `HashSet`, sorting a `List<Book>` via `Collections.sort`, and maintaining a `TreeSet<Book>`.
3. **Hands-on:** Build a small word-frequency counter using `HashMap<String, Integer>` and `computeIfAbsent`/`merge` (research these `Map` convenience methods), then print results sorted by frequency descending using a `Comparator`.
4. **Hands-on:** Reproduce and then correctly fix `ConcurrentModificationException` using both `Iterator.remove()` and `removeIf(...)`.
5. **Conceptual:** Explain, referencing Topics 2 and 5, why `ArrayDeque` and `ArrayList` share a common design philosophy advantage over `LinkedList`, connecting this to CPU cache locality specifically.
6. **Synthesis:** Design a small task-scheduling system using `PriorityQueue<Task>` where `Task implements Comparable<Task>` (ordered by urgency), demonstrating tasks being processed in priority order regardless of insertion order.

## What's Next

Module 10 gave you the complete, practical Java Collections Framework — the single most-used part of everyday Java development. Throughout this module, you saw `<E>`, `<K,V>` type parameters used constantly without a full explanation of the mechanism behind them. **Module 11 — Generics** now delivers that full explanation: type erasure, bounded type parameters, wildcards (`? extends`, `? super`), and exactly why `List<Integer>` and `List<String>` are, at the bytecode level, more similar than you might expect.

---

**Previous:** [08 — The `Collections` Utility Class & Concurrent Collections Preview](08-Collections-Utility-Class-And-Concurrent-Collections-Preview.md) · **Module Overview:** [00 — Module Overview](00-Module-Overview.md)

**Type "Continue" to begin Module 11 — Generics.**
