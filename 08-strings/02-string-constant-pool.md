# The String Constant Pool

## Learning Objectives

- Understand exactly how and where the String Constant Pool lives, and how it relates to Module 02's memory model
- Precisely predict `==` behavior for every common String creation pattern
- Use `intern()` correctly, and understand what it actually does
- Never again be surprised by String equality behavior

## Prerequisites

[01 — String Immutability](01-string-immutability.md), Module 03 Topic 6 (`Integer` cache — the direct conceptual predecessor to this topic), Module 07 Topic 3 (`equals()`)

## Motivation

This is the single most practically important topic in this entire module — `String == String` behaving inconsistently is, by a wide margin, the most common real "gotcha" every Java developer eventually hits. If you internalized Module 03, Topic 6's `Integer` cache lesson, this topic will feel like a direct, expected extension of the exact same underlying idea, applied to Java's most heavily used type.

## Problem Statement

Strings are created **constantly** in real programs — string literals appear throughout source code in huge numbers. If every single string literal produced a brand-new object, memory usage would balloon needlessly, since many literals across a large codebase are frequently **identical** (`"GET"`, `"true"`, `"UTF-8"`, error messages, etc., often repeated dozens or hundreds of times across a codebase). Recall Topic 1: because `String` is immutable, sharing identical String objects across many variables is provably **safe** — no one can ever mutate a shared instance and corrupt another reference's view of it. The JVM exploits this safety with a dedicated optimization: the **String Constant Pool**.

## Concept: The String Constant Pool

> The **String Constant Pool** (technically part of the **Heap** in modern JVMs — moved there from the Method Area/PermGen as of Java 7, a genuinely important historical detail) is a special, dedicated storage area where the JVM keeps **exactly one copy** of every distinct String literal value, reusing it automatically for every identical literal encountered anywhere in the program.

```java
String s1 = "hello";
String s2 = "hello";
System.out.println(s1 == s2);   // true !! -- BOTH variables point to the SAME pooled object
```

```
              String Constant Pool (part of the Heap, since Java 7)
              ┌───────────────────────┐
              │     "hello"  (ONE object)  │◀────────┐
              └───────────────────────┘         │
                                                     │
    s1 ──────────────────────────────────────────┘
    s2 ──────────────────────────────────────────┘

    BOTH s1 and s2 point to the EXACT SAME pooled "hello" object
```

**Why is this exactly analogous to Module 03, Topic 6's `Integer` cache?** Both are the **exact same underlying optimization idea**: since the objects involved are **immutable** (Topic 1; and Module 03's wrapper classes are also immutable, for the same reasons), it's provably safe to silently share a single instance across many references — and the JVM/JDK exploits this safety specifically for extremely commonly-created values (small integers; string literals) to save memory. If you understood the `Integer` cache, you already understand the *conceptual* core of this topic — only the mechanism's specifics differ.

## `new String(...)` — Deliberately Bypassing the Pool

```java
String s1 = "hello";                // uses the POOL -- reuses the existing "hello", if present
String s2 = new String("hello");      // explicitly creates a BRAND NEW object, NOT from the pool

System.out.println(s1 == s2);          // false !! -- DIFFERENT objects
System.out.println(s1.equals(s2));      // true -- SAME logical content
```

```
  String Constant Pool                        Regular Heap (NOT pooled)
  ┌───────────────────────┐               ┌───────────────────────┐
  │     "hello"  (pooled)      │               │   "hello" (a SEPARATE,   │
  └───────────────────────┘               │    genuinely NEW object)  │
             ▲                                └───────────────────────┘
             │                                             ▲
             s1                                              s2
```

