# Multidimensional Arrays

## Learning Objectives

- Understand that Java's "2D arrays" are actually arrays of arrays, precisely
- Create and use both rectangular and jagged (non-rectangular) 2D arrays
- Correctly write nested loops to traverse multidimensional arrays

## Prerequisites

[01 — Array Fundamentals](01-array-fundamentals.md), Module 04 Topic 4 (nested loops)

## Motivation

Grids, matrices, game boards, and tabular data all naturally want a two-dimensional structure. Java doesn't have a genuinely "native" 2D array type the way some languages do — understanding that it's actually **arrays of arrays** under the hood explains both its flexibility (jagged arrays, Topic 2's highlight) and a few genuinely surprising behaviors.

## Concept: A "2D Array" Is an Array of Arrays

```java
int[][] grid = new int[3][4];   // 3 rows, 4 columns
```

**What this actually creates**, precisely: an array of length 3, where **each element is itself a separate `int[]` array of length 4**:

```
 grid (outer array, length 3)
 ┌──────────────────────────┐
 │ grid[0] ──▶ [int[4]: 0,0,0,0]  │
 │ grid[1] ──▶ [int[4]: 0,0,0,0]  │
 │ grid[2] ──▶ [int[4]: 0,0,0,0]  │
 └──────────────────────────┘

 Each grid[i] is its OWN, INDEPENDENT array object on the Heap --
 "grid" is really just an array of REFERENCES to other arrays (Module 06, Topic 1's reference model)
```

```java
grid[0][0] = 1;    // row 0, column 0
grid[1][2] = 99;     // row 1, column 2
System.out.println(grid[1][2]);   // 99
```

## Creation and Initialization Syntax

```java
int[][] grid1 = new int[3][4];                    // rectangular, all elements defaulted to 0

int[][] grid2 = {                                    // ARRAY LITERAL syntax -- each inner {} is a ROW
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
};

int[][] grid3 = new int[3][];   // creates the OUTER array (3 slots), but each INNER array is
                                    // still null -- must be created SEPARATELY (see jagged arrays, below)
```

## Traversal with Nested Loops (Recall Module 04, Topic 4)

```java
int[][] grid = {
    {1, 2, 3},
    {4, 5, 6}
};

for (int row = 0; row < grid.length; row++) {           // grid.length = number of ROWS (2)
    for (int col = 0; col < grid[row].length; col++) {     // grid[row].length = number of COLUMNS in THIS row
        System.out.print(grid[row][col] + " ");
    }
    System.out.println();
}
```

**Critical detail: `grid[row].length`, not `grid.length`, gives the column count** — precisely *because* each row is its own independent array object, with its own independent `.length`. This is exactly why the inner loop's bound must be recomputed per row (`grid[row].length`) — a detail that matters enormously once you reach jagged arrays below, where different rows genuinely have different lengths.

**The enhanced for-each loop (Module 04, Topic 4) also works, nested identically:**
```java
for (int[] row : grid) {          // each 'row' IS one of the inner int[] arrays
    for (int value : row) {
        System.out.print(value + " ");
    }
    System.out.println();
}
```

## Jagged Arrays — Rows of Different Lengths

Because each row is genuinely its own **independent** array object, **rows don't need to have the same length at all**:

```java
int[][] jagged = new int[3][];         // outer array of 3 (each element currently null)
jagged[0] = new int[]{1};                 // row 0 has 1 element
jagged[1] = new int[]{1, 2, 3};             // row 1 has 3 elements
jagged[2] = new int[]{1, 2};                  // row 2 has 2 elements
```

```
 jagged
 ┌──────────────────────┐
 │ jagged[0] ──▶ [1]            │
 │ jagged[1] ──▶ [1, 2, 3]        │
 │ jagged[2] ──▶ [1, 2]             │
 └──────────────────────┘

  A genuinely NON-rectangular, "jagged" (staircase-shaped) structure --
  impossible to represent in languages with a truly native, fixed-shape 2D array type
```

**Why does this matter, practically?** Real-world data is frequently naturally jagged — a triangular number pattern, a list of students each with a different number of grades, a sparse matrix representation. Because Java's "2D array" is fundamentally just "array of array references" (Module 06, Topic 1's reference model, once again), this flexibility comes entirely for free, with no special syntax needed beyond what you already know from ordinary array creation.

**Traversal must account for jagged rows correctly:**
```java
for (int row = 0; row < jagged.length; row++) {
    for (int col = 0; col < jagged[row].length; col++) {   // MUST use jagged[row].length, NOT a fixed value
        System.out.print(jagged[row][col] + " ");
    }
    System.out.println();
}
```

