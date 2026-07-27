# Break, Continue & Labeled Statements

## Learning Objectives

- Use `break` and `continue` correctly, and explain precisely what each one exits/skips
- Understand why `break` and `continue` inside nested loops only affect the *innermost* loop by default
- Use labeled `break`/`continue` to control an outer loop from inside a nested one
- Know when labeled statements are the right tool, and when they're a readability smell

## Prerequisites

[04 — For Loops](04-For-Loops.md), [03 — While & Do-While Loops](03-While-And-Do-While-Loops.md)

## Motivation

`break` and `continue` give you fine-grained control over loop execution beyond what the loop's own condition expresses — "stop entirely, right now" or "skip the rest of this iteration, but keep looping." Both are simple individually; the genuinely tricky part — and a real interview favorite — is precisely how they behave inside **nested** loops, which is where labeled statements come in.

## `break` — Exit the Loop Immediately

```java
for (int i = 0; i < 10; i++) {
    if (i == 5) {
        break;         // immediately exits the ENTIRE for loop, skipping i = 5 through 9
    }
    System.out.println(i);
}
System.out.println("Done");
```
**Output:**
```
0
1
2
3
4
Done
```
`break` immediately terminates the **entire loop** (not just the current iteration) — execution jumps to the first statement **after** the loop's closing brace, and the loop's condition is never checked again.

You've actually already seen `break` used in a different context — the classic `switch` statement (Topic 2) — where it plays a directly analogous role: "exit this construct immediately, don't continue processing further."

## `continue` — Skip to the Next Iteration

```java
for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) {
        continue;       // skip the rest of THIS iteration's body, go straight to the update step
    }
    System.out.println(i);
}
```
**Output:**
```
1
3
5
7
9
```
`continue` skips the **remainder of the current iteration's body only** — execution jumps directly to the loop's **update** step (in a `for` loop) or **condition check** (in a `while`/`do-while` loop), and the loop **continues** running normally from there. Unlike `break`, the loop is **not** terminated.

```
break:

┌────────────────────────────────┐
│ Loop Body                      │
│                                │
│ ...                            │
│ break;                         │──────────────▶ Exit the Loop
│                                │
│ (Remaining statements          │
│  are never executed)           │
└────────────────────────────────┘
```

```
continue:

┌────────────────────────────────┐
│ Loop Body                      │
│                                │
│ ...                            │
│ continue;                      │──────────────▶ UPDATE (for)
│                                │               or
│ (Remaining statements          │               CONDITION (while/do-while)
│  are never executed)           │
└────────────────────────────────┘                         │
                                                          ▼
                                                  Next Iteration
```

**A genuinely common mistake worth calling out explicitly:** in a `for` loop specifically, `continue` jumps to the **update** step, not directly back to the condition check — meaning the counter still increments correctly even when `continue` fires. This is precisely *why* the earlier for-loop trace ("update runs after the body, before the next condition check," Topic 4) matters here: understanding that exact order is what lets you correctly predict that `continue` in a `for` loop doesn't accidentally skip the update and cause an infinite loop.

## `break`/`continue` in Nested Loops — The Default Behavior

**By default, `break` and `continue` only affect the *innermost* enclosing loop** — this is the single most important, most commonly tested fact about these two keywords:

```java
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) {
            break;    // exits ONLY the inner (j) loop -- the outer (i) loop is completely unaffected
        }
        System.out.println("i=" + i + " j=" + j);
    }
}
```
**Output:**
```
i=0 j=0
i=1 j=0
i=2 j=0
```
The inner loop breaks out as soon as `j == 1`, every single time — but the **outer** loop keeps running all three of its own iterations regardless, completely unaware that the inner loop was interrupted. If your intent was actually "stop **everything**, both loops, entirely" — plain `break` cannot express that. This is exactly the gap **labeled statements** fill.

## Labeled `break` — Exiting an Outer Loop From Within an Inner One

