# Interfaces & Abstract Classes

## Learning Objectives

- Define and use abstract classes and interfaces correctly
- Know the full, precise comparison between them
- Understand default and static interface methods (Java 8+), and why they were added
- Understand exactly how Java resolves the interface-multiple-inheritance Diamond Problem
- Apply the "program to an interface, not an implementation" principle

## Prerequisites

[03 — Abstraction](03-Abstraction.md), [04 — Inheritance](04-Inheritance.md)

## Motivation

Topic 3 promised you'd learn Java's two dedicated abstraction mechanisms — here they are, in full. The abstract-class-vs-interface question is a perennial interview classic, and the reasoning behind Java 8's addition of default methods (a genuinely significant language change) is one of the best examples in this entire course of a deliberate, well-reasoned language evolution decision.

## Problem Statement

You need a way to say "every type in this family **must** provide certain behavior," without necessarily providing (or being able to provide) a full implementation at the point where you declare that requirement. Sometimes you also want to share **some** common implementation across a family of related classes, while still leaving other parts unimplemented, to be filled in differently by each concrete subclass.

## Abstract Classes

```java
abstract class Shape {
    protected String color;                    // regular field -- CAN have state

    Shape(String color) {                        // regular constructor -- CAN have one
        this.color = color;
    }

    abstract double area();                       // ABSTRACT method -- NO body, subclasses MUST implement it

    void describe() {                              // CONCRETE method -- fully implemented, inherited as-is
        System.out.println(color + " shape with area " + area());
    }
}

class Circle extends Shape {
    private double radius;

    Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }

    @Override
    double area() {                                // MUST provide this -- Shape declared it abstract
        return Math.PI * radius * radius;
    }
}
```

- **`abstract class`** cannot be instantiated directly (`new Shape(...)` is a compile error) — it exists specifically to be **extended**.
- Can contain **both** abstract methods (no body, must be implemented by subclasses) **and** concrete methods (fully implemented, inherited as-is).
- Can have **fields with actual state**, constructors, and any access modifier on its members — it behaves, structurally, almost exactly like a regular class, except it can't be instantiated and can declare unimplemented abstract methods.
- A class can `extends` only **one** abstract class (it's still a *class* — Topic 4's single-inheritance rule fully applies).

## Interfaces

```java
interface Drawable {
    void draw();                                   // implicitly public and abstract
    double MAX_SIZE = 1000.0;                        // implicitly public, static, AND final
}

class Circle implements Drawable {
    @Override
    public void draw() {                             // must be explicitly public here
        System.out.println("Drawing a circle");
    }
}
```

