# Pattern Matching

## Learning Objectives

- Use `instanceof` pattern matching to eliminate redundant casts
- Use `switch` pattern matching, including type patterns, guarded patterns (`when`), and `null` handling
- Destructure records directly in a pattern using record patterns
- Combine sealed classes (Topic 2) with `switch` pattern matching to get compiler-checked exhaustiveness

## Prerequisites

Module 05 Topic 5 (Polymorphism — `instanceof` and casting), Module 04 Topic 2 (`switch` statement/expression), Topic 1 (Records) and Topic 2 (Sealed Classes) of this module

## Motivation

Module 05, Topic 5 taught you the classic `instanceof`-then-cast idiom: check the type, then **manually cast** to use it. That cast is redundant work — the `instanceof` check already proved the type; you're just telling the compiler something it already just verified. **Pattern matching**, delivered incrementally from Java 16 through Java 21, removes that redundancy across `instanceof` and `switch`, and — combined with Topic 1's records and Topic 2's sealed classes — enables something classic Java never had: **destructuring** a type directly in a pattern, with the compiler proving you've handled every case.

## `instanceof` Pattern Matching (Java 16)

```java
// The classic way -- Module 05, Topic 5
if (obj instanceof String) {
    String s = (String) obj;      // redundant cast -- you just proved it's a String
    System.out.println(s.length());
}

// Pattern matching -- Java 16+
if (obj instanceof String s) {    // "s" is bound automatically, already cast
    System.out.println(s.length());
}
```

**How it works internally:** `s` is a **pattern variable** — the compiler proves, via flow analysis, exactly which branches `s` is definitely a `String` and safely in scope. This flow-sensitivity extends usefully into `&&` conditions:

```java
if (obj instanceof String s && s.length() > 5) {   // s is in scope for the && clause too
    System.out.println(s);
}
```

And, less obviously, into the **negative** branch when the positive branch exits early:

```java
if (!(obj instanceof String s)) {
    return;
}
System.out.println(s.length());   // s is STILL in scope here -- the compiler proved
                                    // that reaching this line means the instanceof was true
```

## `switch` Pattern Matching (Java 21)

Module 04, Topic 2's `switch` only ever matched exact constant values. **Pattern matching for `switch`**, finalized in **Java 21** (2023), lets each `case` match a **type**, not just a value:

```java
static String describe(Object obj) {
    return switch (obj) {
        case Integer i -> "int: " + i;
        case String s  -> "string: " + s;
        case null      -> "it's null";        // switch pattern matching can match null directly
        default        -> "something else: " + obj;
    };
}
```

**Why is matching `null` directly inside `switch` a big deal?** Recall Module 04, Topic 2's warning: a classic `switch` on a reference type **throws `NullPointerException` immediately** if the selector is `null`, before any `case` is even considered — one of that topic's flagged common mistakes. `case null ->` lets you handle that possibility **explicitly and safely**, inside the `switch` itself, instead of requiring an external `if (obj == null)` guard beforehand.

## Guarded Patterns — `when`

A pattern can be refined with an additional boolean condition using `when`:

```java
static String categorize(Object obj) {
    return switch (obj) {
        case Integer i when i < 0  -> "negative int";
        case Integer i when i == 0 -> "zero";
        case Integer i             -> "positive int";
        case String s when s.isEmpty() -> "empty string";
        case String s               -> "non-empty string";
        default -> "other";
    };
}
```

**Why not just use `&&` inside the case body?** A guarded pattern keeps the **condition and the match together, at the case label itself** — readable top-to-bottom as "match this type, further narrowed by this condition," and it correctly participates in **exhaustiveness checking** (below): the compiler knows an `Integer` guarded by `when i < 0` doesn't cover *all* integers, so a plain `case Integer i ->` is still required to catch the rest.

## Record Patterns — Destructuring (Java 21)

This is where Topic 1 (Records) and pattern matching combine directly. A record pattern lets you **match a record's type AND extract its components in one step**:

```java
record Point(int x, int y) { }

static String describe(Object obj) {
    return switch (obj) {
        case Point(int x, int y) when x == y -> "on the diagonal";
        case Point(int x, int y)             -> "point at " + x + "," + y;
        default -> "not a point";
    };
}
```

`Point(int x, int y)` is a **record pattern**: it matches "is this a `Point`?" and simultaneously **destructures** it into `x` and `y` — no manual `p.x()`/`p.y()` accessor calls (Topic 1) needed. Record patterns **nest**, letting you match and destructure arbitrarily deep structures in one expression:

```java
record Point(int x, int y) { }
record Line(Point start, Point end) { }

static double length(Object obj) {
    return switch (obj) {
        case Line(Point(var x1, var y1), Point(var x2, var y2)) ->
            Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2));
        default -> 0;
    };
}
```

`var` inside a record pattern lets the compiler infer each component's type — combining Module 03, Topic 1's local-variable type inference with destructuring.

## Exhaustive `switch` Over Sealed Types — The Full Payoff

This is precisely where Topic 2's sealed classes and `switch` pattern matching combine to deliver something classic Java never offered:

```java
sealed interface Shape permits Circle, Square, Triangle { }
record Circle(double radius) implements Shape { }
record Square(double side) implements Shape { }
record Triangle(double base, double height) implements Shape { }

static double area(Shape shape) {
    return switch (shape) {
        case Circle(double r)        -> Math.PI * r * r;
        case Square(double s)        -> s * s;
        case Triangle(double b, double h) -> 0.5 * b * h;
        // NO default needed -- and NO default ALLOWED to silently hide a missing case
    };
}
```

**Because `Shape` is `sealed` (Topic 2) with exactly three permitted subtypes, the compiler knows, with total certainty, the complete list of possible cases.** If a case is missing, this is a **compile error** — "the switch expression does not cover all possible input values." If a fourth shape (`Hexagon`) is later added to `Shape`'s `permits` list, **every exhaustive `switch` over `Shape` across the entire codebase immediately fails to compile** until updated — turning Module 05's silent-runtime-gap problem into an impossible-to-miss compile-time signal.

```
 Classic instanceof chain              Exhaustive sealed switch
 --------------------------           --------------------------
 if/else if/else if/...                switch (shape) { case ...; case ...; }
 no compiler check of                  compiler PROVES every sealed
 completeness                          subtype is handled
 new subtype added ->                  new subtype added ->
 silent runtime gap                    COMPILE ERROR everywhere it matters
```

## Real-World Analogy

Classic `instanceof`-and-cast is like **checking someone's ID, then asking them to state their own name out loud again** even though you just read it — redundant work. Pattern matching is like **the ID check itself handing you their name directly** — no redundant restatement. Exhaustive `switch` over a sealed type is like **a form with a truly fixed, printed list of checkboxes** (not a blank "other" line) — it is *structurally impossible* to submit the form leaving a real option unconsidered, because the compiler is checking the printed list against your code, every single time it compiles.

## Advantages

- Eliminates redundant casting after `instanceof` checks, and redundant accessor calls after record type checks.
- `switch` pattern matching handles `null` explicitly, closing a well-known `NullPointerException` trap (Module 04, Topic 2).
- Exhaustive `switch` over sealed types (Topic 2) gives compiler-**proven** completeness — a category of bug (missed case) becomes a compile error instead of a runtime surprise.
- Record patterns make deeply nested data structures readable to destructure in a single expression.

## Disadvantages / Trade-offs

- Deeply nested record patterns can become visually dense — readability should still guide how much destructuring to inline in one pattern versus extracting named variables.
- Exhaustiveness checking only provides its full guarantee over genuinely `sealed` hierarchies (Topic 2) — over ordinary open classes, a `default` branch is still required and the same old completeness risk remains.

## Best Practices

- Prefer pattern-matching `instanceof`/`switch` over the classic cast idiom in all new code — it's strictly safer and shorter.
- Combine sealed hierarchies (Topic 2) with exhaustive `switch` specifically for closed, fixed-case domain modeling — this is the single biggest payoff of this entire topic.
- Use guarded patterns (`when`) instead of nesting `if` statements inside a `case` body, to keep exhaustiveness checking accurate.

## Common Mistakes

- Adding an unnecessary `default` branch to an exhaustive sealed `switch` — this silently defeats the compiler's exhaustiveness checking for future additions to the hierarchy (a new subtype would now silently fall into `default` instead of causing a compile error).
- Forgetting that `case null ->` must be handled explicitly (or a `default` branch must cover it) in `switch` pattern matching over reference types — omitting both is a compile error, a deliberate improvement over the classic `switch`'s silent `NullPointerException`.
- Writing overly deep record-pattern nesting that hurts readability more than it helps — extract intermediate variables when nesting gets hard to read at a glance.

## Interview Questions

1. **Q: What does `if (obj instanceof String s)` do differently from the classic `instanceof`-then-cast idiom?**
   A: It binds `s` as an already-cast, flow-scoped pattern variable directly from the `instanceof` check, eliminating the redundant manual cast — available since Java 16.

2. **Q: How does `switch` pattern matching (Java 21) handle `null` differently from a classic `switch`?**
   A: A classic `switch` on a reference type throws `NullPointerException` immediately if the selector is `null`. Pattern-matching `switch` lets you write `case null ->` to handle it explicitly and safely inside the `switch` itself.

3. **Q: How do sealed classes (Topic 2) and `switch` pattern matching combine to provide a guarantee classic Java never had?**
   A: Because a sealed type's complete set of permitted subtypes is known to the compiler, a `switch` over it can be checked for **exhaustiveness** at compile time — a missing case is a compile error, and adding a new subtype later breaks compilation everywhere an exhaustive switch needs updating, rather than silently leaving a runtime gap.

4. **Q: What is a record pattern, and what does it do in one step that classic code needs two steps for?**
   A: A record pattern like `case Point(int x, int y) ->` simultaneously checks the type AND destructures its components into `x` and `y` — classic code would need a type check followed by separate calls to `p.x()`/`p.y()`.

## Summary

- **`instanceof` pattern matching** (Java 16) eliminates redundant casts with flow-scoped pattern variables.
- **`switch` pattern matching** (Java 21) allows type-based cases, explicit `null` handling, and `when` guarded conditions.
- **Record patterns** (Java 21) destructure a record's components directly in a pattern, and nest for deep structures.
- Combined with **sealed classes** (Topic 2), exhaustive `switch` gives compiler-**proven** completeness over a closed set of cases — turning a classic silent runtime gap into a compile error.