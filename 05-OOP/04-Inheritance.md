# Inheritance

## Learning Objectives

- Use `extends` and `super` correctly
- Understand the IS-A relationship and how to recognize when inheritance is (and isn't) appropriate
- Understand exactly why Java restricts classes to single inheritance, mechanically, not just as a rule
- Understand constructor chaining across an inheritance hierarchy at a preview level

## Prerequisites

[01 — Introduction to OOP](01-Introduction-To-OOP.md)

## Motivation

Inheritance is the pillar most associated with "code reuse" in casual descriptions of OOP — but overusing it is one of the most common real design mistakes newer developers make (a theme Topic 7 addresses directly with "favor composition over inheritance"). This topic teaches both the mechanics and the judgment for when inheritance is genuinely the right tool.

## Problem Statement

Suppose you're modeling employees at a company: `Manager`, `Engineer`, and `Intern` all share common data (`name`, `salary`, `employeeId`) and common behavior (`calculatePay()`), but each also has type-specific behavior (`Manager` approves budgets, `Engineer` writes code). Without inheritance, you'd duplicate the shared fields/methods across every one of these classes independently — a direct violation of good software design (DRY — "Don't Repeat Yourself"), and a maintenance nightmare if a shared field/behavior ever needs to change.

## Concept: Inheritance and the IS-A Relationship

> **Inheritance** lets one class (the **subclass**/**child class**) acquire the fields and methods of another class (the **superclass**/**parent class**), while adding its own additional fields/methods or overriding inherited behavior (Topic 5).

Inheritance models an **IS-A relationship**: "a `Manager` **IS-A** `Employee`," "a `Dog` **IS-A** `Animal`." This is the litmus test for whether inheritance is the *conceptually correct* tool — if you can't honestly say "X IS-A Y" in plain English, inheritance is probably the wrong relationship (Topic 7 covers the alternative, HAS-A, in depth).

```java
class Employee {
    protected String name;
    protected double baseSalary;

    Employee(String name, double baseSalary) {
        this.name = name;
        this.baseSalary = baseSalary;
    }

    double calculatePay() {
        return baseSalary;
    }
}

class Manager extends Employee {          // Manager IS-A Employee
    private double bonus;

    Manager(String name, double baseSalary, double bonus) {
        super(name, baseSalary);            // calls the Employee (superclass) constructor
        this.bonus = bonus;
    }

    @Override
    double calculatePay() {
        return super.calculatePay() + bonus;  // reuse the parent's logic, then extend it
    }
}
```

```java
Manager m = new Manager("Priya", 80000, 15000);
System.out.println(m.calculatePay());   // 95000 -- combines inherited AND Manager-specific logic
System.out.println(m.name);              // "Priya" -- 'name' is INHERITED from Employee, not redefined
```

## `extends` and `super` — The Mechanics

- **`extends`** declares the inheritance relationship: `class Manager extends Employee` means `Manager` is a subclass of `Employee`.
- **`super`** (used two ways):
  1. **`super(...)`** — as the first statement of a subclass constructor, explicitly calls the superclass's constructor, ensuring the inherited part of the object is properly initialized before the subclass adds its own initialization. (If omitted, Java automatically inserts an implicit `super()` call to the superclass's no-argument constructor — full constructor-chaining depth: Module 06.)
  2. **`super.methodName(...)`** — calls the superclass's version of a method, even when the subclass has overridden it (as shown in `calculatePay()` above) — lets a subclass **extend**, rather than completely replace, inherited behavior.

## What Gets Inherited

A subclass inherits its superclass's **non-private** fields and methods automatically. `private` members are **not directly accessible** by name in the subclass (consistent with Topic 2's access-modifier rules) — this is precisely *why* `protected` exists: it's specifically designed to expose members to subclasses (even in different packages) while still restricting general public access.

```
                            Employee (Superclass)

                  ┌──────────────────────────────────┐
                  │ Fields                           │
                  │ • protected name                 │
                  │ • protected baseSalary           │
                  │                                  │
                  │ Methods                          │
                  │ • calculatePay()                 │
                  └─────────────────┬────────────────┘
                                    │
                                    │ extends
                                    ▼
                           Manager (Subclass)

                  ┌──────────────────────────────────┐
                  │ Inherited                        │
                  │ • name                           │
                  │ • baseSalary                     │
                  │ • calculatePay()                 │
                  │                                  │
                  │ Additional Members               │
                  │ • private bonus                  │
                  │ • overridden calculatePay()      │
                  └──────────────────────────────────┘
```