**`new String("hello")` deliberately, explicitly asks for a genuinely new, separate object** — bypassing the automatic pooling that plain literal syntax gets. This is precisely, mechanically why `s1 == s2` is `false` here, while `s1.equals(s2)` is correctly `true` (same logical content, different objects — exactly Module 07, Topic 3's `equals()`-vs-identity distinction, applied concretely).

> **Practical note:** `new String(...)` is almost never actually needed or recommended in modern code — it exists primarily for this exact kind of teaching example, and for a small number of legacy/specialized use cases (like deliberately forcing a non-pooled instance, or certain encoding-conversion constructors). In real, everyday code, always just use literal syntax (`"hello"`) unless you have a specific, deliberate reason not to.

## String Concatenation — Compile-Time vs. Runtime, A Genuinely Subtle Distinction

```java
String s1 = "hel" + "lo";              // COMPILE-TIME constant expression (both parts are literals)
String s2 = "hello";
System.out.println(s1 == s2);            // true !! -- the COMPILER combines "hel" + "lo" into "hello"
                                            // AT COMPILE TIME (Module 03, Topic 7's compile-time constants!),
                                            // so s1 is ALSO just a pooled literal, identical to s2
```

```java
String part = "hel";
String s3 = part + "lo";                // RUNTIME concatenation -- 'part' is a VARIABLE, not a literal
String s4 = "hello";
System.out.println(s3 == s4);             // false !! -- runtime concatenation ALWAYS creates a
                                             // brand-new String object (via StringBuilder internally --
                                             // Topic 4), which is NEVER automatically pooled
```

**Why the difference?** `"hel" + "lo"` involves **only** literals — the compiler can fully evaluate this at compile time (exactly Module 03, Topic 7's compile-time constant concept, extended to Strings), producing a single, already-known literal `"hello"` that gets pooled normally. `part + "lo"`, involving a **variable**, cannot be evaluated until the program actually runs — the JVM must genuinely construct a new String object at runtime (internally, via `StringBuilder`, Topic 4), and runtime-constructed Strings are **never** automatically added to the pool.

## `intern()` — Manually Requesting Pooling

```java
String s3 = new String("hello").intern();   // explicitly requests the POOLED version
String s4 = "hello";
System.out.println(s3 == s4);                 // true !! -- intern() returns the POOLED instance
```

**What `intern()` actually does:** it checks the String Constant Pool for an existing entry with the exact same content; if found, it returns **that pooled instance**; if not found, it **adds** the current String's content to the pool and returns a reference to that newly-pooled entry. `intern()` is the manual, explicit escape hatch for **opting a runtime-constructed String into pooling** — useful in specific, deliberate memory-optimization scenarios (e.g., deduplicating many repeated runtime-built Strings, like parsed tokens from a large file that happen to repeat often), but rarely needed in typical, everyday application code.

## The Complete Decision Table

| Creation pattern | Pooled? | `==` compared to an identical literal |
|---|---|---|
| `"hello"` (literal) | Yes, automatically | `true` |
| `"hel" + "lo"` (both parts compile-time constants) | Yes (compiler folds it into one literal) | `true` |
| `new String("hello")` | **No** | `false` |
| `part + "lo"` (where `part` is a variable) | **No** | `false` |
| `new String("hello").intern()` | Yes (explicitly requested) | `true` |

## The Correct, Universal Rule — Never Break It

> **Always use `.equals()` (never `==`) to compare String content — no exceptions, ever, in real code.**

The table above exists purely to build your *understanding* of the mechanism — **not** as a guide for when `==` happens to be "safe" to use for Strings. Even the "safe-seeming" cases (two identical literals) are fragile in practice: refactoring a literal into a variable, changing a compile-time-constant expression into a runtime one, or simply not being 100% certain how a given String was constructed **anywhere** in a real codebase, can silently flip `==`'s result from `true` to `false` without any other visible code change. `.equals()` is **always** correct, regardless of how the Strings were constructed — there's no legitimate reason to ever prefer `==` for String content comparison in real, production code.

## Real-World Analogy

Think of the String Constant Pool like a **library's single reference copy of a commonly requested book** — if ten different people ask for "the dictionary" (an identical literal, requested repeatedly throughout the program), the library hands out **the exact same physical reference copy** each time, rather than printing ten brand-new copies. But if someone explicitly walks in and asks the library to **specifically print them their own personal copy** (`new String(...)`), they get a genuinely separate, distinct physical book — even though its contents are word-for-word identical to the shared reference copy. `intern()` is like later **donating that personal copy back to be recognized as equivalent to the shared reference copy** going forward.

## Advantages

- Significant memory savings for programs with many repeated identical string literals (extremely common in real, large codebases).
- Made entirely safe by `String`'s immutability (Topic 1) — sharing pooled instances carries zero risk of one reference's "mutation" corrupting another's view.

## Disadvantages / Trade-offs

- The pooling behavior's inconsistency (literal vs. `new`, compile-time vs. runtime concatenation) is a genuine, real source of confusion and bugs for anyone relying on `==` instead of `.equals()`.
- `intern()`, if overused on many distinct runtime-generated strings, can itself bloat the pool and hurt performance — it's a deliberate, situational optimization tool, not a default habit.

## Best Practices

- **Always compare String content with `.equals()`**, never `==` — treat this as an absolute, exception-free rule.
- Avoid `new String(...)` in ordinary code — plain literal syntax is simpler, automatically pooled, and virtually always sufficient.
- Reserve `intern()` for specific, measured memory-optimization scenarios, not as routine practice.

## Common Mistakes

- Comparing Strings with `==` and having it "happen to work" during testing (because both were literals), then failing unpredictably in production once one of them originates from user input, file reading, or network data (which are never pooled by default).
- Assuming `s1 == s2` reliably indicates equal content for Strings, mirroring Module 03, Topic 6's `Integer` cache trap in an even more commonly encountered form.
- Forgetting that runtime concatenation (`variable + "literal"`) never produces an automatically-pooled result, even if the final content happens to match an existing pooled literal exactly.

## Interview Questions

1. **Q: Why does `String s1 = "hello"; String s2 = "hello"; s1 == s2` evaluate to `true`?**
   A: Both literals are automatically placed in (or retrieved from) the String Constant Pool — since identical literals always resolve to the exact same pooled object, `s1` and `s2` end up referencing the same instance, making identity comparison (`==`) return `true`.

2. **Q: Why does `new String("hello") == "hello"` evaluate to `false`?**
   A: `new String(...)` explicitly, deliberately creates a brand-new object outside the pool, regardless of whether an identical literal already exists in the pool — so it's never the same object as the pooled literal, even though its content is logically equal (`.equals()` would return `true`).

3. **Q: What does `String.intern()` do?**
   A: It checks the String Constant Pool for an existing entry matching the String's content; if found, returns that pooled instance; if not, adds the current content to the pool and returns a reference to the newly pooled entry. It's the manual mechanism for opting a runtime-constructed String into pooling.

4. **Q: Should you ever use `==` to compare Strings in real code?**
   A: No — always use `.equals()`. Even cases where `==` "happens" to return the expected result (like two identical literals) are fragile and can silently break under refactoring or when a String originates from a non-literal source (user input, file/network data), which is never automatically pooled.

## Summary

- The **String Constant Pool** (part of the Heap since Java 7) stores one shared instance per distinct literal value, safely enabled by `String`'s immutability (Topic 1) — directly analogous to Module 03, Topic 6's `Integer` cache.
- Literal syntax (and compile-time-constant concatenation) uses the pool automatically; `new String(...)` and runtime concatenation do not.
- `intern()` manually requests pooling for a runtime-constructed String.
- **Always use `.equals()`, never `==`, for String content comparison** — this is an absolute rule, not a situational judgment call.

## Exercises

1. Predict the `==` result for each of these pairs, and explain your reasoning for each: (a) `"abc" == "abc"`, (b) `new String("abc") == "abc"`, (c) `("a" + "b" + "c") == "abc"`, (d) `(someVariable + "c") == "abc"` where `someVariable = "ab"`.
2. Explain, precisely, why comparing Strings with `==` might "pass" during a developer's local testing but fail once real user input is involved.
3. Write a short program demonstrating `intern()` making a `new String(...)`-created object become `==`-equal to an existing pooled literal.

---

**Previous:** [01 — String Immutability](01-string-immutability.md) · **Next:** [03 — String Methods & API](03-string-methods-and-api.md)
