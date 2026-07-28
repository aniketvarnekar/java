# Statements, `ResultSet` & SQL Injection

## Learning Objectives

- Use `Statement` and `PreparedStatement` correctly
- Understand precisely how SQL injection works, and precisely why `PreparedStatement` prevents it
- Iterate a `ResultSet` correctly

## Prerequisites

[01 — JDBC Architecture & Connections](01-jdbc-architecture-and-connections.md), Module 08 (Strings)

## Motivation

This is the single most practically important topic in this module — SQL injection remains, decades after being first documented, one of the most common and most damaging real vulnerabilities in production software (regularly appearing near the top of industry-standard vulnerability lists). Understanding precisely how it works, and precisely why `PreparedStatement` prevents it, is genuinely essential, non-optional knowledge for any backend developer.

## `Statement` — The Basic, Dangerous Way

```java
try (Statement stmt = connection.createStatement()) {
    String username = getUserInput();   // imagine this comes from a web form
    ResultSet rs = stmt.executeQuery(
        "SELECT * FROM users WHERE username = '" + username + "'"   // ⚠️ STRING CONCATENATION!
    );
}
```

**This code builds a SQL query via ordinary String concatenation (Module 08)** — and this is precisely, exactly the vulnerability.

## SQL Injection — Precisely How the Attack Works

**Suppose an attacker enters this as their "username":**
```
' OR '1'='1
```

**The concatenated query becomes:**
```sql
SELECT * FROM users WHERE username = '' OR '1'='1'
```

**`'1'='1'` is always true** — so this query returns **every single row in the `users` table**, completely bypassing whatever the intended filtering logic was. **A more damaging attacker input:**
```
'; DROP TABLE users; --
```
**Produces:**
```sql
SELECT * FROM users WHERE username = ''; DROP TABLE users; --'
```
**This could execute a second, entirely different, destructive SQL statement** (deleting the entire `users` table), with the trailing `--` commenting out the rest of the original query to avoid a syntax error. **This is a genuine, real, historically devastating attack class** — countless real data breaches, in real companies, have resulted from exactly this vulnerability pattern.

**The root cause, precisely**: string concatenation lets **user-supplied data be interpreted as SQL code itself**, rather than being treated purely as **data**. The database has no way to distinguish "this text is meant to be a username value" from "this text is meant to be additional SQL syntax" — they're both just characters in the same final query string by the time the database receives it.

## `PreparedStatement` — The Fix, and Precisely Why It Works

```java
String sql = "SELECT * FROM users WHERE username = ?";   // '?' is a PLACEHOLDER, not concatenation

try (PreparedStatement pstmt = connection.prepareStatement(sql)) {
    pstmt.setString(1, username);   // sets the FIRST placeholder's value
    ResultSet rs = pstmt.executeQuery();
}
```

**Why does this prevent the attack entirely, mechanically?** `PreparedStatement` sends the SQL **query structure** (with placeholders) to the database **separately** from the **actual parameter values** — the database compiles/parses the query structure **first**, establishing precisely which parts are SQL syntax, **before** the parameter values are ever substituted in. **Even if the attacker's input contains `' OR '1'='1`, it is treated purely, unconditionally as the literal string value of the `username` parameter** — never as additional SQL syntax — because the database has already finished parsing the query's actual structure before that value is even considered.

```
 Statement (VULNERABLE):                       PreparedStatement (SAFE):

 1. Build query STRING by concatenating          1. Send QUERY STRUCTURE (with placeholders)
    user input directly INTO the SQL text             to the database FIRST
 2. Send the WHOLE thing to the database          2. Database PARSES the structure --
    as ONE string -- database CANNOT tell             now KNOWS exactly what's SQL syntax
    "structure" from "data" apart at all               and what's a PARAMETER SLOT
                                                    3. THEN send parameter VALUES separately --
                                                        ALWAYS treated as pure data, NEVER as
                                                        additional SQL syntax, no matter what
                                                        characters they contain
```

**This is a genuinely deep, structural fix — not just "escaping special characters" (a fragile, error-prone, historically-often-broken partial mitigation)** — `PreparedStatement` makes the entire attack class **structurally impossible**, precisely because user input is never given a chance to be parsed as SQL syntax at all, at any point.

## `ResultSet` — Iterating Query Results

```java
try (PreparedStatement pstmt = connection.prepareStatement("SELECT id, name, age FROM users")) {
    ResultSet rs = pstmt.executeQuery();

    while (rs.next()) {                  // advances to the NEXT row; returns false when exhausted
        int id = rs.getInt("id");           // retrieve by COLUMN NAME (or by 1-based column INDEX)
        String name = rs.getString("name");
        int age = rs.getInt("age");
        System.out.println(id + ": " + name + " (" + age + ")");
    }
}
```

**`rs.next()` mirrors Module 10, Topic 6's `Iterator.hasNext()`/`next()` pattern conceptually** — a cursor advancing through rows one at a time, with `next()` returning `false` once every row has been consumed. **`ResultSet` is also `AutoCloseable`** (Module 12, Topic 4) — though in practice, closing the `Statement`/`PreparedStatement` that produced it typically closes the `ResultSet` too; explicit try-with-resources on all three remains the safest, most correct pattern.