## Every Class Ultimately Extends `Object`

A crucial, universal fact previewed here (full depth: Module 07): **every class in Java, whether you write `extends` explicitly or not, ultimately inherits from `java.lang.Object`**. If you don't write `extends SomethingElse`, the compiler implicitly treats your class as `extends Object`. This is *why* every single Java object — regardless of what class it is — already has methods like `toString()`, `equals()`, and `hashCode()` available, without you ever defining them: they're inherited from `Object`, the root of Java's entire class hierarchy.

```
                            Object   (the ultimate root of EVERY class hierarchy)
                              │
              ┌───────────────┼───────────────┐
              ▼                               ▼
          Employee                            (any other class you write)
              │
              ▼
          Manager
```

## Why Java Restricts Classes to Single Inheritance

This is one of the most commonly asked "why" questions in Java, with a precise, mechanical answer — not just an arbitrary rule:

**The Diamond Problem.** Imagine Java allowed a class to extend *two* classes:

```
        ClassA                    ClassB
          │  has method doWork()     │  has method doWork()  (a DIFFERENT implementation)
          └───────────┬──────────────┘
                      ▼
                   ClassC  extends ClassA, ClassB   (HYPOTHETICAL -- not legal Java)
```

If `ClassC` calls `doWork()`, and it didn't override it itself — **which** `doWork()` should run: `ClassA`'s version, or `ClassB`'s version? Both are equally "inherited." This genuine ambiguity is called the **Diamond Problem**, and different languages that *do* allow multiple class inheritance (like C++) resolve it with complex, often confusing rules (explicit scope resolution, virtual inheritance, etc.).

**Java's designers made a deliberate simplicity/safety trade-off (echoing Module 01, Topic 3's "Simple" feature discussion): disallow multiple class inheritance entirely, eliminating the Diamond Problem's ambiguity by construction, rather than adding complex disambiguation rules to resolve it.**

```java
class ClassC extends ClassA, ClassB { }   // COMPILE ERROR: 'class' can only extend ONE class
```

**But then how does Java let you compose behavior from multiple sources?** This is exactly what **interfaces** (Topic 6) solve — a class **can** implement multiple interfaces, and modern Java interfaces can even provide default method implementations. The Diamond Problem *can* theoretically arise with multiple interfaces too (if two interfaces provide conflicting default method implementations) — but Java handles this case with a precise, much simpler compiler rule (forcing the implementing class to explicitly resolve the conflict), covered fully in Topic 6, rather than the deep ambiguity multiple *class* inheritance would create for **state** (fields), not just behavior (methods) — the truly hard part of the Diamond Problem that interfaces mostly sidestep by (traditionally) not holding instance state at all.

## Inheritance Is Not Always the Right Tool

A single class can only `extends` **one** superclass — this is a genuine, permanent design constraint. Combined with the IS-A litmus test from earlier, this leads directly to a critical design principle, explored fully in Topic 7: **inheritance should be reserved for genuine IS-A relationships, and even then, used judiciously** — because once you commit a class to extending a particular superclass, you've used up its one available "class inheritance slot," permanently shaping its position in the hierarchy.

```java
// BAD inheritance -- Stack IS NOT genuinely "a kind of" ArrayList in any meaningful conceptual sense,
// even though this compiles and technically "works" by reusing ArrayList's methods:
class Stack<T> extends ArrayList<T> {
    void push(T item) { add(item); }
    T pop() { return remove(size() - 1); }
    // PROBLEM: Stack now ALSO inherits ArrayList's add(int, T), get(int), remove(int), etc. --
    // methods that let callers violate a Stack's fundamental LIFO discipline entirely!
}
```
This is a real, classic, historically-cited example of inheritance misuse — full discussion of the better alternative (composition) is Topic 7.

## Real-World Analogy

