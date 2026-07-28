# Array Fundamentals

## Learning Objectives

- Declare, create, and use arrays correctly, in every common syntax variation
- Understand precisely how arrays are stored in memory, connecting directly to Module 02
- Understand array default values and fixed-size behavior
- Understand exactly why/when `ArrayIndexOutOfBoundsException` occurs

## Prerequisites

Module 02 Topic 3 (Runtime Data Areas), Module 03 Topic 2 (Primitive Data Types)

## Motivation

Arrays are Java's most fundamental data structure — every other collection type (Module 10) is either built directly on top of an array internally, or exists specifically to overcome an array's limitations (Topic 4 makes this explicit). Understanding arrays precisely, at the memory level, makes everything in Module 10 feel like a natural extension rather than new magic.

## Problem Statement

You frequently need to store a **fixed-size collection of same-typed values** — a list of exam scores, a fixed set of RGB color components, a grid of cells. Declaring a separate variable for each value (`int score1, score2, score3, ...`) doesn't scale and can't be looped over. Java needs a single, indexed structure that holds many values of the same type together.

## Concept: What an Array Actually Is

> An **array** is an **object** (Module 05, Topic 1) that holds a **fixed number** of values, all of the **same type**, stored in **contiguous, indexed slots**, accessed via a zero-based integer index.

**A genuinely important, easy-to-miss fact: arrays are objects.** Even an array of primitives (`int[]`) is itself an object, living on the **Heap** (Module 02, Topic 3) — the array variable you declare is a **reference**, exactly like any other object reference, not the actual data itself.

## Declaration and Creation

```java
int[] scores;                    // DECLARATION only -- 'scores' is null, no array exists yet
scores = new int[5];               // CREATION -- allocates a 5-element int array on the HEAP

int[] grades = new int[5];          // declaration + creation combined
int[] values = {10, 20, 30};          // ARRAY LITERAL -- declares, creates, AND initializes in one step
int[] more = new int[]{1, 2, 3};        // equivalent, explicit form (needed when not a simple declaration)
```

**Alternative (legacy C-style) syntax**, still legal but discouraged in modern Java style:
```java
int scores[];    // legal, but int[] scores is STRONGLY preferred -- the type "int[]" belongs with
                   // the TYPE, not scattered onto the variable name, for readability
```

## Memory Layout — The Precise Picture

```java
int[] scores = new int[5];
scores[0] = 90;
scores[1] = 85;
```

```
 STACK (Module 02, Topic 3)              HEAP (Module 02, Topic 3)
 ┌─────────────────────┐              ┌────────────────────────────┐
 │ scores = 0xA1B2  ────────┼─────────────▶│  int[5] array OBJECT           │
 │  (a REFERENCE)             │              │  length = 5   (see below)       │
 └─────────────────────┘              │  [0]=90 [1]=85 [2]=0 [3]=0 [4]=0│
                                          └────────────────────────────┘
```

**This is exactly Module 06, Topic 1's reference model, applied to arrays**: `scores` on the Stack holds a reference (address); the actual array data — every one of its 5 slots, stored **contiguously** in memory — lives on the Heap as a single object. Passing an array to a method follows the **exact same** pass-by-value-of-the-reference rules from Module 06, Topic 1: a method can mutate the array's contents (visible to the caller, since both share the reference), but reassigning the local parameter to a different array has no effect on the caller.

## Fixed Size — A Defining, Permanent Characteristic

**An array's length is fixed at creation time and can never change.** This is the single most important structural fact about arrays, and directly motivates Topic 4 of this module:

```java
int[] scores = new int[5];
scores.length;             // 5 -- note: length is a FIELD, not a method (no parentheses!)
// there is NO way to "add a 6th element" to this array -- its size is permanently fixed at 5
```

**Why `length` is a field, not a method (`.length`, not `.length()`)**: this is a genuinely common beginner mix-up, since `String.length()` (Module 08) *is* a method. Arrays predate a lot of Java's later API conventions and simply expose `length` as a public field directly — worth memorizing as a specific, arbitrary-feeling exception rather than trying to derive it from a general rule.

## Default Values — Recap and Extension

Recall Module 03, Topic 2: when you create an array with `new`, every slot is automatically initialized to that type's **default value** — exactly the same defaulting rule that applies to instance/static fields (Module 03, Topic 1), now applied uniformly across every slot:

```java
int[] ints = new int[3];        // [0, 0, 0]
boolean[] bools = new boolean[3]; // [false, false, false]
String[] strs = new String[3];     // [null, null, null]  -- String is a reference type!
```

## Bounds Checking — `ArrayIndexOutOfBoundsException`

```java
int[] scores = new int[5];       // valid indices: 0, 1, 2, 3, 4
scores[5];                          // throws ArrayIndexOutOfBoundsException: Index 5 out of bounds for length 5
scores[-1];                           // throws ArrayIndexOutOfBoundsException: Index -1 out of bounds for length 5
```

**Recall Module 01, Topic 3's "Robust" feature**: Java performs **runtime bounds checking** on every array access — attempting to read or write outside the valid `0` to `length - 1` range always throws a catchable exception (Module 12 covers exception handling in full), rather than silently reading/corrupting adjacent memory the way unchecked C array access famously can. This is a genuine, concrete safety guarantee, not just a theoretical one — it directly prevents an entire historical class of memory-corruption bugs and security vulnerabilities common in lower-level languages.

