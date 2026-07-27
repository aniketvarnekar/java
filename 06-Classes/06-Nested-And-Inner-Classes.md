# Nested & Inner Classes

## Learning Objectives

- Distinguish all four kinds of nested classes: static nested, inner, local, and anonymous
- Know precisely when each is appropriate
- Understand exactly how a (non-static) inner class holds an implicit reference to its enclosing instance

## Prerequisites

All prior topics in this module

## Motivation

You'll encounter nested classes constantly in real Java code — especially anonymous classes (common in older event-handling and callback code) and static nested classes (common in builder patterns and helper types). This topic is more reference-oriented than deeply conceptual — the goal is recognition and correct usage, not memorizing every rule.

## Problem Statement

Sometimes a class is only meaningful in the context of, or tightly coupled to, another specific class — a `Node` class that only makes sense inside a `LinkedList`, or a one-off comparator needed for a single sorting call. Declaring these as fully independent, top-level classes can add unnecessary namespace clutter and obscure the tight coupling relationship. Java provides **nested classes** — classes defined inside another class — for exactly these situations.

## The Four Kinds of Nested Classes

```
┌────────────────────────────────────────────────────────────────────────────────────┐
│                                 NESTED CLASSES                                     │
│                                                                                    │
│  ┌────────────────────────────────┐      ┌────────────────────────────────┐        │
│  │ STATIC NESTED CLASS            │      │ INNER CLASS (non-static)       │        │
│  │ (no outer instance             │      │ (HOLDS a reference to          │        │
│  │  reference needed)             │      │  a specific outer              │        │
│  │                                │      │  instance)                     │        │
│  └────────────────────────────────┘      └────────────────────────────────┘        │
│                                                                                    │
│  ┌────────────────────────────────┐      ┌────────────────────────────────┐        │
│  │ LOCAL CLASS                    │      │ ANONYMOUS CLASS                │        │
│  │ (defined INSIDE a              │      │ (no name at all,               │        │
│  │  method body)                  │      │  declared + instantiated       │        │
│  │                                │      │  in ONE expression)            │        │
│  └────────────────────────────────┘      └────────────────────────────────┘        │
└────────────────────────────────────────────────────────────────────────────────────┘
```

### 1. Static Nested Class

```java
class LinkedList {
    private Node head;

    static class Node {          // STATIC nested class -- doesn't need an OUTER LinkedList instance
        int value;
        Node next;
    }
}

LinkedList.Node n = new LinkedList.Node();   // created WITHOUT any LinkedList instance at all
```

