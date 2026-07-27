# Transactions & Connection Pooling

## Learning Objectives

- Use JDBC transactions (`commit`/`rollback`) correctly
- Understand the ACID properties at a working level
- Understand why connection pooling exists, directly paralleling Module 15's thread-pool reasoning

## Prerequisites

[01 — JDBC Architecture & Connections](01-JDBC-Architecture-And-Connections.md), Module 15 Topic 5 (Executors/thread pools), Module 12 (Exceptions)

## Motivation

Real-world database operations frequently need "all or nothing" guarantees — transferring money between two accounts must never leave one account debited without the other being credited. This closing topic covers transactions (the mechanism providing that guarantee) and connection pooling (the practical, performance-critical infrastructure every real application needs) — completing your JDBC foundation.

## The Problem: Multi-Step Operations That Must Be Atomic

```java
// Transferring money -- TWO separate SQL statements that MUST both succeed, or NEITHER should:
pstmt1.executeUpdate();   // debit account A
pstmt2.executeUpdate();     // credit account B

// ⚠️ What if the program crashes, or an exception occurs, BETWEEN these two statements?
//    Account A is debited, but account B was NEVER credited -- money simply VANISHES!
```

**By default, JDBC connections are in "auto-commit" mode** — each individual statement is immediately, independently committed to the database the instant it executes. This is precisely the problem above: two related statements, each auto-committed separately, offer **no guarantee that both succeed together** — exactly Module 15, Topic 2's "race condition" concern, but for **database consistency** instead of in-memory shared state, and exactly Module 12's "guarantee cleanup even if an exception occurs" concern, applied to multi-step database operations.

## Transactions — `commit()` and `rollback()`

```java
try {
    connection.setAutoCommit(false);   // DISABLE auto-commit -- take manual control

    pstmt1.executeUpdate();   // debit account A -- NOT yet permanently saved
    pstmt2.executeUpdate();     // credit account B -- NOT yet permanently saved

    connection.commit();          // BOTH statements become permanent, TOGETHER, atomically
} catch (SQLException e) {
    connection.rollback();          // UNDOES both statements entirely -- as if NEITHER ever happened
    throw e;
} finally {
    connection.setAutoCommit(true);   // restore default behavior for future use of this connection
}
```

**This is precisely Module 12's exception-handling philosophy, applied to database operations**: `commit()` makes every statement since the last commit **permanent, together, as one atomic unit** — `rollback()` (typically called from a `catch` block, exactly Module 12, Topic 1's pattern) **undoes every statement since the last commit entirely**, as if none of them had ever executed at all. **There is no possible intermediate state where account A is debited but account B isn't** — either both changes are saved together, or neither is.

## The ACID Properties — A Working-Level Introduction

| Property | Meaning |
|---|---|
| **Atomicity** | A transaction's statements all succeed together, or all fail together — no partial completion (exactly the money-transfer example above) |
| **Consistency** | A transaction moves the database from one valid state to another valid state, never violating defined rules/constraints |
| **Isolation** | Concurrent transactions (from different connections/threads) don't interfere with each other's intermediate, uncommitted state — directly analogous to Module 15's entire concurrency-hazard discussion, now at the database level |
| **Durability** | Once committed, a transaction's changes survive even a subsequent system crash — permanently, reliably saved |

**Isolation, specifically, is genuinely, directly analogous to Module 15's entire concurrency chapter** — just as Module 15 covered race conditions and visibility problems between **threads** sharing memory, database transaction isolation addresses race-condition-like problems between **concurrent transactions** sharing the same underlying data, with databases offering configurable isolation levels representing different points on a similar safety/performance trade-off curve to what Module 15 explored for in-memory concurrency.

## Why Connection Pooling Exists — Directly Paralleling Module 15, Topic 5

Recall Topic 1's key fact: **creating a `Connection` is genuinely expensive** (real network/authentication overhead). Recall Module 15, Topic 5's exact reasoning for `ExecutorService`: **thread creation has real overhead, so reuse a managed pool of threads instead of creating one per task.**

**Connection pooling applies precisely this same reasoning to database connections:**

```java
// WITHOUT pooling -- pays connection-establishment cost on EVERY single request:
Connection conn = DriverManager.getConnection(url, user, pass);   // expensive, EVERY time
// ... use it ...
conn.close();

// WITH pooling (e.g., HikariCP, the modern standard connection pool library) --
// connections are ESTABLISHED ONCE, then REUSED across many requests:
Connection conn2 = dataSource.getConnection();   // returns an ALREADY-ESTABLISHED connection
                                                     // from the pool -- essentially FREE, fast
// ... use it ...
conn2.close();   // does NOT actually close the underlying connection -- RETURNS it to the pool,
                    // ready for the NEXT request to reuse
```

```
     WITHOUT pooling                              WITH pooling (HikariCP, etc.)

 Request 1: CREATE connection (slow!)         Pool: [conn1, conn2, conn3, conn4, conn5]
            ... use ...                                (created ONCE, at application startup)
            DESTROY connection
 Request 2: CREATE connection (slow!)         Request 1: BORROW conn1 from pool (fast!)
            ... use ...                                  ... use ...
            DESTROY connection                            RETURN conn1 to pool
 Request 3: CREATE connection (slow!)         Request 2: BORROW conn2 from pool (fast!)
            ... use ...                                  ... use ...
            DESTROY connection                            RETURN conn2 to pool
```

**A genuinely important detail, worth calling out explicitly**: when using a pool, `connection.close()` does **NOT** actually terminate the underlying database connection — it **returns** the connection to the pool, ready for immediate reuse by the next request. This is precisely why using try-with-resources (Module 12, Topic 4) remains exactly correct and important even with pooling — `close()`'s specific *effect* changes (return to pool, rather than genuine termination), but the *discipline* of always closing (always returning resources) remains exactly as essential as ever.

