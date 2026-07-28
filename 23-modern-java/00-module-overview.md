# Module 23 — Modern Java

## Module Goal

Throughout this course, features from Java 8 through 21 were taught **in place**, at the exact point they belonged conceptually — `var` alongside Module 03's type system, switch expressions alongside Module 04's control flow, text blocks alongside Module 08's Strings, Virtual Threads and Structured Concurrency as the capstone of Module 15's concurrency model. That was deliberate: a feature makes the most sense next to the problem it solves, not in a separate "new stuff" pile.

This module has two jobs. First, it teaches the remaining major language features that didn't yet have a natural home: **Records** (Java 16), **Sealed Classes** (Java 17), and **Pattern Matching** for `instanceof` and `switch` (Java 16–21), including **Record Patterns** — three features that were deliberately saved for here because they build on each other and are best understood together. Second, it **consolidates** the entire "modern Java" story into one place: a single map of every significant feature from Java 8 to Java 25, what problem each one solved, which version introduced it, and — critically — a direct link back to the module where it was taught in full, so this module works as the connective tissue of the whole course rather than a duplicate of it.

## Topics Covered in This Module

1. **[Records](01-records.md)** — the boilerplate problem records solve, syntax, compact constructors, canonical vs. custom constructors, what the compiler generates for you, and records vs. classes vs. Lombok.
2. **[Sealed Classes](02-sealed-classes.md)** — restricting inheritance on purpose, `sealed`/`permits`/`non-sealed`, and why sealed hierarchies unlock exhaustiveness checking.
3. **[Pattern Matching](03-pattern-matching.md)** — `instanceof` pattern matching, `switch` pattern matching, record patterns (destructuring), guarded patterns (`when`), and exhaustive `switch` over sealed types.
4. **[Modern Java Recap & What's New Through Java 25](04-modern-java-recap-and-whats-new.md)** — a consolidated timeline of every major feature since Java 8, pointers back to where each was taught, and full first-time coverage of features that don't warrant a standalone topic: Sequenced Collections, Stream Gatherers, unnamed variables/patterns, Scoped Values, the Foreign Function & Memory API, and the status of String Templates.
5. **[Module Summary](05-module-summary.md)** — consolidated recap.

## Prerequisites

- Module 05 (OOP), Topic 2 (Encapsulation) and Topic 4 (Polymorphism) — records and sealed classes are refinements of concepts you already know.
- Module 07 (Objects), Topic 2 (`equals()`/`hashCode()`) — records auto-generate exactly what Module 07 taught you to write by hand.
- Module 06 (Classes), Topic 2 (Constructors) — compact constructors are a variant of what you already know.
- Module 04 (Control Flow), Topic 2 (`switch` statement/expression) — pattern matching extends `switch` further.
- Module 15 (Concurrency), Topic 8 (Virtual Threads & Structured Concurrency) — revisited here as part of the consolidated timeline.

## How to Study This Module

Read Topics 1–3 in order — Records, then Sealed Classes, then Pattern Matching — because Topic 3's most powerful form (exhaustive `switch` with record patterns) is literally a combination of Topics 1 and 2. Topic 4 is deliberately structured as a reference map: skim the recap table for features you've already mastered, and read closely only the sections marked "first coverage here." By the end of this module you will have an explicit, dated mental timeline of the entire language's evolution from Java 8 (2014) to Java 25 (2025) — which is one of the most common things interviewers probe for when hiring for modern codebases.