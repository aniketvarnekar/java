# Module 20 — JDBC

## Module Goal

Nearly every real backend application talks to a database. JDBC (Java Database Connectivity) is the standard, vendor-neutral API Java has provided for this since 1.1 (1997) — and, despite modern ORMs (Hibernate, Spring Data) sitting on top of it, understanding JDBC directly is what lets you actually understand what those frameworks are doing underneath, debug them effectively, and write raw SQL access code when a framework doesn't fit. This module covers connections, statements (and the SQL injection vulnerability they can prevent or enable), and transactions.

## Topics Covered in This Module

1. **[JDBC Architecture & Connections](01-JDBC-Architecture-And-Connections.md)** — the driver-based architecture that makes JDBC vendor-neutral, and `Connection`.
2. **[Statements, `ResultSet` & SQL Injection](02-Statements-ResultSet-And-SQL-Injection.md)** — `Statement` vs. `PreparedStatement`, and precisely how the latter prevents SQL injection.
3. **[Transactions & Connection Pooling](03-Transactions-And-Connection-Pooling.md)** — `commit`/`rollback`, the ACID properties, and why connection pooling exists — directly extending Module 15's thread-pool reasoning.
4. **[Module Summary, Interview Questions & Exercises](04-Module-Summary-Exercises.md)** — consolidated recap, quiz, and practice problems.

## Prerequisites

- Module 12 (Exceptions), Topic 4 (try-with-resources — every JDBC resource is `AutoCloseable`).
- Module 15 (Concurrency), Topic 5 (Executors/thread pools — connection pooling directly mirrors this reasoning).
- Module 16 (JVM Internals), Topic 4 (Reflection — JDBC drivers are discovered via a Reflection-based mechanism).

## How to Study This Module

Topic 2's SQL injection coverage is the single most practically important, real-world-consequential piece of this module — genuinely one of the most common, most damaging real security vulnerabilities in production software, and one Java's `PreparedStatement` makes straightforward to prevent entirely. Topic 3's connection pooling discussion is deliberately framed as a direct parallel to Module 15, Topic 5 — the exact same "reuse expensive resources instead of creating them per-use" reasoning, applied to database connections instead of threads.

---

**Previous module:** [19 — Networking](../19-Networking/00-Module-Overview.md) · **Next:** [01 — JDBC Architecture & Connections](01-JDBC-Architecture-And-Connections.md)
