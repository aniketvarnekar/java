# Exception Fundamentals & Hierarchy

## Learning Objectives

- Understand the complete `Throwable` class hierarchy
- Write correct basic `try`/`catch`/`finally` blocks
- Understand exactly how the JVM searches for a matching `catch` block, including multiple catches and inheritance

## Prerequisites

Module 05 Topic 4 (Inheritance), Module 01 Topic 3 ("Robust" feature)

## Motivation

You've encountered exceptions constantly since early in this course, always reactively ("this throws X"). This topic builds the proactive, structural understanding: what an exception actually *is* as a Java object, the full class hierarchy it belongs to, and precisely how control flow jumps when one is thrown.

## Problem Statement

Programs encounter error conditions constantly — a file that doesn't exist, a network call that times out, an array index that's out of bounds, invalid user input. Recall Module 01, Topic 3's "Robust" feature: Java's philosophy is to **fail predictably and safely**, rather than crash unpredictably or silently corrupt state (contrast with C, where many of these conditions produce undefined behavior or require manually checking return codes after every single operation). **Exceptions are the structured mechanism Java uses to achieve this.**

## Concept: What an Exception Actually Is

> An **exception** is an ordinary Java **object** — an instance of a class extending `Throwable` — that represents an abnormal condition, **"thrown"** at the point it occurs and **"caught"** by code elsewhere prepared to handle it, with the JVM automatically unwinding the call stack in between.

## The Complete `Throwable` Hierarchy

```
                                    Object
                                       │
                                  Throwable
                                       │
                   ┌───────────────────┴───────────────────┐
                   ▼                                         ▼
                 Error                                    Exception
          (serious, unrecoverable                              │
           JVM/system-level problems)          ┌────────────────┴────────────────┐
                   │                              ▼                                  ▼
     ┌─────────────┼─────────────┐        RuntimeException                  (all other Exception
     ▼             ▼             ▼        (UNCHECKED)                        subclasses --
OutOfMemoryError StackOverflowError  ...        │                              CHECKED)
                                          ┌───────┼───────┬──────────────┐            │
                                          ▼       ▼       ▼              ▼            ▼
                              NullPointerException  ArrayIndexOutOf  ClassCastException  IOException
                                                      BoundsException                     SQLException
                                                                                           (etc.)
```

**Every single exception you've encountered in this course inherits from `Throwable`** — the root of this entire hierarchy, and (per Module 07, Topic 1) itself ultimately inheriting from `Object`.

### `Error` — Serious, Typically Unrecoverable Problems

`Error` and its subclasses (`OutOfMemoryError`, `StackOverflowError` — both from Module 02, Topic 3!) represent conditions **outside normal application control** — typically JVM-level or system-level failures your application code has no realistic way to recover from. **You should almost never catch `Error`** — if the JVM has run out of memory or a thread's stack has overflowed, attempting to gracefully "handle" and continue is usually futile or actively dangerous (the JVM may already be in a compromised state).

### `Exception` — Conditions a Well-Written Program CAN Reasonably Handle

`Exception` and its subclasses represent conditions your application **can and should** anticipate and handle — a missing file, invalid input, a failed network call. This branch splits further into **checked** and **unchecked** exceptions — the subject of Topic 2, previewed here: `RuntimeException` and its subclasses are **unchecked**; everything else under `Exception` is **checked**.

## Basic `try`/`catch`/`finally`

```java
try {
    int result = 10 / 0;               // throws ArithmeticException
    System.out.println("Never reached");
} catch (ArithmeticException e) {
    System.out.println("Caught: " + e.getMessage());   // "Caught: / by zero"
} finally {
    System.out.println("Always runs");
}
```

**Output:**
```
Caught: / by zero
Always runs
```

- **`try`**: the block of code that might throw an exception.
- **`catch`**: runs **only if** an exception of the matching type (or a subtype — inheritance applies here, exactly like polymorphism, Module 05, Topic 5) is thrown inside the `try` block. Execution jumps **immediately** to the matching `catch`, skipping any remaining code in the `try` block entirely.
- **`finally`**: runs **always** — whether an exception was thrown or not, whether it was caught or not, even if the `try`/`catch` block contains a `return` statement (full depth: Topic 3).

## How the JVM Finds a Matching `catch` — Precisely

When an exception is thrown, the JVM searches **enclosing `catch` blocks, in order**, looking for the **first** one whose declared type **matches** the thrown exception's actual type — where "matches" means the thrown exception **is** that type, or a **subtype** of it (Module 05, Topic 4's IS-A relationship, applied directly):

```java
try {
    throw new ArrayIndexOutOfBoundsException("bad index");
} catch (NullPointerException e) {
    System.out.println("NPE caught");          // does NOT match -- unrelated exception type
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("AIOOBE caught");           // MATCHES -- this one runs
} catch (Exception e) {
    System.out.println("Generic Exception caught");   // never reached -- the AIOOBE catch already matched
}
```

**Multiple `catch` blocks are checked top to bottom, and only the FIRST matching one runs** — this is precisely why **catch blocks must be ordered from MOST specific to LEAST specific**:

```java
try {
    // ...
} catch (Exception e) {                          // ⚠️ WRONG ORDER -- this catches EVERYTHING first!
    System.out.println("Generic");
} catch (ArrayIndexOutOfBoundsException e) {      // COMPILE ERROR: unreachable catch block --
    System.out.println("Specific");                 // this can NEVER be reached, since Exception
}                                                     // already matches every possible thrown exception
```

