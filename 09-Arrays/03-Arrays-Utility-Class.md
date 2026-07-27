# The `Arrays` Utility Class

## Learning Objectives

- Use `java.util.Arrays`'s most important static methods correctly
- Understand why `println(array)` doesn't print what beginners expect, and what to use instead
- Understand `copyOf`/`copyOfRange` as the correct way to "resize" an array

## Prerequisites

[01 — Array Fundamentals](01-Array-Fundamentals.md)

## Motivation

Arrays themselves (Topics 1–2) provide only the bare minimum: indexed access and `.length`. Nearly everything else useful you'd want to do with an array — sort it, search it, compare it, copy it, print it meaningfully — lives in the separate `java.util.Arrays` utility class, following a pattern you'll see repeatedly in Java's standard library (a core data type plus a companion utility class of `static` helper methods — recall Module 06, Topic 4's "static methods" discussion).

## Why `println(array)` Doesn't Print What You'd Expect

```java
int[] nums = {1, 2, 3};
System.out.println(nums);   // [I@1b6d3586   -- NOT "[1, 2, 3]"!
```

Recall Module 07, Topic 2: `println` calls `toString()`. **Arrays don't override `toString()`** — they use `Object`'s default, unhelpful implementation. `[I@1b6d3586` is actually a real, decodable format: `[` means "array," `I` means "of type `int`" (a JVM-internal type-descriptor convention), followed by the default hash code. **This is a genuinely common, real beginner surprise** — precisely because it's so natural to assume printing an array would show its contents, the way printing a `List` (Module 10) does.

## `Arrays.toString()` — The Fix

```java
import java.util.Arrays;

int[] nums = {1, 2, 3};
System.out.println(Arrays.toString(nums));   // [1, 2, 3]

int[][] grid = {{1, 2}, {3, 4}};
System.out.println(Arrays.toString(grid));      // [[I@..., [I@...]  -- still wrong for NESTED arrays!
System.out.println(Arrays.deepToString(grid));    // [[1, 2], [3, 4]]  -- CORRECT for multidimensional arrays
```

**Why does `Arrays.toString` still fail for 2D arrays?** Recall Topic 2: a 2D array is an array of array **references**. `Arrays.toString` prints each **element** — for a 2D array, each element is itself an array, and printing an array (again) falls back to the same unhelpful default representation, one level deep. **`Arrays.deepToString`** recursively handles this, correctly printing every nested level — always use `deepToString` for multidimensional arrays, `toString` for single-dimensional ones.

## `Arrays.sort()`

```java
int[] nums = {5, 2, 8, 1, 9};
Arrays.sort(nums);                    // MUTATES 'nums' IN PLACE -- sorted ascending
System.out.println(Arrays.toString(nums));   // [1, 2, 5, 8, 9]

Arrays.sort(nums, 1, 4);                // sorts only the SUBRANGE [1, 4) -- indices 1, 2, 3
```

**`Arrays.sort` mutates the array in place** and returns `void` — a different convention from `String`'s methods (Module 08), which always return a *new* object rather than mutating. This is a direct, practical consequence of arrays being genuinely **mutable** (Topic 1) — unlike `String`, there's no immutability constraint preventing in-place modification here.

## `Arrays.binarySearch()`

```java
int[] sorted = {1, 3, 5, 7, 9};       // MUST already be sorted -- binarySearch REQUIRES this precondition
int index = Arrays.binarySearch(sorted, 7);   // 3 -- the INDEX where 7 was found
int missing = Arrays.binarySearch(sorted, 4);   // a NEGATIVE number -- indicates "not found"
                                                   // (specifically: -(insertion point) - 1)
```

**Why does `binarySearch` require a pre-sorted array?** Binary search works by repeatedly halving the search range, relying entirely on the assumption that "everything to the left is smaller, everything to the right is larger" — an assumption that's only valid if the array is genuinely sorted. Calling it on an unsorted array produces **undefined, meaningless results** (no exception, just silently wrong answers) — a real, important precondition to respect. This is precisely why `binarySearch` achieves logarithmic time complexity, dramatically faster than a linear scan through every element for large arrays — a first, concrete taste of algorithmic complexity reasoning that becomes fully explicit in Module 10.

## `Arrays.fill()`

```java
int[] arr = new int[5];
Arrays.fill(arr, 7);                  // [7, 7, 7, 7, 7] -- fills EVERY slot with the given value
Arrays.fill(arr, 1, 3, 0);              // fills only the SUBRANGE [1, 3) -- [7, 0, 0, 7, 7]
```

## `Arrays.equals()` — Correct Array Comparison

```java
int[] a = {1, 2, 3};
int[] b = {1, 2, 3};

System.out.println(a == b);              // false -- different array OBJECTS (Module 06, Topic 1)
System.out.println(a.equals(b));           // false !! -- arrays don't override equals() either,
                                              // so it falls back to Object's IDENTITY check (Module 07,
                                              // Topic 3) -- exactly the SAME trap as un-overridden equals()!
System.out.println(Arrays.equals(a, b));     // true -- CORRECT, compares CONTENT element by element
```

**This is precisely Module 07, Topic 3's lesson, applied directly to arrays**: since arrays don't override `equals()` (just like `toString()`), `.equals()` on two arrays defaults to identity comparison, exactly like the un-overridden `Object.equals()` you learned about — **never use `.equals()` (or `==`) to compare array contents; always use `Arrays.equals()`** (or `Arrays.deepEquals()` for multidimensional arrays, following the same pattern as `deepToString`).

## `Arrays.copyOf()` and `Arrays.copyOfRange()` — "Resizing" an Array

Recall Topic 1: an array's size can **never** actually change. "Resizing" always means creating a **new** array:

