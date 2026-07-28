# `equals()` and `hashCode()`

## Learning Objectives

- Recall and precisely apply the difference between `==` and `.equals()` (first introduced in Module 03)
- Override `equals()` correctly, satisfying its full formal contract
- Understand and apply the `equals`/`hashCode` contract, and explain exactly what breaks when it's violated
- Write a correct `hashCode()` implementation, and understand why hash-based collections require one

## Prerequisites

[02 — `toString()`](02-tostring.md), Module 03 Topic 6 (the `Integer` cache `==` pitfall)

## Motivation

This is the single most important topic in this module, and one of the most consequential in the entire course — getting `equals()`/`hashCode()` wrong doesn't just cause obviously broken behavior; it causes **silent, intermittent** bugs in `HashMap`/`HashSet` (Module 10) that can be genuinely difficult to diagnose. This topic is worth mastering completely, not just skimming.

## Recap: `==` vs. `.equals()` (From Module 03)

Recall Module 03, Topic 6: `==` on object references compares **identity** (are these two references pointing at the exact same object?), never logical value equality. `Object`'s **default** `equals()` implementation actually does exactly the same thing:

```java
// Object's default equals():
public boolean equals(Object obj) {
    return (this == obj);   // identity comparison, by default!
}
```

**This means, without an override, `.equals()` and `==` behave identically** — both just check "same object?" — which is almost never what you actually want when comparing two objects for logical equality (e.g., "are these two `Point`s at the same coordinates?", not "are they the literal same object in memory?").

```java
class Point {
    int x, y;
    Point(int x, int y) { this.x = x; this.y = y; }
}

Point p1 = new Point(3, 4);
Point p2 = new Point(3, 4);
System.out.println(p1 == p2);         // false -- different objects
System.out.println(p1.equals(p2));      // ALSO false! -- Object's default equals() is just ==
```

Even though `p1` and `p2` are logically "the same point," `equals()` returns `false` because it hasn't been overridden — it's still using `Object`'s default identity check.

## Overriding `equals()` Correctly

```java
class Point {
    int x, y;
    Point(int x, int y) { this.x = x; this.y = y; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;                          // (1) same object -> trivially equal
        if (o == null || getClass() != o.getClass()) return false;  // (2) null or different type -> not equal
        Point p = (Point) o;                                    // (3) safe cast, now that type is confirmed
        return x == p.x && y == p.y;                             // (4) compare the ACTUAL relevant fields
    }
}
```

Now `p1.equals(p2)` correctly returns `true`. Walking through the four steps, each with a real reason:
1. **Identity shortcut** — if it's literally the same object, it's trivially equal; also a small performance optimization, avoiding unnecessary field comparisons.
2. **`null` and type check** — `equals()` must never throw `NullPointerException` (checking `o == null` first), and must return `false` for a fundamentally different, unrelated type rather than throwing `ClassCastException`.
3. **Safe cast** — only performed after confirming the type matches, so it can never fail.
4. **Field-by-field comparison** — the actual logical equality check, comparing whatever fields define "sameness" for this class.

## The `equals()` Contract — A Formal, Compiler-Unenforced Requirement

`Object`'s documentation (and the Java Language Specification) defines a strict **contract** that any `equals()` override **must** satisfy — the compiler does **not** check or enforce any of this; violating it produces code that compiles fine but behaves incorrectly, sometimes only in rare, hard-to-reproduce circumstances:

| Property | Requirement |
|---|---|
| **Reflexive** | `x.equals(x)` must always be `true` |
| **Symmetric** | `x.equals(y)` must equal `y.equals(x)` — never true one direction and false the other |
| **Transitive** | If `x.equals(y)` and `y.equals(z)`, then `x.equals(z)` must also be `true` |
| **Consistent** | Repeated calls to `x.equals(y)` must keep returning the same result, as long as neither object's relevant state changes |
| **Non-null** | `x.equals(null)` must always return `false`, never throw |

**Why does symmetry matter so much in practice?** A surprisingly easy way to accidentally violate it: using `instanceof` instead of `getClass()` for the type check, combined with inheritance:

```java
class Point {
    int x, y;
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;   // ⚠️ uses instanceof, not getClass()
        Point p = (Point) o;
        return x == p.x && y == p.y;
    }
}

class ColorPoint extends Point {
    String color;
    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) return false;
        ColorPoint cp = (ColorPoint) o;
        return super.equals(o) && color.equals(cp.color);
    }
}

Point p = new Point(1, 2);
ColorPoint cp = new ColorPoint(1, 2, "red");

p.equals(cp);    // true  -- cp IS-A Point (instanceof passes), coordinates match, Point.equals ignores color
cp.equals(p);     // false -- p is NOT a ColorPoint (instanceof fails)

// p.equals(cp) != cp.equals(p)  --  SYMMETRY VIOLATED!
```

