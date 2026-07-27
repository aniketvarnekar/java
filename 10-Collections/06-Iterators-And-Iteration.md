# Iterators & Iteration

## Learning Objectives

- Understand precisely what powers the for-each loop, connecting back to Module 04 and Topic 1's `Iterable` preview
- Use `Iterator` and `ListIterator` directly, including safe removal during iteration
- Understand fail-fast iteration and `ConcurrentModificationException` precisely — why it happens and how to avoid it

## Prerequisites

[01 — Collections Framework Overview](01-Collections-Framework-Overview.md), Module 04 Topic 4 (for-each loop)

## Motivation

You've used for-each loops constantly since Module 04 without knowing exactly what makes them work for any `Collection`. This topic reveals the mechanism — and, critically, explains one of the most commonly encountered runtime surprises in real Java code: `ConcurrentModificationException`, thrown by code that looks completely innocent at first glance.

## The `Iterator` Interface — What Powers For-Each

```java
Iterator<String> it = names.iterator();
while (it.hasNext()) {
    String name = it.next();
    System.out.println(name);
}
```

**This `while` loop is EXACTLY what the enhanced for-each loop compiles down to**, for any `Iterable` (Topic 1):

```java
for (String name : names) {
    System.out.println(name);
}
```

**is compiler-rewritten, behind the scenes, into essentially:**

```java
Iterator<String> it = names.iterator();   // Iterable.iterator() -- the ONE method Iterable requires
while (it.hasNext()) {
    String name = it.next();
    System.out.println(name);
}
```

**This is the complete, precise mechanism behind Topic 1's `Iterable` preview**: any type implementing `Iterable<E>` must provide exactly one method, `iterator()`, returning an `Iterator<E>` — an object with `hasNext()` (is there more?) and `next()` (retrieve the next element, advancing the internal position). The for-each loop is purely **syntactic sugar** the compiler generates for you, over this exact `Iterator` pattern — precisely analogous to how varargs (Module 06, Topic 1) is syntactic sugar over an array parameter, or how `switch` expressions (Module 04, Topic 2) are sugar over more explicit control flow.

## `Iterator.remove()` — The ONLY Safe Way to Remove During Iteration

```java
List<Integer> nums = new ArrayList<>(List.of(1, 2, 3, 4, 5, 6));
Iterator<Integer> it = nums.iterator();
while (it.hasNext()) {
    int n = it.next();
    if (n % 2 == 0) {
        it.remove();       // removes the CURRENT element -- SAFE during iteration
    }
}
System.out.println(nums);   // [1, 3, 5]
```

`Iterator` provides a `remove()` method specifically for safely removing the **most recently returned** element **during** active iteration. This works because the iterator itself tracks the collection's internal state and updates its own bookkeeping consistently as part of the removal — unlike calling the collection's own `remove()` method directly while iterating (shown next), which does **not** coordinate with an in-progress iteration at all.

## `ConcurrentModificationException` — Why It Happens, Precisely

```java
List<Integer> nums = new ArrayList<>(List.of(1, 2, 3, 4, 5, 6));
for (int n : nums) {
    if (n % 2 == 0) {
        nums.remove(Integer.valueOf(n));   // ⚠️ modifying the LIST directly, DURING for-each iteration
    }
}
// throws ConcurrentModificationException !!
```