## Three (or More) Dimensions

The same "array of arrays (of arrays...)" principle extends indefinitely:

```java
int[][][] cube = new int[2][3][4];   // an array of 2 elements, each a 2D array of [3][4]
cube[0][1][2] = 99;
```

**Practically, arrays beyond 2 dimensions are relatively rare in typical application code** — flagged here for completeness, since the underlying principle (arrays of arrays of arrays...) is identical no matter how many levels deep you go.

## Real-World Analogy

Think of a rectangular 2D array like a **fixed-size apartment building where every single floor has exactly the same number of units** — floor 1 has 4 units, floor 2 has 4 units, every floor identical in shape. A **jagged array** is like a building where **each floor was independently designed with its own, different number of units** — floor 1 has 3 units, floor 2 has 7, floor 3 has 1 — entirely legitimate, since (per Module 06, Topic 1's reference model) each floor is really just its own separate, independently-sized structure, connected to the building only by an elevator shaft (the outer array's references) rather than a physically uniform floor plan repeated throughout.

## Advantages

- Jagged array support comes entirely free from Java's "array of array references" design — no special language feature needed.
- Nested for-each/indexed loops (Module 04) apply directly and naturally to multidimensional traversal, with no new syntax to learn.

## Disadvantages / Trade-offs

- The "arrays of arrays" model means a truly rectangular 2D array still involves one extra layer of reference indirection per row, compared to languages with a genuinely flat, single-block 2D array representation — a minor, generally negligible performance nuance beyond this course's scope to fully quantify.
- Jagged arrays' flexibility means you must always use `array[row].length` rather than assuming a fixed column count — forgetting this is a real, common bug source when adapting rectangular-array code to handle jagged data.

## Best Practices

- Always compute inner-loop bounds from `array[row].length`, never a hardcoded or outer-derived constant, to correctly support both rectangular and jagged arrays uniformly.
- Use array literal syntax (`{{...}, {...}}`) for small, fixed, known-at-compile-time grids; use `new Type[rows][]` plus individual row assignment for genuinely jagged, dynamically-sized data.

## Common Mistakes

- Assuming all rows have the same length and using `grid[0].length` (or a hardcoded number) as the inner loop bound for every row — breaks immediately for jagged arrays.
- Forgetting that `new int[3][]` creates an outer array of `null` rows, requiring each row to be separately created before use (exactly analogous to Topic 1's object-array `null`-slot trap).
- Confusing `grid.length` (number of rows) with the number of columns.

## Interview Questions

1. **Q: Does Java have a "true," native 2D array type?**
   A: No — a Java "2D array" is really an array of references to other (1D) arrays. This is why `int[][]` traversal requires `grid[row].length` for the column count (rather than a single, fixed value), and why jagged (non-rectangular) arrays are possible at all.

2. **Q: What is a jagged array, and why is it possible in Java without any special syntax?**
   A: An array where different rows have different lengths — possible because each row is genuinely its own independent array object, connected to the outer array only via ordinary object references (Module 06, Topic 1), so nothing requires them to share a uniform length.

3. **Q: What does `new int[3][]` create, and what must happen before it's usable?**
   A: An outer array of 3 elements, each currently `null` (no inner array assigned yet) — each row must be individually created (e.g., `arr[0] = new int[5];`) before it can be accessed, or a `NullPointerException` results, exactly analogous to an uninitialized object array slot (Topic 1).

## Summary

- Java's multidimensional arrays are **arrays of arrays** — `int[][]` is an array of `int[]` references, not a genuinely native flat 2D structure.
- This design enables **jagged arrays** (rows of different lengths) with zero special syntax.
- Correct traversal always uses `array[row].length` for the inner bound, never a fixed/hardcoded value, to correctly handle both rectangular and jagged cases.
- The same "array of arrays" principle extends to three or more dimensions, though this is rare in typical application code.

## Exercises

1. Create a jagged `int[][]` representing a triangular number pattern (row `i` has `i + 1` elements, e.g., row 0 has 1 element, row 1 has 2, etc., for 5 rows), and print it using correctly-bounded nested loops.
2. Explain precisely why `grid[row].length` (not `grid[0].length` or `grid.length`) must be used for the inner loop bound in general multidimensional array traversal code.
3. Draw the memory diagram (in this topic's style) for `int[][] grid = {{1, 2}, {3, 4, 5}};`, showing the outer array and each independent inner array object.

---

**Previous:** [01 — Array Fundamentals](01-array-fundamentals.md) · **Next:** [03 — The `Arrays` Utility Class](03-arrays-utility-class.md)
