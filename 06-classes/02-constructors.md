# Constructors

## Learning Objectives

- Write constructors correctly, including multiple overloaded constructors
- Understand exactly what the compiler does (and doesn't) provide automatically
- Chain constructors within the same class using `this(...)`
- Understand why constructors are a distinct language concept, not just "a method that happens to run first"

## Prerequisites

[01 — Class Members: Fields & Methods](01-class-members-fields-and-methods.md), Module 05 Topic 4 (Inheritance — `super(...)` preview)

## Motivation

You've already used constructors throughout Modules 05–06 without a dedicated explanation. This topic makes every rule explicit: what the compiler generates for you automatically, when it stops doing so, and how to avoid duplicating initialization logic across multiple constructors using chaining.

## Problem Statement

When an object is created, it usually needs to start in a valid, fully-initialized state — recall Module 05, Topic 2's encapsulation principle: an object should never be observable in an invalid state. Fields need real values assigned as part of construction, not left at their (Module 03, Topic 1) default zero/null values and hoped to be set correctly by whoever happens to use the object afterward.

## Concept: What a Constructor Is

> A **constructor** is a special block of code, with the **same name as the class** and **no return type at all** (not even `void`), that runs **automatically, exactly once**, at the moment an object is created via `new`, responsible for bringing that object into a valid initial state.

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {     // CONSTRUCTOR -- same name as the class, no return type
        this.x = x;
        this.y = y;
    }
}

Point p = new Point(3, 4);    // the constructor runs HERE, automatically, as part of 'new'
```

## Why a Constructor Is NOT "Just a Method"

This is a genuinely important distinction, not pedantry:

| | Constructor | Regular Method |
|---|---|---|
| Name | Must exactly match the class name | Any valid identifier |
| Return type | **None at all** — not even `void` | Required (`void` if nothing returned) |
| Called how | Only implicitly, via `new` | Explicitly, by name, on an object/class |
| Can be inherited? | **No** — a subclass never inherits a superclass's constructors | Yes (subject to access modifiers — Module 05, Topic 2) |
| Purpose | Bring a *new* object into a valid initial state, exactly once | Define ongoing behavior, callable any number of times |

**A constructor's fundamental job is unique: it's the one piece of code the language *guarantees* runs before anyone else can touch the object at all** — this is precisely what lets Module 05, Topic 2's encapsulation principle be enforced from the very first moment an object exists, not just after-the-fact.

## The Default Constructor — What the Compiler Does Automatically

If you write **no constructor at all**, the compiler automatically generates a **default constructor** — a no-argument constructor that does nothing beyond calling the superclass's no-argument constructor (implicitly, via an invisible `super()` — Module 05, Topic 4):

```java
public class Point {
    private int x;
    private int y;
    // NO constructor written -- the compiler generates:
    //   public Point() { super(); }
}

Point p = new Point();   // works -- uses the COMPILER-GENERATED default constructor
```

**The instant you write ANY constructor yourself, the compiler stops generating the default one entirely:**

```java
public class Point {
    private int x, y;
    public Point(int x, int y) { this.x = x; this.y = y; }
    // NO default constructor exists anymore -- you wrote ONE constructor, so the
    // compiler assumes you're handling construction deliberately
}

Point p = new Point();    // COMPILE ERROR: Point does not have a constructor matching ()
```

**Why does the compiler behave this way?** If you've already written a constructor expressing *your* intended way(s) to construct the object, the compiler assumes you've deliberately decided how construction should work — silently continuing to also offer a no-argument default alongside your custom constructor(s) could let objects be created in states you never intended to allow (e.g., a `Point` with no coordinates at all, when you specifically designed it to always require `x` and `y`).

## Constructor Overloading

Just like regular methods (Module 05, Topic 5), constructors can be **overloaded** — multiple constructors, same class, different parameter lists — giving callers multiple valid ways to construct an object:

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public Point() {          // overloaded constructor -- defaults to the origin
        this(0, 0);              // CONSTRUCTOR CHAINING -- see below
    }
}

Point origin = new Point();       // uses the no-arg constructor
Point custom = new Point(3, 4);    // uses the two-arg constructor
```

## Constructor Chaining with `this(...)`

**`this(...)`**, when used as the **very first statement** inside a constructor, calls **another constructor in the same class** — letting you avoid duplicating initialization logic across multiple overloaded constructors:

```java
public class Employee {
    private String name;
    private double salary;
    private String department;

    public Employee(String name, double salary, String department) {
        this.name = name;
        this.salary = salary;
        this.department = department;
    }

    public Employee(String name, double salary) {
        this(name, salary, "Unassigned");    // CHAINS to the three-arg constructor above
    }

    public Employee(String name) {
        this(name, 0.0);                       // CHAINS to the two-arg constructor,
                                                    // which itself chains to the three-arg one
    }
}
```

