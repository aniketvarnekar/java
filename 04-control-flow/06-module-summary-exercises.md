# Module 04 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-module-overview.md) has been covered:

- [x] **If-Else & Conditional Logic** — `if`/`else`/`else-if`, the strict `boolean`-only condition rule, the dangling-else problem and why braces solve it, nested conditionals vs. `else-if` chains
- [x] **Switch Statement & Expression** — classic `switch` and fall-through (dangerous vs. intentional use), legal switch subjects and the compile-time-constant case-label rule, the modern `switch` expression (arrow syntax, `yield`, exhaustiveness checking), and why Java kept both forms
- [x] **While & Do-While Loops** — condition-first vs. condition-last execution, the "runs zero times" vs. "runs at least once" distinction, accidental vs. intentional infinite loops, loop-variable scope
- [x] **For Loops** — exact three-part execution order, `for` as syntactic sugar over `while`, the enhanced for-each loop and when it's preferable, nested loops and multiplicative iteration counts
- [x] **Break, Continue & Labeled Statements** — precise behavior of each, the innermost-loop-only default, labeled `break`/`continue` for multi-level loop control, when labels are (and aren't) the right tool

## Practical Connections

- **Modern `switch` expressions** (Topic 2) are used constantly in real Java 17+/21+ codebases for mapping enum-like states to behavior — e.g., mapping an HTTP status code range to a response category, or an order status enum to a UI label — with the compiler's exhaustiveness checking catching forgotten cases when a new status is added later.
- **For-each loops** (Topic 4) are the default iteration style throughout the entire Java Collections Framework (Module 10) — nearly all real-world iteration over `List`/`Set`/`Map` entries in production code uses for-each or the closely related Streams API (Module 18), not indexed `for` loops.
- **Labeled breaks** (Topic 5) appear routinely in real search/validation code — e.g., validating every field of every row in a batch of records and stopping at the first invalid one found, across nested loops.
- **`do-while` menu loops** (Topic 3) are a direct, real pattern in CLI tools and REPL-style applications — you'll use exactly this shape again when you build interactive command-line programs later in this course.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| `while` vs `do-while` | `while` checks its condition first (may run zero times); `do-while` checks last (always runs at least once). |
| Classic `switch` statement vs. modern `switch` expression | The classic form is a statement with fall-through-by-default and no produced value; the modern arrow form is an expression, produces a value, never falls through, and supports exhaustiveness checking. |
| `break` vs `continue` | `break` exits the entire loop immediately; `continue` skips only the rest of the current iteration and proceeds to the next one. |
| Unlabeled vs. labeled `break`/`continue` in nested loops | Unlabeled affects only the innermost enclosing loop; labeled targets a specifically named outer loop directly. |
| Indexed `for` vs. enhanced for-each | Indexed `for` gives you the index (needed for writing back, backward iteration, lockstep iteration); for-each is simpler and avoids off-by-one bugs but provides no index at all. |

## Consolidated Interview Questions (Module 04)

1. Why does `if (someInt)` fail to compile in Java, unlike in C or JavaScript?
2. In an ambiguous, brace-less nested `if`/`else`, which `if` does the `else` bind to?
3. What is fall-through in a classic `switch` statement, and what's one legitimate, intentional use for it?
4. What are the key improvements the modern `switch` expression (Java 14+) makes over the classic statement?
5. What does `yield` do inside a `switch` expression's block branch?
6. What's the fundamental difference between `while` and `do-while`, in terms of when the condition is checked?
7. Give an example of a problem where `do-while` is the more natural fit than `while`.
8. What is the exact execution order of a classic `for` loop's three header parts?
9. Is a `for` loop functionally different from a `while` loop? What's the one genuine practical difference?
10. When would you prefer an indexed `for` loop over an enhanced for-each loop?
11. By default, does `break` inside a nested loop affect the outer loop too?
12. What problem do labeled `break`/`continue` solve, and what was the workaround before labels existed?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** From memory, draw the dangling-else example from Topic 1 and explain, without re-reading, exactly which `if` the `else` binds to and why.
2. **Hands-on:** Write a modern `switch` expression that maps a `char` grade (`'A'` through `'F'`) to a GPA `double` value, including a `default` branch that throws an `IllegalArgumentException` for invalid input.
3. **Hands-on:** Write a `do-while` loop implementing a simple number-guessing game loop structure (pseudocode is fine for the "guess" input — focus on the loop shape itself), explaining why `do-while` fits this problem better than `while`.
4. **Hands-on:** Write nested loops printing a multiplication table (1-10), then modify it with a labeled `break` to stop the entire table generation early if any product exceeds 50.
5. **Conceptual:** Explain, referencing Module 03's compile-time constants, why `case someVariable:` is illegal in a `switch` but `case SOME_FINAL_CONSTANT:` is legal.
6. **Conceptual:** A colleague writes a `for` loop counter as `float i = 0.0f; for (; i != 1.0f; i += 0.1f)`. Explain, referencing Module 03's floating-point precision discussion, why this loop may never terminate as expected.
7. **Synthesis:** Write a method that searches a 2D `int[][]` grid for a target value using nested loops and a labeled `break`, returning the `{row, col}` position where found, or `{-1, -1}` if not found anywhere in the grid.

## What's Next

Module 04 completed your control-flow toolkit: branching and repetition. Combined with Module 03's data vocabulary, you now have everything needed to write real, non-trivial procedural logic. **Module 05 — OOP** is where Java's true identity begins: classes, objects, and the four pillars (Encapsulation, Abstraction, Inheritance, Polymorphism) that everything else in this course — Collections, Exceptions, Streams, Concurrency — is built on top of.

---

**Previous:** [05 — Break, Continue & Labeled Statements](05-break-continue-and-labeled-statements.md) · **Module Overview:** [00 — Module Overview](00-module-overview.md)

**Type "Continue" to begin Module 05 — OOP.**