- **`interface`** declares a pure **contract** — traditionally, only method signatures (no body) and constants, with **no instance state (fields) at all**.
- Every method in an interface is **implicitly `public abstract`** (unless it's `default`, `static`, or `private` — see below) — you never write those modifiers explicitly.
- Every field in an interface is **implicitly `public static final`** — meaning interfaces can only ever declare **constants**, never genuine mutable instance state.
- A class can **`implements` multiple interfaces simultaneously** — this is the key structural difference from abstract classes, covered fully below.

## Default and Static Interface Methods (Java 8+) — Why They Were Added

Before Java 8, interfaces could **only** declare abstract method signatures — zero implementation was ever allowed. This created a **real, painful problem** for evolving the standard library: if the JDK team wanted to add a new method to the widely-implemented `List` interface (say, `forEach`), **every single class in existence that implemented `List`** — across the entire Java ecosystem, including code the JDK team had never seen — would **immediately fail to compile**, since none of them would have implemented the new method. This made evolving core interfaces after their initial release essentially impossible without breaking the world.

**Java 8's solution: `default` methods** — interface methods **with a body**, providing a default implementation that implementing classes automatically inherit **without being forced to implement it themselves**:

```java
interface Greetable {
    String getName();                               // still abstract -- must be implemented

    default void greet() {                            // DEFAULT method -- has a body, OPTIONAL to override
        System.out.println("Hello, " + getName() + "!");
    }
}

class Person implements Greetable {
    private String name;
    Person(String name) { this.name = name; }

    @Override
    public String getName() { return name; }
    // greet() is NOT implemented here -- it's INHERITED from Greetable's default implementation
}
```

```java
Person p = new Person("Aniket");
p.greet();   // prints "Hello, Aniket!" -- using Greetable's DEFAULT implementation, unmodified
```

**This is precisely how the JDK itself added `forEach`, `stream()`, and many other methods to existing collection interfaces in Java 8** (Module 10/18 will use these constantly) **without breaking a single existing implementation anywhere in the Java ecosystem** — a real, historically significant engineering achievement, directly enabling the Streams API (Module 18) and Lambda expressions (Module 17) to integrate cleanly into pre-existing interfaces like `List` and `Comparator`.

**Static interface methods** (also Java 8+) are utility methods that belong to the interface itself, not to any implementing instance — used for interface-related helper/factory logic:

```java
interface Greetable {
    static Greetable of(String name) {                // STATIC method -- called as Greetable.of(...), 
        return () -> name;                               // NOT on an instance
    }
    String getName();
}
```

**Private interface methods** (Java 9+) allow interfaces to have internal helper methods, used only by their own `default`/`static` methods, not exposed to implementers at all — a small, later refinement extending the same evolution.

## The Full Comparison: Abstract Class vs. Interface

| Aspect | Abstract Class | Interface |
|---|---|---|
| Instantiable directly? | No | No |
| Can have instance fields (state)? | **Yes** | No — only `public static final` constants |
| Can have constructors? | **Yes** | No |
| Method implementations allowed? | Yes, freely (regular concrete methods) | Only via `default`/`static`/`private` methods (Java 8+/9+) |
| Access modifiers on members | Any (`private`, `protected`, `public`, package-private) | Implicitly `public` (unless `private` helper method, Java 9+) |
| How many can a class extend/implement? | **Only one** (`extends`) — single inheritance | **Multiple** (`implements`) — no limit |
| Represents | "IS-A," typically with shared state and partial implementation | A capability/contract — "CAN-DO," typically stateless |
| Typical use case | A family of closely related classes sharing common state/logic, with some behavior deliberately left for subclasses to define | An unrelated variety of classes that all need to guarantee a specific capability (e.g., `Comparable`, `Runnable`, `Serializable`) |

## Why Java Provides BOTH — Not Just One

This directly answers the question posed at the end of Topic 3: these two tools solve **genuinely different design problems**, and neither can fully replace the other:

- **Abstract classes** are the right tool when you have a **genuine, shared IS-A family** that also needs **shared state and partial shared implementation** — e.g., all `Shape`s share a `color` field and a `describe()` implementation, while each specific shape must define its own `area()`.
- **Interfaces** are the right tool when you need to guarantee a **capability across otherwise-unrelated classes** — a `Duck`, an `Airplane`, and a `PaperAirplane` share almost nothing in common as *types*, but could all reasonably `implements Flyable`, each flying via wildly different mechanisms, with **zero** shared state or implementation needed between them.

## How Interfaces Achieve "Multiple Inheritance" Safely

Recall Topic 4's Diamond Problem: Java disallows multiple **class** inheritance specifically because of ambiguous **state** conflicts. Interfaces traditionally hold **no state at all**, which is precisely why implementing multiple interfaces is safe — there's no field-conflict version of the Diamond Problem to worry about.

But Java 8's `default` methods reintroduced a *narrower* version of the Diamond Problem — for **behavior**, not state:

```java
interface A {
    default void greet() { System.out.println("Hello from A"); }
}
interface B {
    default void greet() { System.out.println("Hello from B"); }
}

class C implements A, B {   // COMPILE ERROR: class C inherits unrelated defaults for greet() from A and B
    // MUST explicitly override greet() to resolve the conflict:
    @Override
    public void greet() {
        A.super.greet();      // explicitly choose (or combine) whichever behavior is intended
    }
}
```

**Java's rule here is precise and much simpler than C++'s multiple-inheritance rules:** if a class implements two interfaces that provide **conflicting** `default` implementations for the same method, the class is **required** to override that method explicitly and resolve the conflict itself — the compiler refuses to silently guess. This gives Java the flexibility of "multiple behavioral inheritance" via interfaces, **without** ever silently resolving a genuine ambiguity — it forces the *human* to make an explicit choice exactly where ambiguity exists, rather than papering over it with an implicit rule that might not match the developer's actual intent.

## "Program to an Interface, Not an Implementation"

This is a foundational, widely-cited OOP design principle, made concrete by everything in this topic and Topic 5:

```java
// LESS FLEXIBLE:
ArrayList<String> names = new ArrayList<>();     // the VARIABLE's type is tied to one specific implementation

// MORE FLEXIBLE (program to the interface):
List<String> names = new ArrayList<>();            // the variable's type is the general CONTRACT --
                                                        // the concrete implementation can be swapped
                                                        // (e.g., to LinkedList) with ONE line changed,
                                                        // and every use of 'names' elsewhere is unaffected
```

**Why does this matter, concretely?** Declaring variables, parameters, and return types using the most general interface/abstract type that satisfies your actual needs (rather than a specific concrete class) maximizes polymorphism's benefit (Topic 5): calling code becomes decoupled from *which specific implementation* is actually in use, and that implementation can be swapped later — for performance tuning, testing (substituting a test-double implementation), or simply better-fitting behavior — with minimal ripple effects through the rest of the codebase. You'll apply this principle constantly starting in Module 10 (Collections).

## Real-World Analogy

Think of an **abstract class like a partially-completed form template** — some sections are already filled in and shared by everyone using the template (concrete methods, fields), while other sections are explicitly left blank with a label saying "you must fill this in" (abstract methods) — and you can only ever be handed *one* such template to complete. Think of an **interface like a certification/license** — a "Certified Electrician" license doesn't dictate *how* you learned electrical work or what tools you personally own (no shared state/implementation); it just guarantees you can perform certain tasks reliably. And critically, a person can hold **multiple, entirely unrelated certifications simultaneously** (electrician AND scuba diving instructor) — exactly like a class implementing multiple, unrelated interfaces.

## Advantages

- Abstract classes let a family of related classes share real state and implementation, reducing duplication beyond what interfaces alone can offer.
- Interfaces provide safe, unlimited "multiple inheritance of type/capability," letting unrelated classes guarantee shared behavior without the Diamond Problem's state-conflict risk.
- `default` methods let interfaces evolve over time without breaking existing implementers — a genuinely significant, real engineering capability the JDK itself relies on constantly.

## Disadvantages / Trade-offs

- Overusing `default` methods can blur the traditional, clean "interface = pure contract, no implementation" mental model — a real, ongoing design tension in modern Java.
- Choosing between an abstract class and an interface isn't always obvious for a new design — genuinely requires the judgment this topic aims to build, not a mechanical rule.

## Best Practices

- Default to interfaces for defining capabilities/contracts, especially across otherwise-unrelated classes; reach for abstract classes specifically when you need genuine shared state and partial shared implementation across a closely related family.
- Program to interfaces (or abstract supertypes) in variable/parameter/return-type declarations wherever practical, to maximize flexibility and testability.
- Use `default` methods primarily for genuine backward-compatible interface evolution, not as a general-purpose way to avoid writing abstract methods.

## Common Mistakes

- Believing interfaces can never have any implementation at all — true only through Java 7; since Java 8, `default`/`static` methods (and Java 9+ `private` methods) can carry real implementation.
- Forgetting interface fields are implicitly `public static final` — attempting to give an interface genuine, mutable instance state is simply not possible.
- Assuming a class can `extends` multiple abstract classes — it cannot; only interfaces support "implementing" multiple types at once.

## Interview Questions

1. **Q: What's the fundamental difference between an abstract class and an interface?**
   A: An abstract class can hold real instance state, constructors, and freely-mixed abstract/concrete methods, but a class can only extend one. An interface traditionally holds no instance state (only constants), and a class can implement any number of interfaces simultaneously — interfaces define a capability/contract rather than a stateful IS-A family.

2. **Q: Why were default methods added to interfaces in Java 8?**
   A: To let existing interfaces (like `List`, `Comparator`) gain new methods without breaking every class that already implemented them across the entire Java ecosystem — a default implementation is automatically inherited unless a class chooses to override it, enabling backward-compatible interface evolution. This was essential infrastructure for cleanly introducing the Streams API and Lambdas.

3. **Q: If a class implements two interfaces that both provide a conflicting `default` method with the same signature, what happens?**
   A: A compile error — Java requires the implementing class to explicitly override the method and resolve the conflict itself (optionally calling `InterfaceName.super.method()` to select or combine specific behavior), rather than silently guessing which default should apply.

4. **Q: What does "program to an interface, not an implementation" mean, and why does it matter?**
   A: Declaring variables/parameters/return types using the most general interface/abstract type that satisfies your needs, rather than a specific concrete class — this decouples calling code from a specific implementation choice, letting that implementation be swapped later (for performance, testing, or better fit) with minimal ripple effects elsewhere in the codebase.

## Summary

- **Abstract classes**: can hold state/constructors/concrete methods, support single inheritance only, model a genuine shared IS-A family.
- **Interfaces**: traditionally stateless (only constants), support multiple implementation, model a capability/contract; since Java 8, can carry `default`/`static` (and Java 9+ `private`) method implementations specifically to enable safe interface evolution.
- Conflicting `default` methods from multiple interfaces force an explicit compile-time resolution — Java never silently guesses.
- "Program to an interface, not an implementation" is a foundational design principle this topic's mechanics directly enable.

## Exercises

1. Design (in code) an abstract class `Vehicle` with shared state (`speed`) and a concrete `honk()` method, plus an abstract `accelerate()` method — then implement two subclasses. Explain why this scenario suits an abstract class better than an interface.
2. Design (in code) an interface `Flyable` with a single abstract `fly()` method, implemented by two completely unrelated classes (e.g., `Bird` and `Drone`). Explain why this scenario suits an interface better than an abstract class.
3. Reproduce the two-interfaces-with-conflicting-default-methods example from this topic, and write the explicit override needed to resolve the conflict, using `InterfaceName.super.methodName()` to call one specific interface's version.
4. Explain, precisely, why interface fields being implicitly `public static final` makes the interface-multiple-implementation Diamond Problem fundamentally less dangerous than the class-multiple-inheritance version from Topic 4.

---

**Previous:** [05 — Polymorphism](05-Polymorphism.md) · **Next:** [07 — Association, Aggregation & Composition](07-Association-Aggregation-Composition.md)
