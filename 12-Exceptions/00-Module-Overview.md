# Module 12 — Exceptions

## Module Goal

Every module since Module 01 has occasionally mentioned an exception in passing — `NullPointerException`, `ArrayIndexOutOfBoundsException`, `ClassCastException`, `ConcurrentModificationException`, `CloneNotSupportedException`. This module gives Java's exception-handling model its full, dedicated treatment: the complete `Throwable` hierarchy, the genuinely controversial checked-vs-unchecked distinction, precise `try`/`catch`/`finally` semantics (including several surprising edge cases), and — finally delivering on Module 07's preview — the complete mechanics of try-with-resources and `AutoCloseable`.

## Topics Covered in This Module

1. **[Exception Fundamentals & Hierarchy](01-Exception-Fundamentals-And-Hierarchy.md)** — the `Throwable` tree (`Error` vs. `Exception`), and basic `try`/`catch`/`finally` control flow.
2. **[Checked vs. Unchecked Exceptions](02-Checked-Vs-Unchecked-Exceptions.md)** — the deep, genuinely debated distinction, why Java introduced checked exceptions, and the real controversy around them.
3. **[Try-Catch-Finally Deep Dive](03-Try-Catch-Finally-Deep-Dive.md)** — multi-catch, `finally`'s absolute execution guarantee (and its surprising interaction with `return`), and exception chaining.
4. **[Try-With-Resources & `AutoCloseable`](04-Try-With-Resources-And-AutoCloseable.md)** — the complete mechanics of deterministic resource cleanup, finally delivering on Module 07, Topic 5's preview in full.
5. **[Custom Exceptions & Best Practices](05-Custom-Exceptions-And-Best-Practices.md)** — designing your own exception types, exception chaining/`cause`, and real anti-patterns to avoid.
6. **[Module Summary, Interview Questions & Exercises](06-Module-Summary-Exercises.md)** — consolidated recap, quiz, and practice problems.

## Prerequisites

- Module 07 (Objects), especially Topic 5 (Object Lifecycle & GC — the `AutoCloseable` preview this module completes).
- Module 05 (OOP), especially Topic 4 (Inheritance — the exception hierarchy is itself a class hierarchy).
- Module 01 (Introduction), especially Topic 3 ("Robust" — exceptions are the concrete mechanism behind that feature).

## How to Study This Module

Topic 2 is where this module earns its depth — the checked/unchecked distinction is one of Java's most debated design decisions, and understanding *both* the original reasoning and the real, legitimate criticisms it's received over 25+ years makes you far more equipped to reason about exception design in your own code than simply memorizing "checked extends Exception, unchecked extends RuntimeException." Topic 4 directly resolves the `finalize()` cautionary tale from Module 07, Topic 5.

---

**Previous module:** [11 — Generics](../11-Generics/00-Module-Overview.md) · **Next:** [01 — Exception Fundamentals & Hierarchy](01-Exception-Fundamentals-And-Hierarchy.md)
