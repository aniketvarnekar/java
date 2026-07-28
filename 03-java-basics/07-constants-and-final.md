# Constants & `final`

## Learning Objectives

- Use `final` correctly on local variables, fields, and (preview) methods/classes
- Understand exactly what `final` guarantees — and what it does NOT guarantee
- Understand compile-time constants and their special treatment by the compiler
- Follow the `SCREAMING_SNAKE_CASE` convention correctly and know why it exists

## Prerequisites

[01 — Variables & Identifiers](01-variables-and-identifiers.md), [02 — Primitive Data Types](02-primitive-data-types.md)

## Motivation

`final` is one of Java's most misunderstood keywords — many developers correctly know "it means you can't change it" but stumble the moment they're asked "does that mean the object itself is immutable?" (it doesn't, and this is a real, common interview trap explored below). Getting `final` precisely right now also sets up Module 08 (Strings) and Module 23 (Records), both of which lean heavily on the concept of immutability.

## Problem Statement

Sometimes you want a variable whose value, once set, must **never change again** — for genuine constants (`MAX_RETRY_COUNT`), for values that should be tamper-proof once initialized (a configuration loaded at startup), or simply to let the compiler catch an accidental reassignment as an error instead of a silent bug.

## Concept: The `final` Keyword

`final` can be applied to variables (local, instance, static), and — previewed here, fully covered in Modules 05–06 — also to methods and classes, with a related but distinct meaning in each case.

### `final` on a Variable

```java
final int MAX_RETRIES = 3;
MAX_RETRIES = 5;   // COMPILE ERROR: cannot assign a value to final variable MAX_RETRIES
```

Once a `final` variable is assigned, any attempt to reassign it is a **compile-time error** — not a runtime check, not a warning, a hard compilation failure.

### `final` Does NOT Mean "Immutable Object" — A Critical Distinction

This is the single most important, most commonly tested nuance of `final`:

> **`final` fixes the *reference* (or primitive value) a variable holds. It does NOT make the *object* that reference points to immutable.**

```java
final List<String> names = new ArrayList<>();
names.add("Aniket");     // PERFECTLY LEGAL -- we're mutating the OBJECT, not reassigning the variable
names.add("Priya");       // also legal

names = new ArrayList<>();  // COMPILE ERROR -- THIS reassigns the variable itself, which final forbids
```

```
           final Reference Variable                         ArrayList Object (Heap)

┌──────────────────────────────────────┐      ┌──────────────────────────────────────┐
│ final List<String> names             │      │ ArrayList                            │
│                                      │      │                                      │
│ names = 0xA1B2 ──────────────────────┼─────▶│ ["Aniket", "Priya"]                  │
│                                      │      │                                      │
│ The reference cannot be changed      │      │ The object remains mutable           │
│ to point to another object.          │      │                                      │
│                                      │      │ names.add("Rahul");      ✓ Allowed   │
│                                      │      │ names.remove("Priya");   ✓ Allowed   │
└──────────────────────────────────────┘      └──────────────────────────────────────┘

                names = new ArrayList<>();      ✗ Not Allowed
```

