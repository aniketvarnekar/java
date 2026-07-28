# Annotations & Dynamic Proxies

## Learning Objectives

- Write and use custom annotations correctly, including retention and target
- Understand how a framework actually reads annotations at runtime (tying directly to Topic 4)
- Understand Dynamic Proxies precisely, and how they implement Spring-style AOP behavior

## Prerequisites

[04 — Reflection](04-reflection.md), Module 05 Topic 6 (Interfaces)

## Motivation

You've used annotations constantly (`@Override`, Module 05, Topic 5; `@FunctionalInterface`, coming in Module 17) without knowing how to write your own or how a framework actually *acts* on one. This topic covers both — and reveals Dynamic Proxies, the specific mechanism that makes Spring's `@Transactional`, logging aspects, and similar "behavior injected around a method call" features actually work.

## Custom Annotations

```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)   // controls WHEN this annotation is available (see below)
@Target(ElementType.METHOD)             // controls WHERE this annotation can be applied
public @interface Test {                  // 'public @interface' -- defines a NEW annotation type
    String description() default "";        // an annotation can have "elements" (like parameters)
}
```

```java
class Calculator {
    @Test(description = "Verifies addition works")
    void testAdd() {
        // ...
    }
}
```

**`@interface` declares a genuinely new annotation type** — structurally similar to declaring an interface (Module 05, Topic 6), but with special compiler treatment. **Elements** (like `description` above) work like configurable parameters attached to each usage.

## `@Retention` — Controlling Annotation Lifetime

| Retention Policy | Meaning |
|---|---|
| `SOURCE` | Discarded by the compiler — exists only in source code (e.g., `@Override`, purely a compile-time check) |
| `CLASS` | Retained in the `.class` file's bytecode (Topic 1), but **not** loaded/available at runtime (rare, default if unspecified) |
| `RUNTIME` | Retained in bytecode **and** available via Reflection (Topic 4) at runtime — **this is the retention level frameworks actually need** |

