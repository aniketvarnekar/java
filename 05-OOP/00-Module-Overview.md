# Module 05 — Object-Oriented Programming (OOP)

## Module Goal

This is the module where Java's true identity comes into focus. Module 01 told you Java is "object-oriented" — this module explains **what that actually means**, mechanically and philosophically: how classes model real-world (and abstract) concepts, why Java's four pillars (Encapsulation, Abstraction, Inheritance, Polymorphism) exist, and how the JVM (Module 02) actually executes polymorphic behavior at runtime. Every module after this one — Collections, Exceptions, Streams, Concurrency — is written using OOP constructs and OOP thinking. This is the conceptual foundation the rest of Core Java stands on.

> **Scope note:** this module teaches OOP *concepts* — the pillars, relationships, and design vocabulary. The full *mechanics* of writing classes (constructors, `static`, `this`, initialization order) are covered in Module 06 (Classes), and object lifecycle/`equals`/`hashCode` in Module 07 (Objects). You'll see minimal class syntax here just enough to illustrate each concept — don't worry if you don't yet know every keyword; Modules 06–07 fill in every remaining detail immediately afterward.

## Topics Covered in This Module

1. **[Introduction to OOP](01-Introduction-To-OOP.md)** — the procedural-to-object-oriented paradigm shift, what a class and object actually are, and why Java committed to this paradigm.
2. **[Encapsulation](02-Encapsulation.md)** — bundling data with behavior, access modifiers, and why "hiding" data is a feature, not a limitation.
3. **[Abstraction](03-Abstraction.md)** — hiding complexity behind a simple interface, and how it differs from (and complements) encapsulation.
4. **[Inheritance](04-Inheritance.md)** — code reuse and IS-A relationships via `extends`, `super`, and why Java restricts classes to single inheritance.
5. **[Polymorphism](05-Polymorphism.md)** — compile-time polymorphism (overloading) vs. runtime polymorphism (overriding), and exactly how the JVM performs dynamic method dispatch.
6. **[Interfaces & Abstract Classes](06-Interfaces-And-Abstract-Classes.md)** — the two mechanisms for abstraction and contracts, a full comparison, and how interfaces let Java achieve multiple-inheritance-like behavior safely.
7. **[Association, Aggregation & Composition](07-Association-Aggregation-Composition.md)** — HAS-A relationships between objects, and why "favor composition over inheritance" is a foundational design principle.
8. **[Module Summary, Interview Questions & Exercises](08-Module-Summary-Exercises.md)** — consolidated recap, quiz, and practice problems.

## Prerequisites

- Module 02 (JVM), especially Topic 3 (Runtime Data Areas — Stack vs. Heap) and Topic 4 (Execution Engine — JIT) — polymorphism's runtime mechanism builds directly on both.
- Modules 03–04 (Java Basics, Control Flow).

## How to Study This Module

Topics 2–5 are the classic "four pillars" — study them in order, since each builds vocabulary the next uses. Topic 5 (Polymorphism) is the deepest and most interview-critical — it's where "OOP theory" meets "how the JVM actually runs your code," tying directly back to Module 02. Topic 6 resolves a question every learner eventually asks ("why can't a class extend two classes?") with a precise, mechanical answer, not just a rule to memorize.

---

**Previous module:** [04 — Control Flow](../04-Control-Flow/00-Module-Overview.md) · **Next:** [01 — Introduction to OOP](01-Introduction-To-OOP.md)
