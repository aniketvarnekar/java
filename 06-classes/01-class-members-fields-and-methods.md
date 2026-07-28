# Class Members: Fields & Methods

## Learning Objectives

- Structure a class's fields and methods correctly and idiomatically
- Use varargs correctly, and know how they relate to arrays
- Understand, with full certainty, that Java is **always** pass-by-value — including precisely what that means for object references
- Recognize the method-design conventions expected in professional Java code

## Prerequisites

Module 05 (OOP), Module 02 Topic 3 (Runtime Data Areas)

## Motivation

"Is Java pass-by-value or pass-by-reference?" is one of the most-debated, most-often-wrongly-answered questions about Java on the internet. This topic settles it definitively, with a worked memory trace, because getting this wrong leads directly to real, confusing bugs the moment you start writing methods that take object parameters.

## Class Anatomy, Formalized

```java
public class Order {
    // FIELDS (state) -- what the object KNOWS
    private String orderId;
    private double total;
    private List<String> items;

    // CONSTRUCTORS -- how the object is BUILT (Topic 2)
    public Order(String orderId) {
        this.orderId = orderId;
        this.items = new ArrayList<>();
    }

    // METHODS (behavior) -- what the object CAN DO
    public void addItem(String item, double price) {
        items.add(item);
        total += price;
    }

    public double getTotal() {
        return total;
    }
}
```

A class is just fields (state) + constructors (Topic 2) + methods (behavior), all bundled together — exactly Module 05's encapsulation principle, made into concrete syntax. Convention (Module 03, Topic 8) orders these consistently: fields first, then constructors, then methods — a widely followed, though not compiler-enforced, layout that makes any class predictable to scan.

## Method Signatures — Precisely

A method's **signature** consists of its **name** plus its **parameter types, in order** (the return type is *not* part of the signature — you cannot overload two methods that differ *only* in return type):

```java
void process(int x) { }
void process(double x) { }        // OK -- different parameter type, different signature
int process(int x) { }             // COMPILE ERROR -- same signature as the first (return type doesn't count)
```

## Varargs — Variable-Length Argument Lists

```java
int sum(int... numbers) {          // "..." marks numbers as VARARGS
    int total = 0;
    for (int n : numbers) {
        total += n;
    }
    return total;
}

sum();               // legal -- numbers becomes an empty int[]
sum(1);                // numbers becomes int[]{1}
sum(1, 2, 3);           // numbers becomes int[]{1, 2, 3}
sum(new int[]{1, 2});    // you can also pass an actual array directly
```

**What varargs actually is, under the hood:** `int... numbers` is compiled to **exactly** `int[] numbers` — varargs is purely **compile-time syntactic sugar** that lets the *caller* pass a comma-separated list of arguments instead of manually constructing an array first. Inside the method body, `numbers` is a perfectly ordinary array, with no special behavior at all.

**Rules:** a method can have **at most one** varargs parameter, and it **must be the last** parameter in the list:

```java
void log(String tag, int... values) { }    // legal -- varargs is LAST
void log(int... values, String tag) { }     // COMPILE ERROR -- varargs must be last
```

**Why must it be last?** If varargs could appear anywhere, the compiler couldn't unambiguously determine where the variable-length portion ends and any subsequent fixed parameters begin — requiring it to be last removes this ambiguity entirely, by construction.

`System.out.printf(...)` and `String.format(...)` (methods you may have already used) are real, everyday examples of varargs in the standard library.

## Java Is ALWAYS Pass-by-Value — No Exceptions

This is the single most important, and most commonly misunderstood, fact in this entire topic. **Java has exactly one parameter-passing mechanism: pass-by-value.** There is no pass-by-reference in Java, ever, for anything — not for primitives, and not for objects.

**The subtlety that causes all the confusion:** for object-typed parameters, **the "value" being passed is the reference itself (the Heap address) — not the object it points to.** This was previewed in Module 02, Topic 3 — here is the complete, worked-through consequence of that fact.

### Case 1: Primitives — Unambiguous

```java
void increment(int x) {
    x = x + 1;
}

int number = 5;
increment(number);
System.out.println(number);   // still 5 -- completely unaffected
```
`x` is a **copy** of `number`'s value, on the callee's own Stack frame (Module 02, Topic 3). Modifying `x` inside `increment` has zero effect on the caller's original `number`.

### Case 2: Objects — Mutating the Object's State DOES Affect the Caller

```java
class Counter { int count; }

void incrementCounter(Counter c) {
    c.count = c.count + 1;      // mutating the OBJECT this reference points to
}

Counter myCounter = new Counter();
myCounter.count = 5;
incrementCounter(myCounter);
System.out.println(myCounter.count);   // 6 !! -- the caller's object WAS changed
```

**Why does this happen, if Java is pass-by-value?** Because the **value** copied into `c` is the **reference** (Heap address) — both `myCounter` (caller's variable) and `c` (callee's parameter) are **separate variables holding a copy of the same address**, both pointing at the **exact same object** on the Heap. Mutating fields **through** that shared address (`c.count = ...`) is visible to anyone else holding a reference to that same object — including the caller.

```
 Caller's Stack Frame:                 Callee's Stack Frame:                 Heap:

┌────────────────────────────┐        ┌────────────────────────────┐        ┌──────────────────┐
│ myCounter = 0xA1B2 ────────┼────────┼────▶ c = 0xA1B2            ├───────▶│ Counter Object   │
│ (the reference)            │        │ (a COPY of the SAME        │        │ count = 6        │
│                            │        │  reference/address)        │        │                  │
└────────────────────────────┘        └────────────────────────────┘        └──────────────────┘

        BOTH point to the SAME object — mutating fields through EITHER reference
                        is visible through BOTH references.
```

