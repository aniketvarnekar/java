# Records

## Learning Objectives

- Explain the specific boilerplate problem records solve, and why it existed for 20+ years before being fixed
- Write records correctly, including compact constructors and custom methods
- Know exactly what the compiler generates for you, and what a record cannot do
- Compare records against ordinary classes, and against the Lombok library's `@Data` approach

## Prerequisites

Module 07 Topic 2 (`equals()`/`hashCode()`), Module 06 Topic 2 (Constructors), Module 05 Topic 2 (Encapsulation)

## Motivation

Recall Module 07, Topic 2: to write a correct, simple, immutable data-holder class in "classic" Java — say, a `Point` with `x` and `y` — you had to hand-write: private final fields, a constructor, getters, `equals()`, `hashCode()`, and `toString()`. That's roughly **30-40 lines of code to represent two numbers.** None of it is interesting; all of it is mechanical, derivable entirely from the field list, and yet every line was a place a mistake could hide (Module 07, Topic 2's warning about `equals()`/`hashCode()` contracts being easy to violate by hand). **Records**, introduced as a preview in Java 14 (2020) and finalized in **Java 16** (2021), let the compiler generate all of it for you from a single line.

## The Problem, Concretely

```java
// The "classic" way -- Module 06/07's techniques, applied in full
public final class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() { return x; }
    public int getY() { return y; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Point)) return false;
        Point p = (Point) o;
        return x == p.x && y == p.y;
    }

    @Override
    public int hashCode() {
        return Objects.hash(x, y);
    }

    @Override
    public String toString() {
        return "Point[x=" + x + ", y=" + y + "]";
    }
}
```

```java
// The record way -- Java 16+
public record Point(int x, int y) { }
```

