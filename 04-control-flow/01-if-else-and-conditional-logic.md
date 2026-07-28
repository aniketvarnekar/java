# If-Else & Conditional Logic

## Learning Objectives

- Write `if`, `if-else`, and `else-if` chains correctly
- Understand exactly how Java resolves nested/ambiguous `if`-`else` pairings
- Know when braces are optional, and why omitting them is a real, documented bug source

## Prerequisites

Module 03 (Java Basics), especially Topic 5 (Operators — `boolean` expressions)

## Motivation

`if`/`else` is likely already familiar from other languages — the value here is precision about Java-specific rules: exactly what counts as a valid condition, how the compiler resolves ambiguous nesting, and a real, famous production bug (Apple's "goto fail" bug, in C, but the exact same shape of mistake is possible in Java) caused by exactly the brace-omission issue covered below.

## Problem Statement

A program frequently needs to do different things depending on runtime data — charge a different shipping rate depending on order weight, show a different UI state depending on login status, etc. `if`/`else` is the fundamental branching construct that makes this possible.

## Concept: `if`, `if-else`, `else-if`

```java
if (condition) {
    // runs only if condition is true
}
```

```java
if (age >= 18) {
    System.out.println("Adult");
} else {
    System.out.println("Minor");
}
```

```java
if (score >= 90) {
    grade = 'A';
} else if (score >= 80) {
    grade = 'B';
} else if (score >= 70) {
    grade = 'C';
} else {
    grade = 'F';
}
```

**A critical, Java-specific rule inherited from Module 03, Topic 5:** the condition inside `if (...)` must be an expression of type `boolean` — **exactly**. Unlike C/JavaScript/Python, there is **no implicit conversion** from `int` or any other type to `boolean`:

```java
int flag = 1;
if (flag) { ... }    // COMPILE ERROR in Java: incompatible types: int cannot be converted to boolean
```

This directly reinforces the robustness rationale from Module 03: eliminating an entire historical bug category (accidental `=` instead of `==`, "truthy" values behaving unpredictably) by making the type system refuse to compile anything but a genuine `boolean`.

## `else-if` Is Not a New Keyword — It's Just Nesting

A commonly-missed detail: `else if` is **not special Java syntax**. It's simply an `if` statement used as the entire body of an `else` clause:

```java
if (score >= 90) {
    grade = 'A';
} else {
    if (score >= 80) {          // this "if" IS the entire body of the outer "else"
        grade = 'B';
    } else {
        grade = 'C';
    }
}
```

The `else if score >= 80` form you're used to writing is just this same structure, with the (fully optional) braces around the inner `if` omitted for readability — proving `else if` chains are, structurally, just nested `if`/`else`. This matters for understanding the next section precisely.

## The Dangling-Else Problem

Consider:
```java
if (a > 0)
    if (b > 0)
        System.out.println("Both positive");
else
    System.out.println("??? Which if does this belong to ???");
```

**Java's rule (shared with C, C++, and most C-family languages):** an `else` is always matched to the **nearest, most recently unmatched `if`** — regardless of indentation. In the example above, the `else` binds to the **inner** `if (b > 0)`, **not** the outer `if (a > 0)`, even though the indentation visually (and misleadingly) suggests otherwise:

```
if (a > 0)                          <- outer if, UNMATCHED (no else attached to it!)
    if (b > 0)                      <- inner if, this is the one the else actually binds to
        System.out.println("Both positive");
    else                             <- binds to the NEAREST unmatched if (the inner one)
        System.out.println("??? Which if does this belong to ???");
```

If `a = 5` and `b = -3`: the outer `if (a > 0)` is true, so we enter it. The inner `if (b > 0)` is false, so we go to its `else` — printing the confusing message, **even though `a` genuinely is positive**. Indentation alone lied to the reader about the actual behavior.

**The fix — always use braces**, even for single-statement bodies:

```java
if (a > 0) {
    if (b > 0) {
        System.out.println("Both positive");
    }
} else {
    System.out.println("a is not positive");
}
```

## Why Braces Matter — Beyond Just Dangling-Else

Even without nested `if`s, omitting braces on single-statement bodies is a real, historically significant bug source, because it's dangerously easy to *later* add a second statement, assuming (incorrectly) that it's still part of the conditional body:

```java
if (isAdmin)
    grantAccess();
    logAdminAction();   // ⚠️ LOOKS like part of the if, but it's NOT --
                          // it runs UNCONDITIONALLY, every single time, regardless of isAdmin!
```
Only `grantAccess();` is actually controlled by the `if` — `logAdminAction();` runs unconditionally, since Java (unlike Python) uses braces, **not indentation**, to define a block's boundaries. This exact class of bug (a security-relevant statement silently executing unconditionally due to a missing brace) has caused real, famous production security vulnerabilities in C-family code.

**Best practice, stated directly and without exception: always use braces `{ }` for every `if`/`else` body, even single-statement ones.** Nearly every professional Java style guide mandates this for exactly the reasons demonstrated above.

## Nested Conditionals vs. `else-if` Chains — Choosing the Right Shape

```java
// Nested (implies a hierarchical/dependent relationship between conditions)
if (user.isLoggedIn()) {
    if (user.isAdmin()) {
        showAdminPanel();
    }
}

// else-if chain (implies mutually exclusive, parallel alternatives)
if (grade == 'A') {
    bonus = 100;
} else if (grade == 'B') {
    bonus = 50;
} else if (grade == 'C') {
    bonus = 10;
}
```

**Why does the shape matter, not just correctness?** Nesting communicates "condition B is only meaningful/relevant *given* condition A" (a genuine dependency). An `else-if` chain communicates "exactly one of these mutually exclusive branches applies" (parallel alternatives). Choosing the shape that matches your actual logic's meaning — not just whichever happens to produce correct output — is a real code-quality signal, and makes intent obvious to a future reader without them needing to trace through the logic first.

## Real-World Analogy

Think of `if`/`else` like a **flowchart decision diamond** — you've probably drawn one in a school or planning context. The dangling-else problem is exactly like a hand-drawn flowchart where two arrows visually *appear* to both point away from the same diamond, but the actual, authoritative rule (drawn precisely) is that one specific arrow belongs to the *nearest* diamond, not the one that merely looks closest on paper. Braces in Java are like **explicitly re-drawing that diamond's boundary box** so there's no room for a reader (or the compiler) to misjudge which diamond an arrow belongs to.

## Advantages

- Java's refusal to implicitly convert non-`boolean` types to `boolean` conditions eliminates an entire, historically real bug category at compile time.
- Mandatory-in-practice braces (by convention, strongly enforced by style guides/linters) eliminate the dangling-else and unconditional-execution bug classes described above.

## Disadvantages / Trade-offs

- Deep nesting of `if`/`else` can become genuinely hard to read ("arrow code" / the "pyramid of doom") — a real, common code-quality issue addressed by techniques like early returns (guard clauses), covered practically once you reach Module 06 (Classes/methods).

## Best Practices

- Always use braces, even for single-statement bodies — no exceptions, regardless of how "obviously safe" a particular case looks.
- Prefer `else-if` chains for mutually exclusive, parallel alternatives; reserve genuine nesting for cases where an inner condition truly only makes sense given an outer one.
- Consider early-return "guard clauses" (`if (!valid) return;` at the top of a method) instead of wrapping an entire method body in one large `if` block — reduces nesting depth and improves readability (revisited with full context in Module 06).

## Common Mistakes

- Omitting braces and later adding a second statement, unaware it now runs unconditionally.
- Misjudging which `if` a bare, brace-less `else` binds to in nested conditionals — always resolved by "nearest unmatched `if`," regardless of indentation.
- Writing `if (booleanExpression == true)` instead of simply `if (booleanExpression)` — redundant, and specifically `if (booleanExpression == false)` is both redundant and less readable than `if (!booleanExpression)`.

## Interview Questions

1. **Q: Does Java allow `if (someInt)` the way C does?**
   A: No — Java's `if` condition must be an expression of type `boolean` exactly; there is no implicit conversion from `int` (or any other type) to `boolean`. This is a deliberate robustness decision (Module 01/03) eliminating a historical class of C-family bugs.

2. **Q: In a nested `if`/`else` without braces, which `if` does a bare `else` bind to?**
   A: The **nearest, most recently unmatched `if`** — regardless of the source code's indentation, which can visually mislead a reader into assuming a different (incorrect) association.

3. **Q: Why do most Java style guides mandate braces even for single-statement `if` bodies?**
   A: To eliminate two real bug classes: the dangling-else ambiguity in nested conditionals, and the risk of a later-added second statement silently running unconditionally outside the intended conditional body, since Java uses braces (not indentation) to define block boundaries.

## Summary

- `if`/`else`/`else-if` require a genuine `boolean` condition — Java performs no implicit truthy/falsy conversion.
- `else if` is not special syntax — it's a nested `if` used as an `else` clause's entire body, with braces conventionally omitted for readability.
- An `else` without braces binds to the **nearest unmatched `if`**, which can visually contradict misleading indentation — always use braces to remove this ambiguity entirely.
- Choose nested `if` vs. `else-if` chains based on whether conditions are genuinely dependent (nested) or mutually exclusive alternatives (chain).