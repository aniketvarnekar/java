# Module 12 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-Module-Overview.md) has been covered:

- [x] **Exception Fundamentals & Hierarchy** — the complete `Throwable`/`Error`/`Exception` tree, basic `try`/`catch`/`finally`, exact catch-matching rules and unreachable-catch compile errors
- [x] **Checked vs. Unchecked Exceptions** — the mechanical distinction, the original design reasoning, and a full, balanced treatment of the genuine, real controversy
- [x] **Try-Catch-Finally Deep Dive** — multi-catch, `finally`'s absolute guarantee, the `return`/`throw`-inside-`finally` footgun, and precise stack-unwinding mechanics
- [x] **Try-With-Resources & `AutoCloseable`** — the complete compiler-generated mechanism, multi-resource reverse-order closing, suppressed exceptions, and the full resolution of Module 07's `finalize()` story
- [x] **Custom Exceptions & Best Practices** — designing meaningful custom exceptions, exception chaining/`cause`, and every major real-world anti-pattern

## Practical Connections

- **Every Spring REST controller's `@ExceptionHandler`** is built directly on this module's type-based catch-matching (Topic 1) and custom exception design (Topic 5) — translating domain exceptions into appropriate HTTP error responses.
- **JDBC and Hibernate/JPA** (Module 20 upcoming) both wrap low-level checked `SQLException`s into unchecked exceptions using exactly Topic 5's chaining pattern — you now understand precisely why and how.
- **Try-with-resources is the default, expected pattern** for every `Connection`, `PreparedStatement`, `InputStream`, and `OutputStream` in real Java code (Modules 13, 20) — Topic 4's mechanics are foundational, everyday knowledge.
- **Production incident postmortems** frequently trace back to Topic 5's anti-patterns (an empty `catch` block that silently hid a real bug for months) — this module's guidance is directly, practically protective against genuinely common, costly real-world mistakes.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| `Error` vs `Exception` | `Error`: serious, JVM/system-level, generally uncatchable. `Exception`: application-level, handleable, further split into checked/unchecked. |
| Checked vs unchecked | Checked (extends `Exception` directly): compiler-enforced handling. Unchecked (extends `RuntimeException`): no compile-time enforcement. |
| `finally` running vs. `finally`'s `return`/`throw` overriding | `finally` always runs; but if it contains its OWN `return`/`throw`, that silently discards whatever the `try`/`catch` was doing. |
| `Collections.unmodifiableList`-style views vs. try-with-resources' guarantee | Unrelated, but both share the theme of "compiler/runtime-enforced correctness vs. manual discipline" seen throughout this course. |
| Exception chaining vs. losing the cause | Chaining (`super(msg, cause)`) preserves the original exception; failing to chain silently discards it. |

## Consolidated Interview Questions (Module 12)

1. What's the difference between `Error` and `Exception`?
2. How does the JVM decide which `catch` block handles a thrown exception when multiple are present?
3. What's the mechanical difference between checked and unchecked exceptions?
4. What was the original philosophical reasoning behind checked exceptions, and what are the main real-world criticisms?
5. Does `finally` run if `try` contains a `return`? What happens if `finally` also contains a `return`?
6. What is stack unwinding?
7. What interface must a class implement to be usable in try-with-resources?
8. In what order do multiple try-with-resources resources close?
9. What are suppressed exceptions?
10. How does try-with-resources resolve each of `finalize()`'s Module 07 criticisms?
11. What is exception chaining, and why does it matter?
12. Why is an empty `catch` block the most damaging exception anti-pattern?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** Draw the `Throwable` hierarchy from memory, then write out, in one sentence each, the mechanical difference and the intended philosophical difference between checked and unchecked exceptions.
2. **Hands-on:** Write a method with `try { return 1; } finally { System.out.println("x"); }` and a second with `try { return 1; } finally { return 2; }` — predict and verify both outputs.
3. **Hands-on:** Implement a custom `AutoCloseable` resource whose `close()` throws, combined with a `try` block that also throws — catch the result and print both the primary message and every suppressed exception.
4. **Hands-on:** Design a custom `RuntimeException` subclass with at least one structured data field (like `InsufficientFundsException`'s `shortfall`), and a scenario that chains it from an original lower-level exception.
5. **Conceptual:** A colleague writes `catch (Exception e) { }` "just to be safe." Explain, referencing this module's anti-patterns, precisely what could go wrong with this approach.
6. **Synthesis:** Design a small file-processing method that reads a file (try-with-resources), validates its content (throwing a custom checked or unchecked exception — justify your choice using Topic 2's guidance), and chains any low-level `IOException` into your custom exception type correctly.

## What's Next

Module 12 completed Java's error-handling model — you now understand not just how to write `try`/`catch`, but the deep reasoning, controversy, and best practices behind it. **Module 13 — IO** applies this directly: reading and writing files and streams, where try-with-resources (just mastered) is used constantly, alongside the Reader/Writer/InputStream/OutputStream class hierarchies.

---

**Previous:** [05 — Custom Exceptions & Best Practices](05-Custom-Exceptions-And-Best-Practices.md) · **Module Overview:** [00 — Module Overview](00-Module-Overview.md)

**Type "Continue" to begin Module 13 — IO.**