Both compile to a class with **identical semantics**: immutable fields, accessors, a correct `equals()`/`hashCode()` pair (Module 07, Topic 2's contract, satisfied automatically), and a sensible `toString()`. The record version is one line.

## What the Compiler Generates For You

Writing `record Point(int x, int y) { }` causes `javac` to automatically generate:

1. **Private final fields** for `x` and `y`.
2. **A canonical constructor** — `public Point(int x, int y)` — assigning each parameter to its field.
3. **Accessor methods** — named `x()` and `y()` (**not** `getX()`/`getY()` — a deliberate departure from JavaBeans convention, since a record isn't a mutable bean).
4. **`equals()`** — compares all fields (Module 07, Topic 2's `instanceof`+field-comparison pattern, generated correctly every time).
5. **`hashCode()`** — a hash combining all fields, consistent with `equals()` (satisfying Module 07, Topic 2's contract by construction — it is now *impossible* to accidentally violate it).
6. **`toString()`** — in the form `Point[x=1, y=2]`.

```
 record Point(int x, int y) { }
        |
        v
 +----------------------------------------------+
 | final class Point extends Record {            |
 |   private final int x;                        |
 |   private final int y;                         |
 |   public Point(int x, int y) { this.x=x; this.y=y; }
 |   public int x() { return x; }                  |
 |   public int y() { return y; }                   |
 |   public boolean equals(Object o) { ... }          |
 |   public int hashCode() { ... }                     |
 |   public String toString() { ... }                    |
 | }                                                       |
 +----------------------------------------------+
```

**Every record implicitly extends `java.lang.Record`** (an abstract class), which is why records **cannot extend any other class** (Java has no multiple inheritance of state, Module 05, Topic 4) — but a record **can implement interfaces**, exactly like any class.

## Compact Constructors — Validation Without Repeating Yourself

If you need to validate or normalize fields, a **compact constructor** lets you do so without re-declaring the parameter list or writing the field assignments (those are still generated automatically, *after* your compact constructor body runs):

```java
public record Range(int low, int high) {
    public Range {                       // no parameter list -- reuses the record header's
        if (low > high) {
            throw new IllegalArgumentException("low must be <= high");
        }
        // no explicit "this.low = low;" needed -- the compiler still does it after this block
    }
}
```

**Why this form, instead of a normal constructor?** A normal constructor would force you to repeat `this.low = low; this.high = high;` — pure boilerplate the record already promises to eliminate. The compact constructor lets you inject **validation or normalization logic only**, while the mechanical field assignment remains automatic.

You can also define an explicit **canonical constructor** (with the full parameter list) if you need more control, or **additional, overloaded constructors** — but every overloaded constructor must ultimately delegate to the canonical one via `this(...)` (identical to Module 06, Topic 2's constructor chaining rule):

```java
public record Range(int low, int high) {
    public Range {
        if (low > high) throw new IllegalArgumentException("low must be <= high");
    }

    public Range(int single) {           // overloaded constructor
        this(single, single);             // must delegate to the canonical constructor
    }
}
```

## Records Can Have Methods, Static Fields, and Static Methods

A record is not limited to just data — it can have any instance methods, static fields, and static methods a normal class can, **except** it cannot declare additional **instance** fields beyond the ones in its header (that would break the "record = exactly its header's data" guarantee):

```java
public record Point(int x, int y) {
    public double distanceFromOrigin() {          // ordinary instance method -- allowed
        return Math.sqrt(x * x + y * y);
    }

    public static final Point ORIGIN = new Point(0, 0);   // static field -- allowed

    // private int cachedValue;   // COMPILE ERROR -- extra instance field not allowed
}
```

## Overriding Generated Methods

You can override any generated method (`toString()`, `equals()`, `hashCode()`, or an accessor) if the default isn't right for your case — the compiler simply doesn't generate that particular method for you, and uses your version instead:

```java
public record Password(String value) {
    @Override
    public String toString() {
        return "Password[REDACTED]";   // override -- never leak the real value via logging
    }
}
```

## Records vs. Classes vs. Lombok

| | Ordinary class | Record | Lombok `@Data`/`@Value` |
|---|---|---|---|
| Boilerplate | You write everything | Compiler generates it | Annotation processor generates it |
| Requires a build-tool dependency | No | No — built into the language | **Yes** — an external library |
| Mutability | Your choice | **Always immutable** fields | Your choice (`@Data` is mutable, `@Value` is immutable) |
| Can extend a class | Yes | **No** (implicitly extends `Record`) | Yes |
| IDE/tooling support | Native | Native (it's real bytecode) | Requires plugin support for full IDE integration |
| Best for | Classes with real behavior/mutable state | **Simple, immutable data carriers** | Teams already invested in Lombok pre-Java 16 |

**Why did records make Lombok's `@Value` largely unnecessary for new code?** Lombok's core value proposition — eliminating data-class boilerplate — is now a **first-class, compiler-verified language feature**, with no external dependency, no annotation processor, and no "magic" a new team member has to learn a separate library to understand. Lombok remains relevant for older codebases already using it, or for features records don't cover (like builder-pattern generation).

## When NOT to Use a Record

- **Mutable state is required** — a record's fields are always `final`; if you need to change a value after construction, you need an ordinary class (Module 06's mutable objects).
- **You need to extend another class** — records cannot extend classes (only implement interfaces).
- **JPA/Hibernate entities** (a common real-world case) — most JPA versions require a no-arg constructor and mutable fields for the persistence provider to construct and populate entities via reflection (Module 16, Topic 4) — a fundamental mismatch with a record's immutability guarantee. (Newer Hibernate versions have limited, evolving record support, but classic mutable entities remain the norm.)
- **The class represents genuine behavior, not just data** — if a class is mostly methods and only incidentally holds a field or two, a record's "I am fundamentally a data tuple" semantics are misleading.

## Real-World Analogy

A hand-written data class is like **building custom furniture from raw lumber every single time** — fully flexible, but you re-cut the same basic joints (constructor, `equals`, `hashCode`) from scratch on every project. A record is like **ordering a flat-pack kit for a well-defined shape** — you specify the dimensions (the field list) once, and the standard, correct assembly (every generated method) happens automatically, precisely, and identically every time.

## Advantages

- Eliminates data-class boilerplate entirely, removing an entire category of Module 07, Topic 2 `equals()`/`hashCode()` mistakes by construction.
- Immutability by default (Module 06's thread-safety benefits) with zero extra effort.
- Built into the language — no external dependency, fully supported by every IDE and tool natively.
- Concise, self-documenting: `record Point(int x, int y)` states its entire contract in one line.

## Disadvantages / Trade-offs

- Always immutable — genuinely the wrong tool when mutable state is a requirement.
- Cannot extend a class, which occasionally conflicts with existing class hierarchies.
- Accessor naming (`x()` not `getX()`) breaks JavaBeans convention, which some older reflection-based frameworks (older JPA versions, some JavaBeans-reliant UI frameworks) assume.

## Best Practices

- Default to a record for any class that is fundamentally "a fixed bundle of data" with no mutable state — this is now the idiomatic modern choice.
- Use compact constructors for validation, not complex logic — keep the compact constructor short and focused on invariants.
- Override `toString()` explicitly for records containing sensitive data (passwords, tokens) rather than relying on the generated version.

## Common Mistakes

- Trying to add extra instance fields beyond the header — this is a compile error, not a runtime surprise, but it trips up newcomers expecting record syntax to work like a class body.
- Forgetting that record accessors are `x()`, not `getX()` — code (or frameworks) written assuming JavaBeans naming will not find them via reflection unless explicitly adapted.
- Using a record for a JPA entity without checking your specific Hibernate/JPA version's actual level of record support — this remains a common, genuine source of runtime surprises in real projects.

## Interview Questions

1. **Q: What specific problem do records solve, and which Java version finalized them?**
   A: They eliminate the boilerplate of hand-writing a constructor, accessors, `equals()`, `hashCode()`, and `toString()` for simple immutable data-holder classes — finalized in Java 16 (2021), after a Java 14 preview.

2. **Q: What does a compact constructor do differently from a normal constructor in a record?**
   A: It omits the parameter list (reusing the record header's) and omits explicit field assignments (the compiler still performs them, after the compact constructor's body runs) — it exists purely to add validation/normalization logic without repeating boilerplate.

3. **Q: Why can't a record extend another class?**
   A: Every record implicitly extends `java.lang.Record`, and Java has no multiple inheritance of state (Module 05, Topic 4) — a record can still implement any number of interfaces, though.

4. **Q: Why are records generally unsuitable as JPA/Hibernate entities?**
   A: Most JPA implementations construct and populate entities reflectively via a no-arg constructor and mutable field/setter access — fundamentally incompatible with a record's immutability and lack of a no-arg constructor, though support is evolving in newer versions.

## Summary

- **Records** (`record Name(fields...) { }`), finalized in **Java 16**, eliminate data-class boilerplate by having the compiler generate the constructor, accessors, `equals()`, `hashCode()`, and `toString()`.
- Records are **always immutable** and implicitly extend `java.lang.Record`, so they cannot extend another class but can implement interfaces.
- **Compact constructors** add validation without repeating field assignments; explicit canonical or overloaded constructors are also possible.
- Records are the right tool for simple, immutable data carriers — the wrong tool for mutable state, class hierarchies, or (usually) JPA entities.

## Exercises

1. Rewrite a hand-written `Point` class (with fields, constructor, getters, `equals()`, `hashCode()`, `toString()`) as a one-line record, and verify the behavior is equivalent.
2. Write a `Range(int low, int high)` record with a compact constructor that throws `IllegalArgumentException` if `low > high`.
3. Explain, from memory, why records cannot have additional instance fields beyond their header, and why that restriction is essential to what a record *is*.
4. Explain why a typical JPA entity is usually a poor fit for a record.

---

**Previous:** [00 — Module Overview](00-Module-Overview.md) · **Next:** [02 — Sealed Classes](02-Sealed-Classes.md)
