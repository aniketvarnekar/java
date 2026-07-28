# For Loops

## Learning Objectives

- Understand the classic three-part `for` loop's exact execution order
- Rewrite any `for` loop as an equivalent `while` loop, and explain why they're interchangeable
- Use the enhanced for-each loop for iterating over arrays (full collection depth in later modules)
- Write nested loops correctly and reason about their total iteration count

## Prerequisites

[03 — While & Do-While Loops](03-while-and-do-while-loops.md)

## Motivation

The `for` loop is the workhorse of "repeat this a known/computable number of times" logic — counting, iterating over indices, processing fixed-size collections. Its three-part header packs a lot of meaning into a small space; this topic makes every part of that header, and its exact execution order, fully explicit.

## Problem Statement

A `while` loop can express any repetition, but for the extremely common pattern of "initialize a counter, repeat while some condition on it holds, update it each iteration," writing all three of those pieces separately (as `while` requires) is repetitive and scatters logically-related code across the loop body. The `for` loop packages exactly this common pattern into one compact, conventional header.

## The Classic Three-Part `for` Loop

```java
for (int i = 0; i < 5; i++) {
    System.out.println("i = " + i);
}
```

The header has exactly three parts, separated by semicolons:

```
for ( INITIALIZATION ; CONDITION ; UPDATE ) { BODY }
       runs ONCE,       checked      runs AFTER
       before anything    BEFORE     each body
       else               each body   execution
                           execution
```

## Exact Execution Order — Traced Step by Step

```java
for (int i = 0; i < 3; i++) {
    System.out.println("i = " + i);
}
```

```
1. INITIALIZATION runs ONCE:              int i = 0;
2. CONDITION checked:                      is i < 3?   (0 < 3, YES)
3. BODY runs:                               prints "i = 0"
4. UPDATE runs:                              i++       (i becomes 1)
5. CONDITION checked again:                   is i < 3?   (1 < 3, YES)
6. BODY runs:                                  prints "i = 1"
7. UPDATE runs:                                 i++       (i becomes 2)
8. CONDITION checked again:                      is i < 3?   (2 < 3, YES)
9. BODY runs:                                     prints "i = 2"
10. UPDATE runs:                                   i++       (i becomes 3)
11. CONDITION checked again:                        is i < 3?   (3 < 3, NO) -- loop exits
```
**Output:**
```
i = 0
i = 1
i = 2
```

**The most commonly missed detail:** the **update** step (`i++`) runs **after** the body, but **before** the next condition check — not "before the body." A surprising number of learners initially assume the update happens right before the body's *next* run starts, which is functionally the same moment, but reasoning about it as "update, then check" (rather than "check, then body, then update") leads to correct predictions in trickier cases (like when `continue` is involved — Topic 5).

## `for` Is Just Syntactic Sugar Over `while`

This is a genuinely important, unifying insight: **every classic `for` loop can be mechanically rewritten as an exactly equivalent `while` loop**, proving `for` adds no new *capability* — only more compact, conventional syntax for an extremely common pattern:

```java
// This for loop:
for (int i = 0; i < 5; i++) {
    System.out.println(i);
}

// ...is EXACTLY equivalent to this while loop:
int i = 0;              // the initialization, moved before the loop
while (i < 5) {          // the condition, unchanged
    System.out.println(i);
    i++;                  // the update, moved to the END of the body
}
```

**One real, meaningful difference:** the `for` loop's initialized variable (`int i` in the header) is scoped **only to the loop itself** (including its condition and update, but not code after the loop) — while the `while` version's `i` remains in scope in the surrounding block after the loop ends too. This is a genuine, practical reason to prefer `for` when a counter variable has no legitimate use outside the loop: it keeps your variable's scope as tight as possible, which is a real, general best practice (minimize scope), not just Java-specific idiom.

## Omitting Parts of the `for` Header

All three parts are technically optional (though the semicolons are always required):

```java
int i = 0;
for (; i < 5; i++) {   // initialization omitted -- 'i' already declared/initialized above
    System.out.println(i);
}
```

```java
for (int i = 0, j = 10; i < 5; i++, j--) {   // MULTIPLE variables/updates, comma-separated
    System.out.println(i + " " + j);
}
```

```java
for (;;) {              // ALL THREE parts omitted -- an explicit infinite loop
    // equivalent to while (true) { ... } -- requires an explicit break/return to ever exit
}
```

## The Enhanced For-Each Loop (Preview)

For iterating over **every element** of an array or collection, without needing an explicit index at all, Java provides a second, distinct `for` syntax — the **enhanced for loop** (sometimes called "for-each"), introduced in Java 5:

```java
int[] scores = {85, 92, 78, 90};

for (int score : scores) {     // "for each int (named score) IN scores"
    System.out.println(score);
}
```

**Why does this exist alongside the classic `for` loop, rather than replacing it?** The classic `for` loop's three-part header is built around index-based iteration — genuinely necessary when you need the **index itself** (to modify the array in place, compare adjacent elements, iterate backward, skip elements, or iterate multiple arrays in lockstep). The for-each loop is built specifically for the extremely common, simpler case: "do something with every element, in order, and I don't care about the index at all" — it's more concise and, critically, **eliminates an entire class of off-by-one indexing bugs** (`i <= length` instead of `i < length`, starting at `1` instead of `0`, etc.) for exactly the cases where an index isn't actually needed.

```java
// Classic for -- needed here because we DO need the index (to know WHICH position to modify)
for (int i = 0; i < scores.length; i++) {
    scores[i] = scores[i] + 5;   // bonus points -- requires knowing the index to write back
}

// For-each -- perfect here, we only need to READ each value, index is irrelevant
int total = 0;
for (int score : scores) {
    total += score;
}
```

