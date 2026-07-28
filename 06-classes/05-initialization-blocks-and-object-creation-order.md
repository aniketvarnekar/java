# Initialization Blocks & Object Creation Order

## Learning Objectives

- Use instance initializer blocks correctly
- Recite, precisely and completely, the exact order of every initialization step when an object is created
- Trace this full order through an inheritance hierarchy, including the interaction with class-level initialization from Module 02

## Prerequisites

All prior topics in this module; Module 02 Topic 2 (Class Loader Subsystem — class initialization); Module 05 Topic 4 (Inheritance)

## Motivation

This topic is the payoff of everything Module 06 has built so far. Individually, you now understand fields, constructors, `this`, and `static` — but a genuinely deep understanding requires knowing the **exact, complete order** in which all of it actually executes. This is one of the most reliably asked "trace the output" style interview questions in Java, precisely because it requires synthesizing multiple topics correctly at once.

## Instance Initializer Blocks

Beyond the static blocks from Topic 4, Java also has **instance initializer blocks** — a block of code (no `static` keyword) that runs as part of **every single object's construction**, before the constructor body:

```java
class Order {
    private String id;
    private List<String> items;

    {                                    // INSTANCE INITIALIZER BLOCK
        items = new ArrayList<>();         // runs for EVERY object, before the constructor body
        System.out.println("Instance block running");
    }

    Order(String id) {
        this.id = id;
        System.out.println("Constructor running");
    }
}
```

```java
new Order("A1");
```
**Output:**
```
Instance block running
Constructor running
```

**Why would you use this instead of just putting the same code directly in the constructor?** Mainly: (1) shared setup logic that should run **regardless of which overloaded constructor is used** (Topic 2) — putting it in an instance block guarantees it runs no matter which constructor a caller chooses, without needing every constructor to explicitly chain via `this(...)`; (2) it's occasionally used for more complex field initialization requiring multiple statements, similar to why static blocks exist for class-level fields (Topic 4). **In modern, everyday practice, instance blocks are relatively rare** compared to just initializing fields inline or inside a constructor — but they exist, you'll encounter them in real/legacy code, and they matter for the complete ordering picture below.

## The Complete, Precise Object Creation Order

This is the single most important synthesis in this entire module. When `new SomeClass(...)` executes for the **very first time** a class is used (triggering class initialization per Module 02), the **full** order is:

```
═══════════════════ CLASS-LEVEL (happens ONCE per class, ever) ═══════════════════
1. Static fields are given their DEFAULT values (0, null, false, etc.)     [Module 02: Preparation]
2. Static field initializers AND static blocks run, in TEXTUAL ORDER       [Module 02: Initialization]

═══════════════════ OBJECT-LEVEL (happens EVERY time 'new' is called) ═══════════════════
3. Instance fields are given their DEFAULT values (0, null, false, etc.)
4. The constructor's FIRST LINE runs: either an explicit/implicit
   super(...) [-> which recursively repeats steps 3-6 for the SUPERCLASS
   first, all the way up the hierarchy] OR an explicit this(...)
   [-> which jumps to a different constructor in the SAME class instead]
5. Instance field initializers AND instance initializer blocks run,
   in TEXTUAL ORDER (interleaved in the order they appear in the source)
6. The REST of the constructor's body runs (everything after the
   super(...)/this(...) call)
```

**The single most important, most commonly tested fact buried in this order:** **instance field initializers and instance blocks run BEFORE the constructor's own body — but AFTER the superclass's entire construction (steps 3-6) has already fully completed.** This ordering is not arbitrary — it's what guarantees a subclass's constructor body can safely rely on the superclass portion of the object already being fully, validly initialized.

## A Complete Worked Trace

```java
class Animal {
    static { System.out.println("1. Animal static block"); }
    { System.out.println("3. Animal instance block"); }
    Animal() {
        System.out.println("4. Animal constructor");
    }
}

class Dog extends Animal {
    static { System.out.println("2. Dog static block"); }
    { System.out.println("5. Dog instance block"); }
    Dog() {
        System.out.println("6. Dog constructor");
    }
}

public class Test {
    public static void main(String[] args) {
        new Dog();
    }
}
```

