# Array vs. `ArrayList`

## Learning Objectives

- Fully compare arrays and `ArrayList`
- Understand precisely why `ArrayList` exists, given arrays already exist
- Understand `Arrays.asList()`'s important quirks
- Know when to genuinely still prefer a raw array

## Prerequisites

All prior topics in this module

## Motivation

This topic is the payoff of the entire module — everything you've learned about arrays' fixed size, default `toString`/`equals`, and manual resizing via `Arrays.copyOf` directly explains **why** `ArrayList` (the first class you'll study in depth in Module 10) exists at all, and precisely what problems it solves.

## Problem Statement

Recall Topic 1's defining fact: **an array's size is permanently fixed at creation.** But most real-world "collections" of data — a shopping cart's items, a list of search results, a chat's messages — need to **grow and shrink dynamically** as the program runs. You genuinely don't know the final size in advance. Arrays, by themselves, cannot do this.

## `ArrayList` — A Preview (Full Depth: Module 10)

```java
import java.util.ArrayList;
import java.util.List;

List<String> names = new ArrayList<>();   // starts EMPTY -- no fixed size specified
names.add("Aniket");                         // grows to size 1
names.add("Priya");                            // grows to size 2
names.remove("Aniket");                          // shrinks back to size 1
names.size();                                       // 1 -- a METHOD, not a field (unlike array.length!)
```

`ArrayList` is a class from `java.util` (Module 10's primary subject) that behaves like a genuinely **resizable** array — you can add and remove elements freely, and it manages its own internal capacity automatically.

## How `ArrayList` Actually Achieves "Dynamic" Resizing

**This is the direct payoff of Topic 3's `Arrays.copyOf` discussion**: `ArrayList` doesn't have any magic the language withholds from you — internally, it maintains a genuine, ordinary **array** as its backing storage. When you `add()` an element and the backing array is already full, `ArrayList` performs conceptually exactly what `Arrays.copyOf` does: **allocates a new, larger array, copies every existing element into it, and discards the old one** — entirely transparently, without you ever needing to think about it.

```
 ArrayList internally, conceptually:

 add("A")  -> backing array: ["A", _, _, _]           (capacity 4, size 1)
 add("B")  -> backing array: ["A", "B", _, _]           (capacity 4, size 2)
 add("C")  -> backing array: ["A", "B", "C", _]           (capacity 4, size 3)
 add("D")  -> backing array: ["A", "B", "C", "D"]           (capacity 4, size 4 -- FULL)
 add("E")  -> capacity exceeded! ArrayList allocates a NEW, LARGER array (often ~1.5x growth),
              copies "A","B","C","D" into it, THEN adds "E":
              backing array: ["A", "B", "C", "D", "E", _]     (capacity 6, size 5)
```

**This is precisely why `ArrayList` "just works" for dynamic sizing, and why understanding raw arrays first (this entire module) makes `ArrayList`'s internal behavior — and its performance characteristics — fully transparent rather than mysterious**, once you reach Module 10's full treatment (including exactly when this resizing cost matters for performance).

## The Full Comparison

| Aspect | Array | `ArrayList` |
|---|---|---|
| Size | Fixed at creation, permanent | Dynamic — grows/shrinks automatically |
| Can hold primitives directly? | **Yes** (`int[]`, `double[]`, etc.) | **No** — only objects; primitives require autoboxing (Module 03, Topic 6) into wrapper types (`ArrayList<Integer>`, never `ArrayList<int>`) |
| Length/size access | `.length` (a **field**) | `.size()` (a **method**) |
| Adding/removing elements | Impossible — must create a new array | `.add(...)`, `.remove(...)` — built in, direct |
| Default `toString()`/`equals()` | Unhelpful (`Object` defaults, Topic 3) | Meaningful, correctly overridden (`[Aniket, Priya]`, correct content-based `equals()`) |
| Type safety with generics | N/A (arrays predate generics; have their own type-checking model) | Full generic type safety (Module 11) |
| Performance for fixed-size, known-in-advance data | Slightly better (no resizing overhead, minimal memory footprint) | Slightly more overhead (capacity management, object wrapper boxing for primitives) |
| Multidimensional support | Native (`int[][]`, Topic 2) | Requires `List<List<Integer>>` — more verbose |

## `Arrays.asList()` — A Genuinely Important Quirk

```java
Integer[] arr = {1, 2, 3};
List<Integer> list = Arrays.asList(arr);   // looks like it creates a normal, resizable List...

list.add(4);       // throws UnsupportedOperationException !!
```

**Why does this throw an exception?** `Arrays.asList()` does **not** create a genuine, independent `ArrayList` — it creates a **fixed-size list view directly backed by the original array**. Since the underlying array (Topic 1) can never actually change size, the returned list **structurally cannot support `add`/`remove`** either — attempting either throws `UnsupportedOperationException`. **You CAN, however, use `.set(index, value)`** on it, since that mutates an existing slot's content without changing the size — exactly what the backing array itself permits.

```java
list.set(0, 99);      // LEGAL -- mutates the existing array slot in place
System.out.println(arr[0]);   // 99 !! -- the ORIGINAL array 'arr' is affected too, since the
                                 // List is a VIEW over the SAME backing array, not a copy
```

**This is a genuinely common, real interview/practical gotcha** — `Arrays.asList()` is useful specifically for quickly wrapping an existing array as a `List` for read-only-size purposes (e.g., passing it to a method expecting a `List`), but it is **not** a substitute for `new ArrayList<>(...)` when genuine add/remove flexibility is needed. **To get a truly independent, fully mutable `ArrayList` from an array**, wrap the result: `new ArrayList<>(Arrays.asList(arr))`.

## When to Genuinely Still Prefer a Raw Array

Given `ArrayList`'s convenience, when does a raw array remain the better choice?

- **Performance-critical code with a known, fixed size** — arrays avoid `ArrayList`'s minor overhead (capacity management, autoboxing for primitives) and offer the tightest possible memory footprint, especially significant for very large primitive arrays (Module 03, Topic 2's memory-optimization discussion, applied at scale).
- **Working with primitives at scale** — `int[]` stores primitives directly and compactly (Topic 1); `ArrayList<Integer>` must autobox every single element into a full `Integer` object (Module 03, Topic 6), a real, measurable memory and performance cost for large primitive datasets.
- **Interfacing with APIs that specifically require arrays** — some legacy or low-level APIs (and `main`'s own `String[] args`, Module 01!) are defined in terms of arrays directly.
- **Genuinely fixed-size, small, simple data** — RGB values, a Cartesian coordinate pair, days of the week — where dynamic resizing is never conceptually needed at all.

**In the overwhelming majority of everyday application code** — business logic, collections of domain objects, anything whose size isn't known/fixed in advance — **`ArrayList` (and the broader Collections Framework, Module 10) is the correct, idiomatic default.** This module's deep array knowledge remains valuable precisely because it explains *why* `ArrayList` behaves the way it does and *when* its overhead genuinely matters enough to reach for a raw array instead.

## Real-World Analogy

Think of an array like a **shipping container of a fixed, pre-ordered size** — extremely efficient and compact for exactly the cargo it was built for, but if you need to add more cargo than it holds, you must get an entirely new, larger container and transfer everything over (exactly `Arrays.copyOf`). Think of `ArrayList` like a **modular storage unit rental** — you can add or remove boxes freely as your needs change, and the rental company (the `ArrayList` implementation) transparently handles moving everything to a bigger unit behind the scenes when you outgrow the current one — genuinely convenient, at the cost of a small amount of overhead compared to owning your own perfectly-sized container from the start.

## Advantages of `ArrayList` Over Raw Arrays

- Dynamic resizing eliminates the entire "I don't know the final size in advance" problem arrays cannot solve.
- Rich, built-in methods (`add`, `remove`, `contains`, and far more — Module 10) vs. arrays' bare-minimum indexed access.
- Correct, meaningful `toString()`/`equals()` out of the box (unlike arrays' unhelpful `Object` defaults, Topic 3).
- Full generic type safety (Module 11).

## Disadvantages / Trade-offs

- Cannot hold primitives directly — autoboxing overhead (Module 03, Topic 6) for large primitive datasets.
- Slightly more memory and CPU overhead than a raw, perfectly-sized array, due to capacity management.
- `Arrays.asList()`'s fixed-size-view quirk is a genuine, real trap if not understood precisely.

## Best Practices

- Default to `ArrayList`/`List` (Module 10) for general application-level collections whose size isn't fixed/known in advance.
- Use raw arrays specifically for large primitive datasets, performance-critical fixed-size data, or multidimensional numeric data (Topic 2).
- Never assume `Arrays.asList()` returns a fully mutable list — wrap it in `new ArrayList<>(...)` if add/remove is needed.

## Common Mistakes

- Calling `.add()` on a list returned directly from `Arrays.asList()`, triggering `UnsupportedOperationException`.
- Using `ArrayList<Integer>` (with heavy autoboxing overhead) for very large, performance-critical primitive numeric datasets where a plain `int[]` would be significantly more efficient.
- Assuming arrays and `ArrayList` are interchangeable everywhere — they have genuinely different capabilities and appropriate use cases.

## Interview Questions

1. **Q: Why can't arrays grow or shrink, while `ArrayList` can?**
   A: An array's size is fixed permanently at creation (JVM-level constraint, Topic 1). `ArrayList` isn't a fundamentally different kind of storage — internally, it maintains its own ordinary backing array, and when more capacity is needed, it allocates a new, larger array and copies existing elements into it (conceptually identical to `Arrays.copyOf`), entirely transparently to the caller.

2. **Q: Can an `ArrayList` hold primitive `int` values directly?**
   A: No — generic collections require object type parameters (Module 03, Topic 6; Module 05, Topic 6's generics preview). `ArrayList<Integer>` autoboxes each `int` into an `Integer` wrapper object, unlike a raw `int[]`, which stores primitives directly and compactly.

3. **Q: What does `Arrays.asList(someArray)` actually return, and what's the common mistake developers make with it?**
   A: A fixed-size `List` **view** directly backed by the original array — not an independent, fully resizable `ArrayList`. Calling `.add()` or `.remove()` on it throws `UnsupportedOperationException`, since the underlying array can never change size; `.set(...)` works, but mutates the original array too, since it's a view, not a copy.

## Summary

- Arrays are fixed-size, low-level, primitive-capable, but come with minimal built-in functionality and unhelpful default `toString`/`equals`.
- `ArrayList` provides dynamic resizing, rich built-in methods, and correct default behavior — internally implemented using an ordinary backing array plus `Arrays.copyOf`-style reallocation, exactly the mechanism you now fully understand.
- `Arrays.asList()` returns a fixed-size view backed by the original array — not a substitute for `new ArrayList<>(...)` when true resizability is needed.
- Default to `ArrayList`/`List` for general application code; reach for raw arrays specifically for performance-critical, large-scale primitive data or genuinely fixed-size use cases.

## Module-Wide Quick Revision

- Arrays are objects on the Heap — fixed size, contiguous, zero-based indexed, runtime bounds-checked (Topic 1).
- 2D+ arrays are arrays of array references, enabling jagged arrays with zero special syntax; always use `array[row].length` for inner bounds (Topic 2).
- `java.util.Arrays` provides `toString`/`deepToString`, `sort`, `binarySearch` (requires sorted input), `fill`, `equals`/`deepEquals`, `copyOf`/`copyOfRange` — arrays don't override `toString`/`equals` themselves (Topic 3).
- `ArrayList` provides genuine dynamic resizing, internally built on the exact same `Arrays.copyOf`-style reallocation mechanism; `Arrays.asList()` returns a fixed-size view, not a true `ArrayList` (this topic).

## Common Pitfalls (Module-Wide)

- Confusing `array.length` (field) with `list.size()` (method).
- Printing/comparing arrays directly instead of using `Arrays.toString`/`Arrays.equals`.
- Assuming `Arrays.asList()` returns a fully mutable list.
- Forgetting jagged array rows can have different lengths, and using a fixed inner-loop bound.