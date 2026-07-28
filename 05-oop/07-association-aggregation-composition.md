# Association, Aggregation & Composition

## Learning Objectives

- Distinguish association, aggregation, and composition precisely
- Understand "favor composition over inheritance" as an actionable design principle, not just a slogan
- Recognize ownership and lifecycle dependency as the key distinguishing factor between these relationship types

## Prerequisites

[04 — Inheritance](04-inheritance.md)

## Motivation

Topic 4 taught IS-A relationships (inheritance). Most real-world object relationships are actually **HAS-A** relationships instead — a `Car` HAS-A `Engine`, a `Library` HAS-A collection of `Book`s. This topic gives you the vocabulary and judgment for modeling these correctly, and closes the loop on a principle mentioned but not yet fully justified in Topic 4: **favor composition over inheritance**.

## Problem Statement

Topic 4 ended with a cautionary example: `Stack extends ArrayList` — inheritance misused purely for code reuse, without a genuine IS-A relationship, resulting in a `Stack` that leaks unwanted `ArrayList` behavior (`add(int, T)`, `get(int)`) that violates its own intended discipline. The alternative design needs its own vocabulary — this topic provides it.

## The Three HAS-A Relationship Types

### 1. Association — The Most General "Uses/Knows-About" Relationship

> **Association**: two independent objects that are related and interact, but neither owns or controls the other's lifecycle. Both can exist entirely independently.

```java
class Student {
    void enrollIn(Course course) {   // Student USES/interacts with Course
        // ...
    }
}
class Course { /* ... */ }
```

A `Student` and a `Course` are associated — a student enrolls in courses, and a course has enrolled students — but a `Course` can exist with zero students, and a `Student` can exist without being enrolled in any course. Neither's existence depends on the other.

### 2. Aggregation — A Weaker "HAS-A," Independent Lifecycles

> **Aggregation**: a special case of association representing a genuine **whole-part** relationship ("HAS-A"), but where the **part can exist independently of the whole** — the "part" is not owned exclusively by the "whole."

```java
class Department {
    private List<Professor> professors;   // Department HAS-A list of Professors (aggregation)
}
```

