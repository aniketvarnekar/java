# Module 05 Summary

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-module-overview.md) has been covered:

- [x] **Introduction to OOP** — the procedural-to-OOP paradigm shift, precise class-vs-object definition tied to Module 02's Method Area/Heap, four-pillars preview
- [x] **Encapsulation** — access modifiers (all four levels, precisely), genuine invariant-protection vs. "fake encapsulation," JavaBeans convention
- [x] **Abstraction** — precise distinction from encapsulation, abstraction as a multi-layered spectrum (tying back to Module 01/02's JDK/bytecode layering), preview of abstract classes/interfaces
- [x] **Inheritance** — `extends`/`super` mechanics, IS-A relationship test, the Diamond Problem and why Java restricts classes to single inheritance, the `Stack extends ArrayList` anti-pattern
- [x] **Polymorphism** — overloading (compile-time/static binding) vs. overriding (runtime/dynamic binding), the JVM's method-table dispatch mechanism, JIT inline-caching optimization tied to Module 02, upcasting/downcasting, `instanceof` pattern matching
- [x] **Interfaces & Abstract Classes** — full mechanical and conceptual comparison, `default`/`static`/`private` interface methods and why Java 8 added them, the interface Diamond Problem resolution, "program to an interface"
- [x] **Association, Aggregation & Composition** — HAS-A relationship types by lifecycle-ownership strength, full justification of "favor composition over inheritance"

## Practical Connections

- **Spring Framework's entire dependency injection model** is built on "program to an interface, not an implementation" (Topic 6) — Spring beans are typically wired by interface type, letting implementations be swapped (including for testing, via mock implementations) without touching dependent code.
- **The Java Collections Framework** (Module 10) is a textbook application of this module: `List`, `Set`, `Map` are interfaces (Topic 6); `ArrayList`, `HashSet`, `HashMap` are concrete implementations; polymorphism (Topic 5) is exactly what lets you write `List<String> list = new ArrayList<>();` and have all your code work unchanged if you later switch to `LinkedList`.
- **JPA/Hibernate entity design** relies heavily on composition (Topic 7) — an `Order` entity composed of `LineItem`s is a direct, everyday application of the composition patterns taught here, and ORM frameworks specifically model aggregation vs. composition semantics (cascade delete behavior) using exactly this vocabulary.
- **Every REST API framework's request-handling interfaces** (`Runnable`, `Callable`, HTTP handler interfaces) rely on Topic 6's default-method-enabled interface evolution and Topic 5's dynamic dispatch to let framework code invoke your application-specific implementations polymorphically.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| Encapsulation vs. Abstraction | Encapsulation controls *who* can access data (protection); abstraction controls *how much* a user needs to understand to use something (simplification). Complementary, not synonyms. |
| Overloading vs. Overriding | Overloading: same class, different parameter lists, resolved at compile time. Overriding: subclass, identical signature, resolved at runtime based on actual object type. |
| Abstract Class vs. Interface | Abstract class: state + constructors + single inheritance. Interface: traditionally stateless, multiple implementation, contract-focused; `default` methods (Java 8+) added limited implementation capability specifically for safe evolution. |
| Aggregation vs. Composition | Aggregation: the "part" can exist independently of the "whole." Composition: the "part"'s lifecycle is entirely bound to the "whole." |
| Upcasting vs. Downcasting | Upcasting (specific → general) is implicit and always safe. Downcasting (general → specific) requires an explicit cast and risks `ClassCastException` if the actual object isn't really that subtype. |

## What's Next

Module 05 gave you OOP's conceptual and design foundation. **Module 06 — Classes** now goes deep into the *mechanics* this module deliberately kept light: constructors (including constructor overloading and chaining), the `static` keyword in full depth, the `this` keyword, instance/static initialization blocks, and the precise order in which all of this runs when an object is created — completing the picture Module 02's class-loading discussion started.