**Why this distinction exists, precisely:** `final` operates at the level of the **variable binding** (Module 02's Stack-slot-or-field concept) — it locks *what the variable points to*, nothing more. Whether the *object itself* is mutable is an entirely separate property, controlled by that object's own class design (does it expose methods that change its internal state?). A class that's genuinely immutable by design — like `String` (Module 08) — has **no** methods that mutate its own state at all, regardless of whether any particular reference to it is `final`. `final` and "immutable class design" are related but fully independent concepts, and conflating them is a very common, real misconception.

### `final` on Primitives — Genuinely Immutable

For primitives specifically, there's no "object vs. reference" distinction to worry about — a `final` primitive variable's *value* truly can never change, full stop, since a primitive variable directly *is* its value (Topic 2):

```java
final int MAX_SCORE = 100;   // this value can NEVER change, ever -- no nuance here, unlike objects
```

### `final` Local Variables

```java
void process(final int retries) {   // parameters can be final too
    // retries = retries + 1;   // would be a COMPILE ERROR
}
```

**Why mark a local variable/parameter `final` if you weren't going to reassign it anyway?** Two real reasons: (1) it's a **compiler-enforced documentation** signal — "this value is intentionally fixed for the rest of this method, trust that as you read the code below" — genuinely useful in longer methods; (2) historically (pre-Java 8), local variables captured by an anonymous inner class or lambda were **required** to be `final` (or "effectively final" — see below) — a rule you'll fully understand once you reach Module 06 (Classes) and Module 17 (Functional Programming).

> **"Effectively final" (Java 8+):** a local variable that is *never* reassigned after its initial assignment is treated by the compiler as **"effectively final"**, even without the explicit `final` keyword — and can still be captured by a lambda/anonymous class exactly as if it had been explicitly marked `final`. This is a real, modern Java relaxation of the older, stricter, explicit-`final`-only rule — full context in Module 17.

### `final` on Fields — Must Be Assigned Exactly Once

```java
class Account {
    final String accountNumber;   // no initializer here -- MUST be assigned in every constructor

    Account(String accountNumber) {
        this.accountNumber = accountNumber;   // assigned exactly once, here
    }
}
```

A `final` instance field can be assigned either at declaration, or in **every** constructor (but not both, and not conditionally-sometimes in a constructor) — the compiler performs the same definite-assignment analysis introduced in Topic 1, extended to guarantee a `final` field is assigned **exactly once**, on every possible construction path, before the constructor finishes. (Full depth on constructors: Module 06.)

## Compile-Time Constants — A Special Case

A `final` variable that is (a) a primitive or `String`, and (b) initialized with a value the compiler can fully compute **at compile time** (a literal, or an expression made only of other compile-time constants), is treated specially by the compiler as a **compile-time constant**:

```java
final int MAX_RETRIES = 3;               // a genuine compile-time constant
final String APP_NAME = "InventoryPro";    // also a compile-time constant
```

**What makes this special?** The compiler **inlines** the literal value directly at every usage site, in the bytecode itself, rather than generating a runtime field lookup. This is exactly what makes `switch` statements on `int`/`String` constants work (Module 04), and is why, back in Module 02's class-loading discussion, merely referencing a `static final` compile-time constant of another class does **not** trigger that other class's initialization — the compiler already baked the actual value directly into your bytecode at compile time, with no runtime dependency on the other class at all.

```java
class Config {
    static final int VERSION = 5;   // compile-time constant
}

class Demo {
    public static void main(String[] args) {
        System.out.println(Config.VERSION);   // does NOT trigger Config's class initialization --
                                                  // javac already inlined the literal '5' right here
    }
}
```

## Naming Convention: `SCREAMING_SNAKE_CASE`

Recall from Topic 1: the convention for constants (`static final` fields specifically) is **ALL_CAPS with underscores between words**:

```java
public static final int MAX_CONNECTIONS = 100;
public static final String DEFAULT_ENCODING = "UTF-8";
```

**Why specifically `static final`, and why this casing?** `static` ensures there's exactly one shared copy (Module 02's Method Area), appropriate for a value that's conceptually "the same constant everywhere," not per-object. `final` prevents it from ever being reassigned. The `SCREAMING_SNAKE_CASE` casing convention lets any reader instantly distinguish "this is a fixed constant" from an ordinary variable or field at a glance, with zero other context needed — directly echoing the naming-convention rationale from Topic 1.

## Real-World Analogy

