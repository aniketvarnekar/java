# Functional Interfaces

## Learning Objectives

- Define "functional interface" precisely
- Understand why this concept is the prerequisite that makes lambda expressions possible at all
- Use `@FunctionalInterface` correctly
- Recognize functional interfaces you've already used throughout this course

## Prerequisites

Module 05 Topic 6 (Interfaces, `default`/`static` methods), Module 06 Topic 6 (Anonymous classes)

## Motivation

Before Topic 2 shows you lambda syntax, you need to understand the concept that makes lambdas possible **at all** — a lambda isn't a floating, type-less block of code; it's always shorthand for implementing one very specific *kind* of interface. Skipping this leads to a common, real confusion: "why can't I use a lambda here?"

## Concept: What Makes an Interface "Functional"

> A **functional interface** is any interface with **exactly one abstract method**.

```java
interface Greetable {
    void greet();   // exactly ONE abstract method -- this IS a functional interface
}

interface Calculator {
    int calculate(int a, int b);   // exactly ONE abstract method -- ALSO functional
}
```

**Recall Module 05, Topic 6's Java 8 `default`/`static` method addition**: an interface can have `default` and `static` methods with real bodies, **and still be functional**, as long as it has exactly **one** abstract (unimplemented) method:

```java
interface Greetable {
    void greet();                                  // the ONE abstract method
    default void greetLoudly() {                     // default methods DON'T count against the "one" rule
        greet();
        System.out.println("!!!");
    }
}
```

**Why exactly one, specifically?** A lambda expression provides **only a body of code** — no method name at all. For the compiler to know **which** method that code is implementing, the target interface must have **exactly one** unimplemented method to match it against — if there were two or more abstract methods, the compiler would have no way to know which one your lambda's body is supposed to fill in.

## `@FunctionalInterface` — A Compiler-Enforced Guarantee

```java
@FunctionalInterface
interface Greetable {
    void greet();
}
```

