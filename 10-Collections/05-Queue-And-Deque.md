# Queue & Deque

## Learning Objectives

- Use `Queue` and `Deque` correctly, including their exception-throwing vs. `null`/`boolean`-returning method pairs
- Use `PriorityQueue` and understand its ordering mechanism
- Understand why `ArrayDeque` is generally preferred over `LinkedList` for stack/queue use cases

## Prerequisites

[02 — The `List` Interface & Implementations](02-List-Interface-And-Implementations.md), [07 — `Comparable` & `Comparator`](07-Comparable-And-Comparator.md) *(PriorityQueue section references this — read that topic first if unfamiliar with ordering)*

## Motivation

Queues model an extremely common real-world pattern: processing items in a specific order (first-in-first-out, or priority-based) — task schedulers, breadth-first graph traversal, print job queues, message processing. This topic covers Java's dedicated interfaces for exactly this pattern.

## The `Queue` Interface — FIFO (First-In-First-Out)

```java
Queue<String> queue = new LinkedList<>();   // LinkedList implements Queue too! (Topic 1's hierarchy)
queue.offer("first");     // adds to the back (like add(), but returns false instead of throwing
                             // if the queue has a fixed capacity and is full)
queue.offer("second");
queue.poll();                // removes AND returns the FRONT element ("first") -- returns null if empty
queue.peek();                  // returns the FRONT element WITHOUT removing it -- returns null if empty
```

**`Queue` deliberately provides TWO parallel sets of methods** for the same core operations — one set that **throws an exception** on failure, one that returns a **special value** (`null`/`false`) instead:

| Operation | Throws on failure | Returns special value on failure |
|---|---|---|
| Insert | `add(e)` | `offer(e)` (returns `false`) |
| Remove | `remove()` | `poll()` (returns `null`) |
| Examine (peek) | `element()` | `peek()` (returns `null`) |

**Why does this deliberate duplication exist?** Different situations call for different failure handling: sometimes an empty/full queue is a genuine, exceptional programming error you want loudly flagged (`add`/`remove`/`element`); sometimes it's a completely normal, expected outcome you want to handle gracefully in your own logic without exception-handling ceremony (`offer`/`poll`/`peek`). **In practice, `offer`/`poll`/`peek` are used far more often**, since "the queue happened to be empty right now" is rarely a genuine error condition in typical queue-processing code.

## The `Deque` Interface — Double-Ended Queue

```java
Deque<String> deque = new ArrayDeque<>();
deque.addFirst("A");        // add to the FRONT
deque.addLast("B");           // add to the BACK
deque.removeFirst();            // remove from the FRONT
deque.removeLast();               // remove from the BACK
deque.peekFirst();                  // look at the FRONT without removing
deque.peekLast();                     // look at the BACK without removing
```

`Deque` (pronounced "deck," short for "double-ended queue") supports efficient insertion/removal at **both** ends. Because of this, `Deque` can naturally serve **two** different roles:

```java
// As a QUEUE (FIFO -- first in, first out):
deque.addLast("A"); deque.addLast("B");     // enqueue at the back
deque.removeFirst();                            // dequeue from the front -- "A" comes out first

// As a STACK (LIFO -- last in, first out):
deque.push("A");   // equivalent to addFirst
deque.push("B");
deque.pop();          // equivalent to removeFirst -- "B" comes out first (the LAST one pushed)
```

**`Deque` is Java's officially recommended replacement for the legacy `java.util.Stack` class** — `Stack` (like `Vector`/`Hashtable`, extends `Vector` in fact) is an old, synchronized-by-default, largely obsolete class. Modern code uses `Deque`'s `push`/`pop`/`peek` methods for stack-like LIFO behavior instead.

## `PriorityQueue` — Ordered by Priority, Not Insertion Order

```java
Queue<Integer> pq = new PriorityQueue<>();
pq.offer(5);
pq.offer(1);
pq.offer(3);
pq.poll();   // 1 -- NOT "5" (insertion order) -- the SMALLEST element comes out first, by default
pq.poll();   // 3
pq.poll();   // 5
```

`PriorityQueue` does **not** preserve insertion order at all — instead, every `poll()` retrieves the element with the **highest priority** (by default, natural ordering — the *smallest* value first, using `Comparable`, Topic 7). Internally, it's implemented using a **binary heap** (a different tree-like structure from `TreeMap`'s Red-Black Tree, optimized specifically for "always efficiently retrieve the minimum/maximum" rather than full sorted-order traversal) — giving O(log n) insertion and removal, with O(1) peek at the highest-priority element.

```java
Queue<String> pq2 = new PriorityQueue<>(Comparator.reverseOrder());   // MAX-first instead of min-first
                                                                          // (Topic 7 covers Comparator fully)
```

**Why is this genuinely useful?** Task schedulers (process the most urgent task first, regardless of when it was submitted), Dijkstra's shortest-path algorithm (always expand the currently-nearest unvisited node next), event simulation (always process the next chronological event) — all fundamentally need "always give me the current highest-priority item," which `PriorityQueue` provides efficiently and directly.

## `ArrayDeque` vs. `LinkedList` for Queue/Deque Use

Both implement `Deque`. Recall Topic 2's `ArrayList`-vs-`LinkedList` cache-locality lesson — the **exact same reasoning applies here**:

| | `ArrayDeque` | `LinkedList` |
|---|---|---|
| Internal structure | A resizable circular array | A chain of individually-linked nodes (Topic 2) |
| Cache locality | **Excellent** | Poor |
| Memory overhead per element | Minimal | Higher (extra node/reference overhead) |
| Can be `null`? | **No** — throws `NullPointerException` on inserting `null` | Yes |
| Implements `List` too? | No — `Deque` only | **Yes** — dual-purpose |

**Modern guidance: prefer `ArrayDeque` over `LinkedList` for pure queue/stack/deque use cases** — exactly mirroring Topic 2's `ArrayList`-over-`LinkedList` guidance, for exactly the same cache-locality reasons. `LinkedList` remains useful specifically when you genuinely need **both** `List` and `Deque` capabilities simultaneously on the same object — a comparatively rare, specific need.

## Real-World Analogy

Think of `Queue` like a **single checkout line at a store** — first person in line is served first (FIFO). Think of `Deque` like a **line where people can also join or leave from the front** (perhaps someone realizes they forgot an item and steps briefly to the front, or the express lane occasionally lets someone cut in) — genuinely double-ended access. Think of `PriorityQueue` like an **emergency room's triage system** — patients aren't seen in arrival order at all; whoever currently has the most urgent condition is always seen next, regardless of when they walked in.

## Advantages

- `Queue`'s dual exception/special-value method pairs let you choose the right failure-handling style for your specific situation.
- `Deque`'s double-ended flexibility lets one interface serve both queue (FIFO) and stack (LIFO) roles.
- `PriorityQueue` provides efficient, automatic "always retrieve the highest-priority item" behavior — a genuinely common, important real-world need.
- `ArrayDeque`'s cache-friendly array-backed structure makes it the modern, efficient default over legacy `Stack`/`LinkedList`-as-a-deque.

## Disadvantages / Trade-offs

- `PriorityQueue` does not support efficient arbitrary-position removal or full sorted-order iteration (iterating it directly does **not** yield sorted output — only repeated `poll()` calls do) — a genuinely common, real point of confusion.
- `ArrayDeque` doesn't permit `null` elements — a real, sometimes-surprising constraint compared to `LinkedList`.

## Best Practices

- Prefer `offer`/`poll`/`peek` over `add`/`remove`/`element` for typical queue processing, where an empty/full queue is a normal, expected condition.
- Use `ArrayDeque` as your default `Deque`/stack/queue implementation; reserve `LinkedList` for cases genuinely needing both `List` and `Deque` behavior together.
- Never use the legacy `java.util.Stack` class in new code — use `Deque`'s `push`/`pop`/`peek` instead.
- Remember `PriorityQueue`'s iteration order is **not** sorted — only sequential `poll()` calls retrieve elements in priority order.

## Common Mistakes

- Assuming `PriorityQueue`'s iterator (or a direct `toString()`/for-each) produces sorted output — it doesn't; only `poll()` in sequence does.
- Using the legacy `Stack` class instead of `Deque`, missing out on better performance and modern API design.
- Inserting `null` into an `ArrayDeque`, triggering an unexpected `NullPointerException`.

## Interview Questions

1. **Q: Why does `Queue` provide both `add`/`offer`, `remove`/`poll`, and `element`/`peek` pairs?**
   A: The first of each pair throws an exception on failure (empty/full queue), appropriate when that's a genuine programming error; the second returns a special value (`null`/`false`) instead, appropriate when an empty/full queue is a normal, expected condition to handle gracefully without exception overhead — the latter is used far more often in practice.

2. **Q: How does `PriorityQueue` determine which element `poll()` returns?**
   A: The element with the highest priority — by default, the smallest according to natural ordering (`Comparable`, Topic 7), or according to a supplied `Comparator`. It's implemented internally with a binary heap, giving O(log n) insertion/removal and O(1) peek at the highest-priority element; it does not preserve insertion order at all.

3. **Q: Why is `ArrayDeque` generally preferred over `LinkedList` for stack/queue use cases?**
   A: Exactly the same reasoning as `ArrayList` over `LinkedList` (Topic 2) — `ArrayDeque`'s array-backed structure provides significantly better real-world CPU cache locality and lower per-element memory overhead than `LinkedList`'s scattered node-chain structure.

## Summary

- `Queue` provides FIFO access with dual exception-throwing/special-value method pairs (`add`/`offer`, `remove`/`poll`, `element`/`peek`).
- `Deque` supports efficient access at both ends, serving as both a queue (FIFO) and a stack (LIFO, via `push`/`pop`), and is the modern replacement for the legacy `Stack` class.
- `PriorityQueue` always retrieves the highest-priority element (not insertion order), using a binary heap internally for O(log n) operations — its iteration order is not sorted.
- Prefer `ArrayDeque` over `LinkedList` for queue/stack/deque use cases, for the same cache-locality reasons established in Topic 2.

## Exercises

1. Implement a simple "undo" stack using `Deque`'s `push`/`pop` methods, and explain why this is preferred over the legacy `Stack` class.
2. Given a stream of task priorities (e.g., `[5, 1, 8, 3, 9, 2]`), use a `PriorityQueue` to process and print them in priority order (smallest first), and explain why simply iterating the `PriorityQueue` directly would NOT produce this same sorted output.
3. Explain why `ArrayDeque` is generally preferred over `LinkedList` for a simple task queue, referencing Topic 2's cache-locality argument directly.

---

**Previous:** [04 — The `Map` Interface & Implementations](04-Map-Interface-And-Implementations.md) · **Next:** [06 — Iterators & Iteration](06-Iterators-And-Iteration.md)
