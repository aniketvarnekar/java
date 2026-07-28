# Object Cloning

## Learning Objectives

- Understand `Cloneable` and `clone()`, including why the design is widely considered broken
- Distinguish shallow copy from deep copy precisely, with a worked memory trace
- Know the modern, recommended alternative: copy constructors (and copy factory methods)

## Prerequisites

[01 — The `Object` Class](01-the-object-class.md), Module 02 Topic 3 (Heap, references)

## Motivation

`clone()` is a genuinely important topic to understand — not primarily because you should use it (you mostly shouldn't, and this topic explains exactly why), but because it's a well-known cautionary tale about API design, it appears in legacy code you'll encounter, and understanding *why* it's broken deepens your understanding of references and object identity from Module 02.

## Problem Statement

Sometimes you need an independent **copy** of an object — a new object with the same field values, but changing the copy should **not** affect the original. Simply doing `Point p2 = p1;` does **not** create a copy at all — recall Module 02, Topic 3: this just copies the **reference**, so `p1` and `p2` end up pointing at the **exact same** object.

```java
Point p1 = new Point(3, 4);
Point p2 = p1;              // NOT a copy -- p2 is just ANOTHER reference to the SAME object
p2.x = 999;
System.out.println(p1.x);    // 999 !! -- p1 "changed" too, because there was only ever ONE object
```

## `Cloneable` and `clone()` — How It's Supposed to Work

```java
class Point implements Cloneable {
    int x, y;
    Point(int x, int y) { this.x = x; this.y = y; }

    @Override
    public Point clone() {
        try {
            return (Point) super.clone();     // calls Object's protected clone() method
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(e);        // "can't happen" -- we DID implement Cloneable
        }
    }
}

Point p1 = new Point(3, 4);
Point p2 = p1.clone();           // a GENUINE, separate copy
p2.x = 999;
System.out.println(p1.x);         // 3 -- p1 is unaffected, as expected
```

## Why This Design Is Widely Considered Broken

This is a genuinely well-known, widely-cited example of a design mistake in Java's own standard library — worth understanding precisely, not just accepting as folklore:

1. **`Cloneable` is a "marker interface" with no methods at all** — it doesn't declare `clone()`, or anything else. It exists purely as a flag that `Object`'s `clone()` implementation checks *at runtime* — if a class calls `super.clone()` without implementing `Cloneable`, it throws `CloneNotSupportedException`, a **checked exception** (full depth: Module 12) you must handle even though, logically, you'd expect this to be a compile-time concern, not a runtime one. This inverts Module 05, Topic 6's abstraction principle — a contract that should be visible and enforced at compile time is instead only checked at runtime, silently.

2. **`Object.clone()` is `protected`**, meaning your override must explicitly widen it to `public` to be generally usable — an unusual, easy-to-get-wrong requirement most other Java APIs don't impose.

3. **`clone()` performs a *shallow* copy by default** (explained fully below) — which is frequently **not** what's actually needed, silently producing subtly incorrect behavior unless you're specifically aware of this and handle it yourself.

4. **It completely bypasses the normal constructor call** — `clone()` doesn't run any constructor at all, which conflicts with the careful, deliberate object-creation guarantees you learned in Module 06, Topic 5 (every object should be built through its constructor, ensuring validated initial state) — cloning sidesteps this entirely.

## Shallow Copy vs. Deep Copy — The Real Danger

`Object`'s default `clone()` behavior copies each field's **value** — for primitives, this is a genuine independent copy; **for object-reference fields, it copies only the reference**, not the object it points to. This is called a **shallow copy**.

```java
class Person {
    String name;
    List<String> hobbies;    // a REFERENCE field
}

Person original = new Person();
original.name = "Aniket";
original.hobbies = new ArrayList<>(List.of("Reading", "Coding"));

Person copy = original.clone();   // SHALLOW copy

copy.hobbies.add("Gaming");        // mutating the LIST through the COPY's reference...
System.out.println(original.hobbies);  // ...also shows "Gaming"! Both objects share the SAME List!
```

```
Original                                 Shallow Copy

┌────────────────────────────┐          ┌────────────────────────────┐
│ name = "Aniket"            │          │ name = "Aniket"            │
│ hobbies = 0xA1B2 ──────────┼──────────┼───────── hobbies = 0xA1B2  │
└────────────────────────────┘          └────────────────────────────┘
               │                                    │
               └────────────────┬───────────────────┘
                                ▼
                 ┌──────────────────────────────┐
                 │ ["Reading",                  │
                 │  "Coding",                   │
                 │  "Gaming"]                   │
                 └──────────────────────────────┘

name (String): copied independently
hobbies (List): SAME reference copied (shared object)
```

**This is exactly the reference-copying behavior from Module 02, Topic 3 and Module 06, Topic 1, applied to `clone()` specifically** — copying a reference field copies the *address*, not the object it addresses, so both the original and the "clone" end up sharing the exact same underlying mutable object. Mutating that shared object through **either** reference is visible through **both**.

**A `deep` copy**, by contrast, recursively copies referenced objects too, so the copy and original share **no** mutable state at all:

```java
@Override
public Person clone() {
    try {
        Person copy = (Person) super.clone();
        copy.hobbies = new ArrayList<>(this.hobbies);   // manually create a NEW, independent List
        return copy;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError(e);
    }
}
```
Now `copy.hobbies` and `original.hobbies` are genuinely separate `ArrayList` objects — mutating one has zero effect on the other.

**The fundamental danger:** `clone()`'s *default* behavior is shallow, but developers frequently *assume* it produces a fully independent deep copy — this mismatch between expectation and actual behavior is a real, historically common source of subtle bugs, especially for classes with reference-typed fields (which is most non-trivial classes).

## The Modern, Recommended Alternative: Copy Constructors

Given everything above, the overwhelming modern consensus (including guidance from Java's own creators and standard style guides) is: **avoid `Cloneable`/`clone()` entirely.** Instead, use a **copy constructor** — an ordinary constructor that takes an instance of the same class and copies its state:

```java
class Person {
    String name;
    List<String> hobbies;

    Person(String name, List<String> hobbies) {
        this.name = name;
        this.hobbies = hobbies;
    }

    Person(Person other) {                                // COPY CONSTRUCTOR
        this.name = other.name;
        this.hobbies = new ArrayList<>(other.hobbies);      // explicit, VISIBLE deep copy of the List
    }
}

Person original = new Person("Aniket", new ArrayList<>(List.of("Reading")));
Person copy = new Person(original);    // clear, explicit, uses a REAL constructor
copy.hobbies.add("Gaming");
System.out.println(original.hobbies);   // ["Reading"] -- correctly UNAFFECTED
```

**Why is this better, precisely?** (1) It uses an **ordinary, familiar constructor** — no marker interfaces, no checked exceptions, no `protected`-method-widening ceremony. (2) The copying logic (shallow vs. deep, for each field) is **fully visible and explicit** in the constructor body — nothing is hidden or silently defaulted. (3) It **does** run through normal object construction (Module 06, Topic 5's careful, validated initialization order), unlike `clone()`, which bypasses it entirely. A closely related alternative is a **static copy factory method** (`Person.copyOf(original)`), following the same principle with slightly different calling syntax.

## Real-World Analogy

Think of a **shallow copy** like **photocopying a folder's cover page, but leaving the same physical documents inside both folders** — you now have two folder covers, but if someone pulls a document out of (or scribbles on a document inside) the "original" folder, that exact same physical document is missing/altered when you check the "copy" folder too, since there was only ever **one** set of actual documents, just two covers pointing at them. A **deep copy** — best achieved via a deliberate, explicit copy constructor — is like actually **photocopying every single document inside as well**, giving you two fully independent, self-contained folders that can be modified without any interference between them.

## Advantages of Copy Constructors Over `clone()`

- No checked-exception ceremony, no marker interface, no `protected`-method-widening requirement.
- Copying logic is fully explicit and visible, field by field — no silent, easy-to-miss shallow-copy default.
- Runs through genuine, normal object construction, preserving Module 06's careful initialization guarantees.

## Disadvantages / Trade-offs

- Copy constructors must be written explicitly for every class that needs copying support (no automatic, inherited "just call `clone()`" shortcut) — though this is arguably a feature, not a cost, given the explicitness benefit above.
- `clone()` remains present throughout the JDK and countless existing codebases (e.g., `ArrayList.clone()`) — you must still understand it to work with and reason about legacy/existing code correctly, even while avoiding it in new code.

## Best Practices

- **Prefer copy constructors (or static copy factory methods) over `Cloneable`/`clone()`** in new code — this is the strong, well-established modern consensus.
- If you do need to reason about or use `clone()` on an existing class (like `ArrayList`), always check its documentation for whether it performs a shallow or deep copy — never assume.
- When writing a copy constructor, be deliberate and explicit about every reference-typed field: does it need a genuine deep copy, or is sharing acceptable for this specific field (e.g., an immutable field is always safe to share, since it can never be mutated through either reference anyway)?

## Common Mistakes

- Assuming `clone()` always produces a fully independent deep copy — its default behavior is shallow, and deep copying (for reference fields) must be handled explicitly.
- Forgetting to implement `Cloneable` before calling `super.clone()`, resulting in a runtime `CloneNotSupportedException` instead of a compile-time error.
- Not overriding `clone()`'s return type covariantly (returning the specific subtype, as shown in the `Point` example) — technically optional, but a widely followed best practice (available since Java 5's covariant return types) that avoids forcing every caller to cast the result themselves.

## Interview Questions

1. **Q: Why is Java's `Cloneable`/`clone()` design widely considered a mistake?**
   A: `Cloneable` is a marker interface with no methods, so the contract is only enforced at runtime (via `CloneNotSupportedException`) rather than compile time; `Object.clone()` is `protected`, requiring awkward widening; the default behavior is a shallow copy, which is frequently not what's actually wanted; and `clone()` completely bypasses normal constructor-based object creation.

2. **Q: What's the difference between a shallow copy and a deep copy?**
   A: A shallow copy copies each field's value directly — for reference-typed fields, this copies only the reference (address), so the original and copy end up sharing the exact same underlying mutable object. A deep copy recursively copies referenced objects too, so the copy and original share no mutable state, and mutating one never affects the other.

3. **Q: What is the modern, recommended alternative to `Cloneable`/`clone()`, and why is it preferred?**
   A: A copy constructor (or static copy factory method) — an ordinary constructor taking an instance of the same class and explicitly copying its state, field by field. It avoids `clone()`'s checked-exception/marker-interface ceremony, makes shallow-vs-deep copying decisions fully explicit and visible per field, and runs through normal, validated object construction rather than bypassing it.

## Summary

- `Point p2 = p1;` never copies an object — it only copies the reference, leaving both variables pointing at the exact same object (Module 02, Topic 3).
- `Cloneable`/`clone()` is Java's original mechanism for genuine object copying, but is widely considered a design mistake: a runtime-only marker interface, a `protected` method requiring widening, and a shallow-copy-by-default behavior that frequently surprises developers.
- **Shallow copy**: reference fields are copied by address, sharing the same underlying object. **Deep copy**: referenced objects are recursively copied too, achieving full independence.
- The modern, recommended alternative is a **copy constructor** (or static copy factory method) — explicit, uses normal construction, and makes shallow-vs-deep decisions visible per field.