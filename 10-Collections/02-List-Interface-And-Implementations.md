# The `List` Interface & Implementations

## Learning Objectives

- Use the `List` interface and its core implementations correctly
- Understand `ArrayList`'s and `LinkedList`'s genuinely different internal structures, and how that drives their performance differences
- Know precisely when each implementation is the right choice
- Know why `Vector` exists and why it's largely obsolete

## Prerequisites

Module 09 (Arrays — `ArrayList`'s array-based mechanics were previewed there), [01 — Collections Framework Overview](01-Collections-Framework-Overview.md)

## Motivation

`List` is, by a wide margin, the most commonly used Collections Framework interface in everyday Java code. This topic gives you the internal structural understanding needed to make a genuinely informed choice between its two major implementations — not just "ArrayList is usually fine," but a precise understanding of *why*.

## The `List` Interface

```java
List<String> names = new ArrayList<>();
names.add("Aniket");           // adds at the END
names.add(0, "Priya");           // adds AT a specific index, shifting others right
names.get(0);                      // "Priya" -- indexed access, like an array
names.set(0, "Rahul");                // replaces the element AT index 0
names.remove(0);                        // removes BY index
names.remove("Aniket");                   // removes BY VALUE (first matching element)
names.indexOf("Aniket");                    // find an element's position
names.size();                                 // current element count
```

`List<E>` adds **index-based** operations on top of `Collection`'s basics — `get`, `set`, `add(index, ...)`, `indexOf` — none of which make sense for `Set` (no defined positions) but are fundamental to `List`'s ordered nature.

## `ArrayList` — Backed by a Resizable Array (Recap, Extended)

Recall Module 09, Topic 4 in full: `ArrayList` maintains an internal array, growing via `Arrays.copyOf`-style reallocation when capacity is exceeded.

```
 ArrayList's internal structure:
 ┌──────────────────────────────┐
 │  backing array: [A, B, C, _, _]  │   <- contiguous, indexed, just like a raw array
 │  size = 3  (elements actually used) │
 │  capacity = 5  (total array length)  │
 └──────────────────────────────┘
```

**Performance characteristics, precisely:**

| Operation | Complexity | Why |
|---|---|---|
| `get(index)` | **O(1)** — constant time | Direct array index calculation, exactly like raw array access (Module 09, Topic 1) |
| `add(element)` at the end | **Amortized O(1)** | Usually just writes to the next free slot; occasionally triggers a full O(n) reallocation+copy, but this cost is spread ("amortized") across many operations |
| `add(index, element)` in the middle | **O(n)** | Every element after `index` must be shifted one position to the right to make room |
| `remove(index)` | **O(n)** | Every element after `index` must be shifted one position to the left to fill the gap |
| `contains(element)` | **O(n)** | Must linearly scan (no shortcut without additional indexing structures) |

## `LinkedList` — Backed by a Doubly-Linked Chain of Nodes

`LinkedList` uses a **fundamentally different internal structure**: instead of one contiguous array, it maintains a chain of individually-allocated **nodes**, each holding a value and references to its **previous** and **next** node:

```
 LinkedList's internal structure:

 head                                                          tail
  │                                                              │
  ▼                                                              ▼
┌───────┐   next  ┌───────┐   next  ┌───────┐   next  ┌───────┐
│ "A"     │ ──────▶│ "B"     │ ──────▶│ "C"     │ ──────▶│ null    │
│ prev=null│◀────── │ prev     │◀────── │ prev     │◀────── │ prev     │
└───────┘         └───────┘         └───────┘         └───────┘

  Each node is its OWN, separately Heap-allocated object (Module 06, Topic 6's
  static nested Node class, applied EXACTLY as previewed there!) -- NOT contiguous
  in memory, connected purely via REFERENCES (Module 06, Topic 1)
```

**This is precisely the `Node` pattern previewed in Module 06, Topic 6's Nested Classes exercises** — `LinkedList`'s actual JDK implementation uses exactly this kind of private, static nested `Node` class internally.

**Performance characteristics, precisely:**

| Operation | Complexity | Why |
|---|---|---|
| `get(index)` | **O(n)** | No direct index calculation possible — must walk the chain node by node from the nearest end |
| `add`/`remove` at the **beginning or end** | **O(1)** | Just re-point a few `next`/`prev` references — no shifting of other elements needed at all |
| `add`/`remove` in the **middle** (given a reference to the node) | **O(1)** for the actual insertion — but **O(n)** to first *find* that position | Re-pointing references is instant; the cost is entirely in the traversal needed to locate the spot |
| `contains(element)` | **O(n)** | Must walk the chain, same as `ArrayList` |

## The Direct Comparison — And Why It Matters

| | `ArrayList` | `LinkedList` |
|---|---|---|
| Internal structure | One contiguous, resizable array | A chain of individually-linked nodes |
| Random access (`get(i)`) | **O(1)** — fast | **O(n)** — slow |
| Insert/remove at start/end | Amortized O(1) at end; O(n) at start (shifting) | **O(1)** at both ends |
| Insert/remove in the middle | O(n) (shifting) | O(n) to locate + O(1) to splice — still effectively O(n) overall |
| Memory overhead per element | Minimal (just the element itself, in the array) | Higher — each node needs extra memory for `prev`/`next` references |
| Cache locality (real-world CPU performance) | **Excellent** — contiguous memory is CPU-cache-friendly | **Poor** — scattered Heap allocations are not cache-friendly |
| Implements `Deque`? | No | **Yes** (previewed in Topic 5) |

**The practical, honest verdict — a genuinely important, often-surprising real-world fact**: despite `LinkedList`'s theoretical O(1) middle-insertion advantage, **`ArrayList` is the correct default choice for the overwhelming majority of real-world use cases**, including many scenarios that naively "sound like" a `LinkedList` win. Why? Because **locating** the middle insertion point is itself O(n) for *both* structures, and `ArrayList`'s excellent CPU cache locality (contiguous memory access is dramatically faster on real hardware than following scattered pointer chains, due to how CPU caches actually work) frequently makes it **faster in practice**, even for workloads with moderate insertion/removal activity, despite `LinkedList`'s better theoretical Big-O for those specific operations.

**When is `LinkedList` genuinely the better choice?** When you have a **reference directly in hand** to the insertion/removal point (via a `ListIterator`, Topic 6) and are doing many such operations **without** needing random access — a genuinely narrower use case than intuition might suggest. **In practice, reach for `ArrayList` by default; only switch to `LinkedList` if you've measured a specific, genuine performance benefit for your actual access pattern.**

## `Vector` — Legacy, Largely Obsolete

```java
List<String> v = new Vector<>();   // works, but rarely the right choice in modern code
```

`Vector` predates the modern Collections Framework (it's from Java 1.0, later retrofitted to implement `List`) and behaves almost identically to `ArrayList`, with **one** difference: **all of `Vector`'s methods are `synchronized`** — exactly Module 08, Topic 4's `StringBuilder` vs. `StringBuffer` story, replayed identically here. `Vector` pays synchronization overhead on every single operation, whether or not the instance is ever actually shared across threads — which, in the overwhelming majority of real use cases, it isn't.

**Modern guidance: never use `Vector` in new code.** Use `ArrayList` for the common single-threaded case, or a proper concurrent collection from `java.util.concurrent` (Module 15) for genuine multi-threaded needs — `Vector`'s simple, blanket synchronization is an outdated, largely superseded approach to thread safety.

## Real-World Analogy

Think of `ArrayList` like a **numbered row of parking spaces** — you can drive directly to space #47 instantly (O(1) random access), but if you need to insert a new car into the middle of an already-full row, every car after that point has to shuffle over one space (O(n) shifting). Think of `LinkedList` like a **treasure hunt chain of notes**, each note telling you where to find the next one — finding note #47 requires following the chain from the start, one note at a time (O(n)), but once you're physically holding a specific note in your hand, inserting a brand-new note right after it is instant (O(1)) — no other notes need to move at all.

## Advantages

- `ArrayList`: fast random access, excellent cache locality, minimal per-element memory overhead — the right default for most use cases.
- `LinkedList`: genuinely O(1) insertion/removal at either end (making it a solid `Deque` implementation, Topic 5), and O(1) splicing once positioned via an iterator.

## Disadvantages / Trade-offs

- `ArrayList`: O(n) insertion/removal anywhere but the end, due to shifting.
- `LinkedList`: O(n) random access, poor cache locality, and real extra per-node memory overhead — its theoretical middle-insertion advantage rarely materializes in practice due to the O(n) cost of *locating* that position in the first place.

## Best Practices

- Default to `ArrayList` unless you have a specific, measured reason to prefer `LinkedList`.
- Always declare as `List<E>` (the interface), not `ArrayList<E>`/`LinkedList<E>` directly (Module 05, Topic 6; Topic 1 of this module) — this lets you swap implementations later with a one-line change if your access patterns genuinely warrant it.
- Never use `Vector` in new code — prefer `ArrayList` (single-threaded) or `java.util.concurrent` collections (Module 15, multi-threaded).

## Common Mistakes

- Assuming `LinkedList` is "generally faster for insertions" without accounting for the O(n) cost of locating the insertion point in typical (non-iterator-based) usage.
- Using `LinkedList` purely out of habit or a vague sense it's "more efficient," without measuring against the actual access pattern.
- Using `Vector` in new code out of unfamiliarity with its obsolescence.

## Interview Questions

1. **Q: What's the internal structural difference between `ArrayList` and `LinkedList`?**
   A: `ArrayList` is backed by a single, contiguous, resizable array (Module 09's mechanics, directly). `LinkedList` is backed by a chain of individually Heap-allocated nodes, each holding a value and references to its previous/next node — genuinely different memory layouts with genuinely different performance characteristics.

2. **Q: Why is `ArrayList` usually the better default choice, despite `LinkedList`'s theoretically better Big-O for middle insertions?**
   A: Locating the insertion point is itself O(n) for both structures in typical usage (no direct reference already in hand), and `ArrayList`'s contiguous memory layout provides dramatically better real-world CPU cache locality than `LinkedList`'s scattered node allocations — frequently making `ArrayList` faster in practice even for moderate insert/remove workloads, despite `LinkedList`'s better theoretical complexity for the insertion step alone.

3. **Q: Why is `Vector` considered obsolete?**
   A: It synchronizes every single method by default, imposing real overhead on every operation whether or not the instance is ever actually shared across threads — in the vast majority of real use cases, it isn't. `ArrayList` (unsynchronized, faster) is the correct choice for single-threaded use; proper `java.util.concurrent` collections (Module 15) are the correct choice for genuine multi-threaded sharing.

## Summary

- `List<E>` adds index-based operations (`get`, `set`, `add(index, ...)`) on top of `Collection`.
- `ArrayList`: array-backed, O(1) random access, O(n) middle insertion/removal, excellent cache locality — the correct default choice.
- `LinkedList`: node-chain-backed, O(n) random access, O(1) insertion/removal at either end (and via iterator splicing), poorer cache locality — a genuinely narrower, specialized use case than intuition suggests.
- `Vector` is obsolete legacy code — always prefer `ArrayList` or a proper `java.util.concurrent` collection instead.

## Exercises

1. Explain, precisely, why `ArrayList.get(index)` is O(1) while `LinkedList.get(index)` is O(n), referencing each structure's actual internal layout.
2. A colleague proposes using `LinkedList` for a program that frequently inserts elements into arbitrary middle positions by index (`list.add(someIndex, value)`, not via an iterator). Explain why this doesn't actually achieve the O(1) insertion benefit `LinkedList` is known for.
3. Explain why `Vector` is considered obsolete, connecting your answer directly to Module 08, Topic 4's `StringBuilder`/`StringBuffer` comparison.

---

**Previous:** [01 — Collections Framework Overview](01-Collections-Framework-Overview.md) · **Next:** [03 — The `Set` Interface & Implementations](03-Set-Interface-And-Implementations.md)