A **label** is an identifier followed by a colon, placed directly before a loop:

```java
outer:                                  // this is a LABEL, naming the loop right below it
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) {
            break outer;                 // exits the OUTER loop directly, skipping ALL remaining
                                            // iterations of BOTH loops entirely
        }
        System.out.println("i=" + i + " j=" + j);
    }
}
System.out.println("Done");
```
**Output:**
```
i=0 j=0
Done
```
`break outer;` immediately terminates the loop **labeled** `outer` — which also, necessarily, terminates every loop nested inside it (there's no way to "keep" an inner loop running once its enclosing outer loop has ended). Execution jumps straight to the first statement after `outer`'s closing brace.

## Labeled `continue` — Skipping an Outer Loop's Current Iteration From Within an Inner One

```java
outer:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) {
            continue outer;    // skips the REST of the outer loop's current iteration (abandoning
                                  // the inner loop entirely for this pass), goes straight to the
                                  // OUTER loop's update step (i++)
        }
        System.out.println("i=" + i + " j=" + j);
    }
}
```
**Output:**
```
i=0 j=0
i=1 j=0
i=2 j=0
```
Here, `continue outer;` abandons the inner loop as soon as `j == 1` (identical trigger to the earlier `break` example) but, instead of exiting everything, it jumps straight to the **outer** loop's next iteration (`i++`, then re-check `i < 3`) — the outer loop keeps going, it's only the *inner* loop's remaining work for this particular outer pass that gets skipped.

## Why Labeled Statements Exist At All

**The problem they solve:** without labels, there is genuinely **no way** to break out of (or continue) anything but the single innermost loop directly surrounding a `break`/`continue` statement. Before labels existed in a language, the only workaround for "exit two nested loops at once" was an awkward boolean "found" flag checked in both loops' conditions — functionally correct, but noticeably more verbose and less direct about actual intent:

```java
// The AWKWARD workaround, without labels:
boolean found = false;
for (int i = 0; i < 3 && !found; i++) {
    for (int j = 0; j < 3; j++) {
        if (j == 1) {
            found = true;
            break;              // still only breaks the inner loop directly...
        }                         // ...the outer loop only stops because of the added !found check
        System.out.println("i=" + i + " j=" + j);
    }
}
```
Labels express the same intent far more directly, without an extra flag variable polluting the loop's condition.

## Best Practices — When to Use Labels, and When Not To

Labeled statements are a **legitimate, real Java feature** — not deprecated, not "bad practice" in any absolute sense — but they should be reached for **deliberately, only for genuinely nested-loop control-flow needs**, not casually:

- **Good use case:** searching a 2D grid/matrix for a specific value, wanting to stop *all* searching the instant it's found (a very common, genuinely idiomatic use of `break` with a label).
- **Questionable use case:** using labels to work around what should really be restructured into a separate, well-named method that simply `return`s early — often, extracting nested-loop logic into its own method and using a plain `return` (instead of a labeled `break`) produces clearer code (full method-design guidance: Module 06).

```java
// Idiomatic labeled-break use case: searching a 2D grid
int target = 7;
int foundRow = -1, foundCol = -1;

search:
for (int row = 0; row < grid.length; row++) {
    for (int col = 0; col < grid[row].length; col++) {
        if (grid[row][col] == target) {
            foundRow = row;
            foundCol = col;
            break search;    // stop searching ENTIRELY the instant it's found -- clean and direct
        }
    }
}
```

## Real-World Analogy

Think of `break` like **pulling a fire alarm inside one specific room** — everyone in that room evacuates immediately, but people in *other, separate rooms* (an outer loop, unaffected by an unlabeled `break`) keep working, unaware anything happened. A **labeled** `break` is like pulling a **building-wide** fire alarm instead — it evacuates the targeted room *and* every room nested inside/dependent on it, all at once, directly, by design.

## Advantages

- `break`/`continue` give precise, direct control over loop execution beyond what a loop's own condition alone can express.
- Labeled statements solve the genuine "control an outer loop from inside a nested one" problem directly, without an awkward extra flag-variable workaround.

## Disadvantages / Trade-offs

- Overusing labeled statements — especially with multiple, deeply nested labels — can genuinely hurt readability; it's a real, if occasionally overstated, "smell" worth being deliberate about, not reflexive.
- `break`/`continue` inside deeply nested conditionals-within-loops can sometimes obscure a method's overall control flow if overused; extracting logic into well-named helper methods (Module 06) is frequently the clearer alternative once nesting gets deep.

## Best Practices

- Reserve labeled `break`/`continue` for genuine multi-level loop control (like the 2D-search example) — not as a default habit.
- Always name labels descriptively (`search:`, `outer:`) rather than something meaningless — a label is still an identifier, and deserves the same naming care as any other (Module 03, Topic 1).
- When labeled logic starts feeling tangled, consider whether extracting the nested loops into their own method (using a plain `return` for early exit) would be clearer — a very common, genuinely good refactor once you reach Module 06.

## Common Mistakes

- Assuming an unlabeled `break`/`continue` inside a nested loop affects the *outer* loop — by default, it only ever affects the *innermost* enclosing loop.
- Forgetting that `continue` (in a `for` loop) still runs the update step before re-checking the condition — mistakenly fearing it causes an infinite loop, when it does not.
- Placing a label on something other than directly the loop it's meant to name (labels must immediately precede the loop statement they label).

## Interview Questions

1. **Q: What's the difference between `break` and `continue`?**
   A: `break` immediately terminates the entire enclosing loop; execution resumes after the loop. `continue` skips only the remainder of the current iteration's body, then proceeds to the loop's next iteration (its update step in a `for` loop, or condition check in `while`/`do-while`) — the loop itself keeps running.

2. **Q: By default, which loop does `break` affect inside a nested loop?**
   A: Only the **innermost** loop directly enclosing it — an outer loop is completely unaffected by an unlabeled `break` inside a nested inner loop.

3. **Q: What problem do labeled `break`/`continue` solve, and what did developers have to do before labels to achieve the same effect?**
   A: They let you target an *outer* loop directly from within a nested inner loop — either exiting it entirely (`break label;`) or skipping to its next iteration (`continue label;`). Without labels, achieving the same effect required an awkward extra boolean "flag" variable checked in the outer loop's own condition, which is functionally equivalent but less direct about actual intent.

## Summary

- `break` exits the entire current loop immediately; `continue` skips only the rest of the current iteration and proceeds to the next one.
- Both, by default, only affect the **innermost** enclosing loop.
- **Labeled** `break`/`continue` (`label:` before a loop, then `break label;`/`continue label;`) let you target an outer loop directly from within nested loops — a legitimate feature for genuine multi-level loop control, best reserved for cases like early-exit searches over nested structures.
- Overusing labels, or reaching for them where a well-named extracted method with a plain `return` would be clearer, is a real readability concern worth being deliberate about.

## Exercises

1. Predict the full printed output of a nested loop (outer `i` from 0-2, inner `j` from 0-2) where an unlabeled `continue` fires when `j == 1` — then predict the output again if it were `continue outer;` (with an appropriate label) instead.
2. Write a labeled-break search over a 2D array (`int[][] grid`) that stops searching entirely the instant it finds a target value, printing the row and column where it was found.
3. Rewrite the "awkward boolean flag" workaround example from this topic using a labeled `break` instead, and explain which version more directly communicates its intent to a reader.
4. Explain precisely why `continue` inside a `for` loop does not risk causing an infinite loop, referencing the exact execution order established in Topic 4.

---

**Previous:** [04 — For Loops](04-For-Loops.md) · **Next:** [06 — Module Summary, Interview Questions & Exercises](06-Module-Summary-Exercises.md)
