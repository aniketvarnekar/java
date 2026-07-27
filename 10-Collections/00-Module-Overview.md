# Module 10 — Collections

## Module Goal

This is one of the largest, most practically important modules in the entire course. The **Java Collections Framework** is what real Java applications actually use, constantly, to store and manipulate groups of data — you will use `List`, `Set`, and `Map` in nearly every Java program you ever write from this point forward. This module builds the Collections Framework directly on top of everything you already know: arrays (Module 09), `equals()`/`hashCode()` (Module 07), and generics preview (Module 05) — nothing here is new magic, it's all a careful assembly of concepts you already have.

## Topics Covered in This Module

1. **[Collections Framework Overview](01-Collections-Framework-Overview.md)** — the complete interface hierarchy, `Collection` vs. `Map`, and `Iterable`.
2. **[The `List` Interface & Implementations](02-List-Interface-And-Implementations.md)** — `ArrayList`, `LinkedList`, `Vector`, and when each is genuinely the right choice.
3. **[The `Set` Interface & Implementations](03-Set-Interface-And-Implementations.md)** — `HashSet`, `LinkedHashSet`, `TreeSet`, and their total dependence on Module 07's `equals()`/`hashCode()` contract.
4. **[The `Map` Interface & Implementations](04-Map-Interface-And-Implementations.md)** — `HashMap` internals (buckets, collision handling, Java 8's treeification), `LinkedHashMap`, `TreeMap`, and the deprecated `Hashtable`.
5. **[Queue & Deque](05-Queue-And-Deque.md)** — `Queue`, `Deque`, `PriorityQueue`, `ArrayDeque`.
6. **[Iterators & Iteration](06-Iterators-And-Iteration.md)** — `Iterator`, `ListIterator`, fail-fast vs. fail-safe, `ConcurrentModificationException`, and what actually powers the for-each loop.
7. **[`Comparable` & `Comparator`](07-Comparable-And-Comparator.md)** — natural ordering vs. custom ordering, and sorting collections correctly.
8. **[The `Collections` Utility Class & Concurrent Collections Preview](08-Collections-Utility-Class-And-Concurrent-Collections-Preview.md)** — `Collections.sort`/`unmodifiableList`/`synchronizedList`, and a preview of `ConcurrentHashMap` (full depth Module 15).
9. **[Module Summary, Interview Questions & Exercises](09-Module-Summary-Exercises.md)** — consolidated recap, quiz, and practice problems.

## Prerequisites

- Module 09 (Arrays) — `ArrayList`'s internal mechanics were already previewed there.
- Module 07 (Objects), especially Topic 3 (`equals()`/`hashCode()`) — `HashSet`/`HashMap` correctness depends entirely on this.
- Module 05 (OOP), especially Topic 6 (Interfaces) — the entire Collections Framework is interface-driven ("program to an interface").

## How to Study This Module

This module is large because Collections genuinely is the single most-used part of everyday Java. Topics 2–5 follow a consistent rhythm: interface first, then implementations compared. Topic 4 (`Map`) deserves particular attention — `HashMap`'s internal bucket/treeification mechanics are one of the most commonly asked deep-dive interview topics in all of Java. Topic 6 resolves a subtle but important runtime behavior (`ConcurrentModificationException`) that trips up even experienced developers.

---

**Previous module:** [09 — Arrays](../09-Arrays/00-Module-Overview.md) · **Next:** [01 — Collections Framework Overview](01-Collections-Framework-Overview.md)
