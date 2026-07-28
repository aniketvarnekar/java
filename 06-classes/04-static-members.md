# Static Members

## Learning Objectives

- Use static fields and methods correctly
- Understand precisely when `static` is (and isn't) the right design choice
- Understand static blocks and their relationship to class initialization (Module 02)
- Recognize the singleton pattern as a common, legitimate use of static members

## Prerequisites

[03 — The `this` Keyword](03-the-this-keyword.md), Module 02 Topic 2 (Class Loader Subsystem)

## Motivation

`static` has appeared constantly since `HelloWorld.java`'s `public static void main` — this topic finally gives it a complete, dedicated treatment: what it means for fields, methods, and blocks, and — just as important — the judgment for when reaching for `static` is appropriate versus when it undermines good object-oriented design (Module 05).

## Recap: What `static` Means, Precisely

From Module 02 (Topic 3, Runtime Data Areas) and Module 03 (Topic 1): a `static` member belongs to the **class itself**, not to any individual object — there is exactly **one** copy, shared by every instance, living in the **Method Area/Metaspace**, not the Heap alongside per-object instance fields.

```java
class Counter {
    static int totalCreated = 0;    // ONE shared copy, in the Method Area
    int id;                            // a per-INSTANCE copy, on the Heap, one per object

    Counter() {
        totalCreated++;                // every Counter object shares and updates the SAME totalCreated
        id = totalCreated;
    }
}

Counter c1 = new Counter();   // c1.id = 1, totalCreated = 1
Counter c2 = new Counter();   // c2.id = 2, totalCreated = 2
Counter c3 = new Counter();   // c3.id = 3, totalCreated = 3

System.out.println(Counter.totalCreated);   // 3 -- accessed via the CLASS, not any specific instance
```

```
        Method Area (Shared - One Per Class)               Heap (One Allocation Per Object)

┌──────────────────────────────────────┐      ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Counter.totalCreated = 3             │      │ c1           │  │ c2           │  │ c3           │
│ (Single static variable)             │      │ id = 1       │  │ id = 2       │  │ id = 3       │
└──────────────────────────────────────┘      └──────────────┘  └──────────────┘  └──────────────┘
                 ▲                                     │                │                │
                 └────────────── Shared by all THREE objects ───────────┴────────────────┴──────────┘
```

## Accessing Static Members

**Convention (and best practice): always access a `static` member through the class name**, not through an instance reference — even though Java technically permits the latter:

```java
Counter.totalCreated;    // CORRECT, idiomatic -- makes it immediately clear this is shared, not per-object
c1.totalCreated;          // LEGAL, but discouraged -- looks like an instance field, misleadingly
```

**Why is accessing via an instance discouraged, even though it compiles?** It visually disguises a shared, class-level value as if it were per-object state, misleading anyone reading the code about what's actually happening — a real, if minor, readability/correctness-signaling concern most style guides and linters flag.

## Static Methods

```java
class MathUtils {
    static int square(int n) {
        return n * n;
    }
}

int result = MathUtils.square(5);   // called on the CLASS, no MathUtils object ever created
```

**Recall from Topic 3: static methods have no `this`** — they cannot directly access instance fields or call instance methods (there's no specific object for those to refer to), and cannot be overridden polymorphically the way instance methods can (Module 05, Topic 5) — a static method invocation is resolved at **compile time**, based on the reference's declared type, exactly like overloading (Module 05, Topic 5), not at runtime like true polymorphic dispatch.

```java
class Parent {
    static void greet() { System.out.println("Parent"); }
}
class Child extends Parent {
    static void greet() { System.out.println("Child"); }   // this HIDES, not OVERRIDES, Parent's greet()
}

Parent p = new Child();
p.greet();    // prints "Parent" !! -- resolved by p's DECLARED type (Parent), NOT its actual runtime type
              // (contrast this directly with Module 05, Topic 5's INSTANCE method example, where the
              //  ACTUAL runtime type determined the result -- static methods behave completely differently)
```

This is a genuinely important, precise interview distinction: static methods are **hidden**, not **overridden** — because overriding is fundamentally a runtime-polymorphism concept (Module 05, Topic 5), and static methods, having no `this` and no dynamic dispatch, simply don't participate in that mechanism at all.

## When to Use `static` — And When Not To

**Good uses of `static`:**
- **Utility/helper methods with no dependency on any object's state** — `Math.sqrt(x)`, `MathUtils.square(n)` above. These are stateless, pure functions that don't conceptually "belong" to any specific object.
- **Constants** (Module 03, Topic 7) — `public static final int MAX_RETRIES = 3;` — genuinely shared across the entire class, not per-object.
- **Counters/shared state that's genuinely meant to be tracked across ALL instances collectively** — like `totalCreated` above.
- **Factory methods** — `static` methods that construct and return instances of their own class (a common, legitimate pattern, e.g., `LocalDate.of(2024, 1, 15)` from the standard library).

**Poor uses of `static` (a real, common beginner over-application):**
```java
class BankAccount {
    static double balance;   // ⚠️ WRONG -- this makes EVERY BankAccount object SHARE the SAME balance!
}

BankAccount acc1 = new BankAccount();
BankAccount acc2 = new BankAccount();
acc1.balance = 1000;
System.out.println(acc2.balance);   // 1000 !! -- acc2's balance changed too, since there's only ONE
                                       // shared 'balance', not one per account -- almost certainly NOT
                                       // what was intended for a per-account balance
```
**This is a genuinely common, real beginner mistake:** reaching for `static` out of confusion about scope, when the field was actually meant to be **per-object** (an ordinary instance field). The test to apply: **"should every single instance of this class share and see the exact same value for this field, or does each instance need its own independent value?"** If the latter, it must be a regular instance field, never `static`.

## Static Initialization Blocks

Recall Module 02, Topic 2: class initialization runs static initializer blocks. Here's the full, dedicated syntax:

```java
class Configuration {
    static final Map<String, String> DEFAULTS;

    static {                                    // STATIC INITIALIZATION BLOCK
        DEFAULTS = new HashMap<>();               // runs ONCE, when the class is initialized
        DEFAULTS.put("timeout", "30s");            // (recall Module 02: triggered by "active use" --
        DEFAULTS.put("retries", "3");                //  NOT necessarily at program startup)
    }
}
```

**Why use a static block instead of a simple inline initializer** (`static Map<String, String> DEFAULTS = new HashMap<>();`)**?** A static block is necessary when initialization requires **multiple statements** or **logic** (loops, conditionals, exception handling) that a single inline expression can't express — inline initializers are limited to a single expression per field, while a static block can contain arbitrary code.

**Multiple static blocks in one class run in the textual order they appear** — exactly like the single-statement inline initializers they're an extension of.

## The Singleton Pattern — A Legitimate, Common Use of `static`

A design pattern (a well-known, reusable solution to a common design problem) ensuring **exactly one instance** of a class ever exists across the entire program, using `static` deliberately:

```java
class DatabaseConnection {
    private static DatabaseConnection instance;    // the ONE shared instance, held statically

    private DatabaseConnection() { }                 // PRIVATE constructor -- prevents 'new' from OUTSIDE

    public static DatabaseConnection getInstance() {
        if (instance == null) {
            instance = new DatabaseConnection();
        }
        return instance;
    }
}

DatabaseConnection db1 = DatabaseConnection.getInstance();
DatabaseConnection db2 = DatabaseConnection.getInstance();
System.out.println(db1 == db2);   // true -- BOTH refer to the exact same, single object
```

**Why is `instance` `static`?** Precisely because the entire point is "exactly one, shared across everyone" — the same reasoning as `totalCreated` above, but taken to its logical conclusion of "only ever one total, period." The `private` constructor is what actually **enforces** this — since outside code can never call `new DatabaseConnection()` directly, `getInstance()` is the **only** path to obtaining a reference, guaranteeing there's never more than one.

> **A brief, honest caveat, worth knowing even at this stage:** the Singleton pattern, while legitimate and historically extremely common, is also somewhat controversial in modern software design — it introduces global, shared mutable state (a real testing and coupling concern) and has been partly superseded by dependency injection frameworks (like Spring) that manage single-instance ("singleton-scoped") objects more flexibly, without this manual pattern. It's worth knowing deeply because you'll see it constantly in existing code and interviews, while also being aware modern practice often prefers framework-managed alternatives.

## Real-World Analogy

Think of a `static` field like a **shared family whiteboard in a kitchen** — every family member (object instance) reads from and writes to the *same* whiteboard; if one person updates it, everyone sees the update immediately, because there's only ever one whiteboard, not one per family member. An **instance** field, by contrast, is like each family member's own **personal notebook** — everyone has their own, and writing in your notebook has zero effect on anyone else's.

## Advantages

- Enables genuinely shared state/utility logic without needing an unnecessary object instance.
- Static factory methods and the Singleton pattern provide controlled, deliberate ways to manage instance creation.
- Constants (`static final`) are the correct, idiomatic way to represent values genuinely shared across a whole class (Module 03, Topic 7).

## Disadvantages / Trade-offs

- Overusing `static` for fields that should genuinely be per-instance is a real, common bug source (the `BankAccount.balance` mistake above).
- Static methods can't be tested via polymorphic substitution (mocking) as easily as instance methods, since they're not part of any object's dynamically-dispatched interface — a real, practical concern in unit testing (relevant once you explore testing practices beyond this Core Java course's scope).
- Excessive reliance on static/global state (as in an overused Singleton) can make code harder to test and reason about in isolation.

## Best Practices

- Apply the test: "should every instance share this exact value, or does each need its own?" — if every instance, `static`; if each its own, an instance field.
- Access static members via the class name, never through an instance reference.
- Use static blocks specifically when initialization logic requires more than a single expression.

## Common Mistakes

- Making a field `static` that was actually meant to be per-object state (the classic `BankAccount.balance` mistake).
- Assuming static methods can be overridden polymorphically — they're hidden, not overridden, and resolved by the reference's declared type, not the object's actual runtime type.
- Attempting to access instance fields/methods directly from within a static method — impossible, since there's no `this`.

## Interview Questions

1. **Q: What's the difference between a static field and an instance field?**
   A: A static field has exactly one shared copy for the entire class, living in the Method Area — every instance sees and shares the same value. An instance field has its own independent copy per object, living on the Heap as part of that specific object.

2. **Q: Can static methods be overridden?**
   A: No — they can only be **hidden**. Unlike instance methods (which use runtime dynamic dispatch based on the object's actual type — Module 05, Topic 5), a static method call is resolved at compile time based on the reference's *declared* type, since static methods have no `this` and don't participate in polymorphic dispatch at all.

3. **Q: How does the classic Singleton pattern use `static` to guarantee only one instance ever exists?**
   A: It stores the single instance in a `private static` field, and makes the constructor `private` so no outside code can call `new` directly — the only way to obtain a reference is through a `public static getInstance()` method, which creates the instance once (if it doesn't already exist) and always returns that same shared reference thereafter.

## Summary

- A **static member** belongs to the class itself — one shared copy, in the Method Area, not per-object on the Heap.
- **Static methods** have no `this`, cannot access instance state directly, and are **hidden** (not overridden) by subclasses — resolved at compile time by declared type, unlike instance method dispatch.
- Use `static` for utility methods, genuine constants, and state deliberately meant to be shared across every instance — never for what should be per-object state.
- **Static initialization blocks** handle multi-statement class-level initialization logic, running once at class initialization (Module 02).
- The **Singleton pattern** is a legitimate, common use of `static` + a `private` constructor to guarantee exactly one instance exists, though modern frameworks often provide more flexible alternatives.

## Exercises

1. Identify and fix the bug in this class, explaining precisely why the original is wrong: `class ShoppingCart { static List<String> items = new ArrayList<>(); void addItem(String item) { items.add(item); } }` (assume each cart should have its own independent items).
2. Write a static utility class `StringUtils` with a static method `isBlank(String s)`, and explain why making this method `static` (rather than an instance method requiring a `StringUtils` object) is the correct design choice.
3. Predict the output of the `Parent`/`Child` static method hiding example in this topic, and explain precisely why it differs from what Module 05, Topic 5's instance-method polymorphism example would produce with the same structure.
4. Implement a simple Singleton `Logger` class with a `private` constructor and a `static getInstance()` method, and explain what would go wrong (in terms of the guarantee it provides) if the constructor were `public` instead.

---

**Previous:** [03 — The `this` Keyword](03-the-this-keyword.md) · **Next:** [05 — Initialization Blocks & Object Creation Order](05-initialization-blocks-and-object-creation-order.md)
