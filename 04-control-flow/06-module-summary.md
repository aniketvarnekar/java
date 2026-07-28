# Module 04 Summary

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

## What's Next

Module 04 completed your control-flow toolkit: branching and repetition. Combined with Module 03's data vocabulary, you now have everything needed to write real, non-trivial procedural logic. **Module 05 — OOP** is where Java's true identity begins: classes, objects, and the four pillars (Encapsulation, Abstraction, Inheritance, Polymorphism) that everything else in this course — Collections, Exceptions, Streams, Concurrency — is built on top of.