**Why does this produce a compile error, rather than just silently never running?** The compiler performs **reachability analysis** — since `ArrayIndexOutOfBoundsException IS-A Exception` (Module 05, Topic 4), any exception matching the second, more specific `catch` would **already** have matched the first, broader one — making the second `catch` block provably, permanently unreachable. Java flags this as a genuine, real error rather than silently allowing dead code, directly echoing Module 01, Topic 3's "Robust" philosophy of catching real mistakes as early as possible.

## `getMessage()`, `getCause()`, `printStackTrace()`

```java
try {
    Integer.parseInt("abc");
} catch (NumberFormatException e) {
    System.out.println(e.getMessage());     // "For input string: \"abc\""
    e.printStackTrace();                       // prints the FULL call chain that led to this exception
}
```

`Throwable` (Module 07, Topic 1's method-list pattern, applied here) provides several useful inherited methods: `getMessage()` (a human-readable description, often set via the exception's constructor), `printStackTrace()` (prints the exact sequence of method calls that led to the exception — invaluable for debugging), and `getCause()` (used for exception chaining — full depth Topic 5).

## Real-World Analogy

Think of the `Throwable` hierarchy like a **hospital's triage classification**. `Error` is like a **catastrophic, building-wide emergency** (a fire, a structural collapse) — not something any individual doctor (your application code) is expected or equipped to personally resolve; the appropriate response is evacuation, not treatment. `Exception` is like an **individual patient's medical condition** — something a doctor (your `catch` block) is specifically trained and equipped to diagnose and treat, provided they're actually the right kind of specialist for that particular condition (the matching `catch` type). `finally` is like the **hospital's mandatory post-visit checkout procedure** — it happens for every single patient, regardless of what treatment (if any) they received, or whether they were successfully treated at all.

## Advantages

- Structured exception handling separates error-handling logic from normal program logic, improving readability compared to manually checking return codes after every operation.
- The type-based `catch` matching system, combined with inheritance, lets you handle related error conditions with varying degrees of specificity, exactly like polymorphism.
- `finally`'s absolute guarantee provides a reliable place for cleanup logic, regardless of how a `try` block actually exits.

## Disadvantages / Trade-offs

- Exception handling has real performance overhead compared to simple conditional checks (though modern JVMs have optimized this substantially) — not a concern for typical application-level exception use, but relevant for extremely hot-path code (Module 22).
- Overly broad `catch (Exception e)` blocks can silently swallow genuinely unexpected problems if not used carefully — a real anti-pattern covered fully in Topic 5.

## Best Practices

- Order `catch` blocks from most specific to least specific — the compiler enforces this for provably unreachable cases, but remain deliberate about ordering even when it wouldn't be a hard error.
- Never catch `Error` (or its subclasses) in typical application code — these represent conditions your code generally cannot meaningfully recover from.
- Use `getMessage()`/`printStackTrace()`/logging frameworks to preserve diagnostic information — never silently swallow an exception without at least logging it (a real, common anti-pattern, fully covered in Topic 5).

## Common Mistakes

- Ordering `catch` blocks from general to specific, triggering an "unreachable catch block" compile error.
- Assuming `finally` might not run if the `try` block has a `return` statement — it always runs (full nuance: Topic 3).
- Catching and silently ignoring exceptions (an empty `catch` block) — hiding real bugs rather than handling them.

## Interview Questions

1. **Q: What's the difference between `Error` and `Exception`?**
   A: Both extend `Throwable`. `Error` represents serious, typically unrecoverable JVM/system-level problems (like `OutOfMemoryError`) that application code generally shouldn't attempt to catch/handle. `Exception` represents conditions a well-written program can reasonably anticipate and handle — further split into checked and unchecked subtypes (Topic 2).

2. **Q: How does the JVM decide which `catch` block handles a thrown exception, when multiple are present?**
   A: It checks `catch` blocks in the order they're written, top to bottom, running the **first** one whose declared type matches the thrown exception's actual type or any of its supertypes — exactly following the IS-A relationship from inheritance (Module 05, Topic 4).

3. **Q: Why does ordering catch blocks from general (`Exception`) to specific (`ArrayIndexOutOfBoundsException`) cause a compile error?**
   A: The compiler performs reachability analysis — since the specific exception type IS-A the general one, any exception matching the specific catch would already have matched the broader, earlier one, making the specific catch block provably unreachable dead code.

## Summary

- Every exception is an object extending `Throwable`, which splits into `Error` (serious, generally uncatchable) and `Exception` (application-handleable, further split into checked/unchecked — Topic 2).
- `try` marks risky code; `catch` handles a matching exception type (checked top-to-bottom, first match wins); `finally` always runs.
- Catch blocks must be ordered most-specific to least-specific; violating this is a compile error, not just a logic bug, since Java's compiler proves unreachability via the inheritance hierarchy.

## Exercises

1. Draw the `Throwable` hierarchy from memory, correctly placing `Error`, `Exception`, `RuntimeException`, and at least three specific exception types you've encountered in this course.
2. Write a `try`/`catch`/`finally` block that divides by zero, catches `ArithmeticException`, and prints a message from `finally` — predict and verify the exact output order.
3. Explain, precisely, why the compiler flags an "unreachable catch block" error for badly-ordered catch clauses, referencing Module 05's inheritance/IS-A concept directly.

---

**Previous:** [00 — Module Overview](00-module-overview.md) · **Next:** [02 — Checked vs. Unchecked Exceptions](02-checked-vs-unchecked-exceptions.md)
