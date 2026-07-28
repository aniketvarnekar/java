# Why Generics — Introduction

## Learning Objectives

- Understand the concrete, real problem generics solve, with a before/after comparison
- Write and use your own generic classes
- Understand raw types, and why using them is discouraged in modern code

## Prerequisites

Module 10 (Collections), Module 05 Topic 6 (Interfaces)

## Motivation

You've used `List<String>` since Module 09 without knowing precisely what problem the `<String>` part solves. This topic shows you the **actual pre-2004 Java code** this replaced — once you see how genuinely worse it was, generics stop feeling like arbitrary syntax and start feeling like an obviously necessary fix.

## Problem Statement: Life Before Generics (Pre-Java 5, 2004)

Before generics, collections held plain `Object` references — there was no way to tell a `List` "this should only ever contain `String`s":

```java
// PRE-GENERICS CODE (how Java actually looked before 2004):
List names = new ArrayList();     // NO type parameter -- a "raw type"
names.add("Aniket");
names.add(42);                       // ⚠️ COMPILES FINE -- nothing stops you from adding an int here!

String first = (String) names.get(0);   // MANUAL CAST required -- get() returns plain Object
String second = (String) names.get(1);    // COMPILES FINE... but THROWS ClassCastException AT RUNTIME!
                                              // (because names.get(1) is actually an Integer, not a String)
```

**Two genuine, serious problems, both now visible:**
1. **No compile-time type safety** — nothing prevented adding an `Integer` to a list "intended" for `String`s; the mistake was purely a matter of programmer discipline and comments, never enforced by the compiler.
2. **Manual, unsafe casting everywhere** — every single retrieval required an explicit cast back to the "intended" type, and that cast could fail at **runtime**, often far away (in both code location and time) from the actual mistake (adding the wrong type), making the bug's root cause genuinely hard to trace.

## The Generics Solution

```java
List<String> names = new ArrayList<>();
names.add("Aniket");
names.add(42);                // COMPILE ERROR NOW: incompatible types: int cannot be converted to String

String first = names.get(0);    // NO CAST NEEDED -- the compiler already KNOWS this is a String
```

**Generics let a class be written once, generically, parameterized by a type you specify at the point of use** — with the compiler enforcing that type consistently everywhere, and eliminating manual casts entirely. This directly extends Module 01's "statically typed" and "Robust" themes: an entire category of runtime `ClassCastException` bugs is converted into **compile-time** errors instead — caught immediately, at the exact line where the mistake was actually made, rather than potentially much later, at some unrelated retrieval site.

## Writing Your Own Generic Class

```java
public class Box<T> {          // T is a TYPE PARAMETER -- a placeholder for "whatever type is specified"
    private T content;

    public void set(T content) {
        this.content = content;
    }

    public T get() {
        return content;
    }
}
```

```java
Box<String> stringBox = new Box<>();
stringBox.set("Hello");
String s = stringBox.get();      // no cast needed

Box<Integer> intBox = new Box<>();
intBox.set(42);
Integer i = intBox.get();          // ALSO no cast needed -- same Box CLASS, different TYPE

stringBox.set(42);   // COMPILE ERROR -- this Box was declared to hold String, not Integer
```

**`T`** (by convention — see below) is a **type parameter**: a placeholder, filled in with a real, concrete type (`String`, `Integer`, `Employee`, anything) at the point where `Box` is actually used. **The exact same `Box` class definition works for every possible type** — you write the logic **once**, generically, and the compiler generates the appropriate type-checking for every specific usage.

## Naming Conventions for Type Parameters

Recall Module 03, Topic 8's naming conventions — generics have their own, additional convention: **single uppercase letters**, by strong tradition:

| Letter | Conventional meaning |
|---|---|
| `T` | Type (general purpose) |
| `E` | Element (used in collections, e.g., `List<E>`) |
| `K`, `V` | Key, Value (used in maps, e.g., `Map<K, V>`) |
| `R` | Result (often a return type) |
| `N` | Number |

**Why single letters, specifically?** Purely convention, to make type parameters instantly visually distinct from ordinary class names (which follow PascalCase, Module 03, Topic 8) at a glance — a `T` or `E` is unmistakably a placeholder, never confusable with a real, concrete type like `String` or `Employee`.

## Multiple Type Parameters

```java
public class Pair<K, V> {
    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey() { return key; }
    public V getValue() { return value; }
}

Pair<String, Integer> entry = new Pair<>("age", 30);
```

**This is exactly the shape `Map.Entry<K, V>` (Module 10, Topic 4) uses internally** — you're now equipped to understand precisely how that type is defined, not just how to use it.

## Generic Interfaces

Recall `Comparable<T>` and `Comparator<T>` from Module 10, Topic 7 — both are **generic interfaces**, defined using exactly this same mechanism:

