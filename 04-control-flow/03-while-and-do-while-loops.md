# While & Do-While Loops

## Learning Objectives

- Write `while` and `do-while` loops correctly
- Understand precisely when each executes its body relative to its condition check
- Recognize and avoid accidental infinite loops
- Understand loop variables' scope and lifetime, tying back to Module 02

## Prerequisites

[01 — If-Else & Conditional Logic](01-if-else-and-conditional-logic.md)

## Motivation

Loops are how a program does the same kind of work repeatedly without you writing that work out by hand N times. `while` and `do-while` are the two simplest loop forms — the distinction between them (condition-first vs. condition-last) is small in syntax but meaningfully changes behavior, and is a genuine, common source of off-by-one-style bugs when chosen carelessly.

## Problem Statement

Sometimes you need to repeat an action **while some condition remains true** — read input until the user quits, retry a network call until it succeeds or a limit is hit, process items until a queue is empty. The exact number of repetitions often isn't known in advance (unlike a `for` loop, Topic 4, which is usually used when the repeat count *is* known or easily computed).

## `while` Loop — Condition Checked FIRST

```java
int count = 0;
while (count < 5) {
    System.out.println("Count: " + count);
    count++;
}
```

**Execution model:** check the condition → if `true`, run the body, then check the condition again → repeat → if `false` (at any check), skip the body and exit the loop entirely.

```
                 ┌──────────────────────────┐
                 │     Check Condition      │◄─────────────────────┐
                 └────────────┬─────────────┘                      │
                              │                                    │
                         True │                                    │
                              ▼                                    │
                 ┌──────────────────────────┐                      │
                 │      Execute Body        │──────────────────────┘
                 └────────────┬─────────────┘
                              │
                        False (Condition)
                              ▼
                       ┌────────────────┐
                       │ Exit the Loop  │
                       └────────────────┘
```

**Key consequence: a `while` loop's body might run ZERO times**, if the condition is already `false` the very first time it's checked:

```java
int count = 10;
while (count < 5) {
    System.out.println("This never prints");
}
// program continues here immediately -- the body never ran even once
```

## `do-while` Loop — Condition Checked LAST

```java
int count = 0;
do {
    System.out.println("Count: " + count);
    count++;
} while (count < 5);
```

**Execution model:** run the body **first**, unconditionally → *then* check the condition → if `true`, run the body again → repeat → if `false`, exit.

```
                 ┌──────────────────────────┐
                 │      Execute Body        │◄─────────────────────┐
                 └────────────┬─────────────┘                      │
                              │                                    │
                              ▼                                    │
                 ┌──────────────────────────┐                      │
                 │     Check Condition      │                      │
                 └────────────┬─────────────┘                      │
                         true │                                    │
                              └────────────────────────────────────┘
                         false
                          │
                          ▼
                   ┌────────────────┐
                   │ Exit the Loop  │
                   └────────────────┘
```

**Key consequence: a `do-while` loop's body ALWAYS runs at least once**, even if the condition is `false` from the very start:

```java
int count = 10;
do {
    System.out.println("This prints ONCE, even though count is already >= 5");
} while (count < 5);
```

## The Precise Difference, and Why It Matters

| | `while` | `do-while` |
|---|---|---|
| Condition checked | **Before** the body's first run | **After** the body's first run |
| Minimum number of executions | **0** (may never run) | **1** (always runs at least once) |
| Best suited for | "Keep doing this *as long as* X is true," where X might already be false | "Do this, *then keep repeating* while X remains true" — the body is meant to happen at least once by design |

**Concrete example of the difference mattering:** a menu-driven program showing options and processing user input is a classic `do-while` use case — you always want to **show the menu at least once**, before you have any input to check a condition against:

```java
int choice;
do {
    System.out.println("1. New Order  2. View Orders  3. Exit");
    choice = readUserChoice();     // (details not shown -- reads user input)
    // ... process choice ...
} while (choice != 3);
```
Using a plain `while` here would require awkwardly duplicating the menu-display/input-reading logic once *before* the loop (to have something to check) and again *inside* it — `do-while`'s "always run once" guarantee avoids that duplication naturally, because it matches the actual shape of the problem: "show the menu, get input, and keep repeating that until the user picks exit."

## Avoiding Accidental Infinite Loops

A loop runs forever if its condition never becomes `false` — almost always because the code responsible for eventually making it `false` is missing, wrong, or unreachable:

```java
int i = 0;
while (i < 10) {
    System.out.println(i);
    // ⚠️ FORGOT: i++;  -- 'i' never changes, condition (i < 10) is ALWAYS true, loops forever
}
```

```java
int i = 10;
while (i > 0) {
    System.out.println(i);
    i++;   // ⚠️ BUG: increments instead of decrements -- moves AWAY from making the condition false
}
```