### Case 3: Reassigning the Parameter Does NOT Affect the Caller

This is the case that fully proves it's genuinely pass-by-value, not pass-by-reference:

```java
void reassign(Counter c) {
    c = new Counter();      // REASSIGNS the LOCAL variable 'c' to point at a brand-new object
    c.count = 999;
}

Counter myCounter = new Counter();
myCounter.count = 5;
reassign(myCounter);
System.out.println(myCounter.count);   // STILL 5 !! -- caller's reference is completely untouched
```

```
 Caller's Stack Frame:           Callee's Stack Frame (After Reassignment):          Heap:

┌────────────────────────────┐   ┌──────────────────────────────────┐      ┌──────────────────┐
│ myCounter = 0xA1B2 ────────┼──X│ c = 0xC3D4                       ├─────▶│ NEW Counter      │
│ (UNCHANGED — still points  │   │ (reassigned to a DIFFERENT       │      │ count = 999      │
│  to the ORIGINAL object)   │   │  object)                         │      └──────────────────┘
└─────────────┬──────────────┘   └──────────────────────────────────┘
              │
              ▼
      ┌──────────────────┐
      │ ORIGINAL Counter │
      │ count = 5        │
      └──────────────────┘

       myCounter still points HERE, completely unaffected by
       whatever 'c' was reassigned to inside the method.
```

**If Java were genuinely pass-by-reference** (like C++'s `&` reference parameters, or how some languages handle all variables), reassigning `c` inside the method **would** change what the caller's `myCounter` points to. **It doesn't** — proving conclusively that only a **copy of the reference's value** was passed in, exactly as Java's actual, single parameter-passing rule (pass-by-value, always) predicts.

## The Correct Way to State This (For Interviews and Precision)

> **"Java is always pass-by-value. For primitives, the value copied is the primitive's actual value. For objects, the value copied is the reference (memory address) — never the object itself. This means a method can mutate the state of the object a reference points to (visible to the caller), but can never make the caller's own variable point somewhere else (reassignment inside the method is purely local)."**

Avoid ever saying "Java is pass-by-value for primitives and pass-by-reference for objects" — this is a **common, technically incorrect** simplification that gets the *mechanism* wrong, even though it sometimes gets the *observed behavior* roughly right for the mutation case. The precise, correct framing is: **always pass-by-value; for objects, the "value" happens to be a reference.**

## Real-World Analogy

Think of an object reference like a **house's street address written on a piece of paper**. If I hand you a **copy** of that address (pass-by-value, exactly what Java does for object references), you can absolutely go to that address and **repaint the house** (mutate the object's fields) — everyone with a copy of that same address will see the repainted house. But if you **write a different address on YOUR copy of the paper** (reassign your local parameter to a new object), that does **nothing** to my original piece of paper — I still have the original address written on mine, completely unaffected by what you scribbled on your copy.

## Advantages of Java's (Always) Pass-by-Value Model

- Simple, consistent, and universal — exactly one rule, no special cases to remember per type.
- Predictable: a method can never silently redirect a caller's variable to point somewhere entirely different — only genuine object mutation (a deliberate, visible operation on shared state) can affect the caller.

## Disadvantages / Trade-offs

- The "reference is passed by value" nuance is genuinely subtle and a very common source of confusion for newcomers (and even experienced developers who never had it explained precisely) — precisely why this topic exists.
- Unlike true pass-by-reference languages, Java has no built-in way for a method to reassign a caller's actual variable — if you need "multiple return values" or "output parameters," you must use a wrapper object, an array, or a return value composed of multiple pieces (covered practically in later modules).

## Best Practices

- Never describe Java as "pass-by-reference for objects" — always describe it as pass-by-value, with the value being a reference for object types.
- Be deliberate about methods that mutate parameter objects' state — document this behavior clearly, since it's a real, visible side effect on the caller's data.
- Order class members conventionally: fields, then constructors, then methods — makes any class predictable and fast to scan for future readers.

## Common Mistakes

- Believing reassigning a parameter inside a method changes the caller's variable — it never does, for any type, in Java.
- Believing Java is "pass-by-reference for objects" — the correct, precise description is always pass-by-value, with a reference as the copied value for object types.
- Forgetting varargs must be the last parameter, and that a method can have at most one.

## Interview Questions

1. **Q: Is Java pass-by-value or pass-by-reference?**
   A: Always pass-by-value — there is no pass-by-reference mechanism in Java at all. For object-typed parameters specifically, the "value" being copied is the reference (Heap address) itself, not the object it points to — this is why a method can mutate an object's state (visible to the caller, since both share the same address) but can never reassign the caller's own variable to point elsewhere.

2. **Q: If I pass an object to a method and the method reassigns the parameter to a new object, does the caller see that change?**
   A: No — reassigning a parameter only changes that method's own local copy of the reference, on its own Stack frame. The caller's original variable is completely unaffected, since only a *copy* of the reference's value was passed in.

3. **Q: What is a varargs parameter, and what is it actually compiled to?**
   A: A parameter declared with `Type... name`, letting callers pass a comma-separated list of arguments instead of manually building an array. It compiles to an ordinary `Type[]` array parameter — varargs is purely compile-time syntactic sugar for the caller's convenience.

## Summary

- A class is fields (state) + constructors (Topic 2) + methods (behavior); conventional ordering aids readability.
- A method's **signature** is its name plus parameter types (not its return type) — return type alone cannot distinguish overloads.
- **Varargs** (`Type... name`) compiles to an ordinary array parameter; must be the last parameter, at most one per method.
- **Java is always pass-by-value.** For objects, the copied value is the reference — enabling visible mutation of shared object state, but never allowing a method to reassign the caller's own variable.