Behaves essentially like a regular top-level class, just namespaced inside another class for organizational purposes — it has **no** implicit connection to any specific outer instance (consistent with `static`'s general meaning from Topic 4: "belongs to the class, not any instance"). This is the **most commonly used** kind of nested class in everyday code — used constantly for tightly-coupled helper types (like `Node` inside a linked data structure) and Builder-pattern classes.

**Why nest it at all, instead of a top-level class?** Purely organizational — it signals "this type is a tightly-coupled implementation detail of `LinkedList`, not meant for independent, general-purpose use elsewhere," and avoids polluting the broader namespace with a generically-named class like `Node` that might clash with unrelated `Node` classes elsewhere in a large codebase.

### 2. (Non-Static) Inner Class

```java
class Engine {
    private int horsepower = 300;

    class Diagnostics {                 // INNER class -- (non-static) -- HOLDS a reference to
        void report() {                    // a SPECIFIC Engine instance
            System.out.println("HP: " + horsepower);   // can access the OUTER instance's fields directly!
        }
    }
}

Engine engine = new Engine();
Engine.Diagnostics diag = engine.new Diagnostics();   // note the unusual syntax: created VIA an Engine instance
diag.report();   // "HP: 300"
```

**The critical difference from a static nested class:** every inner class **instance is implicitly tied to one specific enclosing outer instance**, and can directly access that specific outer instance's fields/methods (as `report()` does with `horsepower` above) — **exactly like the `Outer.this` mechanism previewed in Topic 3**. Mechanically, the compiler secretly gives every inner class instance a hidden reference back to its specific enclosing outer instance — which is precisely why creating one requires **an existing outer instance** (`engine.new Diagnostics()`), unlike a static nested class.

**When to use an inner (non-static) class specifically:** when the nested class's entire purpose is to work *with* a specific outer instance's state, genuinely needing that implicit connection — otherwise, a static nested class (simpler, no hidden outer reference, no requirement for an existing outer instance) is almost always preferable.

### 3. Local Class

```java
void processOrders(List<Order> orders) {
    class OrderValidator {                  // LOCAL class -- defined INSIDE a method body
        boolean isValid(Order o) {
            return o.getTotal() > 0;
        }
    }

    OrderValidator validator = new OrderValidator();
    for (Order o : orders) {
        if (validator.isValid(o)) {
            // ...
        }
    }
}
```

A class defined entirely **inside a method body**, visible and usable only within that method. Genuinely rare in everyday practice — used when a helper class is needed for logic **entirely local to one specific method**, and not worth promoting to a full nested or top-level class. Local classes can access **effectively final** local variables (Module 03, Topic 7) of the enclosing method, similar to how lambdas do (Module 17).

### 4. Anonymous Class

```java
interface Greeter {
    void greet();
}

void demo() {
    Greeter g = new Greeter() {          // ANONYMOUS class -- NO name, declared AND instantiated
        @Override                          // in this ONE expression
        public void greet() {
            System.out.println("Hello!");
        }
    };
    g.greet();
}
```

An anonymous class **declares and instantiates a one-off implementation of an interface (or subclass of a class) in a single expression, with no class name at all**. Historically, this was the standard way to implement simple callback/listener-style interfaces (especially in older Java GUI code, and pre-Java-8 concurrency code using `Runnable`) before **lambda expressions** (Java 8+, Module 17) offered a far more concise alternative for interfaces with exactly one abstract method.

```java
// Anonymous class (pre-Java 8 style):
Runnable r1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running!");
    }
};

// Lambda (Java 8+, MUCH more concise -- full depth: Module 17):
Runnable r2 = () -> System.out.println("Running!");
```

**Why does this course cover anonymous classes at all, if lambdas are now preferred for many cases?** Two reasons: (1) you'll constantly encounter anonymous classes in existing/legacy Java code, and need to recognize what they are; (2) lambdas **only** work for interfaces with exactly one abstract method (called **functional interfaces** — full depth, Module 17) — for anything else (implementing a class with multiple methods to override, or needing genuine per-instance state beyond what a lambda captures), an anonymous class remains the necessary tool even in modern Java.

## Quick Comparison Table

| Kind | Needs outer instance? | Has a name? | Where declared | Typical use |
|---|---|---|---|---|
| Static nested class | No | Yes | Inside a class, `static` | Tightly-coupled helper types (`Node`, Builders) |
| Inner (non-static) class | **Yes** | Yes | Inside a class, no `static` | Needs direct access to a specific outer instance's state |
| Local class | No (but sees enclosing method's effectively-final locals) | Yes | Inside a method body | Rare — logic entirely local to one method |
| Anonymous class | Depends on context | **No** | Inline, in an expression | One-off interface/subclass implementations; legacy callback code (superseded by lambdas for functional interfaces, Module 17) |

## Real-World Analogy

Think of a **static nested class** like a **department within a company that operates independently** — it's organizationally part of the company (namespaced under it) but doesn't need to consult any specific person to function. Think of an **inner (non-static) class** like a **personal assistant assigned to one specific executive** — the assistant's entire role only makes sense in relation to that one specific person, and they have direct, implicit access to that executive's calendar and files. An **anonymous class** is like **hiring a one-time contractor for a single specific task**, without bothering to give them a formal, reusable job title (class name) at all, since they'll never be hired again under that exact arrangement.

## Advantages

- Static nested classes provide clean namespacing for tightly-coupled helper types without cluttering the broader package namespace.
- Inner classes provide direct, implicit access to a specific outer instance's state, useful for genuinely tied-together behavior.
- Anonymous classes (and their modern lambda successor, Module 17) allow concise, one-off implementations without the ceremony of a fully separate, named top-level class.

## Disadvantages / Trade-offs

- Inner (non-static) classes' implicit outer-instance reference can be a source of subtle memory-retention issues (the inner instance keeps its outer instance alive as long as it exists) — a real, if advanced, concern in long-lived object graphs.
- Anonymous classes can hurt readability when overused for anything beyond simple, short implementations — deeply nested anonymous classes are a real, historically common source of hard-to-read legacy code, part of why lambdas (Module 17) were such a welcomed addition.

## Best Practices

- Default to a **static** nested class unless you specifically need access to a specific outer instance's state — it's simpler and avoids the implicit outer-reference overhead entirely.
- Prefer lambda expressions (Module 17) over anonymous classes when implementing a functional interface (single abstract method) in modern Java — reserve anonymous classes for cases lambdas genuinely can't handle.
- Use local classes sparingly — usually a well-named private method, or a properly promoted nested/top-level class, communicates intent more clearly.

## Common Mistakes

- Forgetting that creating a non-static inner class instance requires syntax tied to an existing outer instance (`outer.new Inner()`), unlike a static nested class.
- Reaching for a non-static inner class when a static nested class would suffice — unnecessarily coupling the nested type to a specific outer instance it doesn't actually need.
- Writing a verbose anonymous class implementation for a functional interface, when a concise lambda (Module 17) would express the exact same thing far more clearly.

## Interview Questions

1. **Q: What's the fundamental difference between a static nested class and a (non-static) inner class?**
   A: A static nested class has no connection to any specific outer instance — it can be instantiated independently, exactly like a top-level class, just namespaced inside another class. A non-static inner class instance is implicitly tied to one specific enclosing outer instance, holding a hidden reference back to it, which lets it directly access that outer instance's fields/methods — and requires an existing outer instance to be created.

2. **Q: What is an anonymous class, and how has its common use case evolved since Java 8?**
   A: A class with no name, declared and instantiated in a single expression, typically implementing an interface or extending a class for a one-off use. Since Java 8, lambda expressions provide a far more concise alternative specifically for functional interfaces (single abstract method) — anonymous classes remain necessary for implementing types with multiple methods or needing genuine per-instance state beyond a lambda's capture semantics.

## Summary

- **Static nested class**: no outer-instance connection, behaves like a namespaced top-level class — the most common, simplest choice.
- **Inner (non-static) class**: implicitly tied to one specific outer instance, with direct access to its state — requires an existing outer instance to create.
- **Local class**: defined inside a method body, rare in everyday practice.
- **Anonymous class**: no name, declared and instantiated inline, historically common for one-off interface implementations, largely (but not entirely) superseded by lambdas (Module 17) for functional interfaces.

## Exercises

1. Write a `Tree` class with a `static` nested `Node` class, and explain why `static` is the correct choice here rather than a non-static inner class.
2. Write an `Engine` class with a non-static inner `Diagnostics` class that reads a private `Engine` field directly, and demonstrate the `outer.new Inner()` instantiation syntax.
3. Rewrite the anonymous `Runnable` example from this topic as a lambda expression (even without having studied Module 17 in depth yet, based on the example shown), and explain why the lambda form is more concise.

---

**Previous:** [05 — Initialization Blocks & Object Creation Order](05-Initialization-Blocks-And-Object-Creation-Order.md) · **Next:** [07 — Module Summary, Interview Questions & Exercises](07-Module-Summary-Exercises.md)
