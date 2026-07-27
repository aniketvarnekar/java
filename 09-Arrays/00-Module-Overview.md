# Module 09 — Arrays

## Module Goal

You've used arrays informally since Module 01's `String[] args`. This module makes arrays fully precise: exactly how they're stored in memory (a direct, concrete application of Module 02's Heap model), multidimensional and jagged arrays, the `java.util.Arrays` utility class, and — critically — the full comparison against `ArrayList` that sets up Module 10's entire Collections Framework.

## Topics Covered in This Module

1. **[Array Fundamentals](01-Array-Fundamentals.md)** — declaration, creation, memory layout, fixed size, default values, and runtime bounds checking.
2. **[Multidimensional Arrays](02-Multidimensional-Arrays.md)** — 2D arrays as "arrays of arrays," and jagged (non-rectangular) arrays.
3. **[The `Arrays` Utility Class](03-Arrays-Utility-Class.md)** — `sort`, `binarySearch`, `fill`, `equals`, `toString`, `copyOf`, and more.
4. **[Array vs. `ArrayList`](04-Array-vs-ArrayList.md)** — the complete comparison, and why Java provides both.
5. **[Module Summary, Interview Questions & Exercises](05-Module-Summary-Exercises.md)** — consolidated recap, quiz, and practice problems.

## Prerequisites

- Module 02 (JVM), especially Topic 3 (Runtime Data Areas — Heap, arrays as objects).
- Module 03 (Java Basics), especially Topic 2 (Primitive Data Types) and Topic 4 (Type Conversion).
- Module 04 (Control Flow), especially Topic 4 (For Loops — for-each preview).

## How to Study This Module

This is a comparatively compact module — arrays are conceptually simple once you know "an array is an object on the Heap holding a fixed-size, contiguous sequence of same-typed slots." Topic 1 establishes that model precisely; Topics 2–3 build on it; Topic 4 is the module's most important takeaway, since it directly motivates why Module 10 (Collections) exists at all.

---

**Previous module:** [08 — Strings](../08-Strings/00-Module-Overview.md) · **Next:** [01 — Array Fundamentals](01-Array-Fundamentals.md)
