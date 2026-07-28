# Encapsulation

## Learning Objectives

- Define encapsulation precisely, beyond "hiding data"
- Use all four access modifiers correctly and explain their exact visibility rules
- Write proper getter/setter methods, and understand *why* they're preferable to public fields
- Recognize genuine encapsulation vs. "fake encapsulation" (getters/setters that expose everything anyway)

## Prerequisites

[01 — Introduction to OOP](01-introduction-to-oop.md)

## Motivation

Encapsulation is usually taught as "make fields private, add getters/setters" — a mechanical rule that misses the actual point. This topic teaches the *reasoning*: encapsulation is about protecting an object's ability to guarantee its own internal consistency, which is a much stronger and more useful idea than just "hiding" data for its own sake.

## Problem Statement

Consider a `BankAccount` class with a public `balance` field:

```java
class BankAccount {
    public double balance;
}

BankAccount acc = new BankAccount();
acc.balance = 1000;
acc.balance = -500;    // ⚠️ NOTHING stops this -- any code, anywhere, can set an invalid balance
```

**The core problem:** with a public field, **any code anywhere in the program** can set `balance` to *anything* — including nonsensical or invalid values (negative balances, for instance) — with zero opportunity for the `BankAccount` class itself to validate, react to, or reject that change. The object has **no control over its own internal consistency**.

## Concept: What Encapsulation Actually Is

> **Encapsulation** is the bundling of data with the methods that operate on it, combined with **restricting direct external access** to that data — so that an object's internal state can only be changed through methods the object itself defines and controls, allowing it to enforce its own rules (**invariants**) about what states are valid.

The key word is **invariant** — a condition that must always hold true for an object to be in a valid state (e.g., "a `BankAccount`'s balance must never be negative"). Encapsulation is fundamentally about **protecting invariants**, not merely "hiding" data as an end in itself.

## Access Modifiers — The Mechanism

Java provides four levels of access control, applied to fields, methods, constructors, and classes:

| Modifier | Same class | Same package | Subclass (different package) | Everywhere |
|---|---|---|---|---|
| `private` | ✅ | ❌ | ❌ | ❌ |
| *(default/package-private — no modifier)* | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

```java
public class BankAccount {
    private double balance;    // ONLY accessible from within BankAccount itself

    public double getBalance() {           // PUBLIC read access, via a controlled method
        return balance;
    }

    public void deposit(double amount) {    // PUBLIC write access, but ONLY via a controlled method
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit must be positive");
        }
        balance += amount;
    }

    public void withdraw(double amount) {
        if (amount <= 0 || amount > balance) {
            throw new IllegalArgumentException("Invalid withdrawal amount");
        }
        balance -= amount;
    }
}
```

Now:
```java
BankAccount acc = new BankAccount();
acc.balance = -500;          // COMPILE ERROR: balance has private access in BankAccount
acc.withdraw(999999);          // throws IllegalArgumentException -- the class ITSELF rejects the invalid operation
```

**This is the entire point:** `balance` can now **only** change through `deposit()` and `withdraw()` — methods that `BankAccount` itself defines, and can therefore make sure never produce an invalid state. The invariant "balance is never negative" is now **enforced by the class itself**, everywhere, always — not something every single piece of calling code has to remember to check independently (and inevitably, somewhere, forget to).

**Why four levels, specifically, and not just "public vs. private"?** Each level answers a genuinely different, common real-world design question:
- `private`: "only this class's own code should ever touch this" — the strongest protection.
- *(package-private, default)*: "classes working closely together in the same package/module can share this, but outside code shouldn't" — useful for internal implementation details shared across a small, cohesive group of related classes.
- `protected`: "subclasses (Topic 4 — Inheritance) need access to build on this, even from a different package, but random unrelated code shouldn't" — a deliberate compromise specifically enabling inheritance-based extension.
- `public`: "this is part of the class's intended, external contract — anyone should be able to use it."

## Getters and Setters — Doing It Properly

```java
public class Person {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be blank");
        }
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("Age out of realistic range");
        }
        this.age = age;
    }
}
```

**The naming convention (`getXxx()`/`setXxx()`/`isXxx()` for booleans) is not arbitrary** — it's part of the **JavaBeans convention**, a widely-adopted Java standard that many frameworks and tools (Spring, Hibernate, JSON libraries like Jackson) rely on via reflection (Module 16) to automatically read/write object properties without you writing manual wiring code. Following this convention isn't just stylistic — it's what makes your classes compatible with a huge swath of the Java ecosystem's tooling "for free."

## "Fake Encapsulation" — A Genuine, Common Anti-Pattern

Simply making a field `private` and adding a trivial getter/setter that does nothing but pass the value straight through **provides zero actual protection** — it's syntactically "encapsulated" but philosophically pointless:

```java
// This is NOT meaningfully encapsulated, despite using private + getter/setter:
public class BankAccount {
    private double balance;
    public double getBalance() { return balance; }
    public void setBalance(double balance) { this.balance = balance; }   // ⚠️ no validation at all!
}

acc.setBalance(-500);    // still perfectly legal! The private field + setter provided ZERO real protection
```

**This is a genuinely important, real distinction:** the *mechanism* of encapsulation (private fields, public methods) is not the same as *actually achieving* encapsulation's *purpose* (protecting invariants). A trivial pass-through setter with no validation logic is "encapsulated" in name only. **Real encapsulation requires the class to actually enforce meaningful rules inside its methods** — as the earlier, corrected `BankAccount` example does with its `deposit`/`withdraw` validation.

