# Module 06 — Classes

## Module Goal

Module 05 taught you OOP *concepts* using deliberately minimal class syntax. This module fills in every mechanical detail that was glossed over: how constructors actually work (and chain), what `this` really refers to, what `static` truly means and when to reach for it, the *exact* order everything runs in when an object is created, and how classes can nest inside other classes. By the end, you'll be able to explain precisely what happens, step by step, from `new SomeClass()` to a fully-initialized object sitting on the Heap.

## Topics Covered in This Module

1. **[Class Members: Fields & Methods](01-class-members-fields-and-methods.md)** — a precise look at fields and methods, revisiting method overloading, varargs, and Java's pass-by-value semantics for object references in full, worked detail.
2. **[Constructors](02-constructors.md)** — the default constructor, constructor overloading, and why constructors exist as a distinct language concept rather than just being "a method that sets fields."
3. **[The `this` Keyword](03-the-this-keyword.md)** — disambiguation, `this()` constructor chaining, and what `this` actually *is* at the bytecode level.
4. **[Static Members](04-static-members.md)** — static fields, methods, and blocks; precisely when `static` is the right (and wrong) choice.
5. **[Initialization Blocks & Object Creation Order](05-initialization-blocks-and-object-creation-order.md)** — instance initializer blocks, and the exact, complete order of execution when an object is created — tying directly back to Module 02's class loading.
6. **[Nested & Inner Classes](06-nested-and-inner-classes.md)** — static nested classes, (non-static) inner classes, local classes, and anonymous classes — what each is for, and how they differ.
7. **[Module Summary, Interview Questions & Exercises](07-module-summary-exercises.md)** — consolidated recap, quiz, and practice problems.

## Prerequisites

- Module 05 (OOP) — this module directly extends the class/object vocabulary and inheritance mechanics introduced there.
- Module 02 (JVM), especially Topic 2 (Class Loader Subsystem) and Topic 3 (Runtime Data Areas) — object creation order and static members connect directly to both.

## How to Study This Module

Topics 2–3 (Constructors, `this`) go together — read them back to back. Topic 5 is the payoff of this entire module: it assembles everything from Topics 2–4 into one precise, complete execution trace, and directly resolves lingering questions from Module 02's class-loading discussion (Topic 2, "when exactly does a class initialize?") by extending it to the *object* level. Topic 6 (Nested Classes) is more reference-oriented — useful to know exists and recognize in others' code, less something you'll write constantly as a beginner.

---

**Previous module:** [05 — OOP](../05-oop/00-module-overview.md) · **Next:** [01 — Class Members: Fields & Methods](01-class-members-fields-and-methods.md)
