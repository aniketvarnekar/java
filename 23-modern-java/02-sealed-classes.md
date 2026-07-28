# Sealed Classes

## Learning Objectives

- Explain why unrestricted inheritance is sometimes a liability, not a feature
- Write sealed classes and interfaces using `sealed`, `permits`, and `non-sealed`
- Understand how sealing enables compiler-checked exhaustiveness (the payoff realized fully in Topic 3)
- Compare sealed hierarchies against `enum` and against unrestricted inheritance

## Prerequisites

Module 05 Topic 4 (Polymorphism/Inheritance), Module 05 Topic 6 (Abstract Classes vs. Interfaces)

## Motivation

Module 05, Topic 4 taught you that `public`/`protected` inheritance means **anyone**, anywhere, can extend your class or implement your interface. That openness is usually exactly what you want — it's the entire point of polymorphism. But sometimes it is a **liability**: if you're modeling a genuinely **fixed, closed set of possibilities** — say, a `Shape` that is only ever a `Circle`, `Square`, or `Triangle`, and conceptually can **never** be anything else — unrestricted inheritance lets some other developer (or your future self) add a `Hexagon` subclass a year later, silently, with no compiler warning that every `switch` statement handling `Shape` (Module 04, Topic 2) is now incomplete. **Sealed classes**, finalized in **Java 17** (2021), let you declare exactly which classes are allowed to extend yours — closing the hierarchy deliberately.

## The Problem, Concretely

```java
// Unrestricted -- Module 05, Topic 4's normal inheritance
public abstract class Shape { }
public class Circle extends Shape { }
public class Square extends Shape { }
// Anyone, anywhere, in any file, can add:
public class Hexagon extends Shape { }   // compiler has no objection
```

If code elsewhere does:

```java
double area(Shape s) {
    if (s instanceof Circle c) { return ...; }
    if (s instanceof Square sq) { return ...; }
    throw new IllegalStateException("unknown shape");   // silent runtime landmine
}
```

Adding `Hexagon` doesn't break compilation anywhere — it breaks **at runtime**, the first time someone passes a `Hexagon` into `area()`, and only if that specific code path is exercised. This is precisely the class of bug Module 17's checked-exception philosophy tries to avoid for error handling — except here, there was no compiler mechanism to catch it *at all*.

## Sealed Syntax

```java
public sealed interface Shape permits Circle, Square, Triangle { }

public final class Circle implements Shape {
    private final double radius;
    public Circle(double radius) { this.radius = radius; }
}

public final class Square implements Shape {
    private final double side;
    public Square(double side) { this.side = side; }
}

public final class Triangle implements Shape {
    private final double base, height;
    public Triangle(double base, double height) { this.base = base; this.height = height; }
}
```

- **`sealed`** on `Shape` declares: "only the classes/interfaces named in `permits` may extend/implement me."
- **`permits Circle, Square, Triangle`** is the explicit, closed list. Any attempt to add a fourth implementer elsewhere is a **compile error**, not a silent runtime gap.

**A permitted subclass must itself declare exactly one of three modifiers**, stating what happens **one level further down**:

| Modifier on the subclass | Meaning |
|---|---|
| `final` | The hierarchy closes here — nothing can extend `Circle` |
| `sealed` | `Circle` itself is open only to *its own* explicit `permits` list — the closed hierarchy continues, deliberately, another level |
| `non-sealed` | `Circle` reopens to **unrestricted** inheritance — anyone can extend it from here |

```
 sealed interface Shape
        |
 permits Circle, Square, Triangle
        |
   +----+----+--------+
   |         |         |
 final     final    sealed  <- Triangle chooses to stay closed, one level further
 Circle    Square   Triangle
              |
       permits RightTriangle, EquilateralTriangle
```

**Why require every subclass to explicitly pick one?** Without this rule, an intermediate class could accidentally leave the hierarchy "half-open" with no clear signal — requiring an explicit choice at every level makes the entire hierarchy's shape **fully visible just by reading declarations**, with no gaps.

## `non-sealed` — Deliberately Reopening

```java
public sealed interface Shape permits Circle, Square, Triangle { }

public non-sealed class Triangle implements Shape { }   // Triangle deliberately reopens

public class RightTriangle extends Triangle { }          // anyone can now extend Triangle
```

**Why would you ever want this?** Sometimes only *part* of a hierarchy is genuinely fixed by design, while another part is deliberately meant to be extensible by library consumers — `non-sealed` lets the author state that boundary explicitly, rather than being forced to choose "everything closed" or "everything open" for the whole hierarchy.

## Sealed Classes Can Restrict to the Same File or Module

If all permitted subclasses live in the **same source file**, the `permits` clause can be omitted entirely — the compiler infers it from every `sealed`/`final`/`non-sealed` subtype declared in that file:

```java
public sealed interface Shape { }   // no "permits" needed

final class Circle implements Shape { }
final class Square implements Shape { }
```

This is common for small, self-contained sealed hierarchies where declaring every type in one file is natural and keeps the closed set visually obvious.

## Sealed Classes vs. `enum` vs. Unrestricted Inheritance

