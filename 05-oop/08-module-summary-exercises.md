# Module 05 Summary, Interview Questions & Exercises

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

## Consolidated Interview Questions (Module 05)

1. What is the difference between a class and an object?
2. What is encapsulation, precisely — and why is a private field with a trivial getter/setter not "real" encapsulation?
3. What's the difference between abstraction and encapsulation?
4. Why does Java restrict classes to single inheritance? What is the Diamond Problem?
5. What's wrong with `Stack extends ArrayList`, and what's the better alternative?
6. What's the difference between method overloading and overriding, in terms of resolution timing?
7. How does the JVM actually decide which overridden method to invoke at runtime?
8. Why does downcasting require an explicit cast, and what can go wrong?
9. What's the fundamental difference between an abstract class and an interface?
10. Why were default methods added to interfaces in Java 8?
11. What happens if a class implements two interfaces with conflicting default methods?
12. What's the difference between aggregation and composition?
13. What does "favor composition over inheritance" actually mean — does it mean never use inheritance?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** From memory, list the four pillars of OOP and write a one-sentence, precise (not just a buzzword) definition for each.
2. **Hands-on:** Design and implement a small class hierarchy: an abstract `Shape` class with a `color` field and abstract `area()`/`perimeter()` methods, two concrete subclasses (`Rectangle`, `Circle`), and a `Drawable` interface both implement with a `default` `describe()` method that calls `area()` polymorphically.
3. **Hands-on:** Write a `Garage` class that HAS-A `List<Car>` via composition (not inheritance), with methods to add a car and start all cars — explain why this is composition, not aggregation (are the `Car`s' lifecycles bound to the `Garage`, or independent?).
4. **Conceptual:** A teammate proposes `class Employee extends Person` and also `class Employee extends Company` (hypothetically, if Java allowed it) because an employee "is kind of both." Explain, using the Diamond Problem, exactly why Java disallows this and what the correct alternative design would be (hint: is `Employee IS-A Company`, genuinely?).
5. **Conceptual:** Explain why `List<String> list = new ArrayList<>();` is generally preferred over `ArrayList<String> list = new ArrayList<>();`, referencing both Topic 6 ("program to an interface") and Topic 5 (polymorphism).
6. **Synthesis:** Design a small `PaymentMethod` interface with a `processPayment(double amount)` abstract method, implemented by unrelated classes `CreditCard`, `PayPal`, and `BankTransfer`. Write a `Checkout` class that accepts any `PaymentMethod` and calls `processPayment` polymorphically — explain, referencing Topic 5's JVM mechanism, exactly how the correct implementation gets invoked for each payment type at runtime.

## What's Next

Module 05 gave you OOP's conceptual and design foundation. **Module 06 — Classes** now goes deep into the *mechanics* this module deliberately kept light: constructors (including constructor overloading and chaining), the `static` keyword in full depth, the `this` keyword, instance/static initialization blocks, and the precise order in which all of this runs when an object is created — completing the picture Module 02's class-loading discussion started.

---

**Previous:** [07 — Association, Aggregation & Composition](07-association-aggregation-composition.md) · **Module Overview:** [00 — Module Overview](00-module-overview.md)

**Type "Continue" to begin Module 06 — Classes.**
