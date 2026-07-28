# Polymorphism

## Learning Objectives

- Distinguish compile-time polymorphism (overloading) from runtime polymorphism (overriding)
- Understand exactly how the JVM performs dynamic method dispatch — not just "it works," but the actual mechanism
- Use upcasting and downcasting correctly, including safe downcasting with `instanceof`
- Explain why polymorphism is the pillar that makes the other three actually *useful* at scale

## Prerequisites

[04 — Inheritance](04-inheritance.md), Module 02 Topic 4 (Execution Engine — JIT)

## Motivation

"Polymorphism" literally means "many forms" (Greek: *poly* = many, *morphe* = form) — one name, many behaviors depending on context. This is the pillar where OOP theory meets JVM mechanics directly: understanding *how* the JVM decides, at runtime, which specific method implementation to actually run is both a deep technical fact and one of the most common senior-level interview questions in Java.

## Problem Statement

Suppose you have `Manager`, `Engineer`, and `Intern`, all subclasses of `Employee` (Topic 4), each with their own version of `calculatePay()`. You want to write **one** piece of code — say, a payroll processing loop — that correctly calculates pay for *any* employee, regardless of their specific subtype, **without** writing a separate `if (employee instanceof Manager) ... else if (employee instanceof Engineer) ...` chain for every single subtype. Polymorphism is exactly what makes this possible.

```java
List<Employee> staff = List.of(new Manager(...), new Engineer(...), new Intern(...));
for (Employee e : staff) {
    System.out.println(e.calculatePay());   // NO type-checking needed --
                                               // each call automatically runs the CORRECT
                                               // subtype's own calculatePay() implementation
}
```

## Two Kinds of Polymorphism in Java

### 1. Compile-Time Polymorphism — Method Overloading

> **Overloading**: multiple methods in the **same class** share the **same name** but have **different parameter lists** (different number, type, or order of parameters).

```java
class Calculator {
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; }
    int add(int a, int b, int c) { return a + b + c; }
}
```

**"Compile-time" because the decision of *which* overload to call is made entirely by the compiler**, based purely on the static types of the arguments at the call site — before the program ever runs:

```java
Calculator calc = new Calculator();
calc.add(2, 3);        // compiler picks add(int, int) at COMPILE TIME
calc.add(2.5, 3.5);      // compiler picks add(double, double) at COMPILE TIME
```

This is also called **static binding** or **early binding** — "static" here means "resolved at compile time," directly connected to Module 01's "statically typed" discussion.

**Overload resolution rules (in order of preference):** Java tries, in order: (1) an exact type match, (2) a match via widening primitive conversion (Module 03, Topic 4), (3) a match via autoboxing/unboxing (Module 03, Topic 6), (4) a match via varargs. This ordering itself is a genuinely common, subtle interview topic — for instance, Java prefers widening a `byte` to an `int` over autoboxing it to `Byte`, because widening is checked before boxing.

### 2. Runtime Polymorphism — Method Overriding

> **Overriding**: a **subclass** provides its **own implementation** of a method that's already defined (with the exact same signature) in its **superclass**.

```java
class Employee {
    double calculatePay() { return 50000; }
}

class Manager extends Employee {
    @Override
    double calculatePay() { return 80000; }   // OVERRIDES the parent's version
}
```

```java
Employee e = new Manager();     // the VARIABLE's declared type is Employee...
System.out.println(e.calculatePay());   // ...but this prints 80000 -- Manager's version runs!
```

**This is the crux of runtime polymorphism:** even though the variable `e` is *declared* as type `Employee`, the **actual method that runs is determined by the object's real, runtime type** (`Manager`), not the variable's compile-time declared type. This is also called **dynamic binding** or **late binding** — "resolved while the program is actually running," not at compile time.

**The `@Override` annotation** (seen above) isn't strictly required by the compiler, but is a strongly recommended best practice: it tells the compiler "I intend this to override a superclass method" — if you make a typo in the method signature (accidentally creating an unrelated overload instead of a genuine override), `@Override` turns that silent, easy-to-miss mistake into an immediate, loud compile error.

## How the JVM Actually Performs Dynamic Dispatch (The Mechanism)

This is where Module 02's Execution Engine knowledge pays off directly. **How does the JVM know, at the exact moment `e.calculatePay()` executes, to run `Manager`'s version rather than `Employee`'s?**

Every class's method table (conceptually a **virtual method table**, or "vtable" — informally) is set up during class loading (Module 02, Topic 2), recording, for each overridable method signature, **which class's implementation should actually run for objects of that specific class**:

