# Module 08 — Strings

## Module Goal

`String` is, by a wide margin, the most-used class in Java — and one of the most misunderstood. This module explains *why* Java made the deliberate, somewhat unusual choice to make `String` immutable, exactly how the String Constant Pool works (extending Module 02's Heap/Metaspace and Module 07's `==`-vs-`.equals()` lessons to Java's most common real-world gotcha), the full practical String API, and when/why to reach for `StringBuilder` instead.

## Topics Covered in This Module

1. **[String Immutability](01-string-immutability.md)** — what immutability means precisely, why Java chose it for `String`, and the security/thread-safety/performance benefits it enables.
2. **[The String Constant Pool](02-string-constant-pool.md)** — how string literals are pooled and reused, `intern()`, and the full, precise explanation of `==` vs. `.equals()` for Strings — the single most common Java gotcha.
3. **[String Methods & API](03-string-methods-and-api.md)** — a comprehensive, practical tour of the `String` class's most-used methods.
4. **[StringBuilder & StringBuffer](04-stringbuilder-and-stringbuffer.md)** — the mutable alternative to `String`, why it exists, and `StringBuilder` vs. `StringBuffer` vs. `String` in full comparison.
5. **[String Formatting & Text Blocks](05-string-formatting-and-text-blocks.md)** — `String.format`/`printf`, and a full return to text blocks (previewed in Module 03).
6. **[Module Summary](06-module-summary.md)** — consolidated recap.

## Prerequisites

- Module 02 (JVM), especially Topic 3 (Runtime Data Areas — Heap, Method Area).
- Module 03 (Java Basics), especially Topic 6 (Wrapper Classes — the `==` vs `.equals()` caching pitfall, which this module extends to Strings).
- Module 07 (Objects), especially Topic 3 (`equals()`/`hashCode()`).

## How to Study This Module

Topic 2 is the heart of this module — it directly extends the `Integer` cache lesson from Module 03, Topic 6 into an even more commonly encountered form. If you understood *why* `Integer a = 200; Integer b = 200; a == b` is `false`, Topic 2 will feel like a natural, expected extension rather than a new, arbitrary rule.