A `Department` HAS `Professor`s, but if the `Department` is dissolved (deleted), the `Professor` objects themselves don't need to cease existing — they could transfer to a different department, or exist independently in the system. The whole (`Department`) does **not** control the part's (`Professor`'s) lifecycle.

### 3. Composition — The Strongest "HAS-A," Dependent Lifecycles

> **Composition**: the strongest form of whole-part relationship, where the **part's lifecycle is entirely bound to the whole's** — the part cannot meaningfully exist independently, and is typically created and destroyed *by* the whole.

```java
class House {
    private final Room[] rooms;      // House HAS-A set of Rooms (COMPOSITION)

    House() {
        rooms = new Room[]{ new Room("Living Room"), new Room("Bedroom") };
        // the Rooms are CREATED here, entirely by House itself, and never exposed
        // for independent creation/ownership elsewhere
    }
}
```

A `House`'s `Room`s don't meaningfully exist independently of that specific house — if the `House` object is destroyed, its `Room`s conceptually go with it. The `House` fully **owns** and controls its `Room`s' entire lifecycle, typically creating them itself (often inside its own constructor) rather than receiving them from outside.

## The Distinguishing Factor: Lifecycle Ownership

```
 ASSOCIATION (Weakest)              AGGREGATION                      COMPOSITION (Strongest)

┌────────────┐    ┌────────────┐   ┌────────────┐    ┌────────────┐    ┌────────────────────────────┐
│ Student    │────│ Course     │   │ Department │───◇│ Professor  │    │ House                      │
└────────────┘    └────────────┘   └────────────┘    └────────────┘    │                            │
 Both fully independent             "Whole" doesn't own the            │  ┌────────────┐            │
 Either can exist without           "part"'s lifecycle;                │  │ Room       │            │
 the other at all                   part can outlive the whole         │  └────────────┘            │
                                                                       │                            │
                                                                       │ Created & destroyed        │
                                                                       │ WITH House                 │
                                                                       └────────────────────────────┘
                                                                        "Part" has NO independent
                                                                        existence outside the whole
```

**In UML diagrams** (a standard visual notation for object relationships, worth recognizing even outside this course), aggregation is drawn with a **hollow diamond** ◇ at the "whole" end, and composition with a **filled diamond** ◆ — the visual distinction (hollow vs. filled) is a deliberate mnemonic for "the whole doesn't fully own/fill in the part's lifecycle" vs. "the whole fully owns/fills in the part's lifecycle."

## Composition in Code — What It Actually Looks Like

```java
class Engine {
    void start() { System.out.println("Engine starting..."); }
}

class Car {
    private final Engine engine;   // Car HAS-A Engine (composition)

    Car() {
        this.engine = new Engine();   // Car creates its OWN Engine -- fully owns its lifecycle
    }

    void start() {
        engine.start();               // Car DELEGATES to its Engine, rather than inheriting from it
    }
}
```

**Notice: `Car` does NOT `extends Engine`.** A `Car` is not "a kind of" `Engine` (fails the IS-A test from Topic 4 entirely) — a `Car` **has** an `Engine` as one of its components, and **delegates** to it for engine-related behavior. This delegation pattern — a class holding a reference to another object and forwarding certain calls to it — is composition's core mechanic.

## "Favor Composition Over Inheritance" — Now Fully Justified

This principle, first mentioned as a caution in Topic 4, can now be explained precisely, using everything from Topics 4–6:

**Why composition is often preferable:**

1. **No single-inheritance-slot cost.** Recall Topic 4: a class can `extends` only one superclass. Composition doesn't consume this scarce resource — a `Car` can be composed of an `Engine`, `Wheels`, and a `Transmission` simultaneously, something inheritance alone (limited to one superclass) could never express directly.

2. **No unwanted behavior leakage.** Recall the `Stack extends ArrayList` anti-pattern (Topic 4) — inheriting from `ArrayList` dragged along methods (`add(int, T)`, `remove(int)`) that violate `Stack`'s intended LIFO discipline. A composition-based `Stack` that merely **holds** an internal `ArrayList` (rather than extending it) exposes **only** the methods it deliberately chooses to (`push`, `pop`), with zero unwanted leakage:

```java
class Stack<T> {
    private final List<T> items = new ArrayList<>();   // COMPOSITION -- Stack HAS-A List internally

    void push(T item) { items.add(item); }
    T pop() { return items.remove(items.size() - 1); }
    // NO add(int, T), NO get(int) exposed -- Stack's public interface is EXACTLY what it should be
}
```

3. **Looser coupling, easier to change later.** A composed component (`Engine`) can be swapped for a different implementation (an `ElectricEngine`, perhaps via an interface — Topic 6) far more easily than restructuring an inheritance hierarchy, which tends to be more rigid and harder to safely evolve once established.

**When inheritance genuinely IS still the better choice:** when there's a real, honest IS-A relationship (Topic 4's test) **and** you specifically want polymorphism (Topic 5) — treating a whole family of related types uniformly through their shared supertype. Composition doesn't provide polymorphism the same way — a `Car` that merely *holds* an `Engine` can't be passed anywhere an `Engine` is expected, the way a `Manager` (genuinely IS-A `Employee`) can be passed anywhere an `Employee` is expected. **The principle isn't "never use inheritance" — it's "don't reach for inheritance by default, purely for code reuse, when the relationship isn't genuinely IS-A."**

## Real-World Analogy

Think of **association** like two coworkers who occasionally collaborate — each has a full, independent life outside work, and neither's existence depends on the other. Think of **aggregation** like a **sports team roster** — the team HAS players, but if the team disbands, the players themselves keep existing (they can join a different team). Think of **composition** like the **relationship between your heart and your body** — your heart doesn't have any meaningful independent existence *as your heart* outside of being part of your specific body; it's created with you and its lifecycle is entirely bound to yours.

## Advantages of Composition

- Avoids consuming a class's single-inheritance slot, allowing a class to be built from multiple independent components.
- Prevents unwanted behavior leakage that inheritance can introduce when the IS-A relationship isn't genuine.
- Produces more loosely coupled, more easily evolved designs — a component can typically be swapped without restructuring a class hierarchy.

## Disadvantages / Trade-offs

- Composition doesn't provide polymorphism the way inheritance does — you lose the ability to treat a composed class uniformly alongside its component's type.
- Can require more boilerplate (explicit delegation methods) compared to simply inheriting behavior directly — a real, if usually acceptable, cost.

## Best Practices