```
  Employee's method table:                 Manager's method table (inherits + OVERRIDES):

  ┌────────────────────────────────┐      ┌────────────────────────────────┐
  │ calculatePay -> Employee's     │      │ calculatePay -> Manager's      │
  │                 implementation │      │                 implementation │
  └────────────────────────────────┘      └────────────────────────────────┘
                                                   ▲
                                                   │
                                              overridden!
```

When you call `e.calculatePay()`, the JVM does **not** look at the *variable's* declared type (`Employee`) to decide which code to run. Instead, at runtime, it looks at the **actual object's real class** (found via the object's own internal type information on the Heap — Module 02, Topic 3) and consults **that specific class's** method table to find the correct implementation to invoke — this lookup-and-invoke process is called **dynamic method dispatch** (or "virtual method invocation").

```
 e.calculatePay() called:
        │
        ▼
 JVM checks: what is the ACTUAL runtime type of the object 'e' currently refers to?
        │
        ▼
 It's a Manager object (regardless of e's declared type being Employee)
        │
        ▼
 JVM consults MANAGER's method table for calculatePay
        │
        ▼
 Manager's own calculatePay() implementation runs
```

**Connecting this to Module 02's JIT compiler:** this per-call runtime lookup sounds like it could be slow if it happened, unoptimized, on every single call — and naively, it would be. This is exactly the kind of thing the **JIT compiler** (Module 02, Topic 4) optimizes aggressively: if the JIT observes (through its runtime profiling) that a particular call site *always* resolves to the same concrete implementation in practice (a technique called **monomorphic call-site optimization** / **inline caching**), it can compile that call site as if it were a direct, non-virtual call — and if that assumption later turns out to be wrong (a different subtype shows up), the JIT **de-optimizes** (Module 02, Topic 4) and falls back to the general dynamic dispatch mechanism, then potentially re-optimizes again with corrected assumptions. **This is a direct, concrete example of everything you learned about JIT de-optimization in Module 02 being applied specifically to make polymorphism fast in practice, not just theoretically elegant.**

## Upcasting and Downcasting

Recall Module 03, Topic 4's widening/narrowing vocabulary for primitives — the **exact same conceptual pattern** applies to object references, with a genuinely important mechanical difference:

### Upcasting — Implicit, Always Safe

```java
Manager m = new Manager(...);
Employee e = m;                // UPCASTING: Manager -> Employee, happens automatically/implicitly
```
Treating a more specific type (`Manager`) as its more general supertype (`Employee`) is always safe — every `Manager` genuinely *is* an `Employee` (the IS-A relationship from Topic 4) — so Java allows this without any explicit cast, exactly like widening primitive conversions.

### Downcasting — Explicit, Potentially Unsafe

```java
Employee e = new Manager(...);
Manager m = (Manager) e;         // DOWNCASTING: Employee -> Manager, requires an EXPLICIT cast
```
Going the other direction — treating a general reference as a more specific type — requires an explicit cast, because it **might be wrong**: the object `e` actually refers to might genuinely be a `Manager`, or it might be some *other* `Employee` subtype entirely (`Engineer`, `Intern`) — the compiler can't verify this from the declared type alone.

```java
Employee e = new Engineer(...);   // e's ACTUAL object is an Engineer, not a Manager
Manager m = (Manager) e;            // COMPILES fine (compiler only checks declared types)...
                                       // ...but THROWS ClassCastException AT RUNTIME!
```

**This is directly analogous to Module 03, Topic 4's narrowing-cast data-loss risk** — but for references, the "risk" is a **`ClassCastException`** at runtime instead of silent data loss, since the JVM *does* check the object's actual runtime type at the moment of a downcast, and throws immediately if it's incompatible.

### Safe Downcasting with `instanceof`

```java
if (e instanceof Manager) {
    Manager m = (Manager) e;    // safe -- we've already confirmed the actual type
    // ...
}
```

**Modern Java (16+) simplifies this further with pattern matching for `instanceof`** (full depth: Module 23), combining the check and the cast into one expression:

```java
if (e instanceof Manager m) {    // 'm' is automatically available, already correctly typed,
    System.out.println(m.getBonus());   // inside this block -- no separate explicit cast needed
}
```

## Why Polymorphism Is What Makes the Other Pillars Actually Useful at Scale

This is worth stating directly, as a genuine synthesis: Encapsulation (Topic 2) and Abstraction (Topic 3) let you design clean, well-protected individual classes. Inheritance (Topic 4) lets you build related families of classes. **But it's polymorphism specifically that lets you write code operating on a whole *family* of related types uniformly**, without knowing (or caring) about every specific subtype in advance — including subtypes that don't even exist yet when your code was written. A payroll system written today, using `Employee e; e.calculatePay();`, will correctly handle a brand-new `Consultant extends Employee` subclass added next year, with **zero changes** to the payroll code itself — this is polymorphism's real, practical payoff, and it's precisely what makes large, evolving OOP codebases maintainable over time.

## Real-World Analogy

Think of polymorphism like a **universal electrical outlet standard within one country**: you plug any device — a lamp, a phone charger, a vacuum cleaner — into the exact same kind of wall socket, using the exact same simple action (plug in), and each device does its own, completely different thing once powered, according to its own internal design. The wall socket (your code calling `e.calculatePay()`) doesn't need to know in advance every kind of device (subclass) that might ever be plugged in — it just needs to know "anything plugged in here knows how to handle being powered," and dynamic dispatch is the "electricity" that reaches each specific device's own internal circuitry (implementation) correctly.

## Advantages

- Enables writing general-purpose code that correctly handles any conforming subtype, including ones written after the general code itself.
- Dynamic dispatch, combined with JIT optimization (inline caching), achieves this flexibility with performance that's often nearly indistinguishable from non-polymorphic code in practice.
- `instanceof` pattern matching (modern Java) makes safe downcasting concise and less error-prone than the classic separate-check-then-cast pattern.

## Disadvantages / Trade-offs

- Overload resolution rules (widening vs. boxing vs. varargs preference) can occasionally produce genuinely surprising results for ambiguous or borderline call sites — a real, if narrow, source of confusion.
- Deep polymorphic call chains can make it harder to trace, just by reading source code, exactly which implementation will run for a given call — you often need to know (or check) the actual runtime type, not just read the static call site.

## Best Practices

- Always use `@Override` when overriding a method — it converts silent signature-typo mistakes into loud compile errors.
- Prefer designing code against supertypes/interfaces (Topic 6) rather than concrete subtypes wherever reasonable, to maximize polymorphism's benefit (this is a preview of the "program to an interface, not an implementation" principle).
- Use `instanceof` pattern matching (modern Java) instead of the older separate-check-then-manually-cast pattern for cleaner, safer downcasting.

## Common Mistakes

- Confusing overloading (compile-time, same class, different parameter lists) with overriding (runtime, subclass, identical signature) — they are entirely different mechanisms with entirely different resolution timing.
- Assuming a variable's *declared* type determines which overridden method runs — it's always the object's *actual runtime type* that determines this, never the declaration.
- Downcasting without an `instanceof` check first (or without pattern matching), risking an unhandled `ClassCastException` at runtime.

## Interview Questions

1. **Q: What's the difference between method overloading and method overriding?**
   A: Overloading is multiple methods in the *same class* sharing a name but differing in parameter list — resolved at *compile time* based on argument types (static/early binding). Overriding is a *subclass* providing its own implementation of an inherited method with an identical signature — resolved at *runtime* based on the object's actual type (dynamic/late binding).

2. **Q: If a variable is declared as type `Employee` but actually holds a `Manager` object, which `calculatePay()` implementation runs when you call it?**
   A: `Manager`'s implementation — dynamic method dispatch always resolves based on the object's actual runtime type, never the variable's compile-time declared type.

3. **Q: How does the JVM actually decide which overridden method implementation to invoke at runtime?**
   A: Each class has a method table (set up during class loading) mapping method signatures to the correct implementation for that specific class. At the call site, the JVM inspects the object's actual runtime type and consults that class's method table — this is dynamic method dispatch. The JIT compiler often optimizes this further via inline caching when a call site is observed to consistently resolve to the same implementation, de-optimizing if that assumption is later violated.

4. **Q: Why does downcasting require an explicit cast, and what happens if the cast is invalid?**
   A: Downcasting (general type → specific type) might be wrong — the compiler can't verify from a variable's declared type alone whether the actual object really is the target subtype. An explicit cast signals awareness of this risk; if the object's actual runtime type is genuinely incompatible with the cast target, the JVM throws `ClassCastException` at the moment of the cast.

## Summary

- **Overloading** (compile-time polymorphism): same name, different parameter lists, resolved statically by the compiler based on argument types.
- **Overriding** (runtime polymorphism): subclass redefines an inherited method with an identical signature, resolved dynamically at runtime based on the object's actual type — never the variable's declared type.
- The JVM performs **dynamic method dispatch** by consulting the actual object's class's method table at the call site; the JIT compiler (Module 02) optimizes this heavily via inline caching, with de-optimization as a fallback.
- **Upcasting** (specific → general) is implicit and always safe; **downcasting** (general → specific) requires an explicit cast and risks `ClassCastException`, best guarded with `instanceof` (or modern pattern matching).
- Polymorphism is what lets code written against a general type correctly handle any conforming subtype — including ones that don't exist yet — making it the pillar that makes large, evolving OOP systems maintainable.