**This is precisely why the `getClass() != o.getClass()` check (exact type match) is generally preferred over `instanceof` in `equals()` implementations** — it guarantees symmetry automatically, at the cost of "a `Point` and a `ColorPoint` with identical coordinates are never equal to each other, in either direction" — a trade-off most designs accept deliberately, precisely to preserve the contract.

## `hashCode()` — What It's For

> **`hashCode()`** returns an `int` "summary" of an object's state, used by **hash-based collections** (`HashMap`, `HashSet`, `HashTable` — full depth Module 10) to efficiently determine which internal "bucket" an object belongs in, **before** ever calling `equals()` to confirm an exact match.

`Object`'s default `hashCode()` typically returns a value derived from the object's memory address/identity (implementation-specific — HotSpot's exact algorithm isn't guaranteed by the spec, only that it's consistent for a given object during its lifetime) — meaning, by default, two logically-equal-but-distinct objects get **different** default hash codes, exactly mirroring the default `equals()` problem above.

## THE Contract: `equals()` and `hashCode()` Must Be Overridden TOGETHER

This is the single most important rule in this entire topic, and one of the most frequently tested facts in all of Java:

> **If two objects are equal according to `equals()`, they MUST have the same `hashCode()`.**
>
> (The reverse is NOT required: two unequal objects are *allowed* to share the same hash code — called a "hash collision" — hash-based collections are specifically designed to handle this correctly, as long as the *first* rule is honored.)

**Why must this hold, precisely, and what breaks if it doesn't?** Hash-based collections like `HashMap` use a two-step lookup process: (1) use `hashCode()` to quickly narrow down to a small "bucket" of candidates, (2) use `equals()` to find the exact match *within* that bucket. **If two equal objects have different hash codes, step (1) sends them to *different* buckets** — meaning a `HashMap` (or `HashSet`) can end up storing what should logically be "the same key" twice, in two different buckets, or fail to find an entry that's genuinely present, because it's searching entirely the wrong bucket.

```java
class Point {
    int x, y;
    @Override
    public boolean equals(Object o) { /* ... correct, as shown above ... */ }
    // ⚠️ NO hashCode() override -- still using Object's IDENTITY-based default!
}

Set<Point> points = new HashSet<>();
points.add(new Point(1, 2));
System.out.println(points.contains(new Point(1, 2)));   // FALSE !! -- even though equals() says they're
                                                            // equal, the DIFFERENT default hashCodes send
                                                            // this lookup to entirely the WRONG bucket,
                                                            // so equals() is never even CALLED to compare them
```

**This bug is genuinely insidious** — it compiles perfectly, `equals()` itself works correctly in isolation, and the failure only manifests specifically when the object is used inside a hash-based collection — a real, common, and often confusing real-world bug.

## Overriding `hashCode()` Correctly

```java
class Point {
    int x, y;

    @Override
    public boolean equals(Object o) { /* as shown above */ }

    @Override
    public int hashCode() {
        return Objects.hash(x, y);   // java.util.Objects -- combines multiple fields into ONE hash code
    }
}
```

**`java.util.Objects.hash(Object... values)`** is the standard, idiomatic way to implement `hashCode()` from Java 7 onward — it combines any number of fields' individual hash codes into a single, well-distributed result, using varargs (Module 06, Topic 1) internally. **The critical rule: `hashCode()` must be computed from exactly the same fields `equals()` uses** — this is precisely what guarantees the contract (equal objects → equal hash codes) holds, since both methods are derived from identical underlying state.

```java
public class Point {
    private final int x, y;    // note: final -- see the "mutable fields" warning below

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Point p = (Point) o;
        return x == p.x && y == p.y;     // uses x, y
    }

    @Override
    public int hashCode() {
        return Objects.hash(x, y);         // uses the SAME fields: x, y
    }
}
```

## A Critical, Related Warning: Never Use Mutable Fields in `hashCode()` for Objects Stored in Hash Collections

```java
class Tag {
    String name;   // MUTABLE -- not final, can be changed after construction
    @Override
    public int hashCode() { return Objects.hash(name); }
    @Override
    public boolean equals(Object o) { /* uses name */ }
}

Set<Tag> tags = new HashSet<>();
Tag t = new Tag();
t.name = "urgent";
tags.add(t);                 // stored in the bucket corresponding to hashCode() for "urgent"

t.name = "resolved";          // ⚠️ mutating the field AFTER it's already in the HashSet!

tags.contains(t);              // likely FALSE -- t's hashCode() is now DIFFERENT, so the lookup
                                  // searches the WRONG bucket -- 't' is effectively "lost" inside
                                  // the HashSet, even though it's still technically present in memory
```

**This is precisely why fields used in `hashCode()`/`equals()` are ideally `final`** (Module 03, Topic 7) — if an object's hash-relevant state can never change after construction, this entire category of "lost in the hash collection" bug becomes structurally impossible. This is also a strong, concrete, real-world motivation for immutable objects generally (a design theme that returns with full force in Module 23's Records).

## Real-World Analogy

