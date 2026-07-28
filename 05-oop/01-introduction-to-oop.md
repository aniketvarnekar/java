# Introduction to OOP

## Learning Objectives

- Explain the procedural-to-OOP paradigm shift and the specific problem OOP solves
- Precisely define "class" and "object," and the relationship between them
- Preview the four pillars you'll study in depth for the rest of this module

## Prerequisites

Module 01 (Introduction), especially Topic 3 (Features of Java)

## Motivation

You've been writing Java code since Module 01 without ever defining your own class beyond `HelloWorld`. That's about to change permanently — from here on, essentially all Java code you write and read will be organized around classes and objects. Understanding *why* this paradigm won out, not just its mechanics, will make every remaining module in this course click faster.

## Problem Statement

Before object-oriented programming became dominant, most software was written **procedurally** — a program was a sequence of functions/procedures operating on data that was largely separate from those functions. As programs grew larger (thousands, then millions of lines of code, built by large teams), procedural code developed real, recurring problems:

- **Data and the functions that operate on it live far apart**, making it hard to see, at a glance, everything that can modify a given piece of data — any function, anywhere in the program, could reach in and mutate shared data structures.
- **No natural way to bundle "the same kind of thing"** — modeling a `Car` meant scattering its speed, fuel level, and behavior (`accelerate`, `brake`) across separate variables and free-standing functions, with nothing in the language itself expressing "these all belong together."
- **Reuse was awkward** — extending or specializing existing functionality typically meant copy-pasting and modifying, rather than building on top of what already existed.

## Concept: The Object-Oriented Answer

> **Object-oriented programming (OOP)** organizes a program around **objects** — self-contained units that bundle **data** (state) together with the **behavior** (methods/functions) that operates on that data — rather than around free-standing functions operating on separate data.

### Class vs. Object — The Blueprint Analogy

This is the most fundamental distinction in all of OOP, and it's worth being fully precise about it immediately:

> A **class** is a **blueprint/template** — it defines *what fields* and *what methods* every object of that type will have, but a class itself holds no actual data for any specific real-world thing.
>
> An **object** (also called an **instance**) is a *specific, concrete thing* created **from** a class — it has its own actual, real values stored in the fields the class defined.

```java
class Car {                    // the CLASS -- a blueprint, defines the SHAPE
    String color;                // every Car has a color field...
    int speed;                    // ...and a speed field...

    void accelerate() {            // ...and this behavior
        speed += 10;
    }
}

Car myCar = new Car();          // an OBJECT -- an actual Car, created FROM the blueprint
myCar.color = "Red";              // THIS object's color is "Red"
myCar.speed = 0;

Car yourCar = new Car();         // a DIFFERENT, completely independent object
yourCar.color = "Blue";            // THIS object's color is "Blue" -- unrelated to myCar's
```

```
                        CLASS: Car (Blueprint)
                  (Stored in the Method Area / Metaspace)

                  ┌──────────────────────────────────┐
                  │ Fields                           │
                  │ • color                          │
                  │ • speed                          │
                  │                                  │
                  │ Methods                          │
                  │ • accelerate()                  │
                  └─────────────────┬────────────────┘
                                    │
                     Used to create many objects
                     (Each has its own state)
                  ┌─────────────────┴────────────────┐
                  ▼                                  ▼
      ┌──────────────────────────┐      ┌──────────────────────────┐
      │ myCar (Object on Heap)   │      │ yourCar (Object on Heap) │
      │                          │      │                          │
      │ color = "Red"            │      │ color = "Blue"           │
      │ speed = 0                │      │ speed = 0                │
      └──────────────────────────┘      └──────────────────────────┘
```

**This directly connects to Module 02's Class Loader and Runtime Data Areas:** the `Car` **class** — its structure, method bytecode — is loaded once and lives in the **Method Area/Metaspace**, shared by everyone. Each `Car` **object** you create with `new` gets its own, independent memory on the **Heap**, holding its own actual field values. This is precisely why `myCar.color` and `yourCar.color` can differ — they're two entirely separate Heap allocations, both structured according to the one shared `Car` blueprint.

### Real-World Analogy

Think of a **class like an architectural blueprint for a house**, and an **object like an actual, physical house built from that blueprint**. The blueprint defines "every house of this design has 3 bedrooms, 2 bathrooms, and a front door" — it's a specification, not a livable structure. You can build many actual houses from the same blueprint, and each one is independent — painting one house's front door red doesn't repaint every other house built from the same blueprint. This is *exactly* the relationship between `Car` (the class/blueprint) and `myCar`/`yourCar` (objects/actual houses) above.

## Why Java Specifically Committed to OOP (Revisiting Module 01)

Recall from Module 01, Topic 1: Java is described as "nearly everything is an object" by deliberate design. The reasoning, made concrete now:

- **Bundling data with behavior** (encapsulation, Topic 2) means a `Car` object's `speed` field and the `accelerate()` method that changes it live in *one place*, not scattered — directly solving procedural programming's "data and functions live far apart" problem.
- **Classes as reusable blueprints** mean once `Car` is defined, creating a thousand different car objects requires zero additional class-writing — just `new Car()` a thousand times, each independent.
- **Inheritance and polymorphism** (Topics 4–5) provide structured, language-supported ways to build specialized variations (`SportsCar extends Car`) and write code that works uniformly across a whole family of related types — directly solving procedural programming's "reuse is awkward" problem.

## The Four Pillars — A Preview