**Why does this throw, precisely?** Most standard `Collection` implementations (`ArrayList`, `HashMap`, `HashSet`, etc.) maintain an internal **modification count** (`modCount`) — incremented every time the collection is **structurally modified** (an element added or removed, changing its size — note: merely *changing* an existing element's value, like `list.set(i, x)`, is *not* considered a structural modification). Each `Iterator` created from that collection **captures the `modCount` value at creation time**, and **checks it on every single `next()`/`remove()` call** — if the live collection's `modCount` has changed **unexpectedly** (i.e., changed by something **other** than the iterator's own `remove()` call) since the iterator was created, it immediately throws `ConcurrentModificationException`.

```
 Iterator created:  captures modCount = 0
        │
        ▼
 for-each calls it.next() -- checks: is modCount still 0? YES -- proceeds
        │
        ▼
 nums.remove(...) called DIRECTLY on the list -- modCount becomes 1 !!
 (the ITERATOR was never told about this change)
        │
        ▼
 for-each calls it.next() again -- checks: is modCount STILL 0 (what it expects)?
                                     NO, it's now 1 -- MISMATCH!
        │
        ▼
 THROWS ConcurrentModificationException
```

**This is called "fail-fast" behavior**, and it's a **deliberate design choice**, not a bug or a limitation to work around cleverly: rather than silently producing subtly incorrect, hard-to-diagnose iteration results (skipped elements, duplicated elements, or worse) when a collection is structurally modified mid-iteration by something other than the iterator itself, the JVM **immediately and loudly fails**, exactly at the point of inconsistency — directly echoing Module 01, Topic 3's "Robust" design philosophy (fail predictably and loudly, rather than silently corrupting state).

**Important nuance: this detection is "best-effort," not a strict guarantee** — the JDK documentation explicitly states fail-fast behavior should never be relied upon for correctness (e.g., in genuinely concurrent, multi-threaded modification scenarios, Module 15) — it exists specifically to catch **bugs** (accidental single-threaded modification-during-iteration mistakes) reliably in the vast majority of ordinary cases, not to provide a formal thread-safety guarantee.

## The Correct Ways to Modify During Iteration

```java
// OPTION 1: Use Iterator.remove() directly (shown above) -- the standard, idiomatic fix
Iterator<Integer> it = nums.iterator();
while (it.hasNext()) {
    if (it.next() % 2 == 0) it.remove();
}

// OPTION 2: Use the Collection's own removeIf() (Java 8+) -- often the cleanest modern option
nums.removeIf(n -> n % 2 == 0);   // (lambda syntax -- full depth Module 17; usable now by pattern)

// OPTION 3: Iterate over a COPY, modify the original
for (int n : new ArrayList<>(nums)) {   // iterating a SEPARATE copy -- the original is free to modify
    if (n % 2 == 0) nums.remove(Integer.valueOf(n));
}
```

**`removeIf(...)` (Java 8+) is generally the cleanest, most modern, most idiomatic solution** for "remove all elements matching a condition" — it internally handles the iteration/removal coordination correctly and safely, expressing your actual intent ("remove matching elements") far more directly than a manual `Iterator` loop.

## `ListIterator` — Bidirectional Iteration for `List`

```java
List<String> names = new ArrayList<>(List.of("A", "B", "C"));
ListIterator<String> lit = names.listIterator();
while (lit.hasNext()) {
    String name = lit.next();
    lit.set(name.toLowerCase());   // REPLACES the current element -- ListIterator-only capability
}
// names is now ["a", "b", "c"]

while (lit.hasPrevious()) {          // can iterate BACKWARD too
    System.out.println(lit.previous());
}
```

`ListIterator<E>` extends `Iterator<E>`, adding `hasPrevious()`/`previous()` (bidirectional traversal), `set(e)` (replace the current element in place — a genuine mutation `Iterator` alone cannot do), and `add(e)` (insert during iteration). It's available **only** on `List` (via `list.listIterator()`), not on `Set`/`Map`, since it fundamentally relies on `List`'s defined positional/index structure (Topic 2).

## Real-World Analogy

Think of an `Iterator` like a **bookmark moving through a book, one page at a time**, tracking "which page am I currently on" internally. `ConcurrentModificationException` is like **someone else secretly tearing pages out of the book while you're reading with your bookmark** — the moment you turn to the next page and notice the book's actual page count doesn't match what you expected based on your bookmark's own tracking, you immediately, loudly stop reading rather than risk silently skipping content or reading nonsense pages out of order. `Iterator.remove()` is like **the bookmark itself being allowed to remove the page it's currently marking**, updating its own tracking consistently as it does so — a coordinated, safe removal, unlike someone else tearing pages out behind the bookmark's back.

## Advantages

- Fail-fast detection catches a real, common class of iteration bugs immediately and loudly, rather than allowing silent, subtly incorrect behavior.
- `Iterator.remove()`/`removeIf()` provide safe, well-defined ways to modify a collection during iteration.
- The uniform `Iterator` pattern is exactly why for-each works identically across every `Collection` type and any custom `Iterable` you write.

## Disadvantages / Trade-offs

- `ConcurrentModificationException` is a genuinely common, real runtime surprise for developers who don't yet understand its precise cause — this topic exists specifically to close that gap.
- Fail-fast detection is explicitly "best-effort," not a strict guarantee — it should never be relied upon as an actual thread-safety mechanism (Module 15 covers genuine concurrent-safe collections).

## Best Practices

- Never call a `Collection`'s own `add`/`remove` directly during a for-each loop over that same collection.
- Use `Iterator.remove()` for manual iteration-with-removal, or `removeIf(...)` (Java 8+) for the common "remove matching elements" case — prefer the latter when applicable, for its clarity.
- Use `ListIterator` specifically when you need bidirectional traversal or in-place element replacement during iteration.

## Common Mistakes

- Calling `list.remove(...)` directly inside a for-each loop over `list`, triggering `ConcurrentModificationException`.
- Assuming fail-fast behavior is a reliable thread-safety guarantee — it's explicitly best-effort, meant to catch bugs, not to provide formal concurrency safety.
- Forgetting `ListIterator` is only available on `List`, not `Set`/`Map`.

## Interview Questions

1. **Q: What powers the enhanced for-each loop, and does it work for any type?**
   A: The `Iterator` interface (`hasNext()`/`next()`), obtained via `Iterable.iterator()` — the compiler rewrites for-each into an equivalent `while` loop using this exact pattern. It works for any type implementing `Iterable`, including custom classes, not just built-in Collections.

2. **Q: Why does modifying a `List` directly (e.g., calling `list.remove(x)`) during a for-each loop over that same list throw `ConcurrentModificationException`?**
   A: The collection tracks an internal modification count (`modCount`), incremented on structural changes. The active `Iterator` captured this count at creation and checks it on every `next()` call — a direct modification outside the iterator's own `remove()` changes `modCount` without the iterator's knowledge, causing a detected mismatch that immediately, deliberately throws the exception ("fail-fast") rather than risk silent, incorrect iteration.

3. **Q: What is the correct way to remove elements from a collection while iterating over it?**
   A: `Iterator.remove()` (called on the active iterator, removing the most recently returned element) or, more idiomatically in modern Java, `Collection.removeIf(predicate)` (Java 8+), which handles the coordination safely and internally.

## Summary

- `Iterable.iterator()` returns an `Iterator` (`hasNext()`/`next()`), and the for-each loop is compiler-generated syntactic sugar over exactly this pattern.
- Most collections are **fail-fast**: a captured modification count is checked on every iterator operation, throwing `ConcurrentModificationException` immediately if the collection was structurally modified outside the iterator itself — a deliberate "fail loud, not silent" design choice, best-effort rather than a strict guarantee.
- `Iterator.remove()` and `Collection.removeIf(...)` are the correct, safe ways to remove elements during iteration.
- `ListIterator` extends `Iterator` with bidirectional traversal and in-place `set()`/`add()`, available only on `List`.

## Exercises

1. Reproduce `ConcurrentModificationException` yourself: write a for-each loop over an `ArrayList` that calls `list.remove(...)` directly inside the loop, and observe the exception.
2. Fix the code from Exercise 1 two different ways: using `Iterator.remove()`, and using `removeIf(...)`.
3. Explain, step by step, exactly why the fail-fast check succeeds or fails for each `next()` call in a loop, referencing the `modCount` mechanism precisely.
4. Explain why `ListIterator` is only available on `List`, not `Set` or `Map`, referencing what capabilities it adds beyond `Iterator`.

---

**Previous:** [05 — Queue & Deque](05-Queue-And-Deque.md) · **Next:** [07 — `Comparable` & `Comparator`](07-Comparable-And-Comparator.md)