```java
public interface Comparable<T> {
    int compareTo(T other);
}

public class Employee implements Comparable<Employee> {   // T is filled in as "Employee" HERE
    @Override
    public int compareTo(Employee other) { /* ... */ }
}
```

**This retroactively explains Module 10, Topic 7's `Comparable<Employee>` syntax fully** — `Comparable` is itself a generic interface, and `Employee` is the concrete type substituted for its `T` when `Employee` implements it.

## Raw Types — Legal, But Discouraged

```java
List rawList = new ArrayList();   // legal (for backward compatibility, Module 01, Topic 2) --
                                     // but throws away ALL of generics' type-safety benefits,
                                     // reverting to pre-2004, cast-everywhere, unsafe behavior
```

**Why does raw-type syntax still compile at all, given how clearly worse it is?** Directly, Module 01, Topic 2's backward-compatibility philosophy — millions of lines of genuinely pre-2004 Java code exist, using raw types throughout, and that code must keep compiling on modern JVMs. The compiler even emits an **"unchecked" warning** when raw types are used in modern code, specifically flagging the loss of type safety — but doesn't hard-error, precisely to preserve compatibility with legitimately old code. **In new code, always use a concrete type argument (or, at minimum, the diamond operator `<>`, Java 7+, letting the compiler infer it) — never a bare raw type.**

## Real-World Analogy

Think of a generic class like a **shipping container's blueprint**, deliberately designed to hold "a single type of cargo, whatever you specify" — you order a "container for holding **books**" or a "container for holding **electronics**," and once ordered, that **specific** container is permanently labeled and inspected to ensure only the specified cargo type ever goes in or comes out. Before generics, every container was an unlabeled, "holds anything" container — anyone could load anything into it, and you'd only discover a wrongly-loaded item once you opened the container far away, at the destination (runtime), rather than being stopped at the loading dock (compile time).

## Advantages

- Converts an entire class of runtime `ClassCastException` bugs into immediate compile-time errors — a direct, major robustness win.
- Eliminates manual, repetitive casting at every retrieval site, improving both safety and readability.
- Lets a single class definition work correctly and safely for any type, without code duplication.

## Disadvantages / Trade-offs

- Adds real syntactic complexity (`<T>`, `<K, V>`, and — Topic 3 — wildcards) that takes genuine time to master.
- Raw types remain legal for backward compatibility, meaning the *possibility* of reverting to unsafe, pre-generics behavior still exists if a developer isn't careful or aware.

## Best Practices

- Always specify a concrete type argument (or use the diamond operator `<>` for inference) — never use raw types in new code.
- Follow the single-uppercase-letter naming convention (`T`, `E`, `K`, `V`, `R`) for type parameters.
- Pay attention to any "unchecked" compiler warnings — they specifically flag places where type safety has been lost, usually due to raw type usage.

## Common Mistakes

- Using raw types out of unfamiliarity, unknowingly reverting to pre-2004-style unsafe casting behavior.
- Forgetting that a generic class's type parameter is decided once, at the point of construction (`new Box<String>()`) — attempting to put a different type into that specific instance is always a compile error, by design.

## Interview Questions

1. **Q: What problem did generics solve when introduced in Java 5?**
   A: Before generics, collections held plain `Object` references, with no compile-time way to enforce "this collection should only contain type X" — mistakes were only caught as `ClassCastException` at runtime, often far from the actual bug, and every retrieval required a manual, unsafe cast. Generics let the compiler enforce type consistency and eliminate manual casts, converting an entire class of runtime bugs into immediate compile-time errors.

2. **Q: What is a raw type, and why does it still compile in modern Java?**
   A: Using a generic class/interface without specifying its type parameter (e.g., `List` instead of `List<String>`), reverting to pre-generics, unsafe behavior. It remains legal purely for backward compatibility with pre-2004 code (Module 01, Topic 2's philosophy) — the compiler emits an "unchecked" warning, but doesn't hard-error.

3. **Q: What do `T`, `E`, `K`, `V` conventionally represent in generic type parameters?**
   A: `T` (general Type), `E` (Element, common in collections), `K`/`V` (Key/Value, common in maps), `R` (Result) — a naming convention using single uppercase letters to visually distinguish type parameters from real, concrete class names.

## Summary

- Before generics (pre-Java 5), collections held raw `Object`s, requiring unsafe manual casts and offering zero compile-time type safety.
- Generics let a class be written once, parameterized by a type specified at the point of use, with the compiler enforcing consistency and eliminating manual casts.
- Type parameters follow a single-uppercase-letter naming convention (`T`, `E`, `K`, `V`, `R`).
- Raw types remain legal (backward compatibility) but should never be used in new code.