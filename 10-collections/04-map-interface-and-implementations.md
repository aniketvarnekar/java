# The `Map` Interface & Implementations

## Learning Objectives

- Use the `Map` interface and its core implementations correctly
- Understand `HashMap`'s internal bucket structure precisely, including collision handling
- Understand Java 8's treeification optimization and why it was added
- Choose between `HashMap`, `LinkedHashMap`, and `TreeMap` correctly

## Prerequisites

Module 07 Topic 3 (`equals()`/`hashCode()`), [03 — The `Set` Interface & Implementations](03-set-interface-and-implementations.md)

## Motivation

`HashMap` is arguably the single most important data structure in all of practical Java programming — and its internal bucket mechanism is one of the most commonly asked "explain how this works internally" interview questions in the entire language. This topic gives you that mechanism precisely, building directly on Module 07's `equals()`/`hashCode()` contract and Topic 3's `HashSet`-is-a-`HashMap-wrapper` revelation.

## The `Map` Interface

```java
Map<String, Integer> ages = new HashMap<>();
ages.put("Aniket", 30);           // insert/update a key-value pair
ages.get("Aniket");                  // 30 -- retrieve by key
ages.get("Unknown");                   // null -- NOT an exception, if the key isn't present
ages.getOrDefault("Unknown", 0);         // 0 -- a safer alternative, avoiding null entirely
ages.containsKey("Aniket");                // true
ages.containsValue(30);                      // true
ages.remove("Aniket");                         // removes the entry
ages.size();                                     // number of key-value pairs

for (Map.Entry<String, Integer> entry : ages.entrySet()) {   // the RECOMMENDED iteration pattern
    System.out.println(entry.getKey() + " = " + entry.getValue());
}
```

