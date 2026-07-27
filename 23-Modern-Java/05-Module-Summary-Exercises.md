# Module 23 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-Module-Overview.md) has been covered:

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

## Consolidated Interview Questions (Module 23)

1. What specific problem do records solve, and which Java version finalized them?
2. What does a compact constructor do differently from a normal constructor in a record?
3. Why can't a record extend another class?
4. What specific problem do sealed classes solve, and which Java version finalized them?
5. What must every direct permitted subtype of a sealed type declare, and why?
6. How does `switch` pattern matching handle `null` differently from a classic `switch`?
7. How do sealed classes and `switch` pattern matching combine to provide a guarantee classic Java never had?
8. What is a record pattern, and what does it do in one step that classic code needs two steps for?
9. Walk through Java's major features from Java 8 to Java 21 in order.
10. What problem does `SequencedCollection` solve?
11. Why is `ScopedValue` considered a better fit than `ThreadLocal` for Virtual-Thread-heavy code?
12. What happened to Java's String Templates feature?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** From memory, explain why records auto-generating `equals()`/`hashCode()` eliminates an entire category of bug that Module 07, Topic 2 warned about with hand-written implementations.
2. **Hands-on:** Model a `PaymentMethod` sealed interface with `CreditCard`, `BankTransfer`, and `Cash` as record implementations, then write an exhaustive `switch` expression that processes each variant using record patterns.
3. **Hands-on:** Rewrite a classic `instanceof`-then-cast chain as a `switch` with type patterns and a `when` guard.
4. **Conceptual:** Explain, in your own words, why sealed classes combined with exhaustive `switch` turn a previously silent runtime risk into a compile-time guarantee.
5. **Conceptual:** Explain why `ScopedValue` is a better architectural fit than `ThreadLocal` specifically for a Structured-Concurrency-based (Module 15, Topic 8) service handling millions of Virtual Threads.
6. **Synthesis:** Take a domain you're familiar with (e.g., an e-commerce order's lifecycle) and design it using records for data and a sealed hierarchy for its fixed set of states, then write one exhaustive `switch` that would need to change — and fail to compile until updated — if a new state were added.

## What's Next

Module 23 completed the language's evolution story — from records and sealed classes through pattern matching and the full Java 8-25 timeline, you now have a complete, dated map of modern Java. **Module 24 — Interview Preparation**, the final module of this course, now consolidates everything you've learned across all 23 modules into a focused, structured interview-readiness resource: the most commonly asked Java interview questions organized by topic, coding problems with full solutions and explanations, system-design-adjacent Java questions, and guidance on how to present your knowledge clearly and confidently in a real interview setting.

---

**Previous:** [04 — Modern Java Recap & What's New Through Java 25](04-Modern-Java-Recap-And-Whats-New.md) · **Module Overview:** [00 — Module Overview](00-Module-Overview.md)

**Type "Continue" to begin Module 24 — Interview Preparation.**