## Real-World Analogy

Think of a database transaction like **a single, all-or-nothing shopping cart checkout** — you add several items and confirm payment as **one** combined action; if payment fails partway through, **nothing** is charged and **nothing** ships, rather than some items being charged and shipped while others silently aren't. Think of connection pooling exactly like **Module 15's restaurant-with-a-standing-team-of-employees analogy, reapplied here**: instead of hiring and training (establishing) a brand-new phone line to the kitchen for every single customer order, a small, fixed set of phone lines are kept permanently open and simply handed off, in turn, to whichever server currently needs to relay an order — dramatically more efficient than establishing and tearing down a fresh line for every single interaction.

## Advantages

- Transactions provide genuine, reliable atomicity guarantees for multi-step database operations, preventing exactly the "partial completion" data-corruption scenarios that would otherwise be possible.
- Connection pooling eliminates the repeated, expensive cost of establishing connections per request — directly, substantially improving real application performance and scalability, exactly mirroring Module 15, Topic 5's thread-pool benefit.

## Disadvantages / Trade-offs

- Transactions held open too long (without committing/rolling back promptly) can hold database locks, reducing concurrency for other transactions — a real, practical performance concern in high-throughput systems.
- Connection pools require careful sizing (too few connections bottleneck the application; too many can overwhelm the database server) — a genuine, real tuning consideration in production systems.

## Best Practices

- Always wrap multi-statement, logically-atomic database operations in an explicit transaction (`setAutoCommit(false)`, `commit()`/`rollback()` in a try/catch, matching Module 12's exception-handling pattern).
- Always use a connection pool (like HikariCP) in real applications — never establish a fresh `Connection` per request in production code.
- Keep transactions as short as reasonably possible, to minimize lock contention with other concurrent transactions.

## Common Mistakes

- Performing multi-step, logically-related database operations without an explicit transaction, risking partial completion on failure.
- Establishing a fresh `Connection` per request in a real application instead of using a connection pool, incurring repeated, unnecessary overhead.
- Forgetting that `close()` on a pooled connection returns it to the pool rather than truly closing it — though the practical discipline (always call `close()`, ideally via try-with-resources) remains exactly the same regardless.

## Interview Questions

1. **Q: Why does JDBC's default auto-commit behavior create a real risk for multi-statement operations?**
   A: Each statement commits independently and immediately — if a failure occurs between two logically-related statements (like debiting one account and crediting another), one change can be permanently saved while the other never happens, leaving the database in an inconsistent state.

2. **Q: What do `commit()` and `rollback()` do, and how does this relate to Module 12's exception handling?**
   A: `commit()` makes every statement since the last commit permanent, together, atomically. `rollback()` undoes all of them, as if none had executed — typically called from a `catch` block when an exception occurs mid-transaction, directly mirroring Module 12's exception-handling discipline, now ensuring database consistency rather than just program-level cleanup.

3. **Q: Why does connection pooling exist, and how does `close()`'s behavior change when using a pool?**
   A: Establishing a database connection is genuinely expensive (network/authentication overhead) — directly paralleling Module 15, Topic 5's thread-creation-overhead reasoning, a pool establishes connections once and reuses them across many requests. With pooling, `close()` returns the connection to the pool for reuse rather than actually terminating it — though the discipline of always calling `close()` (via try-with-resources) remains just as essential.

## Summary

- **Transactions** (`setAutoCommit(false)`, `commit()`/`rollback()`) provide atomicity for multi-statement database operations — directly extending Module 12's exception-handling discipline to database consistency guarantees.
- The **ACID properties** (Atomicity, Consistency, Isolation, Durability) describe the reliability guarantees a properly-used transaction provides; Isolation directly parallels Module 15's concurrency-hazard concerns, applied to concurrent database transactions.
- **Connection pooling** reuses established connections across many requests, directly paralleling Module 15, Topic 5's thread-pool reasoning — essential, standard practice in any real production application.

## Module-Wide Quick Revision

- JDBC's driver-based architecture achieves vendor neutrality (program-to-interface, Module 05); `Connection` is genuinely expensive to create, motivating pooling (Topic 3) (Topic 1).
- `Statement`'s string concatenation enables SQL injection; `PreparedStatement` structurally prevents it by separating query structure from parameter values; `ResultSet.next()` mirrors `Iterator` (Topic 2).
- Transactions (`commit`/`rollback`) provide atomicity for multi-statement operations; connection pooling reuses expensive connections, directly paralleling Module 15's thread-pool reasoning (this topic).

## Common Pitfalls (Module-Wide)

- Using `Statement` with concatenated user input instead of `PreparedStatement`.
- Performing multi-step related operations without an explicit transaction.
- Creating a fresh `Connection` per request instead of using a pool.
- Attempting to parameterize table/column names via `PreparedStatement` placeholders.

## Mini Quiz (Module-Wide)

1. How does JDBC achieve database-vendor neutrality?
2. Why does a `Connection` being expensive to create matter practically?
3. How does `PreparedStatement` prevent SQL injection, mechanically?
4. What does `rollback()` do?
5. Why does connection pooling exist?

*(Answers are derivable from Topics 1, 1, 2, 3, and 3, respectively.)*

---

**Previous:** [02 — Statements, `ResultSet` & SQL Injection](02-Statements-ResultSet-And-SQL-Injection.md) · **Next:** [04 — Module Summary, Interview Questions & Exercises](04-Module-Summary-Exercises.md)
