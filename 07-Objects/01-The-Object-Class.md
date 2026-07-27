# The `Object` Class

## Learning Objectives

- Know every public method `java.lang.Object` provides, and what each is for
- Understand `getClass()` and how it relates to Module 02's class loading
- Build a map for the rest of this module — each remaining topic is a deep dive into specific `Object` methods

## Prerequisites

Module 05 Topic 4 (Inheritance — `Object` as the universal root)

## Motivation

Every single method you're about to see is available on **every object you will ever create in Java**, whether you know it or not — because every class inherits from `Object`, explicitly or implicitly (Module 05, Topic 4). Knowing this full method list precisely is knowing the "baseline capability" of absolutely anything in the entire language.

## Concept: `Object` as the Universal Root

```
                            java.lang.Object
                                    │
                 ┌──────────────────┼──────────────────┐
                 ▼                  ▼                  ▼
             String                Employee            (literally every
                                                          other class)
```

`Object` provides a small set of methods that every class automatically has, unless it chooses to override them. Here's the complete public method list, grouped by purpose:

| Method | Purpose | Covered in |
|---|---|---|
| `toString()` | A human-readable text representation of the object | Topic 2 |
| `equals(Object o)` | Logical equality comparison | Topic 3 |
| `hashCode()` | An integer "bucket" summary, used by hash-based collections | Topic 3 |
| `getClass()` | Returns runtime type information (a `Class` object) | This topic |
| `clone()` | Creates a copy of the object | Topic 4 |
| `finalize()` | (Deprecated) ran before GC reclaimed an object | Topic 5 |
| `wait()`, `notify()`, `notifyAll()` | Low-level thread coordination primitives | Preview here, full depth Module 15 |

## `getClass()` — Runtime Type Introspection

```java
Object obj = "Hello";
System.out.println(obj.getClass());          // class java.lang.String
System.out.println(obj.getClass().getName()); // java.lang.String
```

`getClass()` returns a `Class` object — the **exact same in-memory representation created by the Class Loader during Loading** (Module 02, Topic 2)! This is a direct, concrete link back to Module 02: every object carries a reference to the `Class` metadata object that was constructed when its class was first loaded, and `getClass()` is simply how you retrieve it from any given instance at runtime.

**Why does this matter practically?** `getClass()` is the foundation of **Reflection** (Module 16) — the ability to inspect and manipulate a class's structure at runtime. It's also commonly used inside a correct `equals()` implementation (Topic 3) to verify two objects are genuinely the same type before comparing their fields.

```java
class Point {
    int x, y;
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;   // getClass() used HERE
        Point p = (Point) o;
        return x == p.x && y == p.y;
    }
}
```

**`getClass()` returns the object's exact runtime class — never a superclass**, even when accessed through a superclass-typed reference. This is exactly Module 05, Topic 5's dynamic dispatch principle applied to a method call: `getClass()` always reflects the actual object, not the declared reference type.

```java
Employee e = new Manager();       // declared type: Employee, actual type: Manager
System.out.println(e.getClass());  // "class Manager" -- NOT "class Employee"
```

## `wait()`, `notify()`, `notifyAll()` — A Brief Preview

These three methods look out of place on `Object` at first glance — why would *every single object* need thread-coordination methods? The historical answer: **Java's original concurrency design (Java 1.0) built its synchronization primitives directly around every object's built-in "monitor lock"** (covered fully in Module 15) — so `wait`/`notify`/`notifyAll`, which operate on that per-object monitor, were placed on `Object` itself, since *any* object can potentially serve as a synchronization point. **You will not use these directly for a long time** — modern Java concurrency (`java.util.concurrent`, Module 15) provides far higher-level, safer tools for the vast majority of real use cases. This topic simply flags their existence and explains why they're here at all; full mechanics are Module 15's responsibility.

## Real-World Analogy

Think of `Object`'s methods like the **standard equipment that comes with literally every car off any assembly line**, regardless of make or model — a horn, headlights, a VIN (vehicle identification number, analogous to `getClass()`'s type identity). Every car manufacturer (every class you write) can choose to **upgrade** some of this standard equipment for their specific model (override `toString()`, `equals()`, `hashCode()`) — but every car, even one with zero customization, still has *all* of this baseline equipment functioning by default.