Think of a `final` reference variable like a **permanent nameplate glued onto one specific mailbox** — the nameplate itself (which mailbox it's glued to) can never be moved to a different mailbox. But `final` says absolutely nothing about what's allowed to happen **inside** that mailbox — mail can still be added, removed, or rearranged inside it freely, unless the mailbox itself was separately designed (like `String`) to never let its contents change at all.

## Advantages

- Compiler-enforced immutability for variable bindings catches accidental reassignment bugs at compile time, for free.
- Compile-time constants enable real bytecode-level optimizations (inlining) and features like `switch` on constants.
- The `SCREAMING_SNAKE_CASE` convention creates instant visual clarity about what's a fixed constant across the entire Java ecosystem.

## Disadvantages / Trade-offs

- `final` is frequently, incorrectly assumed to guarantee full object immutability — a real, common source of confusion and occasionally real bugs (assuming a `final` collection field can't be mutated, then being surprised when it is).
- Overusing `final` on every single local variable, out of misplaced caution, can add visual noise without proportional benefit — reserve it for genuinely meaningful cases (true constants, or documenting deliberate intent in longer methods).

## Best Practices

- Use `final` for genuine constants (`SCREAMING_SNAKE_CASE`, `static final`) always.
- Use `final` on fields whenever a field's value is genuinely fixed after construction — this is a strong, compiler-enforced signal of your class's design intent (revisited heavily once you reach immutable object design in Module 06 and Records in Module 23).
- Never assume `final` alone makes a field's referenced object immutable — check whether that object's own class exposes any mutating methods.

## Common Mistakes

- Believing `final List<String> list = new ArrayList<>();` means `list` can never be modified — it means `list` can never be **reassigned**; its contents remain fully mutable.
- Forgetting that a `final` instance field must be definitely assigned exactly once on every constructor path — and being confused by the resulting compiler error if a constructor has a code path that skips assigning it.
- Assuming any `final static` field is automatically a compile-time constant eligible for inlining — this specifically requires a primitive or `String` type, initialized with a compile-time-computable expression; a `final static List<String> NAMES = new ArrayList<>();`, for example, is **not** a compile-time constant (it requires actual runtime object construction), even though it's `final` and `static`.

## Interview Questions

1. **Q: Does `final` make an object immutable?**
   A: No — `final` only fixes the variable's *binding* (which object/value it refers to), preventing reassignment. It says nothing about whether the referenced object's internal state can still be mutated through its own methods; that depends entirely on the object's own class design.

2. **Q: What is a compile-time constant in Java, and why does it matter for class initialization?**
   A: A `final` primitive or `String` field initialized with a value fully computable at compile time (a literal or expression of other compile-time constants). The compiler inlines its literal value directly at every usage site in bytecode, meaning merely referencing it elsewhere does not require the defining class to be initialized at runtime at all — the value was already baked in at compile time.

3. **Q: Can a `final` instance field be assigned inside a constructor, or must it be assigned at declaration?**
   A: It can be assigned either at declaration, or within every constructor (but exactly once per construction path, not both/either interchangeably per-instance) — the compiler enforces definite, single assignment via the same analysis used for local variables, extended to guarantee it happens before construction completes.

## Summary

- `final` on a variable prevents **reassignment** of that variable's binding — it does not make the referenced object immutable; those are independent concepts.
- `final` primitives are genuinely, fully immutable values, since a primitive variable *is* its value directly.
- `final` fields must be definitely assigned exactly once, either at declaration or in every constructor.
- A `final` primitive/`String` initialized with a compile-time-computable expression is a **compile-time constant**, inlined directly into bytecode at every usage site — this is what enables `switch` on constants and avoids triggering unnecessary class initialization.
- Constants follow the `SCREAMING_SNAKE_CASE` naming convention, typically as `public static final` fields.

## Exercises

1. Explain, precisely, why `final int[] scores = {1, 2, 3}; scores[0] = 99;` compiles and runs successfully, while `scores = new int[]{4, 5, 6};` on the next line does not compile.
2. Write a class with a `final` instance field assigned inside its constructor, and explain what compiler error you'd get if you forgot to assign it on one constructor code path (assume the class has two constructors).
3. Explain why `public static final int MAX = 100;` is a compile-time constant, but `public static final List<Integer> DEFAULTS = new ArrayList<>();` is not, even though both are `public static final`.

---

**Previous:** [06 — Wrapper Classes & Autoboxing](06-wrapper-classes-and-autoboxing.md) · **Next:** [08 — Comments & Code Style](08-comments-and-code-style.md)
