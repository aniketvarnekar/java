# Module 06 Summary

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

## What's Next

Module 06 completed the full mechanics of how classes and objects are built. **Module 07 — Objects** now focuses on the *object lifecycle* itself: the `Object` class every type inherits from (previewed in Module 05, Topic 4), `equals()` and `hashCode()` (and the critical contract between them), `toString()`, object cloning, and how garbage collection (previewed in Module 02) actually determines when an object's life ends.