- Default to composition (HAS-A) for code reuse; reserve inheritance (IS-A) specifically for genuine type hierarchies where polymorphism is actually needed.
- When designing a new class relationship, explicitly ask: "is this genuinely IS-A (inheritance), or is it actually HAS-A (composition/aggregation)?" — and be honest about the answer, resisting the temptation to force inheritance just to save a bit of delegation boilerplate.
- Distinguish aggregation from composition by asking: "does the 'part' have a meaningful, independent lifecycle apart from this specific 'whole'?" If yes, aggregation; if no, composition.

## Common Mistakes

- Using inheritance purely to reuse code, without checking whether the relationship is genuinely IS-A (revisiting Topic 4's `Stack extends ArrayList` anti-pattern one final time, now with the correct composition-based alternative shown above).
- Confusing aggregation and composition — the distinguishing factor is specifically **lifecycle ownership/dependency**, not just "how tightly related do these feel."
- Believing "favor composition over inheritance" means "never use inheritance" — it means "don't default to inheritance without checking IS-A and whether polymorphism is actually needed."

## Interview Questions

1. **Q: What's the difference between aggregation and composition?**
   A: Both represent HAS-A whole-part relationships, but aggregation's "part" can exist independently of the "whole" and outlive it (e.g., a `Professor` can exist without a `Department`), while composition's "part" has its lifecycle entirely bound to the "whole" — typically created and destroyed by it, with no meaningful independent existence (e.g., a `Room` within a specific `House`).

2. **Q: What does "favor composition over inheritance" mean, and does it mean inheritance should never be used?**
   A: It means code reuse should default to composition (a class holding and delegating to another object) rather than inheritance, unless there's a genuine IS-A relationship where polymorphism is actually needed. It does not mean "never use inheritance" — it means inheritance shouldn't be reached for by default purely to reuse code when the relationship isn't genuinely IS-A.

3. **Q: Why is `Stack extends ArrayList` considered bad design, and how would composition fix it?**
   A: `Stack` isn't genuinely "a kind of" `ArrayList` — inheriting exposes unwanted methods (`add(int, T)`, `get(int)`) that let callers violate `Stack`'s intended LIFO discipline. A composition-based `Stack` instead holds an internal `ArrayList` as a private implementation detail, exposing only `push`/`pop` — the exact, intended public interface, with zero unwanted leakage.

## Summary

- **Association**: two independent objects that interact, with no ownership relationship.
- **Aggregation**: a HAS-A relationship where the "part" can exist independently of the "whole" and outlive it.
- **Composition**: the strongest HAS-A relationship, where the "part"'s lifecycle is entirely bound to the "whole."
- **"Favor composition over inheritance"**: default to composition for code reuse; reserve inheritance for genuine IS-A relationships where polymorphism is actually needed — directly resolving the `Stack extends ArrayList` anti-pattern from Topic 4.

## Module-Wide Quick Revision

- OOP organizes programs around objects bundling data + behavior; class = blueprint, object = instance (Topic 1).
- Encapsulation protects invariants via access modifiers and meaningful validation — not just "private fields" (Topic 2).
- Abstraction hides implementation complexity behind a simple interface — a different, complementary concern from encapsulation (Topic 3).
- Inheritance (`extends`, `super`) models genuine IS-A relationships; Java restricts classes to single inheritance to avoid the Diamond Problem's state-conflict ambiguity (Topic 4).
- Polymorphism: overloading is compile-time/static (resolved by argument types); overriding is runtime/dynamic (resolved by the object's actual type via the JVM's method-table dispatch, JIT-optimized via inline caching) (Topic 5).
- Abstract classes support shared state + single inheritance; interfaces support pure contracts + multiple implementation, with `default` methods (Java 8+) enabling safe interface evolution (Topic 6).
- Association/aggregation/composition model HAS-A relationships by lifecycle dependency strength; composition is generally preferred over inheritance for pure code reuse (this topic).

## Common Pitfalls (Module-Wide)

- Confusing encapsulation (who can access) with abstraction (how much complexity is exposed).
- "Fake encapsulation" — private fields with trivial, validation-free getters/setters.
- Using inheritance purely for code reuse without a genuine IS-A relationship.
- Confusing overloading (compile-time) with overriding (runtime).
- Assuming a variable's declared type (not the object's actual runtime type) determines which overridden method runs.
- Believing interfaces can never have implementation — true only through Java 7; Java 8+ default methods changed this.