# Lambda Expressions

## Learning Objectives

- Write lambda expressions in every common syntactic form
- Understand closures and variable capture precisely, including the "effectively final" rule
- Compare lambdas directly against the anonymous classes they largely replace

## Prerequisites

[01 — Functional Interfaces](01-Functional-Interfaces.md), Module 06 Topic 6 (Anonymous classes), Module 03 Topic 7 ("effectively final" preview)

## Motivation

This is the syntax and semantics topic the rest of the module builds on. You've already seen lambdas used in passing since Module 10 — this topic makes every rule precise, especially variable capture, which is a genuinely common source of confusion (and a compile error) for newcomers.

## Lambda Syntax — Every Common Form

```java
// No parameters:
Runnable r = () -> System.out.println("running");

// One parameter (parentheses optional for exactly one parameter):
Consumer<String> printer = s -> System.out.println(s);
Consumer<String> printer2 = (s) -> System.out.println(s);   // also legal

// Multiple parameters (parentheses required):
Comparator<Integer> cmp = (a, b) -> a - b;

// With explicit types (usually unnecessary -- the compiler infers from context, Topic 1):
Comparator<Integer> cmp2 = (Integer a, Integer b) -> a - b;

// Multi-statement body (requires braces AND an explicit return):
Function<Integer, Integer> square = (x) -> {
    int result = x * x;
    return result;
};

// Single-expression body (no braces, no 'return' -- the expression's value IS the result):
Function<Integer, Integer> square2 = x -> x * x;
```

**The single-expression form (no braces, no `return`) is the idiomatic default for simple lambdas** — it's genuinely more concise, and the expression's value is implicitly the lambda's return value, mirroring the ternary operator's (Module 03, Topic 5) implicit-value nature.

## Lambdas vs. the Anonymous Classes They Replace — A Direct Comparison

Recall Module 06, Topic 6's anonymous class example:

```java
// Anonymous class (pre-Java 8 style):
Runnable r1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running!");
    }
};

// Lambda (Java 8+, the SAME thing, dramatically more concise):
Runnable r2 = () -> System.out.println("Running!");
```

**Beyond conciseness, recall Module 16, Topic 6's deeper explanation**: the anonymous class version genuinely compiles to a **separate `.class` file** at compile time; the lambda version uses `invokedynamic` to synthesize its implementation **at runtime**, the first time that specific call site executes — a real, structural difference under the hood, not just a syntax shortcut, directly explaining why lambdas are generally lighter-weight.

## Closures — Capturing Variables From the Enclosing Scope

```java
int threshold = 10;

Predicate<Integer> isAboveThreshold = n -> n > threshold;   // 'threshold' is CAPTURED from the
                                                                // ENCLOSING scope -- the lambda "closes over" it
```

**A lambda that references a variable from its surrounding scope is called a closure** — it "closes over" that variable, retaining access to it even if the lambda is later invoked in a completely different context (passed to another method, stored for later use, executed on a different thread, Module 15).

## The "Effectively Final" Rule — Precisely, At Last

Recall Module 03, Topic 7's preview: **a lambda can only capture local variables that are either explicitly `final`, or "effectively final"** (never reassigned after their initial assignment, even without the explicit keyword):

```java
int count = 0;
Runnable r = () -> System.out.println(count);   // LEGAL -- 'count' is effectively final
                                                    // (never reassigned anywhere)

int counter = 0;
counter = 5;                                        // REASSIGNED
Runnable r2 = () -> System.out.println(counter);      // COMPILE ERROR: variable counter is
                                                          // accessed from within a lambda expression,
                                                          // needs to be final or effectively final
```

**Why does this restriction exist, precisely?** This connects directly to Module 02, Topic 3's Stack model and Module 15's entire concurrency chapter. **A lambda might genuinely outlive the method call that created it** — it could be stored, passed elsewhere, or run on a completely different thread (Module 15, Topic 1) **long after** the enclosing method's Stack frame (and its local variables, Module 02, Topic 3) has already been popped and destroyed. To make this safe, the lambda **doesn't actually capture the variable itself** — it captures a **snapshot copy of its value**, taken once, at the point the lambda is created. **If the original variable could still be reassigned afterward, the lambda's captured copy and the "real" variable could silently drift apart** — a genuinely confusing inconsistency the effectively-final rule eliminates entirely, by simply disallowing the scenario where this ambiguity could ever arise.

```
 Method's Stack frame (Module 02, Topic 3):        The lambda (might outlive this frame!):
 ┌─────────────────────┐                         ┌─────────────────────────┐
 │  count = 0              │                         │  captured COPY of count = 0 │
 └─────────────────────┘                         └─────────────────────────┘
        │                                                         │
        ▼ (method returns, frame POPPED)                          ▼ (lambda can still be
   count no longer exists AT ALL                                    invoked later, safely,
                                                                       using its own captured copy)
```