**This annotation (Module 16, Topic 5's custom annotation mechanics, applied here by the JDK itself) tells the compiler: "verify this interface has exactly one abstract method, and fail to compile if that's ever violated."**

```java
@FunctionalInterface
interface Greetable {
    void greet();
    void greetAgain();   // COMPILE ERROR: Greetable is not a functional interface -- multiple
}                           // non-overriding abstract methods found
```

**Is `@FunctionalInterface` strictly required for an interface to actually BE functional?** No — the "exactly one abstract method" rule is what genuinely defines a functional interface, regardless of whether this annotation is present. `@FunctionalInterface` is purely a **deliberate, compiler-enforced safety check** — exactly like Module 05, Topic 5's `@Override` — protecting you from **accidentally** breaking an interface's functional status later (e.g., a teammate adding a second abstract method to an interface your codebase relies on being usable with lambdas, without realizing the consequence) by turning that mistake into an immediate, loud compile error rather than a silent, confusing failure somewhere else entirely.

## Functional Interfaces You've Already Used

**This concept isn't new to this module — you've been using functional interfaces since Module 10:**

```java
Runnable r = ...;              // Module 15, Topic 1 -- ONE abstract method: run()
Comparator<T> c = ...;          // Module 10, Topic 7 -- ONE abstract method: compare(T, T)
                                   // (equals() doesn't count -- it's inherited from Object,
                                   //  Module 07, Topic 1, not a genuinely NEW abstract method)
```

**Recognizing this retroactively is genuinely valuable**: `Runnable` and `Comparator` were **always** functional interfaces — you simply hadn't yet learned the vocabulary term, or the lambda shorthand (Topic 2) for implementing them concisely. This is exactly why Module 15, Topic 1 could write `new Thread(() -> ...)` and Module 10, Topic 7 could write `Comparator.comparing(...)` — both relied on this exact mechanism, used before it was formally named.

## Why This Concept Had to Come Before Lambdas Could Exist

This is the module's central "why," stated directly: **Java's designers needed a way to let a block of code be treated as "an implementation of some interface," without requiring the ceremony of a full anonymous class declaration** (Module 06, Topic 6) **every single time.** But Java remained a statically-typed, interface-driven language (Module 05) — a lambda couldn't just be an untyped, free-floating function the way it might be in a more dynamically-typed language. **The functional interface concept is the bridge**: a lambda's **type** is always inferred as "whatever functional interface is expected at this position" — meaning lambdas are fully compatible with, and require no exception to, Java's static type system.

```java
Runnable r = () -> System.out.println("running");
// the lambda's TYPE is inferred as Runnable, because that's what the LEFT SIDE expects --
// the compiler matches the lambda's parameter list/body shape against Runnable's ONE
// abstract method (run(), which takes no parameters and returns void) to verify compatibility
```

## Real-World Analogy

Think of a functional interface like a **standardized job posting with exactly one specific task listed** ("whoever fills this role must be able to `greet()` visitors") — because there's only **one** task described, anyone can walk up and say "I'll do that specific task," without needing to fill out a full, formal employment application (an anonymous class) — they just need to demonstrate they can perform that **one** specific, unambiguous task. A job posting listing **two** different tasks would be genuinely ambiguous for someone trying to informally volunteer for "the task" — which one would they even be agreeing to do?

## Advantages

- Provides the precise, minimal condition needed for lambda expressions to work within Java's static type system, without any special-casing or compromise to that system.
- `@FunctionalInterface` provides a genuine, compiler-enforced safety net against accidentally breaking an interface's lambda-compatibility.

## Disadvantages / Trade-offs

- The "exactly one abstract method" rule is a real constraint — an interface genuinely needing two or more distinct behaviors cannot be used with lambda shorthand, and must fall back to a full implementing class or anonymous class (Module 06, Topic 6).

## Best Practices

- Always annotate interfaces intended for lambda use with `@FunctionalInterface`, exactly the same discipline as always using `@Override`.
- Before assuming a lambda can be used somewhere, check that the target type is genuinely a functional interface (exactly one abstract method).

## Common Mistakes

- Assuming any interface can be implemented with a lambda — only functional interfaces (exactly one abstract method) qualify.
- Forgetting that `default`/`static` methods (and methods inherited from `Object`, like `equals()`) don't count toward the "one abstract method" rule.
- Adding a second abstract method to an interface without realizing it breaks lambda-compatibility everywhere that interface is used that way — precisely what `@FunctionalInterface` protects against.

## Interview Questions

1. **Q: What is a functional interface, precisely?**
   A: An interface with exactly one abstract method — `default`, `static`, and `Object`-inherited methods (like `equals()`) don't count against this rule.

2. **Q: Is `@FunctionalInterface` required for an interface to be usable with a lambda?**
   A: No — the "exactly one abstract method" rule alone determines whether an interface is functional. `@FunctionalInterface` is a compiler-enforced safety annotation that verifies and protects this property, catching accidental violations (like adding a second abstract method) as an immediate compile error.

3. **Q: Why can't a lambda expression be used to implement an interface with two abstract methods?**
   A: A lambda provides only a method body, with no method name — the compiler determines which method it implements by matching it against the target interface's single abstract method. With two or more abstract methods, there would be no way to determine which one the lambda's body is meant to implement.

## Summary

- A **functional interface** has exactly one abstract method (`default`/`static`/`Object`-inherited methods don't count).
- **`@FunctionalInterface`** is a compiler-enforced check verifying and protecting this property, exactly like `@Override`'s role for method overriding.
- Functional interfaces are the conceptual bridge that lets lambda expressions (Topic 2) exist within Java's static type system — a lambda's type is always inferred as the specific functional interface expected at its usage site.
- `Runnable` and `Comparator`, used earlier in this course, were always functional interfaces — this topic gives you the formal vocabulary for what you'd already been using.