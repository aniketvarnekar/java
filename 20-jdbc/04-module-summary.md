# Module 20 Summary

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-module-overview.md) has been covered:

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

## What's Next

Module 20 completed your foundational database connectivity knowledge — you now understand not just how to use JDBC, but precisely why its key safety mechanisms (`PreparedStatement`, transactions) work the way they do, and how modern frameworks (Spring Data, Hibernate) are built on top of this exact foundation. **Module 21 — Modules** covers the Java Platform Module System (JPMS, Java 9+) — how the JDK itself is organized into modules, and how you can modularize your own applications for stronger encapsulation and more reliable dependency management.