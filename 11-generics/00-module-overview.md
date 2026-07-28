# Module 11 — Generics

## Module Goal

Throughout Modules 05–10, you've written `List<String>`, `Map<String, Integer>`, `Comparable<Employee>` constantly, without a dedicated explanation of the `<...>` syntax itself. This module delivers that explanation in full: why generics were introduced (Java 5, 2004 — Module 01, Topic 2's timeline), how to write your own generic classes and methods, bounded type parameters, wildcards, and — the deepest, most interview-relevant topic — **type erasure**, the mechanism that makes generics work at compile time while remaining invisible at runtime.

## Topics Covered in This Module

1. **[Why Generics — Introduction](01-why-generics-introduction.md)** — the pre-generics problem (raw types, unsafe casting), and generic classes/interfaces from first principles.
2. **[Generic Methods](02-generic-methods.md)** — writing your own generic methods, independent of generic classes, and type inference.
3. **[Bounded Types & Wildcards](03-bounded-types-and-wildcards.md)** — `extends`/`super` bounds, wildcards (`?`, `? extends`, `? super`), and the PECS principle.
4. **[Type Erasure](04-type-erasure.md)** — how generics are actually implemented by the compiler, what gets erased, and the real, concrete consequences and limitations this creates.
5. **[Module Summary, Interview Questions & Exercises](05-module-summary-exercises.md)** — consolidated recap, quiz, and practice problems.

## Prerequisites

- Module 10 (Collections) — you've been using generic types extensively; this module explains the mechanism behind them.
- Module 05 (OOP), especially Topic 4 (Inheritance — upcasting/downcasting) and Topic 6 (Interfaces).
- Module 09 (Arrays) — Topic 4 of this module references array-vs-generics limitations directly.

## How to Study This Module

Topics 1–2 build the practical vocabulary; Topic 3 (wildcards, PECS) is the part most learners find genuinely difficult on first exposure — expect to reread it. Topic 4 (type erasure) is the payoff: once you understand that generics are a **compile-time-only** feature, several previously-confusing rules (why you can't do `new T[]`, why `list instanceof List<String>` doesn't compile) become obvious logical consequences rather than arbitrary restrictions.

---

**Previous module:** [10 — Collections](../10-collections/00-module-overview.md) · **Next:** [01 — Why Generics — Introduction](01-why-generics-introduction.md)