**Why `entrySet()` is the recommended iteration pattern (recall Topic 1's preview)**: iterating `keySet()` and calling `.get(key)` separately for each key performs **two** hash lookups per entry (one implicit in the iteration, one explicit in `get`); `entrySet()` retrieves each key **and** its value together, in a single pass — a genuine, real performance difference for large maps.

## `HashMap`'s Internal Structure — The Full Mechanism

This is the heart of this topic. Understanding it precisely is what separates "I use HashMap" from "I understand HashMap."

### The Bucket Array

Internally, `HashMap` maintains an **array of "buckets"** (historically implemented as linked lists per bucket; see the Java 8 treeification note below). The array's size is always a power of 2 (a specific, deliberate implementation choice enabling a fast bitwise operation instead of a slower modulo, for computing bucket indices).

```
 HashMap's internal bucket array (default initial capacity: 16):

 index:   0     1     2     3     4    ...   15
        ┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐       ┌───┐
        │ ∅  │ │ ∅  │ │ ●  │ │ ∅  │ │ ●  │  ...  │ ∅  │
        └───┘ └───┘ └───┘ └───┘ └───┘       └───┘
                       │           │
                       ▼           ▼
                 ["cat"→5]    ["dog"→3] → ["fog"→7]   <- COLLISION: two different keys
                                                            landed in the SAME bucket!
```

### The `put(key, value)` Algorithm, Step by Step

1. Compute `key.hashCode()` (Module 07, Topic 3).
2. Apply an internal **"hash spreading" function** (HotSpot XORs the hash code's upper and lower 16 bits together) — a deliberate step to reduce the chance of collisions for hash codes that only differ in their high-order bits, improving distribution across buckets.
3. Compute the **bucket index**: `spreadHash & (bucketArrayLength - 1)` — a fast bitwise AND, equivalent to `spreadHash % bucketArrayLength` since the array length is always a power of 2, but significantly faster at the CPU level.
4. **If the bucket is empty**, place the new entry there directly.
5. **If the bucket already has entries (a "collision")**, walk the bucket's chain, calling `.equals()` on each existing key to check for a match:
   - **If a match is found** (`equals()` returns `true`), **update** that entry's value (a `put` with an existing key overwrites, it doesn't duplicate).
   - **If no match is found** after checking every entry in the bucket, **append** the new entry to the chain.

### Why Both `hashCode()` AND `equals()` Are Used — Precisely, Now Fully Concrete

This is Module 07, Topic 3's contract, now shown as the **literal, step-by-step algorithm** that depends on it:

- `hashCode()` answers **"which bucket?"** — a fast, O(1) computation narrowing the search from "every entry in the map" down to "just this one bucket's entries."
- `equals()` answers **"which specific entry, within that bucket?"** — since multiple different keys can legitimately land in the same bucket (a **collision** — not a bug, an expected, normal occurrence for any hash-based structure), `equals()` is what distinguishes between them.

**This is exactly why the contract (equal objects → equal hash codes) is non-negotiable**: if two "equal" keys produced different hash codes, `put`/`get` could send them to **different** buckets, meaning `get(key)` might search entirely the wrong bucket and fail to find a value that a logically-identical `put(key, ...)` call had stored elsewhere — precisely the "lost in the hash collection" bug demonstrated concretely in Module 07, Topic 3, now understood as a direct, mechanical consequence of this exact algorithm.

### Load Factor and Resizing

`HashMap` tracks a **load factor** (default `0.75`) — when the number of stored entries exceeds `capacity × loadFactor` (e.g., `16 × 0.75 = 12` entries, by default), the entire bucket array is **resized** (typically doubled) and **every existing entry is rehashed** into the new, larger array.

**Why `0.75`, specifically — a genuine engineering trade-off?** A **lower** load factor (resizing sooner, keeping buckets sparser) reduces collision chains, keeping lookups fast, but wastes more memory on unused bucket slots. A **higher** load factor (resizing later, allowing buckets to fill more) saves memory, but increases collision chain lengths, slowing down lookups. `0.75` is Java's chosen, well-tested balance point between these two competing costs — you can override it via `HashMap`'s constructor if your specific use case has unusual space/time trade-off needs, though this is rarely necessary in practice.

### Java 8's Treeification — A Real, Modern Optimization

**Before Java 8**: a bucket's collision chain was **always** a simple linked list — meaning, in a rare **worst-case** scenario (many keys with poor hash distribution or deliberately crafted hash collisions, a real historical **denial-of-service attack vector** against naive hash-based structures), a single bucket's chain could grow very long, degrading that bucket's lookup from O(1) to **O(n)**.

**Since Java 8**: if a single bucket's chain grows beyond a threshold (**8** entries, by default) **and** the overall table is sufficiently large, that bucket's linked list is dynamically **converted into a balanced Red-Black Tree** (the same underlying structure `TreeSet`/`TreeMap` use, Topic 3) — degrading that bucket's worst-case lookup from O(n) down to **O(log n)** instead. (If the bucket later shrinks back below a lower threshold, it converts back to a simple linked list — the resizing/rehashing process handles this naturally.)

```
 Before Java 8 (bucket with many collisions):     Since Java 8 (treeification kicks in):

 bucket[3] → A → B → C → D → E → F → G → H → I      bucket[3] → [Red-Black Tree: A,B,C,D,E,F,G,H,I]
             (a LONG chain -- O(n) worst case)         (a BALANCED TREE -- O(log n) worst case)
```

**Why does this require `Comparable`/consistent ordering among the colliding keys?** Building a tree requires being able to order elements — HashMap's treeification internally uses each key's hash code as a tie-breaking comparison basis (falling back to identity-based tie-breaking if hash codes are equal too), so this optimization works even for keys that don't themselves implement `Comparable` — a genuinely clever, pragmatic engineering detail.

## `LinkedHashMap` and `TreeMap` — Exactly Parallel to Topic 3's `Set` Story

```java
Map<String, Integer> linked = new LinkedHashMap<>();    // preserves INSERTION order, HashMap performance
Map<String, Integer> sorted = new TreeMap<>();               // ALWAYS sorted by key, O(log n), Red-Black Tree
```

This is **precisely** Topic 3's `HashSet`/`LinkedHashSet`/`TreeSet` story, applied to `Map` — unsurprising, since (Topic 3) `HashSet` and its relatives are literally implemented as thin wrappers around `HashMap`/`LinkedHashMap`/`TreeMap` internally. Every performance characteristic, every ordering guarantee, and every requirement (`TreeMap` needs `Comparable`/`Comparator` **keys**, using `compareTo` rather than `equals()`/`hashCode()` for its internal structure) carries over identically.

## `Hashtable` — Legacy, Obsolete (Exactly Like `Vector`, Topic 2)

```java
Map<String, Integer> old = new Hashtable<>();   // legacy, Java 1.0 -- avoid in new code
```

`Hashtable` predates the modern Collections Framework and, like `Vector` (Topic 2), synchronizes every single method by default — the exact same obsolete pattern. **Modern guidance: never use `Hashtable`** — use `HashMap` for single-threaded code, or `ConcurrentHashMap` (previewed in Topic 8, full depth Module 15) for genuine concurrent access. A further, real distinction: `Hashtable` doesn't permit `null` keys or values at all, while `HashMap` permits one `null` key and any number of `null` values.

## Full Comparison Table

| | `HashMap` | `LinkedHashMap` | `TreeMap` | `Hashtable` |
|---|---|---|---|---|
| Ordering | None guaranteed | Insertion order | Sorted by key | None guaranteed |
| `get`/`put` complexity | O(1) average (O(log n) worst-case since Java 8's treeification) | Same as `HashMap` | O(log n) | O(1) average |
| Thread-safe? | No | No | No | Yes (fully synchronized — obsolete approach) |
| Allows `null` key? | One | One | No (throws NPE on comparison) | No |
| Requires | Correct `equals()`/`hashCode()` on keys | Correct `equals()`/`hashCode()` on keys | `Comparable`/`Comparator` on keys | Correct `equals()`/`hashCode()` on keys |
| Modern status | **Standard default** | Good, specialized choice | Good, specialized choice | **Obsolete — avoid** |

## Real-World Analogy

Think of `HashMap`'s bucket array like a **hotel with numbered floors**, where a guest's room floor is determined by a quick calculation from their name (`hashCode()` + bucket-index math). If two guests happen to be assigned the **same floor** (a collision), the front desk keeps a **list of everyone on that floor**, checking each one's exact identity (`equals()`) to find the right specific guest when someone asks for them by name. Java 8's treeification is like the hotel noticing one particular floor has become **absurdly overcrowded** (many collisions) and, just for that one floor, switching from "a messy list you have to scan one by one" to "an organized directory sorted for fast lookup" — while every other, normally-populated floor keeps using the simple list, since it's perfectly efficient for a small number of guests.

## Advantages

- `HashMap`'s bucket-based design achieves O(1) average-case lookup/insertion — the foundation of countless real-world performance-critical systems.
- Java 8's treeification provides a real, automatic safety net against worst-case collision scenarios, without requiring any code changes from you.
- The `HashMap`/`LinkedHashMap`/`TreeMap` trio mirrors `Set`'s three-way choice exactly, minimizing new concepts to learn.

## Disadvantages / Trade-offs

- `HashMap` provides zero ordering guarantees — a real design constraint when order matters.
- Correctness depends entirely on well-implemented `equals()`/`hashCode()` (Module 07, Topic 3) on key types — getting this wrong causes the exact "lost entry" bug now fully explained mechanically.
- `TreeMap`'s O(log n) is genuinely slower than `HashMap`'s O(1) average — a real cost, only worth paying when sorted order is actually needed.

## Best Practices

- Default to `HashMap` for general key-value storage; use `LinkedHashMap` for insertion-order iteration; use `TreeMap` for sorted-by-key iteration or range queries.
- Never use mutable objects as `HashMap`/`HashSet` keys unless you're certain their hash-relevant fields will never change after insertion (Module 07, Topic 3's mutable-field warning, directly relevant here too).
- Use `getOrDefault(key, defaultValue)` instead of manually checking `containsKey` then calling `get`, when a sensible default exists — cleaner and avoids a redundant lookup.
- Iterate with `entrySet()`, not `keySet()` + `get()`, for better performance on large maps.

## Common Mistakes

- Using a mutable, poorly-implemented `equals()`/`hashCode()` key type in a `HashMap`, causing silent, hard-to-diagnose "missing entry" bugs.
- Assuming `HashMap` iteration order is meaningful — it isn't; use `LinkedHashMap` if order matters.
- Iterating `keySet()` and calling `get()` for each key instead of using `entrySet()`, incurring unnecessary redundant lookups.

## Interview Questions

1. **Q: Explain, step by step, what happens internally when you call `hashMap.put(key, value)`.**
   A: Compute `key.hashCode()`, apply an internal hash-spreading function, compute the bucket index via a bitwise AND against the (power-of-2) bucket array length, then either place the entry directly (empty bucket) or walk the bucket's existing chain calling `equals()` on each entry to check for an existing match — overwriting the value if found, or appending a new entry if not.

2. **Q: Why does `HashMap` need both `hashCode()` and `equals()`?**
   A: `hashCode()` quickly narrows the search to one specific bucket (O(1)); `equals()` then distinguishes between the (possibly multiple) different keys that legitimately collided into that same bucket. Both are necessary because hash codes alone can't guarantee uniqueness — collisions are a normal, expected occurrence.

3. **Q: What is HashMap's load factor, and why is the default 0.75?**
   A: The threshold (as a fraction of capacity) at which the bucket array is resized and rehashed — `0.75` balances memory usage against collision-chain length/lookup speed; lower values reduce collisions at the cost of more memory, higher values save memory at the cost of longer chains.

4. **Q: What is HashMap's Java 8 treeification optimization?**
   A: When a single bucket's collision chain exceeds a threshold (default 8 entries) in a sufficiently large table, that bucket's linked list is converted into a balanced Red-Black Tree, improving that bucket's worst-case lookup from O(n) to O(log n) — a defense against pathological collision scenarios, without requiring the colliding keys to implement `Comparable` themselves.

5. **Q: Why is `Hashtable` considered obsolete?**
   A: Like `Vector`, it synchronizes every method by default, imposing unconditional overhead even when never actually shared across threads. `HashMap` (single-threaded) or `ConcurrentHashMap` (Module 15, genuine concurrent access) are the correct modern choices.

## Summary

- `HashMap` uses a **bucket array**: `hashCode()` + hash-spreading + bitwise-AND determines the bucket; `equals()` distinguishes between colliding keys within that bucket — the direct, mechanical realization of Module 07's `equals`/`hashCode` contract.
- **Load factor** (default 0.75) governs when the bucket array resizes/rehashes, balancing memory against collision-chain length.
- **Java 8's treeification** converts long collision chains (8+ entries) into balanced Red-Black Trees, capping worst-case lookup at O(log n).
- `LinkedHashMap` adds insertion-order iteration; `TreeMap` maintains sorted-by-key order (via `compareTo`/`Comparator`, not `equals()`/`hashCode()`) at O(log n).
- `Hashtable` is obsolete, exactly like `Vector` — use `HashMap` or `ConcurrentHashMap` (Module 15) instead.

## Exercises

1. From memory, walk through the complete `put(key, value)` algorithm step by step, including what happens on a hash collision.
2. Explain, precisely, why a `HashMap` with a poorly-implemented key class's `equals()`/`hashCode()` can silently "lose" entries — connect your answer to Module 07, Topic 3's `HashSet.contains()` failure example.
3. Explain why Java 8's treeification optimization was added, and what specific worst-case scenario it defends against.
4. Given `Map<String, List<String>> groups = new HashMap<>();`, write code that correctly adds a value to the list for a given key, creating a new empty list first if the key isn't already present (hint: research `computeIfAbsent`, a modern `Map` convenience method).

---

**Previous:** [03 — The `Set` Interface & Implementations](03-set-interface-and-implementations.md) · **Next:** [05 — Queue & Deque](05-queue-and-deque.md)