**Deliberate infinite loops also exist and are legitimate** — a server's main request-handling loop, for instance, is *meant* to run forever (until an explicit shutdown signal via `break` or a return/exit elsewhere):

```java
while (true) {
    Request request = waitForNextRequest();
    handleRequest(request);
    if (shouldShutdown(request)) {
        break;      // the deliberate, explicit exit mechanism -- Topic 5 covers break in depth
    }
}
```

**The distinguishing factor between a bug and a legitimate design isn't "does the condition ever become false in the source code" — it's whether the infinite loop is *intentional*, with an explicit, deliberate exit mechanism (`break`, `return`, or process termination) built in.**

## Loop Variable Scope — Tying Back to Module 02

A variable declared **inside** a loop's condition or body has a scope limited to that loop:

```java
while (condition) {
    int temp = computeSomething();   // 'temp' is a fresh LOCAL variable, on the Stack,
                                       // re-created (conceptually) EVERY iteration, and
                                       // completely gone once this iteration's block ends
}
System.out.println(temp);   // COMPILE ERROR -- 'temp' is out of scope here, it doesn't exist
```

This is a direct, practical application of Module 02's JVM Stack model and Module 03's local-variable scope rules: each iteration's `{ }` block is its own scope, and a variable declared inside it exists only for that iteration's Stack-frame lifetime (conceptually — the JVM doesn't literally push a whole new frame per loop iteration, but the scoping/visibility rules behave exactly as if it did).

## Real-World Analogy

Think of a `while` loop like **checking if there's still coffee in the pot *before* pouring a cup** — if the pot's already empty, you never pour at all. Think of a `do-while` loop like **committing to pour one cup first, then checking if there's enough left for another** — you're guaranteed at least one cup, by design, before the check ever happens.

## Advantages

- `while` cleanly expresses "repeat only if a condition holds," including correctly handling the case where it never should run at all.
- `do-while` cleanly expresses "always do this once, then keep repeating conditionally" — matching problems (like menu loops) that are naturally shaped that way, without artificial code duplication.

## Disadvantages / Trade-offs

- Both are easy to accidentally turn into infinite loops if the loop-progress logic (whatever eventually flips the condition to `false`) is missing or incorrect.
- `do-while` is less commonly used in practice than `while` and the `for` loop (Topic 4) — its "runs at least once" guarantee is genuinely needed less often than you might initially expect, so overusing it where a plain `while` would be clearer is a minor but real style concern.

## Best Practices

- Choose `do-while` specifically when your problem genuinely requires "always run at least once" — don't reach for it out of habit; default to `while` unless that specific guarantee is exactly what you need.
- Always double-check that every loop has a clear, reachable path to making its condition eventually `false` (or an explicit `break`), especially after any refactor that touches the loop's body.
- Keep loop bodies focused — if a `while`/`do-while` body grows very large, consider extracting it into a well-named method (full method-design guidance: Module 06).

## Common Mistakes

- Forgetting to update the loop-control variable inside the body, causing an infinite loop.
- Using `while` when the problem actually requires "always run once" behavior, leading to awkward, duplicated setup code before the loop.
- Declaring a variable inside a loop body and then trying to use it after the loop ends, not realizing it's out of scope there.

## Interview Questions

1. **Q: What's the fundamental difference between `while` and `do-while`?**
   A: `while` checks its condition *before* running the body, so the body may run zero times. `do-while` checks its condition *after* running the body, guaranteeing the body runs at least once, even if the condition is false from the start.

2. **Q: Give a concrete example of a problem where `do-while` is a more natural fit than `while`.**
   A: A menu-driven CLI program — you want to display the menu and read user input at least once before you have anything meaningful to check a loop condition against; `do-while` avoids duplicating that display/read logic both before and inside a `while` loop.

3. **Q: How can a `while` loop become infinite by accident, and how is that different from an intentional infinite loop?**
   A: Accidentally, when the code responsible for eventually making the condition false is missing or incorrect (e.g., forgetting to increment a counter). Intentionally, `while (true)` is a legitimate pattern (e.g., a server's request loop) when paired with an explicit, deliberate exit mechanism like `break` or `return` elsewhere in the body — the difference is the presence of a real, reachable exit path.

## Summary

- `while` checks its condition **first** — body may run zero times.
- `do-while` checks its condition **last** — body always runs at least once.
- Choose based on whether your problem genuinely needs the "at least once" guarantee.
- Both require a reachable path to make the condition false (or an explicit `break`) to avoid unintentional infinite loops; intentional infinite loops (`while (true)`) are legitimate when paired with a deliberate exit mechanism.
- Variables declared inside a loop's body are scoped to that loop, consistent with Module 03's local variable scope rules.