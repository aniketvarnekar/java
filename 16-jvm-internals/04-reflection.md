# Reflection

## Learning Objectives

- Use the Reflection API to inspect and invoke code at runtime
- Understand precisely why frameworks (Spring, Hibernate, JUnit) depend on Reflection
- Understand Reflection's real performance and safety costs, and when to avoid it

## Prerequisites

Module 07 Topic 1 (`getClass()`), Module 06 (Constructors, fields, methods)

## Motivation

You've been told throughout this course that `getClass()` (Module 07, Topic 1) "powers Reflection." This topic delivers on that promise fully — Reflection is the mechanism that makes nearly every framework you'll use professionally possible, and understanding it demystifies a huge amount of otherwise-"magic" framework behavior.

## The Core Idea: Inspecting and Manipulating Code at Runtime

> **Reflection** is the ability of a running Java program to **examine and manipulate its own structure and behavior** — inspecting a class's fields/methods/constructors, and even invoking them — **without knowing the exact class at compile time**.

Recall Module 07, Topic 1: `getClass()` returns a `Class` object — the exact same in-memory metadata created during Module 02's class loading. **Reflection is simply the API for reading and acting on that metadata programmatically.**

```java
Class<?> clazz = Class.forName("com.example.Employee");   // load a class BY NAME, as a STRING --
                                                              // 'Employee' is not referenced directly
                                                              // in source code AT ALL!

Object instance = clazz.getDeclaredConstructor().newInstance();   // creates an instance, dynamically

Method method = clazz.getDeclaredMethod("getSalary");
method.setAccessible(true);         // bypass access control (even for private members!)
Object result = method.invoke(instance);   // CALLS the method, dynamically, at runtime
```

**This is genuinely different from every other topic in this course** — everywhere else, you write `employee.getSalary()` directly, and the compiler checks it exists and is accessible. **Reflection bypasses compile-time checking entirely**, looking up and calling members **by name, as strings, at runtime** — with all the flexibility (and risk) that implies.

## Inspecting a Class's Structure

```java
Class<?> clazz = Employee.class;   // the 'class literal' syntax -- also produces a Class object

for (Field field : clazz.getDeclaredFields()) {
    System.out.println(field.getName() + ": " + field.getType());
}

for (Method method : clazz.getDeclaredMethods()) {
    System.out.println(method.getName());
}

for (Constructor<?> ctor : clazz.getDeclaredConstructors()) {
    System.out.println(ctor.getParameterCount());
}
```

**This is precisely how IDEs implement autocomplete for dynamically-loaded plugins, how debuggers inspect object state, and — most importantly for real, everyday backend development — how frameworks discover your classes' structure without you writing any explicit registration/wiring code.**

## Why Frameworks Depend on Reflection — The Real, Concrete Payoff

**Recall Module 05, Topic 2's JavaBeans convention** — you were told frameworks like Spring/Hibernate/Jackson rely on it "via reflection," without detail. Here's the actual mechanism:

```java
// A framework like Jackson, converting JSON to an object, conceptually does something like:
Class<?> clazz = Employee.class;
Object employee = clazz.getDeclaredConstructor().newInstance();   // create an instance

for (String jsonFieldName : jsonObject.keySet()) {
    Field field = clazz.getDeclaredField(jsonFieldName);    // find the MATCHING field, BY NAME
    field.setAccessible(true);
    field.set(employee, jsonObject.get(jsonFieldName));       // set it, dynamically
}
```

**This is precisely why Jackson can deserialize JSON into *any* class you define, without you writing conversion code for each one** — it uses Reflection to discover your class's fields/setters **at runtime**, matching them against JSON keys **by name**. **Spring's dependency injection** works analogously: it scans your classes (via Reflection) for `@Autowired`-annotated fields/constructors (Topic 5 covers annotations fully), and uses Reflection to **construct and wire objects together automatically**, based purely on runtime inspection — code you never had to write yourself.

**This is the single most important, practical takeaway of this entire topic**: **nearly every "magic" framework behavior you'll encounter professionally (dependency injection, ORM field mapping, JSON serialization, test-runner discovery of `@Test` methods) is Reflection, applied systematically.** Understanding Reflection converts "how does Spring know to do that?" from mysterious magic into a concrete, explicable mechanism.

## `setAccessible(true)` — Bypassing Encapsulation (Deliberately, With Real Risk)

Recall Module 05, Topic 2's encapsulation principle: `private` fields are supposed to be inaccessible from outside their class. **Reflection can bypass this entirely**, via `setAccessible(true)` — a genuinely powerful (and genuinely dangerous) capability:

```java
Field privateField = clazz.getDeclaredField("salary");
privateField.setAccessible(true);           // ⚠️ BYPASSES private access control!
privateField.set(employee, 999999.0);          // directly mutates a private field, from OUTSIDE the class
```