## Why Every Object Gets This "For Free"

This connects directly to Module 05, Topic 4: because `Object` sits at the root of every inheritance hierarchy, and inheritance means acquiring a superclass's methods automatically, **every single object in every Java program you'll ever write already has all of these methods available**, whether you think about it or not. This is precisely why `System.out.println(someRandomObject)` never crashes with "method not found," even for a class you just wrote thirty seconds ago with zero methods of your own — `toString()` is *always* there, inherited from `Object`, even if its default output (Topic 2) isn't very useful.

## Advantages

- Guarantees baseline, universal capability (a text representation, an equality check, type introspection) for literally every object, with zero effort from the class author.
- Provides the foundational hooks (`equals`/`hashCode`, `wait`/`notify`) that entire subsystems of Java (Collections, Module 10; Concurrency, Module 15) are built directly on top of.

## Disadvantages / Trade-offs

- `Object`'s **default** implementations (Topics 2–3) are frequently *not* what you actually want for your own classes — relying on them unmodified is a common, real source of bugs (covered in depth in Topics 2–3).
- A few of `Object`'s methods (`finalize()`, and arguably `clone()`) are now considered genuine historical design mistakes (Topics 4–5) — a useful, honest reminder that even foundational, universal APIs can carry design decisions later regretted, and Java's strong backward-compatibility promise (Module 01, Topic 2) means they can't simply be removed.

## Best Practices

- Know this method list well enough to recognize, immediately, when you're relying on `Object`'s default behavior vs. a deliberately overridden one in any class you read.
- Use `getClass()` (not `instanceof`) inside `equals()` implementations when you specifically want to require an **exact** type match, not just "is a subtype of" (full nuance: Topic 3).

## Common Mistakes

- Assuming a custom class has no `toString()`/`equals()`/`hashCode()` behavior at all if you didn't write them — it always does, inherited from `Object`, just probably not the behavior you actually want (Topics 2–3).
- Confusing `getClass()`'s exact-runtime-type behavior with `instanceof`'s "is-a-subtype" behavior — they answer genuinely different questions.

## Interview Questions

1. **Q: What methods does every Java object inherit from `Object`, whether or not the class explicitly extends anything?**
   A: `toString()`, `equals(Object)`, `hashCode()`, `getClass()`, `clone()`, `finalize()` (deprecated), and `wait()`/`notify()`/`notifyAll()`.

2. **Q: What does `getClass()` return, and how does it relate to Module 02's class loading?**
   A: It returns the `Class` object representing the object's exact runtime type — the same in-memory metadata object created by the Class Loader when that class was first loaded (Module 02, Topic 2). It always reflects the object's actual runtime type, never a superclass, even when accessed through a superclass-typed reference.

3. **Q: Why do `wait()`, `notify()`, and `notifyAll()` exist on `Object` rather than some dedicated thread-related class?**
   A: Java's original concurrency model was built around every object having a built-in "monitor lock," making any object a potential synchronization point — these methods operate on that per-object monitor, which is why they were placed directly on `Object` itself. Modern code typically uses higher-level `java.util.concurrent` tools instead (Module 15).

## Summary

- Every Java class ultimately inherits from `java.lang.Object`, gaining a baseline set of universal methods automatically.
- `getClass()` returns the object's exact runtime type, tying directly back to Module 02's class-loading `Class` object.
- `wait()`/`notify()`/`notifyAll()` exist on `Object` due to Java's original per-object-monitor concurrency design — full depth in Module 15.
- The rest of this module is a deep dive into `Object`'s most practically important, most frequently overridden methods.

## Exercises

1. Without looking back, list every public method `Object` provides, and one sentence on what each does.
2. Explain why `e.getClass()` returns `Manager`, not `Employee`, when `Employee e = new Manager();` — connect your answer to a concept from Module 05.
3. Explain why `System.out.println(new Object() {})` (an anonymous subclass of `Object` with nothing in it) doesn't throw a compile or runtime error about a missing `toString()` method.

---

**Previous:** [00 — Module Overview](00-Module-Overview.md) · **Next:** [02 — `toString()`](02-toString.md)