Think of inheritance like **biological taxonomy**: a `Dog` IS-A `Mammal`, which IS-A `Animal`. A dog inherits general mammalian traits (warm-blooded, has fur) without each species needing to redefine "warm-blooded" from scratch — but a dog is also *born from exactly one direct parent species lineage*, not spliced together from two unrelated animal families at once (the biological equivalent of Java's single-class-inheritance restriction) — even though a dog can simultaneously belong to *multiple, independent classifications* (a "domesticated animal," a "quadruped," a "pet") the way a Java class can implement multiple unrelated interfaces at once.

## Advantages

- Genuine, compiler-enforced code reuse for true IS-A relationships, reducing duplication.
- `super` lets a subclass extend (not just replace) inherited behavior, keeping shared logic in one place.
- Single inheritance's simplicity eliminates the Diamond Problem for state/fields entirely, by construction.

## Disadvantages / Trade-offs

- Single inheritance means a class's one "extends slot" is a scarce, permanent resource — using it for a non-genuine IS-A relationship (like the `Stack extends ArrayList` example) forecloses genuinely using it for a better-fitting relationship later, and leaks unwanted inherited behavior.
- Deep inheritance hierarchies (many layers of subclassing) can become genuinely hard to reason about — you must mentally trace up the whole chain to understand a class's full behavior; a recurring, real critique of inheritance-heavy designs, directly motivating Topic 7's composition-based alternative.

## Best Practices

- Apply the IS-A test honestly before using inheritance — "is X genuinely a specialized kind of Y?" If the honest answer is "not really, I just want to reuse some of Y's code," reach for composition (Topic 7) instead.
- Prefer shallow inheritance hierarchies (1-2 levels) over deep ones wherever reasonably possible — easier to reason about, easier to change later.
- Use `protected` deliberately for members subclasses genuinely need, rather than defaulting everything to `protected` "just in case."

## Common Mistakes

- Using inheritance purely for code reuse, without a genuine IS-A relationship — the classic `Stack extends ArrayList` mistake shown above.
- Forgetting that `private` superclass members are not directly inherited/accessible by name in a subclass — a subclass interacts with them only indirectly, through inherited public/protected methods.
- Assuming Java allows multiple class inheritance in any form — it never does; only multiple *interface* implementation is allowed (Topic 6).

## Interview Questions

1. **Q: Why doesn't Java allow a class to extend multiple classes?**
   A: To avoid the Diamond Problem — genuine ambiguity about which superclass's implementation (and especially which superclass's *state/fields*) should apply when two parent classes provide conflicting definitions. Java's designers deliberately chose single-class-inheritance simplicity over the complex disambiguation rules other languages (like C++) use to resolve this ambiguity.

2. **Q: What is the IS-A relationship, and why does it matter for deciding whether to use inheritance?**
   A: IS-A means "this subclass is genuinely a specialized kind of its superclass" (e.g., `Manager IS-A Employee`). It's the litmus test for whether inheritance is conceptually appropriate — using inheritance purely to reuse code, without a genuine IS-A relationship, leads to classes inheriting unwanted/inappropriate behavior (the classic `Stack extends ArrayList` anti-pattern).

3. **Q: What does every Java class inherit from, even without an explicit `extends` clause?**
   A: `java.lang.Object` — every class's hierarchy ultimately roots at `Object`, whether written explicitly or implied by the compiler. This is why every Java object has `toString()`, `equals()`, and `hashCode()` available by default.

4. **Q: What does `super(...)` do when called as the first line of a subclass constructor?**
   A: It explicitly invokes the superclass's constructor, ensuring the inherited portion of the object is properly initialized before the subclass's own constructor logic runs. If omitted, Java implicitly inserts a call to the superclass's no-argument constructor.

## Summary

- **Inheritance** (`extends`) lets a subclass acquire a superclass's fields/methods, modeling a genuine **IS-A relationship**.
- `super(...)` calls the superclass constructor; `super.method(...)` calls the superclass's version of an overridden method, letting a subclass extend rather than replace inherited behavior.
- Every Java class ultimately inherits from `java.lang.Object`, whether stated explicitly or not.
- Java restricts classes to **single inheritance** specifically to avoid the Diamond Problem's genuine ambiguity — a deliberate simplicity/safety trade-off, not an arbitrary limitation.
- Inheritance should be reserved for genuine IS-A relationships; using it purely for code reuse without that relationship is a real, common design mistake, addressed fully via composition in Topic 7.

## Exercises

1. Given `class Vehicle { protected int wheels; void honk() { ... } }`, write a `Motorcycle extends Vehicle` subclass with an appropriate constructor calling `super(...)`, and explain why `Motorcycle IS-A Vehicle` is a reasonable relationship.
2. Explain, precisely, why allowing `class C extends A, B` would create genuine ambiguity if both `A` and `B` declared a field with the same name — not just a method, specifically a *field* (state) — and why this is the harder part of the Diamond Problem that even interfaces mostly avoid.
3. Explain what's conceptually wrong with `class Stack<T> extends ArrayList<T>`, referencing the IS-A test, even though it "works" in the sense of compiling and reusing `ArrayList`'s methods.

---

**Previous:** [03 — Abstraction](03-Abstraction.md) · **Next:** [05 — Polymorphism](05-Polymorphism.md)