```
 new Employee("Aniket")
        │
        ▼  this(name, 0.0)
 Employee(String name, double salary)
        │
        ▼  this(name, salary, "Unassigned")
 Employee(String name, double salary, String department)
        │
        ▼  (the actual field assignments happen here, exactly once)
 name = "Aniket", salary = 0.0, department = "Unassigned"
```

**Why is this valuable?** Without chaining, you'd have to **duplicate** the actual assignment logic in every overloaded constructor — a direct violation of the DRY principle (Module 05, Topic 4), and a real maintenance risk: if the initialization logic ever needs to change (say, adding validation), you'd have to remember to update it in every single overloaded constructor separately, rather than in the one place they all ultimately funnel through.

**Rules:** `this(...)` **must** be the first statement in a constructor (if used at all), and a constructor **cannot** call both `this(...)` and `super(...)` (Module 05, Topic 4) — only one, since both must occupy that same "first statement" position, and every constructor call chain must ultimately bottom out in a `super(...)` call to properly initialize the inherited portion of the object (Topic 5 of this module covers the complete picture, including how `this(...)` and `super(...)` interact with the full object-creation order).

## Why Constructors Can't Be Inherited

A subclass **never** inherits its superclass's constructors — it must define its own (or rely on the compiler-generated default, if applicable) — because a constructor is fundamentally tied to *building exactly that specific class's own state*, not general reusable behavior. What a subclass **can** do is **call** a superclass constructor via `super(...)` (Module 05, Topic 4) as part of building its own, but it cannot simply "inherit" a working constructor the way it inherits an ordinary method.

## Real-World Analogy

Think of a constructor like a **factory assembly line's starting station** — every car (object) coming off this specific line **must** pass through this station first, and it's the *only* station guaranteed to run before the car is handed to anyone. A factory might offer several starting configurations (overloaded constructors — "standard trim," "luxury trim") that share common setup steps, chained together (`this(...)`) so the shared steps are defined exactly once, in one place, rather than duplicated across every trim-level's own separate starting station.

## Advantages

- Guarantees objects are constructed in a valid initial state before any other code can interact with them — foundational to real encapsulation (Module 05, Topic 2).
- Constructor overloading offers callers multiple convenient ways to construct an object.
- `this(...)` chaining eliminates duplicated initialization logic across overloaded constructors.

## Disadvantages / Trade-offs

- Many overloaded constructors (especially with many optional parameters) can become unwieldy — a real, common problem addressed by the "Builder pattern" (a design pattern beyond this course's Core Java scope, but worth knowing exists) and, in modern Java, by Records with default/compact constructors (Module 23).
- Forgetting that writing any constructor removes the compiler-generated default is a common, real source of confusing "no constructor found" compile errors for beginners.

## Best Practices

- Chain overloaded constructors with `this(...)` to keep initialization logic in exactly one place.
- Give every class at least one explicit, deliberate constructor once it has any fields requiring specific initial values — don't rely on the default constructor's implicit no-op behavior for classes with meaningful state.
- Keep constructors focused on initialization — avoid embedding complex business logic inside a constructor; prefer simple, validated assignment (Module 05, Topic 2's encapsulation principle).

## Common Mistakes

- Writing a constructor and then being surprised `new MyClass()` no longer compiles — the default constructor is only generated when **zero** constructors are written at all.
- Attempting to call both `this(...)` and `super(...)` in the same constructor — only one is allowed, and it must be the first statement.
- Duplicating field-assignment logic across multiple overloaded constructors instead of chaining with `this(...)`.

## Interview Questions

1. **Q: What is the default constructor, and when does the compiler generate it?**
   A: A no-argument constructor the compiler automatically generates **only** when a class defines **no constructors at all**; it does nothing beyond implicitly calling the superclass's no-argument constructor. The moment you write any constructor yourself, the compiler stops generating this default entirely.

2. **Q: Why can't a subclass inherit its superclass's constructors?**
   A: A constructor's job is to build exactly its own class's state; it's fundamentally tied to that specific class, not reusable general behavior. A subclass must define its own constructors (or rely on the compiler default), though it can invoke a superclass constructor via `super(...)` as part of its own construction.

3. **Q: What does `this(...)` do inside a constructor, and what are the rules around it?**
   A: It calls another constructor in the **same** class, enabling constructor chaining to avoid duplicating initialization logic. It must be the first statement in the constructor, and a constructor cannot use both `this(...)` and `super(...)` together — only one, since both compete for that same first-statement position.

## Summary

- A **constructor** shares the class's name, has no return type, and runs automatically, exactly once, when an object is created via `new`.
- The compiler generates a **default (no-arg) constructor** only if a class defines **zero** constructors of its own.
- Constructors can be **overloaded**, and **chained** via `this(...)` (must be the first statement) to avoid duplicating shared initialization logic.
- Constructors are never inherited — a subclass defines its own, optionally invoking a superclass constructor via `super(...)`.