**Why does this capability exist at all, given it directly violates Module 05, Topic 2's encapsulation principle?** Frameworks genuinely need this — a JSON deserializer or an ORM needs to populate `private` fields directly (since a well-designed domain class shouldn't need public setters purely to satisfy a framework's internal needs). **This is a deliberate, narrow escape hatch, intended for framework/tooling code, not ordinary application logic** — using `setAccessible(true)` in typical business logic to bypass another class's intended encapsulation is a genuine anti-pattern, not a clever shortcut.

## The Real Costs of Reflection

- **Performance**: reflective method calls and field access are genuinely **slower** than direct calls — the JVM can't apply the same compile-time optimizations (Module 02, Topic 4's JIT inlining, for instance) as easily to reflective calls, since the exact target isn't statically known. Modern JVMs have optimized this substantially over the years, but a real, measurable gap remains for extremely hot-path code.
- **Loses compile-time safety entirely**: `clazz.getDeclaredMethod("getSallary")` (a typo!) compiles perfectly fine — the error (`NoSuchMethodException`) is only discovered **at runtime**, directly contradicting Module 01's "statically typed, catch errors early" theme.
- **Breaks encapsulation** (via `setAccessible`), a genuine, deliberate risk when misused.

## Real-World Analogy

Think of ordinary method calls like **calling a specific person by name, using a phone number you already know and have verified is correct** (compile-time checking). Think of Reflection like **looking someone up in a directory by a name written on a piece of paper, then dialing whatever number the directory lists** — genuinely flexible (you don't need to know the number in advance, and can look up completely different people using the same lookup code), but if the name on the paper is misspelled, you won't discover the mistake until you actually try to dial and it fails — much later than if you'd verified the number was correct up front.

## Advantages

- Enables genuinely generic, reusable framework code (dependency injection, ORM mapping, JSON serialization) that works with **any** class, without needing that class to implement a specific interface or follow rigid boilerplate.
- Powers essential developer tooling (IDEs, debuggers, test runners) that must work with arbitrary, unknown-in-advance code.

## Disadvantages / Trade-offs

- Genuinely slower than direct method/field access, especially relevant in hot-path code.
- Loses compile-time type/existence checking — errors that would normally be compile errors become runtime exceptions instead.
- `setAccessible(true)` deliberately bypasses encapsulation — powerful for legitimate framework use, a genuine anti-pattern if misused in ordinary application code.

## Best Practices

- Reserve Reflection for genuine framework/tooling/library code — ordinary application business logic should almost never need it directly.
- Be aware that using Reflection-heavy frameworks (Spring, Hibernate, Jackson) means accepting their internal Reflection costs — a reasonable, standard trade-off for the productivity they provide.
- Never use `setAccessible(true)` to bypass another class's encapsulation in typical business logic — this defeats the actual purpose of `private` (Module 05, Topic 2) and should be reserved for legitimate framework/tooling needs.

## Common Mistakes

- Using Reflection directly in application code where a normal, compile-time-checked method call would work just as well, unnecessarily sacrificing type safety and performance.
- Assuming Reflection-based framework "magic" is unknowable or unexplainable — it's a concrete, well-understood mechanism, now demystified by this topic.
- Overusing `setAccessible(true)` in ordinary code, defeating encapsulation for no genuine benefit.

## Interview Questions

1. **Q: What is Reflection, and what does it let you do that ordinary Java code cannot?**
   A: The ability of a running program to inspect and manipulate its own classes' structure (fields, methods, constructors) at runtime, including invoking members looked up by name (as strings) rather than referenced directly in source code — bypassing the compile-time type/existence checking ordinary Java code always has.

2. **Q: How does a framework like Jackson (JSON to object) or Spring (dependency injection) actually work internally?**
   A: Via Reflection — Jackson inspects a target class's fields/setters at runtime and matches them against JSON keys by name, setting values dynamically. Spring scans classes for specific annotations (Topic 5) via Reflection, and uses it to construct/wire objects together automatically, without you writing manual registration code.

3. **Q: What are the real costs of using Reflection?**
   A: Genuinely slower performance than direct method/field calls (the JIT can't optimize reflective calls as effectively), loss of compile-time type/existence checking (typos become runtime exceptions instead of compile errors), and the ability to bypass encapsulation via `setAccessible(true)`, which is powerful for legitimate framework use but a real anti-pattern if misused in ordinary code.

## Summary

- **Reflection** lets a running program inspect and invoke a class's structure at runtime, looked up by name rather than referenced directly in source — the concrete mechanism behind `getClass()` (Module 07, Topic 1).
- It's the actual, demystified mechanism behind nearly every "magic" framework behavior: dependency injection, ORM field mapping, JSON (de)serialization, test-runner discovery.
- `setAccessible(true)` deliberately bypasses `private`/`protected` access control — legitimate for framework code, a genuine anti-pattern in ordinary application logic.
- Real costs: slower performance than direct calls, and loss of compile-time type/existence safety.