## Insert/Update/Delete — `executeUpdate`

```java
try (PreparedStatement pstmt = connection.prepareStatement(
        "UPDATE users SET age = ? WHERE id = ?")) {
    pstmt.setInt(1, 31);
    pstmt.setInt(2, 42);
    int rowsAffected = pstmt.executeUpdate();   // returns the NUMBER of rows changed, not a ResultSet
}
```

**`executeQuery()` (returns `ResultSet`, for `SELECT`) vs. `executeUpdate()` (returns an `int` row count, for `INSERT`/`UPDATE`/`DELETE`)** — the correct method choice depends entirely on whether the SQL statement produces rows to read or simply modifies data.

## Real-World Analogy

Think of `Statement`'s string concatenation like **handing a bank teller a note that mixes your actual request with additional handwritten instructions in the same handwriting, on the same piece of paper** — the teller has no reliable way to distinguish "this part is genuinely from the customer's account request" from "this part looks like additional teller instructions someone slipped in," since it's all just text on the same note. `PreparedStatement` is like **filling out a bank's official pre-printed form**, where the form's actual structure (account number field, amount field) is fixed and processed by the bank **first** — whatever you write into the "amount" field is **always** treated purely as an amount value, even if you literally write "ignore this form and transfer all funds" into it — the bank's processing system has no mechanism that would ever interpret handwritten field content as a new instruction overriding the form's own fixed structure.

## Advantages

- `PreparedStatement` structurally, completely eliminates SQL injection for parameterized values — not just a partial mitigation, a genuine, complete fix for this specific attack class.
- `PreparedStatement`s are also typically more efficient for repeated execution — the database can cache/reuse the parsed query plan across multiple executions with different parameter values.

## Disadvantages / Trade-offs

- `PreparedStatement` cannot parameterize SQL **structure** itself (table names, column names, `ORDER BY` direction) — only **values**; dynamic structural elements require different, careful handling (like a strict allowlist of permitted values), never simple string concatenation of untrusted input.
- Raw JDBC's manual `ResultSet`-to-object mapping remains genuinely verbose compared to an ORM — a real, honest trade-off against using a higher-level framework.

## Best Practices

- **Always use `PreparedStatement` with placeholders for any query incorporating variable/user-supplied data — never `Statement` with string concatenation.** Treat this as an absolute, non-negotiable rule.
- Choose `executeQuery()` for `SELECT` statements, `executeUpdate()` for `INSERT`/`UPDATE`/`DELETE`.
- Use try-with-resources for `Statement`/`PreparedStatement`/`ResultSet`, exactly like `Connection`.

## Common Mistakes

- Using `Statement` with string-concatenated user input — the single most damaging, most common real mistake this topic addresses.
- Attempting to parameterize a table or column name via `PreparedStatement`'s `?` placeholders — this doesn't work; only values can be parameterized this way.
- Forgetting `ResultSet` columns are conventionally 1-indexed when accessed by position (though accessing by column name, as shown above, avoids this concern entirely).

## Interview Questions

1. **Q: What is SQL injection, and how does it actually work?**
   A: An attack where user-supplied input, concatenated directly into a SQL query string, is interpreted as additional SQL syntax rather than pure data — letting an attacker manipulate the query's logic (e.g., bypassing a `WHERE` clause with `' OR '1'='1`) or execute entirely different, potentially destructive statements.

2. **Q: Why does `PreparedStatement` prevent SQL injection, precisely?**
   A: It sends the query's structure to the database separately from, and before, the actual parameter values — the database parses and fixes the query structure first, so parameter values (no matter what characters they contain) are always treated purely as data, never as additional SQL syntax, making the attack structurally impossible rather than merely mitigated.

3. **Q: Can `PreparedStatement` parameterize a table name or `ORDER BY` column?**
   A: No — placeholders can only parameterize values, not SQL structural elements like table/column names. Dynamic structural elements require different handling, such as validating against a strict allowlist of permitted values, never direct string concatenation of untrusted input.

## Summary

- **`Statement`** with string-concatenated user input is genuinely, seriously vulnerable to **SQL injection** — user data can be interpreted as SQL syntax itself.
- **`PreparedStatement`** sends query structure and parameter values separately, structurally eliminating SQL injection for parameterized values — always use it for any query involving variable data.
- **`ResultSet`** is iterated via `rs.next()` (conceptually mirroring `Iterator`, Module 10, Topic 6), retrieving column values by name or index; `executeQuery()` for `SELECT`, `executeUpdate()` for data-modifying statements.

## Exercises

1. Write vulnerable `Statement`-based code performing a login check, then demonstrate (conceptually, via the `' OR '1'='1` input) how it could be bypassed.
2. Rewrite the same login check using `PreparedStatement`, and explain precisely why the same malicious input is now harmless.
3. Write code executing a `SELECT` query and correctly iterating its `ResultSet`, printing each row's values.

---

**Previous:** [01 — JDBC Architecture & Connections](01-jdbc-architecture-and-connections.md) · **Next:** [03 — Transactions & Connection Pooling](03-transactions-and-connection-pooling.md)