The rest of this module is organized around what's traditionally called the **"four pillars of OOP"** — four foundational ideas that, together, define what it means for a language/design to be genuinely object-oriented:

```
┌────────────────────────────────────────────────────────────────────────────────────┐
│                              THE FOUR PILLARS OF OOP                               │
│                                                                                    │
│  ┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────────┐      │
│  │ ENCAPSULATION        │  │ ABSTRACTION          │  │ INHERITANCE          │      │
│  │                      │  │                      │  │                      │      │
│  │ • Bundle data and    │  │ • Hide complexity    │  │ • Create new types   │      │
│  │   behavior           │  │ • Expose only        │  │   from existing ones │      │
│  │ • Hide internal      │  │   essential          │  │ • Represents an      │      │
│  │   implementation     │  │   functionality      │  │   IS-A relationship  │      │
│  │ • Topic 2            │  │ • Topic 3            │  │ • Topic 4            │      │
│  └──────────────────────┘  └──────────────────────┘  └──────────────────────┘      │
│                                                                                    │
│                           ┌────────────────────────────────────┐                   │
│                           │ POLYMORPHISM                       │                   │
│                           │                                    │                   │
│                           │ • One interface                    │                   │
│                           │   Multiple implementations         │                   │
│                           │ • Same method, different behavior  │                   │
│                           │ • Topic 5                          │                   │
│                           └────────────────────────────────────┘                   │
└────────────────────────────────────────────────────────────────────────────────────┘
```

| Pillar | One-sentence essence | Covered in |
|---|---|---|
| **Encapsulation** | Bundle data with the behavior that operates on it; control access to internal state | Topic 2 |
| **Abstraction** | Expose only what's necessary; hide implementation complexity behind a simple interface | Topic 3 |
| **Inheritance** | Build new, specialized classes from existing ones, reusing and extending their structure/behavior | Topic 4 |
| **Polymorphism** | Code written against a general type can work correctly with any of its specific subtypes, each behaving according to its own actual implementation | Topic 5 |

## Advantages of OOP

- Bundling data and behavior together (encapsulation) makes large codebases dramatically easier to reason about — you know where to look for everything relevant to a `Car`.
- Inheritance and polymorphism provide structured, compiler-supported mechanisms for code reuse and extensibility, reducing duplication.
- Objects map naturally onto real-world (and many abstract) domain concepts, which tends to make OOP designs easier for teams to discuss and agree on ("a `Customer` has an `Order` which has `LineItem`s").

## Disadvantages / Trade-offs

- OOP can be over-applied — creating deep, unnecessary class hierarchies for problems that would be simpler as plain functions/data (a real, common critique, and part of why Java added functional-programming features in Java 8 — Module 17).
- Learning curve — genuinely understanding all four pillars, and *when* each applies, takes real time and practice, which is exactly why this entire module exists.

## Best Practices

- Start thinking in terms of "what data belongs together, and what behavior operates on it" whenever modeling a new problem — this is the core OOP instinct this module will sharpen over its remaining topics.
- Don't force every single value into its own class — Module 03's primitives exist precisely because not everything benefits from being a full object (Module 01/03's pragmatic performance trade-off).

## Common Mistakes

- Confusing a class with an object — a class is the blueprint (one definition); objects are the many independent instances created from it.
- Assuming all of a class's objects share the *same* field values — each object has its own, independent copy of every instance field (unless that field is `static`, Module 02/03, in which case it's genuinely shared).

## Interview Questions

1. **Q: What is the difference between a class and an object?**
   A: A class is a blueprint/template defining what fields and methods objects of that type will have; it holds no actual data itself. An object is a specific instance created from a class, with its own independent, actual field values, allocated on the Heap.

2. **Q: Why did Java's designers commit to object-oriented programming as the language's core paradigm?**
   A: OOP directly addresses procedural programming's recurring problems at scale: it bundles data with the behavior that operates on it (rather than scattering them), provides classes as reusable blueprints for creating many independent instances, and provides structured mechanisms (inheritance, polymorphism) for code reuse and extensibility that procedural code lacks.

3. **Q: If I create two objects from the same class, do they share the same field values?**
   A: No — each object gets its own, independent allocation on the Heap, with its own copy of every instance field. Only fields declared `static` are shared across all instances of a class, since those live in the Method Area, not per-object on the Heap.

## Summary

- OOP organizes programs around **objects** — bundles of data (state) and the behavior (methods) that operates on it — directly solving procedural programming's "data and functions live far apart" and "reuse is awkward" problems.
- A **class** is a blueprint (structure/behavior definition, shared, lives in the Method Area); an **object** is an independent instance created from that blueprint (actual data, lives on the Heap).
- The **four pillars** — Encapsulation, Abstraction, Inheritance, Polymorphism — are the foundational ideas the rest of this module explores in depth.

## Exercises

1. In your own words, explain the difference between a class and an object, using an analogy other than the blueprint/house one used in this topic.
2. Given a `class Book { String title; boolean isCheckedOut; }`, create (on paper/pseudocode) three independent `Book` objects with different field values, and explain why changing one object's `isCheckedOut` field doesn't affect the others.
3. Without reading ahead, guess (in one sentence each) what you think "Encapsulation," "Abstraction," "Inheritance," and "Polymorphism" mean, based only on their everyday English meanings — we'll revisit your guesses as each topic unfolds.

---

**Previous:** [00 — Module Overview](00-module-overview.md) · **Next:** [02 — Encapsulation](02-encapsulation.md)
