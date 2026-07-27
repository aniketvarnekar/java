# Module 20 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-Module-Overview.md) has been covered:

- [x] **JDBC Architecture & Connections** — the driver-based, vendor-neutral architecture (Module 05's "program to an interface," concretely applied), automatic driver discovery, and `Connection`'s genuine cost
- [x] **Statements, `ResultSet` & SQL Injection** — precisely how SQL injection works, precisely why `PreparedStatement` structurally prevents it, and `ResultSet` iteration
- [x] **Transactions & Connection Pooling** — `commit`/`rollback`, the ACID properties (with Isolation directly paralleling Module 15's concurrency hazards), and connection pooling's direct parallel to Module 15's thread-pool reasoning

## Practical Connections

- **Every Spring Data JPA repository method** (`findById`, `save`, custom `@Query` methods) ultimately generates and executes JDBC `PreparedStatement`s underneath — you now understand exactly what's happening beneath that abstraction.
- **Hibernate/JPA's `@Transactional` annotation** (Module 16, Topic 5's Dynamic Proxy mechanism!) manages exactly the `commit`/`rollback` behavior from Topic 3, automatically, around your annotated methods.
- **HikariCP** (the modern, standard connection pool used by Spring Boot by default) is a production-grade, highly-tuned implementation of exactly Topic 3's connection-pooling concept.
- **Every real-world data breach involving SQL injection** — and there have been many, well-documented, costly ones — traces back to exactly Topic 2's `Statement`-with-string-concatenation vulnerability.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| `Statement` vs `PreparedStatement` | `Statement`: string-concatenated queries, vulnerable to SQL injection. `PreparedStatement`: parameterized queries, structurally immune to it. |
| `executeQuery` vs `executeUpdate` | `executeQuery`: for `SELECT`, returns a `ResultSet`. `executeUpdate`: for `INSERT`/`UPDATE`/`DELETE`, returns an affected-row count. |
| Auto-commit vs. explicit transactions | Auto-commit: each statement independently, immediately permanent. Explicit transaction: multiple statements become permanent together, atomically, via `commit()`/`rollback()`. |
| `close()` without pooling vs. with pooling | Without: genuinely terminates the connection. With: returns it to the pool for reuse — the discipline of always closing remains identical either way. |

## Consolidated Interview Questions (Module 20)

1. How does JDBC achieve database-vendor neutrality?
2. Why is `Class.forName("...")` no longer required for driver loading in modern JDBC?
3. Why is a `Connection` considered an expensive resource?
4. What is SQL injection, and how does it actually work?
5. Why does `PreparedStatement` prevent SQL injection, precisely?
6. Can `PreparedStatement` parameterize a table name?
7. Why does JDBC's default auto-commit behavior create risk for multi-statement operations?
8. What do `commit()` and `rollback()` do?
9. Why does connection pooling exist, and how does `close()`'s behavior change when pooled?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** Explain, from memory, precisely why `PreparedStatement` structurally prevents SQL injection, not just "escapes" dangerous input.
2. **Hands-on:** Write vulnerable `Statement`-based login-check code, then rewrite it correctly with `PreparedStatement`.
3. **Hands-on:** Write a transaction transferring a value between two "accounts" (two rows in a table), correctly using `commit`/`rollback` around a try/catch.
4. **Hands-on:** Write code iterating a `ResultSet` from a `SELECT` query, printing each row.
5. **Conceptual:** Explain why connection pooling exists, directly paralleling it to Module 15, Topic 5's thread-pool reasoning.
6. **Synthesis:** Design a small `UserRepository` class using a connection pool (conceptually), with a `findById` method using `PreparedStatement` and a `transferFunds` method using an explicit transaction — explain each design choice.

## What's Next

Module 20 completed your foundational database connectivity knowledge — you now understand not just how to use JDBC, but precisely why its key safety mechanisms (`PreparedStatement`, transactions) work the way they do, and how modern frameworks (Spring Data, Hibernate) are built on top of this exact foundation. **Module 21 — Modules** covers the Java Platform Module System (JPMS, Java 9+) — how the JDK itself is organized into modules, and how you can modularize your own applications for stronger encapsulation and more reliable dependency management.

---

**Previous:** [03 — Transactions & Connection Pooling](03-Transactions-And-Connection-Pooling.md) · **Module Overview:** [00 — Module Overview](00-Module-Overview.md)

**Type "Continue" to begin Module 21 — Modules.**