**Output:**
```
1. Animal static block
2. Dog static block
3. Animal instance block
4. Animal constructor
5. Dog instance block
6. Dog constructor
```

### Step-by-Step Explanation

1–2. **Class initialization happens first, for the WHOLE hierarchy, superclass before subclass**, and only the very first time either class is actively used (Module 02, Topic 2). `Animal`'s static block runs before `Dog`'s, because `Dog` cannot be initialized until its superclass `Animal` is — this mirrors exactly how `Dog`'s class loading requires `Animal` to already be loaded and initialized first.

3–4. **Object construction begins with `new Dog()`, but `Dog`'s constructor's very first (implicit) action is `super()`** — calling `Animal`'s constructor. This means `Animal`'s **entire** object-level sequence (its own instance blocks, then its own constructor body) runs to completion **before** `Dog`'s object-level sequence even begins. This is why "Animal instance block" and "Animal constructor" both print **before** anything from `Dog`'s object-level initialization.

5–6. **Only after `Animal`'s portion is fully, completely done** does control return to `Dog`'s constructor, which then runs its own instance block, followed by the rest of its own constructor body.

```
new Dog()
   │
   ▼
Dog's constructor called
   │
   ▼  (implicit super() -- ALWAYS the first action, even if not written)
Animal's constructor called
   │
   ▼
Animal's instance block runs              ─┐
   │                                       │  Animal's ENTIRE object-level
   ▼                                       │  sequence completes FIRST,
Animal's constructor body runs             │  before Dog's begins AT ALL
   │                                      ─┘
   ▼  (control returns to Dog's constructor, AFTER super() fully completes)
Dog's instance block runs                 ─┐
   │                                       │  THEN Dog's own object-level
   ▼                                       │  sequence runs
Dog's constructor body (rest) runs         │
                                          ─┘
```

## Why This Order Exists (The "Why")

**Guaranteeing safety and validity, top-down through the hierarchy.** By the time `Dog`'s own instance initialization and constructor body run, the *entire* inherited `Animal` portion of the object is **already guaranteed to be fully, validly constructed**. This means `Dog`'s constructor can safely call inherited methods, read inherited fields, or rely on any invariant `Animal`'s constructor established — with complete confidence that none of it is still in a half-initialized state. If the order were reversed (subclass first, then superclass), a subclass's constructor could potentially observe or interact with an object whose inherited portion **doesn't exist yet** — a genuinely dangerous, inconsistent state that this strict ordering rule exists specifically to prevent.

## Field Initializers vs. Instance Blocks — They're Actually the Same Mechanism

A subtle but important unifying insight: **field initializers (`int x = 5;`) and instance initializer blocks are actually the same underlying mechanism**, and run **interleaved, in the exact textual order they appear** in the source file:

```java
class Example {
    int a = 1;
    { System.out.println("Block 1, a=" + a); }
    int b = 2;
    { System.out.println("Block 2, a=" + a + " b=" + b); }
}
```
**Output when constructed:**
```
Block 1, a=1
Block 2, a=1 b=2
```
This proves field initializers and instance blocks are woven together in **one single sequence, by source order** — not "all field initializers first, then all blocks," or vice versa. The compiler, in effect, gathers every field initializer and every instance block, in the order they textually appear, and copies them **all** into the beginning of **every** constructor (after any `super(...)`/`this(...)` call), which is precisely why they run identically regardless of which overloaded constructor is actually used.

## Real-World Analogy

Think of building a multi-story house: **you cannot start decorating and furnishing the 2nd floor (subclass instance init + constructor) until the entire 1st floor (superclass instance init + constructor) is fully, structurally complete** — the 2nd floor's very existence depends on the 1st floor already being solid and finished. Static blocks are like the **one-time zoning/permit approval process for the entire neighborhood's building design** — it happens once, for the design itself, completely separately from (and before) any individual house on that design actually gets built.

## Advantages

- Guarantees a strict, safe, predictable initialization order across inheritance hierarchies — a subclass constructor can always trust the inherited portion is already fully valid.
- Field initializers and instance blocks running in textual order, shared across every constructor, keeps shared initialization logic consistent regardless of which overloaded constructor is used.

