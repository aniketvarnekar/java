# Module 23 Summary

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-module-overview.md) has been covered:

- [x] **Records** — the boilerplate problem, syntax, compact constructors, what's auto-generated, records vs. classes vs. Lombok, and why records are usually a poor fit for JPA entities
- [x] **Sealed Classes** — restricting inheritance on purpose, `sealed`/`permits`/`non-sealed`, and sealed hierarchies vs. `enum`
- [x] **Pattern Matching** — `instanceof` pattern matching, `switch` pattern matching (including `null` and `when` guards), record patterns/destructuring, and exhaustive `switch` over sealed types
- [x] **Modern Java Recap & What's New Through Java 25** — a full, dated Java 8→25 timeline with pointers to every prior module's coverage, plus first-time coverage of Sequenced Collections, Stream Gatherers, unnamed variables, Scoped Values, the Foreign Function & Memory API, and String Templates' withdrawal

## Practical Connections

- **Spring Boot DTOs and API request/response bodies** are, in modern codebases, almost always written as records (Topic 1) rather than hand-written classes — directly reducing boilerplate in every REST controller.
- **Domain modeling for a fixed set of business states** (an order's status: `Pending`, `Shipped`, `Cancelled`, each carrying different data) is a textbook use case for sealed interfaces (Topic 2) combined with exhaustive `switch` (Topic 3) — every state-handling `switch` in the codebase is compiler-verified complete.
- **High-throughput, thread-per-request-style services** (Module 15, Topic 8's Virtual Threads revisited in Topic 4) increasingly use `ScopedValue` instead of `ThreadLocal` for request-scoped context (request IDs, security principals) precisely because of the memory-leak and mutation risks `ThreadLocal` carries at Virtual-Thread scale.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| Record vs. ordinary class | Record: compiler-generated constructor/accessors/`equals`/`hashCode`/`toString`, always immutable, cannot extend a class. Class: full manual control, can be mutable, can extend another class. |
| Sealed hierarchy vs. `enum` | Sealed: fixed set of **structurally different** types. `enum`: fixed set of **uniform** constants sharing one shape. |
| `instanceof` pattern matching vs. classic cast | Pattern matching binds an already-cast, flow-scoped variable in one step; the classic idiom requires a separate, redundant manual cast. |
| `ThreadLocal` vs. `ScopedValue` | `ThreadLocal`: mutable via `.set()` anywhere, manual cleanup required, leak risk. `ScopedValue`: immutable per binding, automatically and deterministically unbound at scope end — the modern, Virtual-Thread-friendly choice. |
| `collect()` vs. `gather()` | `collect()`: terminal operation producing one final result. `gather()`: intermediate operation, composable with further stream operations, for custom transformations `collect()` can't express as cleanly. |

## What's Next

Module 23 completed the language's evolution story — from records and sealed classes through pattern matching and the full Java 8-25 timeline, you now have a complete, dated map of modern Java. **Module 24 — Interview Preparation**, the final module of this course, now consolidates everything you've learned across all 23 modules into a focused, structured interview-readiness resource: the most commonly asked Java interview questions organized by topic, coding problems with full solutions and explanations, system-design-adjacent Java questions, and guidance on how to present your knowledge clearly and confidently in a real interview setting.