> **A further, more advanced design point** (previewed here, expanded once you reach immutable design patterns in later modules): sometimes the *best* encapsulation is having **no setter at all** — if a value should genuinely never change after an object is constructed, don't provide a way to change it, rather than providing a setter "just in case." This connects directly to `final` (Module 03, Topic 7) and will connect again to immutability and Records (Module 23).

## Real-World Analogy

Think of encapsulation like a **vending machine**. You cannot reach in and directly grab a snack or rearrange the machine's internal inventory (its private state) — you can only interact with it through its defined, public interface: insert money, press a button, receive a snack. The machine's internal logic **guarantees** certain invariants on your behalf — it won't dispense a snack without valid payment, it tracks its own inventory correctly — precisely *because* you're never given direct access to reach in and mess with its internals. If vending machines had "public fields" (an unlocked back panel anyone could open), none of those guarantees could be trusted anymore.

## Advantages

- Protects an object's invariants — guarantees about valid state are enforced in exactly one place (the class itself), not scattered across every piece of calling code.
- Allows a class's *internal* implementation to change freely (e.g., switching from storing `balance` as a `double` to a `BigDecimal` for precision — Module 03, Topic 2) without breaking any code that uses the class's public methods — the public *contract* (`getBalance()`, `deposit()`) can remain stable even as internals evolve.
- Enables meaningful validation, logging, or side effects to be centralized inside setters/mutator methods, rather than duplicated everywhere a field might be changed.

## Disadvantages / Trade-offs

- Adds real boilerplate (getter/setter methods) compared to simply exposing public fields — though modern Java features (Records, Module 23) significantly reduce this ceremony for simple data-holding classes.
- "Fake encapsulation" (trivial pass-through getters/setters with zero validation) provides syntax without substance — a real, common anti-pattern worth actively avoiding, not just a theoretical risk.

## Best Practices

- Default to `private` fields; expose access only through deliberately-designed public methods.
- Every setter (or any mutating method) should validate its inputs against the class's actual invariants — if there's nothing meaningful to validate and no reason the value would ever need to change after construction, consider omitting the setter entirely rather than adding one reflexively.
- Follow the JavaBeans `getXxx`/`setXxx`/`isXxx` naming convention for genuine property-style accessors — it's what makes your classes interoperate smoothly with the broader Java ecosystem's tooling.

## Common Mistakes

- Making every field `private` but then adding a trivial, validation-free getter and setter for every single one — mechanically "encapsulated," but providing none of encapsulation's actual benefit.
- Exposing a mutable object (like an `ArrayList`) directly through a getter, unintentionally allowing callers to mutate your object's internal state indirectly, bypassing your class's own validation entirely (a genuinely subtle, real bug — revisited with a concrete example once you reach Modules 07/10).
- Choosing `public` fields "to save time," without considering that it permanently forfeits the ability to ever add validation or change internal representation without breaking every caller.

## Interview Questions

1. **Q: What is encapsulation, precisely — is it just "making fields private"?**
   A: No — encapsulation is bundling data with the behavior that operates on it, combined with restricting direct external access, so the object itself can enforce its own invariants (rules about what states are valid). Making fields `private` is the *mechanism*; actually validating/enforcing meaningful rules inside the class's methods is what achieves encapsulation's real *purpose*. A private field with a trivial, validation-free setter is not meaningfully encapsulated.

2. **Q: What are Java's four access modifiers, and what does each control?**
   A: `private` (accessible only within the same class), package-private/default (same class + same package), `protected` (same package + subclasses in any package), and `public` (accessible everywhere). Each represents a different, deliberate answer to "who should be allowed to see/use this."

3. **Q: Why is it generally better to expose a class's fields through getter/setter methods rather than as public fields directly?**
   A: Public fields give calling code direct, unrestricted read/write access with zero opportunity for the class to validate changes or enforce invariants. Getter/setter methods (with actual logic inside them) let the class control and validate every state change, and let the class's internal representation evolve independently of its public contract, without breaking existing calling code.

## Summary

- **Encapsulation** bundles data with the behavior that operates on it, and restricts direct external access, so an object can enforce its own invariants — rules about what states are valid.
- Java's four access modifiers (`private`, default/package-private, `protected`, `public`) each represent a deliberate answer to "who should have access."
- Getters/setters following the JavaBeans convention provide controlled access — but only provide *real* encapsulation when they contain meaningful validation logic, not as a trivial pass-through.
- Sometimes the strongest encapsulation is providing no setter at all, for values that should never change after construction — a preview of immutability, revisited in later modules.

## Exercises

1. Rewrite this class to properly encapsulate its state, adding meaningful validation: `class Product { public String name; public double price; }` (assume price must never be negative, and name must never be blank).
2. Explain, in your own words, why a `private` field with a trivial, validation-free getter and setter is not meaningfully encapsulated, even though it technically uses `private` + accessor methods.
3. Explain the practical difference between `protected` and package-private (default) access, and give one concrete scenario where you'd deliberately choose `protected` over `private`.

---

**Previous:** [01 — Introduction to OOP](01-introduction-to-oop.md) · **Next:** [03 — Abstraction](03-abstraction.md)