## Disadvantages / Trade-offs

- This precise ordering — while logical once understood — is genuinely one of the more complex, multi-step things to memorize correctly in all of Core Java, and a common source of "trace the output" confusion for learners (and a classic interview trap).
- A subclass constructor **cannot** access `this` in any meaningful way (including calling overridable instance methods safely) during the earlier phases of superclass construction — calling an overridden method from within a superclass constructor is a genuinely dangerous, real anti-pattern, because the subclass's own fields haven't been initialized yet at that point (a nuanced, advanced pitfall worth being aware of, even if full treatment is beyond this topic's scope).

## Best Practices

- Keep constructors and instance blocks simple and focused purely on establishing valid initial state — avoid calling overridable instance methods from within a constructor (or instance block), since the object may not be fully constructed yet when such a call executes.
- When in doubt about initialization order for a specific piece of code, trace it explicitly using the six-step order in this topic, rather than guessing.
- Prefer simple inline field initializers over instance blocks unless you have a genuine multi-statement initialization need — keeps classes easier to read.

## Common Mistakes

- Assuming a subclass's fields/constructor run before the superclass's — it's always the reverse: superclass fully completes first, all the way up the chain, before the subclass's own object-level initialization begins.
- Forgetting that field initializers and instance blocks are interleaved by **textual source order**, not grouped separately.
- Calling an overridable instance method from a constructor, not realizing the subclass's own fields may not be initialized yet if that method is overridden.

## Interview Questions

1. **Q: What is the complete order of execution when an object of a subclass is created for the first time (triggering class initialization too)?**
   A: (1) Superclass static fields defaulted, then static initializers/blocks run in textual order; (2) subclass static fields defaulted, then its static initializers/blocks run in textual order; (3) for the new object: superclass instance fields defaulted, superclass constructor's `super(...)` (if any, repeating this process further up the hierarchy), superclass instance initializers/blocks in textual order, then the rest of the superclass constructor body; (4) only then: subclass instance fields defaulted, subclass instance initializers/blocks in textual order, then the rest of the subclass constructor body.

2. **Q: Do instance initializer blocks run before or after the constructor body?**
   A: Before — they run immediately after any `super(...)`/`this(...)` call at the start of a constructor, and before the remainder of that constructor's own body, in the exact textual order they (and field initializers) appear in the source.

3. **Q: Why does Java guarantee the entire superclass portion of an object is constructed before the subclass's own initialization begins?**
   A: To guarantee safety — by the time a subclass's own instance initialization/constructor logic runs, it can safely assume the inherited superclass state is already fully, validly established, since nothing in the subclass's own code has executed yet that could depend on an incompletely-initialized inherited portion.

## Summary

- **Instance initializer blocks** (`{ ... }`, no `static`) run for every object, interleaved with field initializers in textual source order, immediately after any `super(...)`/`this(...)` call and before the rest of the constructor body.
- The **complete object creation order**: class-level static initialization (superclass before subclass, once ever) → for each `new`: superclass's entire object-level initialization (fields defaulted, `super()` chain, instance blocks/initializers, constructor body) fully completes → **then** the subclass's own object-level initialization runs.
- This strict ordering guarantees a subclass can always safely assume its inherited state is already valid by the time its own initialization logic executes.
- Calling overridable instance methods from within a constructor is a genuine, real pitfall, since the subclass's own fields may not be initialized yet when such a call executes during superclass construction.

## Exercises

1. Without running any code, trace the complete printed output of a three-level hierarchy (`GrandParent` → `Parent` → `Child`, `new Child()`), where each class has both a static block and an instance block printing its own name — write out the full expected order before checking your answer by actually running it.
2. Explain, in your own words, why field initializers and instance blocks running in strict textual order (rather than "all fields first, then all blocks") matters — construct an example where the order would produce genuinely different results if reversed.
3. Explain the danger of calling an overridable instance method from within a superclass constructor, referencing the object creation order established in this topic.

---

**Previous:** [04 — Static Members](04-static-members.md) · **Next:** [06 — Nested & Inner Classes](06-nested-and-inner-classes.md)