> **Full depth on for-each with arrays specifically is in Module 09 (Arrays); full depth on for-each with `List`/`Set`/`Map` and the `Iterable` interface that powers it under the hood is in Module 10 (Collections).** This preview is enough to read and write simple element-iteration code for the rest of this module and the next few.

## Nested Loops

A loop inside another loop's body — the inner loop runs to completion **every single time** the outer loop executes one iteration:

```java
for (int row = 1; row <= 3; row++) {
    for (int col = 1; col <= 3; col++) {
        System.out.print(row * col + " ");
    }
    System.out.println();   // move to a new line after each inner loop fully completes
}
```
**Output:**
```
1 2 3
2 4 6
3 6 9
```

**Total iteration count = outer count × inner count** — here, 3 outer iterations × 3 inner iterations = 9 total inner-body executions. This multiplicative relationship is important for reasoning about performance (a nested loop over two collections of size N and M does N×M work — directly relevant once you reach algorithmic complexity discussions and Collections in later modules) and is a common source of "why is this so slow" surprises when N and M are both large.

## Real-World Analogy

Think of the classic `for` loop's three-part header like a **standardized recipe card format**: "start with X" (initialization), "keep going as long as Y" (condition), "after each step, do Z" (update) — packaged into one compact, conventional line instead of scattered across a longer recipe. The for-each loop is like a recipe instruction that says **"for each egg in the carton, crack it"** — you genuinely don't care whether it's the 1st or 4th egg, you just want every one of them processed, in order, without tracking a position number at all.

## Advantages

- Compact, conventional syntax for the extremely common "counted repetition" pattern, with tightly-scoped loop variables.
- For-each eliminates off-by-one indexing bugs entirely for the common "process every element" case.
- `for` and `while` being mechanically interchangeable means you only need to deeply understand looping *once* — the rest is syntax familiarity.

## Disadvantages / Trade-offs

- For-each cannot give you the current index, cannot modify the underlying array/collection's structure safely while iterating (a real, specific pitfall covered fully in Module 10), and cannot easily iterate backward or skip elements — the classic indexed `for` remains necessary for these cases.
- Nested loops' multiplicative iteration count can create real, easy-to-overlook performance problems as data sizes grow — worth being deliberately conscious of, especially once you're working with real collections (Module 10) instead of small, fixed examples.

## Best Practices

- Default to for-each whenever you don't need the index — it's more concise and eliminates a real bug class.
- Use the classic indexed `for` specifically when you need the index itself, need to iterate backward, or need to modify the structure being iterated.
- Be deliberately aware of nested loops' multiplicative cost, especially as the sizes of the things being iterated grow — a "small" nested loop over two 10-element arrays is 100 iterations; over two 10,000-element collections, it's 100 million.

## Common Mistakes

- Off-by-one errors in classic `for` loop conditions (`i <= array.length` instead of `i < array.length`, which would throw `ArrayIndexOutOfBoundsException` — full depth: Module 09).
- Assuming for-each gives you an index to work with — it does not; use the classic indexed `for` when the index itself is needed.
- Forgetting that a `for` loop's initialized counter variable is scoped only to the loop, and trying to reference it afterward.

## Interview Questions

1. **Q: What is the exact execution order of a classic `for` loop's three parts?**
   A: Initialization runs once, before anything else. Then: check condition → if true, run body → run update → check condition again → repeat, until the condition is false, at which point the loop exits without running the body again.

2. **Q: Is a `for` loop functionally different from a `while` loop, or just different syntax?**
   A: Purely different, more compact syntax for the same underlying capability — any classic `for` loop can be mechanically rewritten as an equivalent `while` loop. One genuine practical difference: a `for` loop's initialized variable is scoped only to the loop itself, while an equivalent `while` loop's counter (declared before it) remains in scope afterward.

3. **Q: When would you choose a classic indexed `for` loop over an enhanced for-each loop?**
   A: When you need the current index itself (to write back to a specific array position, compare adjacent elements, iterate backward, or process multiple collections in lockstep) — for-each deliberately doesn't expose an index at all, which is exactly why it's simpler and safer for the common "process every element" case, but insufficient when the index is genuinely needed.

## Summary

- The classic `for (init; condition; update)` loop executes: init once → [check condition → body → update] repeated until the condition is false.
- Every `for` loop is mechanically equivalent to a `while` loop; `for`'s advantage is compact syntax and tightly-scoped loop variables.
- The enhanced for-each loop (`for (Type var : collection)`) iterates every element without an index, eliminating a real class of off-by-one bugs — but cannot provide the index, iterate backward, or safely modify structure while iterating.
- Nested loops multiply their iteration counts (outer × inner) — a real, important performance consideration as data sizes grow.

## Exercises

1. Trace through, step by step (as done in this topic's worked example), the exact execution order of `for (int i = 10; i > 0; i -= 2) { System.out.println(i); }`, predicting its full printed output.
2. Rewrite this `for` loop as an exactly equivalent `while` loop, and explain the one meaningful scoping difference between your two versions: `for (int i = 0; i < 10; i++) { if (i % 2 == 0) System.out.println(i); }`
3. Given `int[] values = {3, 6, 9, 12};`, write both a classic indexed `for` loop and an enhanced for-each loop that each print every element — then explain a case where only the indexed version would work (e.g., printing each element's position alongside its value).
4. Predict the total number of times the innermost `System.out.print` statement executes in a nested loop with an outer loop running 4 times and an inner loop running 6 times each — explain the multiplicative relationship.

---

**Previous:** [03 — While & Do-While Loops](03-while-and-do-while-loops.md) · **Next:** [05 — Break, Continue & Labeled Statements](05-break-continue-and-labeled-statements.md)