| | `enum` (Module 09-equivalent concept) | Sealed hierarchy | Unrestricted inheritance |
|---|---|---|---|
| Fixed, closed set of options | Yes — a fixed set of **instances** | Yes — a fixed set of **types** | No — open by default |
| Each option can carry different fields/shape | No — all constants share one shape | **Yes** — each subtype can have completely different fields | Yes |
| Compiler-checked exhaustive `switch` | Yes (Module 04, Topic 2) | **Yes** (Topic 3, this module) | No |
| Best for | A fixed set of simple, uniform constants (`DAY_MONDAY`, `DAY_TUESDAY`, ...) | A fixed set of structurally **different** cases (`Circle` has a radius; `Square` has a side) | Genuinely open-ended extension points (plugins, frameworks) |

**Why not just use `enum` for the `Shape` example?** An `enum` constant is a single, fixed instance — it cannot hold a *different set of fields* per constant in a type-safe way (Module 09's enum limitations). A `Circle` needs a `radius`; a `Triangle` needs a `base` and `height` — genuinely different shapes of data. Sealed classes give you "fixed, closed set" **and** "each case can be structurally different," which is exactly what an algebraic sum type requires — `enum` only gives you the first half.

## Real-World Analogy

Unrestricted inheritance is like **a club with an open guest list** — anyone can show up and claim membership, and the host has no way to know, in advance, the complete list of who might arrive. A sealed hierarchy is like **a wedding with a fixed, named guest list handed to security at the door** — the host (the compiler) knows, with certainty, the complete and final list of everyone who could possibly be present, and can plan (Topic 3's exhaustive `switch`) accordingly.

## Advantages

- Makes a genuinely fixed set of cases **explicit and compiler-enforced**, rather than an unenforced convention.
- Enables exhaustive `switch` checking (Topic 3) — the compiler can prove, at compile time, that every case is handled.
- Documents design intent directly in the type hierarchy: a `sealed` keyword tells every future reader "this list is deliberately closed," with no comment required.

## Disadvantages / Trade-offs

- Genuinely restrictive by design — wrong choice for hierarchies meant to be extended by unknown future code (plugin architectures, public library extension points).
- Every permitted subtype conventionally must be known/visible to the sealed type's author (same module, or accessible package) — not suited to hierarchies spanning independent, unrelated codebases.

## Best Practices

- Use `sealed` whenever a hierarchy represents a genuinely fixed, closed set of structurally different cases known in full at design time.
- Prefer letting the compiler infer `permits` from same-file declarations for small, self-contained hierarchies — explicit `permits` for hierarchies spanning multiple files.
- Combine with Topic 3's exhaustive `switch` pattern matching — this is where sealing's full value is realized.

## Common Mistakes

- Forgetting that every direct subtype **must** declare `final`, `sealed`, or `non-sealed` — omitting one is a compile error, not a default fallback.
- Using `sealed` for a hierarchy that's genuinely meant to be open for external extension (defeats the purpose, and will frustrate consumers of a public API).
- Assuming `enum` and sealed classes are interchangeable — reach for `enum` for uniform constants, sealed classes for structurally different cases.

## Interview Questions

1. **Q: What specific problem do sealed classes solve, and which Java version finalized them?**
   A: They let an author declare a fixed, closed, compiler-enforced list of permitted subtypes — preventing silent, uncontrolled hierarchy extension — finalized in Java 17 (2021).

2. **Q: What must every direct permitted subtype of a sealed type declare, and why?**
   A: Exactly one of `final` (closes the hierarchy there), `sealed` (continues the closed hierarchy one level further), or `non-sealed` (deliberately reopens to unrestricted inheritance) — required so the entire hierarchy's openness/closedness is fully explicit from its declarations alone.

3. **Q: Why might you choose a sealed hierarchy over an `enum` for a fixed set of cases?**
   A: When each case needs a genuinely different structure/fields (e.g., `Circle` needs a radius, `Triangle` needs a base and height) — `enum` constants all share one uniform shape, while sealed subtypes can each be structurally distinct.

## Summary

- **Sealed classes/interfaces** (`sealed`...`permits`), finalized in **Java 17**, let an author declare an explicit, closed, compiler-enforced list of permitted direct subtypes.
- Every permitted subtype must declare `final`, `sealed`, or `non-sealed`, making the hierarchy's full shape explicit.
- Sealed hierarchies model a fixed set of **structurally different** cases — where `enum` only models a fixed set of **uniform** constants.
- Sealing's full value is realized combined with exhaustive `switch` pattern matching (Topic 3).

## Exercises

1. Model a `PaymentMethod` sealed interface with three structurally different implementations: `CreditCard` (number, expiry), `BankTransfer` (IBAN), and `Cash` (no fields). Explain why `enum` would be a poor fit here.
2. Write a small sealed hierarchy where one subtype is `non-sealed`, and explain, in your own words, why you might deliberately choose to reopen just that one branch.
3. Explain, from memory, why every direct subtype of a sealed type must explicitly declare `final`, `sealed`, or `non-sealed` rather than defaulting to one of them.

---

**Previous:** [01 — Records](01-records.md) · **Next:** [03 — Pattern Matching](03-pattern-matching.md)