```java
int[] original = {1, 2, 3};
int[] bigger = Arrays.copyOf(original, 5);        // [1, 2, 3, 0, 0] -- new array, extra slots defaulted
int[] smaller = Arrays.copyOf(original, 2);         // [1, 2] -- new, truncated array

int[] range = Arrays.copyOfRange(original, 1, 3);     // [2, 3] -- copies indices [1, 3), just like
                                                          // String.substring's exclusive-end convention
                                                          // (Module 08, Topic 3)!
```

**This is exactly how `ArrayList` (Topic 4, Module 10) implements its own dynamic growth internally** — when an `ArrayList` needs more capacity than its backing array currently has, it allocates a new, larger array (via a mechanism conceptually identical to `Arrays.copyOf`) and copies everything over — you're seeing, right now, the actual mechanism that powers "dynamically resizable" collections under the hood.

## Quick Reference Table

| Method | Purpose |
|---|---|
| `Arrays.toString(arr)` | Readable string for a 1D array |
| `Arrays.deepToString(arr)` | Readable string for a multidimensional array |
| `Arrays.sort(arr)` | Sorts in place, ascending |
| `Arrays.binarySearch(sortedArr, key)` | Fast search — **requires** a pre-sorted array |
| `Arrays.fill(arr, value)` | Fills every slot with a value |
| `Arrays.equals(a, b)` | Correct content-based equality for 1D arrays |
| `Arrays.deepEquals(a, b)` | Correct content-based equality for multidimensional arrays |
| `Arrays.copyOf(arr, newLength)` | New array, truncated/padded to `newLength` |
| `Arrays.copyOfRange(arr, from, to)` | New array from a subrange (end-exclusive) |
| `Arrays.asList(arr)` | A `List` view backed by the array (Topic 4 covers this and its quirks) |

## Real-World Analogy

Think of `java.util.Arrays` like a **toolbox of specialized instruments kept separate from the raw materials themselves** — the array is the raw lumber; `Arrays.sort`, `Arrays.fill`, `Arrays.copyOf` are the saw, the paint sprayer, and the "cut a fresh matching board" tool, each a separate, purpose-built instrument you reach for rather than something built directly into the lumber itself.

## Advantages

- Provides essential, well-tested operations (sorting, searching, copying, comparing) that raw arrays don't offer natively.
- `copyOf`/`copyOfRange` provide the correct, safe mechanism for "resizing" — directly mirroring how `ArrayList` implements its own dynamic growth internally.

## Disadvantages / Trade-offs

- The need for a separate utility class (rather than these being methods directly on the array) is a real, if minor, ergonomic cost compared to `List`'s richer built-in instance methods (Module 10) — a genuine motivation, among several others, for preferring `List` in many application contexts (Topic 4).
- `binarySearch`'s precondition (pre-sorted input) is easy to forget, and violating it produces silently wrong results with no warning or exception.

## Best Practices

- Always use `Arrays.toString`/`deepToString` for debugging array contents — never rely on `println(array)` directly.
- Always use `Arrays.equals`/`deepEquals` (never `==` or `.equals()`) for array content comparison.
- Never call `Arrays.binarySearch` on an unsorted array — sort first, or use a linear search if sorting isn't otherwise needed.

## Common Mistakes

- Printing an array directly and being confused by `[I@...`-style output instead of its contents.
- Using `==` or `.equals()` to compare array contents, expecting element-wise comparison.
- Calling `binarySearch` on unsorted data and trusting its (silently incorrect) result.

## Interview Questions

1. **Q: Why does `System.out.println(intArray)` not print the array's contents?**
   A: Arrays don't override `Object`'s default `toString()` (Module 07, Topic 2), so `println` falls back to the unhelpful default `ClassName@hashCode`-style format. `Arrays.toString(arr)` (or `deepToString` for multidimensional arrays) must be used to print actual contents.

2. **Q: Why shouldn't you use `.equals()` to compare two arrays' contents?**
   A: Arrays don't override `equals()` either, so it falls back to `Object`'s default identity comparison (Module 07, Topic 3) — always `false` for two distinct array objects, even with identical content. `Arrays.equals()` (or `deepEquals()` for multidimensional arrays) must be used instead.

3. **Q: Why does `Arrays.binarySearch` require the array to already be sorted?**
   A: Binary search works by repeatedly halving the search range based on the assumption that elements are ordered — this assumption is only valid for sorted data. Calling it on unsorted data produces undefined, silently incorrect results, with no exception or warning.

## Summary

- `java.util.Arrays` provides essential operations arrays don't have natively: `toString`/`deepToString`, `sort`, `binarySearch` (requires pre-sorted input), `fill`, `equals`/`deepEquals`, and `copyOf`/`copyOfRange`.
- Never print, compare, or trust arrays' default `Object`-inherited `toString()`/`equals()` behavior — always use the `Arrays` utility equivalents.
- `Arrays.copyOf`/`copyOfRange` are the correct way to "resize" or extract a subrange from an array, since true in-place resizing is impossible (Topic 1) — this is conceptually the exact mechanism `ArrayList` uses internally for its own dynamic growth.

## Exercises

1. Given `int[] nums = {5, 3, 8, 1};`, print it correctly using `Arrays.toString`, then sort it and print again to confirm the in-place mutation.
2. Explain, precisely, why `int[] a = {1,2}; int[] b = {1,2}; a.equals(b)` evaluates to `false`, and show the correct way to check content equality.
3. Write code that "grows" a 3-element array to 5 elements (with the two new slots defaulted), using `Arrays.copyOf`, and explain why this isn't literally resizing the original array.

---

**Previous:** [02 — Multidimensional Arrays](02-Multidimensional-Arrays.md) · **Next:** [04 — Array vs. `ArrayList`](04-Array-vs-ArrayList.md)
