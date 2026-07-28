# Module 06 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-module-overview.md) has been covered:

- [x] **Class Members: Fields & Methods** — method signatures, varargs mechanics, and a fully worked, definitive treatment of Java's always-pass-by-value semantics (including the crucial reference-mutation-vs-reassignment distinction)
- [x] **Constructors** — the default constructor and exactly when it's generated, constructor overloading, `this(...)` chaining, why constructors aren't inherited
- [x] **The `this` Keyword** — field/parameter disambiguation, `this` as an implicit first parameter (mechanically), all legitimate uses including fluent/chainable APIs
- [x] **Static Members** — static fields/methods/blocks, the precise "shared vs. per-instance" test, static method hiding vs. instance method overriding, the Singleton pattern
- [x] **Initialization Blocks & Object Creation Order** — instance blocks, and the complete, precise six-step object creation order across an inheritance hierarchy, fully synthesizing Modules 02, 05, and 06
- [x] **Nested & Inner Classes** — static nested, (non-static) inner, local, and anonymous classes, with clear guidance on when each applies

## Practical Connections

- **Builder-pattern classes** (extremely common in real Java APIs — e.g., `StringBuilder`, HTTP client request builders) rely directly on Topic 3's "return `this`" fluent-chaining pattern.
- **Dependency injection frameworks** like Spring manage "singleton-scoped" beans conceptually similar to Topic 4's Singleton pattern, but handle instance creation/lifecycle through the framework rather than a manually-written `private` constructor + `static getInstance()` — understanding the manual pattern first makes the framework-managed version much easier to grasp.
- **JPA/Hibernate entity classes** commonly rely on precisely the object-creation-order rules from Topic 5 — frameworks that populate objects via reflection (Module 16) need entities to have accessible no-arg constructors specifically because of how that interacts with the initialization order you now understand completely.
- **`Node` classes in every custom data structure implementation** (linked lists, trees, graphs — foundational to technical interviews and some of Module 10's internals) are the textbook real-world use of Topic 6's static nested classes.
- **Legacy Android and Swing/AWT GUI code** is full of anonymous class event listeners (Topic 6) — recognizing this pattern is essential for reading pre-Java-8-style Java code you'll encounter in real, older codebases.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| Pass-by-value vs. "pass-by-reference for objects" | Java is always pass-by-value; for objects, the value copied is the reference itself — mutation through that shared reference is visible to the caller, but reassigning the local parameter is not. |
| Default constructor vs. no-arg constructor you wrote yourself | The "default constructor" specifically means the compiler-generated one, which only exists if you wrote **zero** constructors at all. |
| `this(...)` vs. `super(...)` | `this(...)` chains to another constructor in the *same* class; `super(...)` calls the *superclass's* constructor — a constructor can use only one, as the first statement. |
| Static method hiding vs. instance method overriding | Static methods are resolved at compile time by the reference's *declared* type (hiding); instance methods are resolved at runtime by the object's *actual* type (overriding, Module 05, Topic 5). |
| Static nested class vs. inner (non-static) class | Static nested classes have no connection to any outer instance; inner classes hold an implicit reference to one specific outer instance and require one to be created. |

## Consolidated Interview Questions (Module 06)

1. Is Java pass-by-value or pass-by-reference? Justify precisely, including what happens with object parameters.
2. What is a varargs parameter compiled to under the hood?
3. When does the compiler generate a default constructor, and when does it stop?
4. What does `this(...)` do, and what's the rule about combining it with `super(...)`?
5. What is `this`, mechanically, at the bytecode/method-invocation level?
6. Why can't `static` methods use `this` or access instance fields directly?
7. Can static methods be overridden? What actually happens when a subclass defines a same-signature static method?
8. What is the Singleton pattern, and how does a `private` constructor enforce its guarantee?
9. What is the complete, precise order of execution when a subclass object is created for the first time?
10. Why does Java guarantee the superclass portion of an object is fully constructed before the subclass's own initialization runs?
11. What's the difference between a static nested class and a non-static inner class?
12. How have anonymous classes' common use cases changed since Java 8 introduced lambdas?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** From memory, write out the complete six-step object creation order (Topic 5) for a two-level inheritance hierarchy.
2. **Hands-on:** Write a method that takes a `List<Integer>` parameter, both mutates it (adds an element) and reassigns the local parameter to a new list — verify experimentally which change the caller actually observes, and explain why using Topic 1's pass-by-value model.
3. **Hands-on:** Implement a `Pizza` class with a fluent builder-style API (`addTopping(String)`, `setSize(String)`, each returning `this`), and chain three calls together in one expression.
4. **Hands-on:** Implement a thread-unsafe Singleton `ConfigManager` (private constructor, static `getInstance()`), and explain (in a comment or writeup) one real risk this simple implementation has in a multithreaded context (a preview of concerns fully addressed in Module 15).
5. **Conceptual:** Explain precisely why `Parent p = new Child(); p.staticMethod();` calls `Parent`'s static method even when `p`'s actual object is a `Child` — contrast this explicitly with what would happen if `staticMethod` were an ordinary instance method instead (Module 05, Topic 5).
6. **Synthesis:** Design a small `BinaryTree` class using a `static` nested `Node` class, implementing an `insert(int value)` method — explain why `Node` should be `static` rather than a non-static inner class.

## What's Next

Module 06 completed the full mechanics of how classes and objects are built. **Module 07 — Objects** now focuses on the *object lifecycle* itself: the `Object` class every type inherits from (previewed in Module 05, Topic 4), `equals()` and `hashCode()` (and the critical contract between them), `toString()`, object cloning, and how garbage collection (previewed in Module 02) actually determines when an object's life ends.

---

**Previous:** [06 — Nested & Inner Classes](06-nested-and-inner-classes.md) · **Module Overview:** [00 — Module Overview](00-module-overview.md)

**Type "Continue" to begin Module 07 — Objects.**