**Why does this matter, concretely?** Recall Module 05, Topic 5: `@Override` is checked entirely at **compile time** and then **discarded** — it has zero runtime presence, `SOURCE` retention is sufficient. **A framework annotation like Spring's `@Autowired` or JUnit's `@Test` MUST use `RUNTIME` retention**, since the framework needs to **discover it via Reflection while the program is actually running** (Topic 4's exact mechanism) — this is the direct, concrete link between annotations and Reflection.

## `@Target` — Controlling Where an Annotation Can Be Applied

```java
@Target({ElementType.METHOD, ElementType.FIELD})   // can be applied to methods OR fields, nothing else
```

Common `ElementType` values: `TYPE` (classes/interfaces), `METHOD`, `FIELD`, `CONSTRUCTOR`, `PARAMETER`, `LOCAL_VARIABLE`. **This is compiler-enforced** — attempting to apply a `@Target(ElementType.METHOD)` annotation to a field is a genuine compile error, exactly like Module 05, Topic 5's `@Override` misuse detection.

## Reading Annotations at Runtime — Directly Using Topic 4's Mechanism

```java
Method method = Calculator.class.getDeclaredMethod("testAdd");

if (method.isAnnotationPresent(Test.class)) {
    Test testAnnotation = method.getAnnotation(Test.class);
    System.out.println("Running test: " + testAnnotation.description());
    method.invoke(new Calculator());   // Topic 4's Reflection, invoking the annotated method
}
```

**This is literally, precisely how JUnit finds and runs your `@Test`-annotated methods** — it scans your test class's methods (via Reflection, Topic 4), checks each one for the `@Test` annotation's presence, and invokes the matching ones. **There is no other, separate "magic" mechanism** — annotations plus Reflection, applied together, is the entire story.

## Dynamic Proxies — Injecting Behavior Around Method Calls

**This is a genuinely powerful, specific application of Reflection** — creating an object **at runtime** that implements a given interface, **without writing a concrete class for it at all**, where every method call is intercepted and routed through custom logic:

```java
interface Greeter {
    String greet(String name);
}

class RealGreeter implements Greeter {
    public String greet(String name) { return "Hello, " + name + "!"; }
}

InvocationHandler handler = (proxy, method, args) -> {
    System.out.println("Before calling: " + method.getName());   // LOGGING, injected AROUND the call
    Object result = method.invoke(new RealGreeter(), args);         // delegate to the REAL implementation
    System.out.println("After calling: " + method.getName());
    return result;
};

Greeter proxy = (Greeter) Proxy.newProxyInstance(
    Greeter.class.getClassLoader(),
    new Class<?>[]{Greeter.class},     // the interface(s) to implement
    handler                              // the logic to run for EVERY method call
);

proxy.greet("Aniket");
// prints: "Before calling: greet" / "Hello, Aniket!" (implicitly, via the real greeter) / "After calling: greet"
```

**`Proxy.newProxyInstance(...)` creates a genuinely new object, at runtime, implementing `Greeter`** — but with **no actual `.java` source file or compiled class for this specific proxy ever written by you**. Every method call on it is routed through your `InvocationHandler`, which decides what to do — here, logging before/after, then delegating to the real implementation via Reflection (`method.invoke`, directly reusing Topic 4's mechanism).

## This Is Exactly How Spring's `@Transactional` Works

**This is the single most valuable, concrete payoff of this topic**: Spring's declarative features (`@Transactional`, `@Cacheable`, method-level security checks) work by creating a **Dynamic Proxy** around your actual bean — when you call a method on what you *think* is your own object, you're often actually calling the **proxy**, whose `InvocationHandler`-equivalent logic:
1. Starts a database transaction (for `@Transactional`)
2. Delegates to your **real** method's actual logic (via Reflection, exactly like the `Greeter` example)
3. Commits the transaction if your method succeeds, or rolls it back if an exception propagates out (Module 12's exception model, directly relevant here)

**This is called Aspect-Oriented Programming (AOP)** — injecting cross-cutting behavior (logging, transactions, security checks) **around** existing method calls, **without modifying the original method's own code at all**. Dynamic Proxies (for interface-based beans) — or a related bytecode-generation technique called CGLIB for class-based beans without interfaces, worth knowing exists — are the concrete mechanism Spring uses to implement this.

## Real-World Analogy

Think of annotations like **sticky notes attached to specific parts of a document** — `@Retention(SOURCE)` is a sticky note the editor removes before publishing (relevant only during editing/compilation); `@Retention(RUNTIME)` is a sticky note that gets **printed directly into the final published document**, so anyone reading it later (a framework, via Reflection) can still see and act on the note. A Dynamic Proxy is like a **personal assistant who intercepts every single phone call meant for their boss** — the caller thinks they're talking directly to the boss, but the assistant actually answers first, does some preparatory work (logging who called, checking the boss's calendar), **then** patches the call through to the real boss, and potentially does follow-up work afterward — all without the caller ever needing to know an intermediary was involved at all.

## Advantages

- Custom annotations, combined with Reflection, provide a clean, declarative way to attach metadata to code that frameworks/tools can discover and act on — the foundation of nearly all modern Java framework design.
- Dynamic Proxies enable powerful, non-invasive cross-cutting behavior injection (AOP) without modifying the original class's source code at all.

## Disadvantages / Trade-offs

- Both mechanisms add real "action at a distance" — behavior that isn't visible by reading the annotated/proxied class's own source code alone, which can genuinely complicate debugging and understanding for newcomers to a framework.
- Dynamic Proxies only work for **interface-based** designs directly (a proxy implements an interface) — proxying a concrete class without an interface requires additional techniques (like CGLIB) beyond the JDK's built-in `Proxy` class.

## Best Practices

- Use `RUNTIME` retention specifically and only when a framework/your own Reflection-based code genuinely needs to discover the annotation at runtime; use `SOURCE` for purely compile-time-checked markers.
- Recognize that Spring/Hibernate/JUnit's "magic" behavior is Annotations + Reflection + (often) Dynamic Proxies — a concrete, learnable mechanism, not something to treat as unknowable.
- Design classes intended for AOP-style framework use (like Spring beans needing `@Transactional`) around interfaces where practical, for the cleanest, most standard proxy-based interception.

## Common Mistakes

- Forgetting `@Retention(RetentionPolicy.RUNTIME)` on a custom annotation intended to be read via Reflection, and being confused when `isAnnotationPresent`/`getAnnotation` don't find it.
- Assuming framework "magic" is unexplainable, rather than recognizing it as Annotations + Reflection + Dynamic Proxies, applied systematically.
- Calling a method directly on `this` from within a Spring-managed bean, expecting `@Transactional`-style proxy behavior to apply — since the call bypasses the proxy entirely when made internally rather than through the external, proxied reference (a genuinely common, real Spring gotcha, directly explained by understanding how proxies actually work).

## Interview Questions

1. **Q: What does `@Retention(RetentionPolicy.RUNTIME)` do, and why do framework annotations need it?**
   A: It ensures the annotation is retained in the compiled bytecode and made available for inspection via Reflection while the program is actually running. Framework annotations (like `@Test`, `@Autowired`) need this specifically because the framework discovers and acts on them at runtime, using exactly the Reflection mechanism from Topic 4.

2. **Q: What is a Dynamic Proxy, and what does `Proxy.newProxyInstance(...)` actually create?**
   A: An object, created entirely at runtime, implementing a specified interface, where every method call is routed through a custom `InvocationHandler` — with no concrete, compiled class written for that specific proxy. It's used to inject custom behavior (logging, transactions) around method calls without modifying the original implementation's source.

3. **Q: How does Spring's `@Transactional` annotation actually work, mechanically?**
   A: Spring creates a Dynamic Proxy (or a CGLIB-based proxy for non-interface classes) around your bean. Calls to the proxy's methods are intercepted, starting a database transaction before delegating (via Reflection) to your real method's logic, then committing or rolling back the transaction based on whether the method completed normally or an exception propagated — Aspect-Oriented Programming, implemented via proxying.

## Summary

- **Custom annotations** (`@interface`) attach metadata to code; `@Retention(RUNTIME)` is required for a framework to discover them via Reflection (Topic 4); `@Target` restricts where an annotation can be applied.
- Combined, **Annotations + Reflection** are the complete, concrete mechanism behind test-runner discovery (`@Test`), dependency injection (`@Autowired`), and similar framework "magic."
- **Dynamic Proxies** (`Proxy.newProxyInstance`) create runtime-generated objects implementing an interface, routing every method call through custom interception logic — the mechanism behind Spring's `@Transactional`/AOP-style cross-cutting behavior injection.

## Exercises

1. Write a custom `@Loggable` annotation with `RUNTIME` retention, apply it to a method, and write Reflection-based code that checks for its presence and prints a message before invoking the method.
2. Implement a Dynamic Proxy around a simple interface that logs every method call's name and arguments before delegating to the real implementation.
3. Explain, step by step, how Spring's `@Transactional` annotation and Dynamic Proxies work together to start/commit/rollback a database transaction around your business logic, without you writing that transaction-management code yourself.

---

**Previous:** [04 — Reflection](04-reflection.md) · **Next:** [06 — Method Handles & Modern Internals](06-method-handles-and-modern-internals.md)