**Important, precise clarification**: this rule applies specifically to **local variables and parameters**. Capturing an **instance field** (Module 06) works differently — the lambda captures a reference to the **enclosing object** (`this`) instead, and can freely read/write that field, since the object itself (living on the Heap, Module 02, Topic 3) genuinely does persist independently of any particular method call's Stack frame.

```java
class Counter {
    int count = 0;   // an INSTANCE field, not a local variable
    Runnable incrementer = () -> count++;   // LEGAL -- freely mutates the FIELD, no
                                                // effectively-final restriction applies here at all
}
```

## Real-World Analogy

Think of a lambda's variable capture like **taking a photograph of a whiteboard before leaving the room** — the photo (the captured copy) preserves exactly what the whiteboard showed at that specific moment, and remains perfectly valid and readable even after you've left the room and the whiteboard has since been erased or repurposed for something else entirely (the enclosing method's Stack frame being destroyed). The effectively-final rule is like a **rule against taking that photo of a whiteboard someone is still actively, continuously erasing and rewriting** — if the board kept changing after your photo, your photo and "the current whiteboard" would tell two different, conflicting stories, which is precisely the confusing inconsistency Java's rule prevents by disallowing it.

## Advantages

- Dramatically more concise than the equivalent anonymous class, with a lighter-weight runtime implementation (Module 16, Topic 6).
- Closures let lambdas carry meaningful context from their creation site, enabling powerful, expressive patterns (callback logic, deferred computation) with minimal ceremony.
- The effectively-final rule prevents a genuinely confusing class of "captured value silently changed underneath me" bugs, by construction.

## Disadvantages / Trade-offs

- The effectively-final restriction can occasionally feel limiting, requiring a workaround (like using a single-element array, or an `AtomicInteger`, Module 15, Topic 4, as a mutable holder) for genuinely mutable-across-invocations state — though this need is less common than newcomers often initially assume.
- Multi-statement lambda bodies (requiring explicit braces and `return`) lose some of the single-expression form's elegance — a signal that a named, extracted method might sometimes be clearer than an increasingly complex inline lambda.

## Best Practices

- Prefer the concise, single-expression lambda form whenever the logic is genuinely simple.
- Understand that a lambda captures a value snapshot, not a live reference, for local variables — this is precisely why the effectively-final rule exists, not an arbitrary restriction to work around cleverly.
- Extract genuinely complex, multi-statement lambda bodies into a well-named regular method (referenced via a method reference, Topic 3) when a lambda starts feeling too dense to read comfortably inline.

## Common Mistakes

- Attempting to reassign a captured local variable inside or after creating a lambda that uses it, triggering the effectively-final compile error.
- Assuming a lambda captures a *live* reference to a local variable (as it would for an instance field) — it captures a value snapshot instead.
- Writing overly complex, multi-statement lambda bodies where a named, extracted method would be clearer.

## Interview Questions

1. **Q: Why must local variables captured by a lambda be final or effectively final?**
   A: A lambda might outlive the method call that created it (stored, passed elsewhere, run on another thread), long after the enclosing method's Stack frame and its local variables have been destroyed (Module 02, Topic 3). The lambda captures a snapshot copy of the variable's value, not a live reference — if the original variable could still be reassigned afterward, the captured copy and the "real" variable could diverge confusingly, which the effectively-final rule prevents entirely.

2. **Q: Does the effectively-final restriction apply to instance fields captured by a lambda?**
   A: No — a lambda capturing an instance field actually captures a reference to the enclosing object (`this`), which persists independently on the Heap regardless of any method call's Stack frame, so the field can be freely read and mutated without restriction.

3. **Q: What's the structural difference between a lambda and the equivalent anonymous class, beyond syntax conciseness?**
   A: An anonymous class compiles to a separate, named `.class` file at compile time. A lambda uses `invokedynamic` (Module 16, Topic 6) to synthesize its implementation dynamically at runtime, the first time its call site executes — making lambdas generally more lightweight.

## Summary

- Lambda syntax: `() -> expr`, `param -> expr`, `(p1, p2) -> expr`, or `(params) -> { statements; return value; }` for multi-statement bodies.
- Lambdas are a dramatically more concise, generally lighter-weight (Module 16, Topic 6) replacement for the anonymous classes they conceptually mirror.
- **Closures**: lambdas capture variables from their enclosing scope; local variables/parameters must be **final or effectively final** (a value snapshot, safe against the enclosing Stack frame's destruction); instance fields, by contrast, are captured via the enclosing object reference and remain freely mutable.

## Exercises

1. Write a lambda implementing `Comparator<String>` comparing strings by length, using the single-expression form.
2. Predict and explain the compile error produced by capturing a local variable in a lambda, then reassigning that variable afterward.
3. Write a class with an instance field captured and mutated by a lambda stored as one of its own fields, and explain why this doesn't trigger the effectively-final restriction that would apply to an equivalent local variable.

---

**Previous:** [01 — Functional Interfaces](01-Functional-Interfaces.md) · **Next:** [03 — Method References](03-Method-References.md)