## Arrays of Objects

```java
String[] names = new String[3];   // [null, null, null] -- each slot holds a REFERENCE, defaulted to null
names[0] = "Aniket";                 // now slot 0 references a String object
names[1].length();                     // throws NullPointerException! -- slot 1 is STILL null,
                                          // never assigned
```

**A genuinely common beginner trap**: creating an array of objects (`new Person[5]`) does **not** create 5 `Person` objects — it creates an array of 5 `null` reference slots, each of which must be individually assigned an actual object (`new Person(...)`) before it can be used, or a `NullPointerException` results.

```java
Person[] people = new Person[3];
for (int i = 0; i < people.length; i++) {
    people[i] = new Person("Person " + i);   // must explicitly create AND assign each element
}
```

## Real-World Analogy

Think of an array like a **row of numbered mailboxes, permanently bolted to a wall**, all built at once, all the same size/shape. You can put mail in mailbox #3, check what's in mailbox #0, or empty mailbox #4 — but you can never add mailbox #5 to this specific, already-built row, or remove mailbox #2 from it — the row's total size is fixed the moment it was installed. If the mailboxes are meant to hold "reference slips pointing to a storage locker elsewhere" (an object array) rather than "the actual mail directly" (a primitive array), an empty, unused mailbox holds a blank slip (`null`) — pointing nowhere — until someone explicitly writes a locker number on it.

## Advantages

- Extremely fast, predictable access — since elements are stored contiguously with a fixed size, computing any index's exact memory location is a simple, constant-time calculation (`base address + index × element size`).
- Runtime bounds checking prevents an entire class of memory-corruption bugs common in unchecked languages.
- Minimal memory overhead compared to more flexible collection types (Topic 4) — no extra bookkeeping beyond the elements themselves and a `length`.

## Disadvantages / Trade-offs

- Fixed size is a genuine, real limitation — most real-world "collections" of data need to grow/shrink dynamically, which arrays fundamentally cannot do (directly motivating `ArrayList`, Topic 4).
- Object arrays default every slot to `null`, requiring careful, explicit population before use — a common source of `NullPointerException` for beginners.

## Best Practices

- Always use `Type[] name` (not `Type name[]`) declaration style — the modern, idiomatic convention.
- Remember `.length` is a field (no parentheses) on arrays, while `.length()` is a method on `String` (Module 08) — a small but consistently confusing distinction worth memorizing deliberately.
- Always populate every slot of an object array explicitly before use, to avoid `NullPointerException`.

## Common Mistakes

- Confusing `array.length` (field) with `string.length()` (method) — a genuinely common, easy typo/confusion.
- Assuming `new Person[5]` creates 5 actual `Person` objects — it creates 5 `null` slots.
- Attempting to "resize" an array by any means other than creating an entirely new array (Topic 3 covers `Arrays.copyOf` for this specific purpose).
- Off-by-one errors accessing `array[array.length]` (always invalid — valid indices only go up to `length - 1`).

## Interview Questions

1. **Q: Is an array a primitive or an object in Java?**
   A: An object — even an array of primitives (like `int[]`) is itself an object, living on the Heap, referenced by a variable exactly like any other object reference (Module 02, Topic 3; Module 06, Topic 1).

2. **Q: Can an array's size change after creation?**
   A: No — an array's length is fixed permanently at creation time. "Resizing" always actually means creating a brand-new array (often copying the old contents into it, Topic 3) — never true in-place resizing.

3. **Q: What happens when you create `new Person[5]` — are there 5 `Person` objects?**
   A: No — it creates an array of 5 `null` reference slots. Each slot must be explicitly assigned an actual `Person` object (via `new Person(...)`) before use; accessing an unassigned slot's members throws `NullPointerException`.

4. **Q: What happens when you access an array index outside its valid range?**
   A: The JVM throws `ArrayIndexOutOfBoundsException` at runtime — a deliberate, checked bounds-checking guarantee (Module 01's "Robust" feature) that prevents silent memory corruption, unlike unchecked array access in lower-level languages.

## Summary

- An array is an **object** — a fixed-size, contiguous, indexed sequence of same-typed slots, living on the Heap, accessed via a reference exactly like any other object.
- Array length is **permanently fixed** at creation and exposed via the `.length` **field** (not a method).
- Elements are automatically defaulted (`0`/`false`/`null`, matching Module 03's field-defaulting rules) upon creation.
- Every array access is **runtime bounds-checked**, throwing `ArrayIndexOutOfBoundsException` for invalid indices — a genuine safety guarantee, not just a convenience.

## Exercises

1. Declare and initialize an `int[]` of 5 exam scores using array literal syntax, then write a loop printing each score with its index.
2. Explain, precisely, why `new Person[3]` does not create three usable `Person` objects, and write the code needed to fully populate it.
3. Draw the Stack/Heap memory diagram (in the style used in this topic) for `double[] prices = new double[3];` after assigning `prices[0] = 9.99;`.
4. Predict and explain the exact exception (if any) thrown by each of: `int[] a = new int[3]; a[3] = 1;` and `int[] b = new int[0]; b[0] = 1;`.

---

**Previous:** [00 — Module Overview](00-module-overview.md) · **Next:** [02 — Multidimensional Arrays](02-multidimensional-arrays.md)
