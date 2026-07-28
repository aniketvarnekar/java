# Switch Statement & Expression

## Learning Objectives

- Write a classic `switch` statement correctly, including intentional fall-through
- Explain exactly why fall-through is dangerous, and how the modern `switch` fixes it
- Write modern `switch` **expressions** (Java 14+) with arrow syntax and `yield`
- Know when to reach for `switch` over an `else-if` chain

## Prerequisites

[01 — If-Else & Conditional Logic](01-if-else-and-conditional-logic.md), Module 03 Topic 7 (`final` and compile-time constants)

## Motivation

`switch` has one of the most interesting stories in modern Java's evolution: it started in 1996 as a direct, largely unmodified copy of C's `switch` — including a notorious footgun (fall-through) that has caused real production bugs for decades. Java 14 introduced a **genuinely redesigned** `switch` **expression** specifically to fix this, without removing the old form (Java's backward-compatibility philosophy, Module 01, Topic 2, in direct action). Understanding both forms — and precisely *why* the new one exists — is both a practical skill and a strong interview signal.

## Problem Statement

An `else-if` chain testing the same variable against many possible values repeatedly (`if (day == 1) ... else if (day == 2) ... else if (day == 3) ...`) is verbose and doesn't clearly communicate "we're selecting one of several fixed cases based on a single value." `switch` exists specifically for this "match one value against several fixed alternatives" pattern.

## The Classic `switch` Statement

```java
int day = 3;
String dayName;

switch (day) {
    case 1:
        dayName = "Monday";
        break;
    case 2:
        dayName = "Tuesday";
        break;
    case 3:
        dayName = "Wednesday";
        break;
    default:
        dayName = "Unknown";
        break;
}
```

**What types can you `switch` on?** `byte`, `short`, `char`, `int` (and their wrapper types, via autoboxing/unboxing — Module 03, Topic 6), `String` (since Java 7), and `enum` types (covered when you reach enums). Notably **not** `long`, `float`, `double`, or `boolean` — `switch` is fundamentally about matching against a fixed, small set of discrete, exact values, which floating-point types are unsuitable for (recall Module 03, Topic 2's discussion of floating-point imprecision), and `boolean` only has 2 values, for which `if`/`else` is already the natural, idiomatic tool.

**Case labels must be compile-time constants** (Module 03, Topic 7) — this is exactly why that topic's discussion of compile-time constant inlining matters here: `case someVariable:` is illegal; `case SOME_FINAL_CONSTANT:` is legal, precisely because the compiler needs to know every case value at compile time to build an efficient lookup/jump table.

## Fall-Through — The Classic `switch`'s Most Dangerous Feature

**Without a `break`, execution continues into the NEXT case, regardless of whether that case's value matches:**

```java
int day = 2;
switch (day) {
    case 1:
        System.out.println("Monday");
    case 2:
        System.out.println("Tuesday");
    case 3:
        System.out.println("Wednesday");
        break;
    default:
        System.out.println("Unknown");
}
```
**Output:**
```
Tuesday
Wednesday
```
Execution starts at the matching `case 2:` label, prints `"Tuesday"`, and — since there's **no `break`** after it — **falls through** into `case 3:`'s body too, printing `"Wednesday"` as well, only stopping because `case 3:` finally has a `break`. `case 1:`'s code never runs (execution jumps *directly* to the matching label — fall-through only affects what happens *after* a match, not before it).

```
switch(day) jumps DIRECTLY to the matching case label:

  case 1:  [NEVER REACHED -- day doesn't match 1]
  case 2:  ──▶ ENTRY POINT (day == 2) ── print "Tuesday" ── (no break) ──┐
  case 3:                                                                │◀── falls through here
           print "Wednesday" ── break ── EXIT switch                     │
  default: [never reached, because case 3's break stopped us first]
```

**Why does fall-through exist as the *default* behavior at all — wasn't this an obviously bad design?** It wasn't seen as obviously bad in 1972 (when C introduced it) — fall-through has a genuine, deliberate intentional use case: **grouping multiple case values to share identical behavior**:

```java
switch (day) {
    case 6:
    case 7:
        System.out.println("Weekend");
        break;
    default:
        System.out.println("Weekday");
        break;
}
```
Here, `case 6:` (Saturday) has **no body of its own** — it deliberately falls through into `case 7:`'s body, so both values 6 and 7 share the exact same "Weekend" behavior. This specific, intentional pattern is legitimate and common — the *danger* is purely **accidental** fall-through, from a forgotten `break`, which is what caused decades of real bugs and is precisely what the modern `switch` expression (below) eliminates entirely.

## Why Modern Java Fixed This: The `switch` Expression (Java 14+)

```java
int day = 3;
String dayName = switch (day) {
    case 1 -> "Monday";
    case 2 -> "Tuesday";
    case 3 -> "Wednesday";
    case 6, 7 -> "Weekend";       // comma-separated multiple labels -- the SAFE way to
                                    // express what fall-through used to accomplish
    default -> "Unknown";
};
```

**What changed, precisely:**
1. **`switch` is now an expression, not just a statement** — it directly **produces a value** you can assign, exactly like the ternary operator (Module 03, Topic 5). No more declaring a variable above the switch and assigning it separately in every branch.
2. **The arrow (`->`) form has NO fall-through, ever** — each case's code runs, and the switch immediately exits, no `break` needed and no risk of accidental fall-through.
3. **Multiple case labels can share one arrow directly**, via a comma-separated list (`case 6, 7 ->`) — this is the *safe*, deliberate replacement for the old "stack empty cases to fall through" trick.
4. **Exhaustiveness checking**: for certain switch subjects (notably `enum` types, and — a modern addition — `sealed` type hierarchies, Module 23), the compiler can verify at compile time that every possible case is handled, refusing to compile otherwise if `default` is missing and not every value is covered — catching a real class of "forgot to handle a new enum value" bugs before runtime.

## `yield` — For Multi-Statement Switch Expression Branches

The arrow form's single-expression shorthand (`case 1 -> "Monday";`) works great for simple cases. For a branch needing multiple statements, use a block with `yield` to specify the produced value:

```java
String category = switch (score / 10) {
    case 10, 9 -> "A";
    case 8 -> "B";
    default -> {
        System.out.println("Below B, calculating further...");   // multiple statements allowed
        yield (score >= 60) ? "Passing" : "Failing";               // yield specifies the RESULT VALUE
    }
};
```

`yield` is to a `switch` expression's block branch what `return` is to a method — it specifies "this is the value this branch produces," without exiting the enclosing method.

## Classic `switch` Statement vs. Modern `switch` Expression — Side by Side

| | Classic `switch` statement | Modern `switch` expression (Java 14+) |
|---|---|---|
| Produces a value directly? | No — you must assign inside each branch | **Yes** — the whole `switch` IS an expression |
| Fall-through by default? | **Yes** — must remember `break` in every branch | **No**, with `->` syntax — never falls through |
| Multiple values per branch | Stacked empty `case` labels (relies on fall-through) | Comma-separated: `case 6, 7 ->` |
| Exhaustiveness checking | No | **Yes**, for `enum`/`sealed` subjects (compiler-enforced) |
| Syntax for a case body | `case X: ... break;` | `case X -> ...;` or `case X -> { ... yield ...; }` |

**You can still use `case X:` (colon) syntax inside a `switch` expression too**, but then you're back to needing `yield` in every branch and losing the fall-through protection — the arrow form is unambiguously the recommended modern style; the colon form remains primarily for the classic `switch` **statement** (no produced value) and legacy code.

## Why Java Kept the Old Form Instead of Replacing It

This is a direct, concrete example of the backward-compatibility philosophy from Module 01, Topic 2: millions of lines of existing Java code use the classic `switch` statement, correctly, with intentional fall-through in places. Removing it would break that code. Instead, Java **added** the new expression form alongside the old statement form — both are fully valid, permanently — while clearly establishing the new arrow-based expression form as the modern, recommended default for new code.

## Real-World Analogy

Think of the classic `switch` like an **old elevator with no automatic doors** — if you don't manually press "close doors" (`break`) after your floor, the elevator (execution) just keeps rolling forward to the next floor, and the one after that, whether you meant it to or not. The modern `switch` expression is like a **modern elevator that automatically stops and opens exactly at your selected floor, every time** — no manual "close doors" step required, and no risk of accidentally continuing past your intended stop.

## Advantages of the Modern Switch Expression

- Eliminates accidental fall-through entirely — a real, historically significant bug class, gone by construction.
- Produces a value directly, eliminating a whole category of "declared but not yet assigned" boilerplate and reducing lines of code.
- Compiler-enforced exhaustiveness checking (for `enum`/`sealed` types) catches "forgot to handle a case" bugs at compile time instead of at runtime.

## Disadvantages / Trade-offs

- The classic `switch` statement remains everywhere in existing/legacy code — you must still be able to read and reason about fall-through correctly, even if you never intentionally write it in new code.
- The arrow form's rule that *intentional* multi-value grouping now requires comma-separated labels (rather than the old "stack empty cases" idiom) is a small, real syntax difference to internalize when switching between old and new code.

## Best Practices

- **Write new code using the modern `switch` expression (arrow form) by default** — it's strictly safer for the vast majority of use cases.
- If you must use (or maintain) a classic `switch` statement, **always include a `break` at the end of every case**, even the last one before `default` — protects against bugs introduced later if cases are reordered.
- Always include a `default` branch, even when you believe every case is covered — an explicit `default` (throwing an exception, or logging an unexpected value) turns a silent "nothing happened" bug into a loud, immediate, debuggable failure.

## Common Mistakes

- Forgetting a `break` in a classic `switch` statement, causing unintended fall-through — the single most common `switch`-related bug in Java history.
- Attempting to `switch` on a `long`, `float`, `double`, or `boolean` — none of these are legal `switch` subjects.
- Using a non-compile-time-constant expression as a `case` label — `case` labels must be resolvable at compile time.
- Assuming the arrow form (`->`) allows fall-through the same way the colon form does — it explicitly does not, by design.

## Interview Questions

1. **Q: What is fall-through in a classic Java `switch` statement, and why is it dangerous?**
   A: Without a `break`, execution continues into the next case's code after a match, regardless of whether that next case's value actually matches — a forgotten `break` silently executes unintended code, a real, historically common bug source. It exists as default behavior because it has a legitimate intentional use (grouping multiple case values to share behavior), but accidental fall-through is the actual danger.

2. **Q: What are the key differences between the classic `switch` statement and the modern `switch` expression (Java 14+)?**
   A: The modern expression directly produces a value (usable in an assignment), never falls through with arrow (`->`) syntax, supports comma-separated multiple case labels as the safe replacement for fall-through grouping, and provides compiler-enforced exhaustiveness checking for `enum`/`sealed` type subjects.

3. **Q: What does `yield` do in a `switch` expression, and how does it relate to `return`?**
   A: `yield` specifies the value a multi-statement block branch of a `switch` expression produces — analogous to how `return` specifies a method's result, but without exiting the enclosing method; it exits only the `switch` expression's block branch.

4. **Q: Can you `switch` on a `double`? Why or why not?**
   A: No — legal `switch` subjects are `byte`/`short`/`char`/`int` (and wrappers), `String`, and `enum` types. `switch` is designed for exact matching against a small, discrete, fixed set of values, which floating-point types (`float`/`double`) are unsuitable for due to representational imprecision (Module 03, Topic 2).

## Summary

- The classic `switch` **statement** matches a value against `case` labels and, without an explicit `break`, **falls through** into subsequent cases — dangerous when accidental, occasionally useful when intentional (grouping shared behavior).
- The modern `switch` **expression** (Java 14+) directly produces a value, uses arrow (`->`) syntax with **no fall-through ever**, supports comma-separated multi-value case labels, and provides compiler-enforced exhaustiveness for `enum`/`sealed` subjects.
- `yield` specifies a value from a multi-statement block branch of a switch expression, analogous to `return` for methods.
- Both forms remain fully valid Java (backward compatibility) — but the modern expression form is the recommended default for new code.