Think of `hashCode()` like a **library's shelf-section number** (Fiction, Non-Fiction, Reference — a rough categorization), and `equals()` like actually **checking the book's exact title and author** once you've found the right shelf section. If a librarian shelves the *same* book under **two different section numbers on two different occasions** (violating the equals/hashCode contract — inconsistent hash for equal objects), a patron searching in the *correct* section for that book might come up empty-handed, even though a copy is sitting on a shelf somewhere else in the building — exactly the "lost in the hash collection" bug demonstrated above.

## Advantages of the Contract-Based Design

- Enables extremely fast average-case lookup in hash-based collections (`HashMap`/`HashSet` — Module 10) — near O(1), rather than having to check every single stored element one by one.
- `Objects.hash(...)` and `Objects.equals(...)` provide simple, standard, idiomatic implementations, removing most of the historical boilerplate/error-proneness of writing these by hand.

## Disadvantages / Trade-offs

- The contract is **entirely compiler-unenforced** — violating it produces code that compiles and often appears to work in casual testing, while harboring a real, subtle bug that only manifests under specific conditions (like the hash-collection scenarios shown above).
- Requires real discipline: every time a class's equality-relevant fields change, both `equals()` and `hashCode()` need to be kept in sync — a genuine, ongoing maintenance responsibility.

## Best Practices

- **Always override `equals()` and `hashCode()` together** — never one without the other.
- Use `getClass() != o.getClass()` (not `instanceof`) in `equals()` to guarantee symmetry, unless you have a specific, deliberate reason to allow subtype equality.
- Use `Objects.hash(...)` for `hashCode()`, built from **exactly** the same fields `equals()` compares.
- Prefer `final` fields for anything used in `equals()`/`hashCode()`, especially for objects that will be stored in hash-based collections.
- Modern Java: consider **Records** (Module 23) for simple data-holding classes — they generate a correct, contract-compliant `equals()`/`hashCode()`/`toString()` automatically, eliminating this entire category of hand-written boilerplate and its associated bug risk.

## Common Mistakes

- Overriding `equals()` without overriding `hashCode()` (or vice versa) — the single most common, most consequential mistake in this entire topic.
- Using `instanceof` instead of `getClass()` in `equals()`, risking a symmetry violation across an inheritance hierarchy.
- Computing `hashCode()` from different fields than `equals()` uses — breaking the contract even though both methods individually "look correct."
- Using mutable fields in `hashCode()`/`equals()` for objects stored in `HashMap`/`HashSet`, causing objects to become unfindable after being mutated post-insertion.

## Interview Questions

1. **Q: What is the `equals()`/`hashCode()` contract, precisely?**
   A: If two objects are equal according to `equals()`, they **must** produce the same `hashCode()`. The converse isn't required — unequal objects may share a hash code (a "collision"), which hash-based collections handle correctly, as long as the first rule is always honored.

2. **Q: What breaks if you override `equals()` but not `hashCode()`?**
   A: `Object`'s default identity-based `hashCode()` remains in effect, meaning two objects that `equals()` considers equal will typically have different default hash codes. In a `HashMap`/`HashSet`, this sends logically-equal objects to different internal buckets, causing lookups (`contains`, `get`) to silently fail even when a logically matching object is present, since `equals()` is only checked *within* the (wrong) bucket the hash code points to.

3. **Q: Why is it generally safer to use `getClass()` rather than `instanceof` inside `equals()`?**
   A: `instanceof` can silently break the *symmetry* requirement of the equals contract when subclasses are involved — a superclass instance might consider a subclass instance equal (since the subclass IS-A superclass, `instanceof` passes), while the subclass's own `equals()` rejects the superclass instance (since it's not the more specific subtype). `getClass()` requires an exact type match in both directions, guaranteeing symmetry automatically.

4. **Q: Why should fields used in `equals()`/`hashCode()` ideally be `final`?**
   A: If such a field is mutated after the object has already been inserted into a hash-based collection, the object's hash code can change, causing subsequent lookups to search the wrong bucket — effectively "losing" the object inside the collection, even though it's still technically present in memory. Making these fields `final` makes this entire bug category structurally impossible.

## Summary

- `Object`'s default `equals()` is just `==` (identity); overriding it requires satisfying a strict, compiler-unenforced **contract**: reflexive, symmetric, transitive, consistent, and never throwing on `null`.
- Using `getClass()` (not `instanceof`) in `equals()` guards against a common, subtle symmetry violation in inheritance hierarchies.
- The **`equals`/`hashCode` contract**: equal objects **must** produce equal hash codes — violating this silently breaks `HashMap`/`HashSet` lookups, since they use `hashCode()` to locate a bucket before ever calling `equals()`.
- `hashCode()` must be derived from **exactly** the same fields `equals()` uses, ideally computed with `Objects.hash(...)`, and ideally from `final` fields to prevent post-insertion mutation bugs.
- Modern Java Records (Module 23) generate correct, contract-compliant implementations of both automatically.