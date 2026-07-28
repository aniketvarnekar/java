# Custom Exceptions & Best Practices

## Learning Objectives

- Design and implement your own exception types correctly
- Use exception chaining (`cause`) to preserve full diagnostic context across abstraction layers
- Recognize and avoid the real, common exception-handling anti-patterns

## Prerequisites

All prior topics in this module

## Motivation

Every professional Java codebase defines its own exception types — generic `RuntimeException`/`Exception` messages don't communicate domain-specific failure conditions clearly. This closing topic gives you the tools to design exceptions that genuinely help the next developer (often future you) understand exactly what went wrong and why.

## Designing a Custom Exception

```java
public class InsufficientFundsException extends RuntimeException {   // UNCHECKED (Topic 2's guidance --
    private final double shortfall;                                     // this is a business-logic condition,
                                                                            // not necessarily requiring EVERY
    public InsufficientFundsException(double shortfall) {                  // caller to handle it explicitly)
        super("Insufficient funds: short by $" + shortfall);
        this.shortfall = shortfall;
    }

    public double getShortfall() {
        return shortfall;
    }
}
```

```java
class BankAccount {
    private double balance;

    void withdraw(double amount) {
        if (amount > balance) {
            throw new InsufficientFundsException(amount - balance);
        }
        balance -= amount;
    }
}
```

**A custom exception is just an ordinary class (Module 06) extending `Exception` (checked) or `RuntimeException` (unchecked, per Topic 2's guidance) — nothing more exotic than that.** Extending `RuntimeException`/`Exception` (rather than implementing `Comparable`, `List`, or any other interface, Module 05, Topic 6) automatically inherits `Throwable`'s full method set (`getMessage()`, `getCause()`, `printStackTrace()`, `getSuppressed()` — Topics 1, 3, 4) — you get all of that "for free" through ordinary inheritance (Module 05, Topic 4).

**Why extra fields (`shortfall` here) are genuinely valuable**: beyond a human-readable message, a well-designed custom exception can carry **structured, programmatically-usable data** about the failure — letting `catch` blocks make informed decisions (e.g., "if the shortfall is small, prompt for a top-up; if large, reject outright") rather than parsing a message string, which is fragile and considered a real anti-pattern.

## Exception Chaining — Preserving the Original `cause`

A genuinely common real-world scenario: a low-level operation fails with one exception type, but the calling code needs to translate it into a higher-level, more meaningful exception type for **its own** callers — without losing the original diagnostic information:

```java
class DataAccessException extends RuntimeException {
    public DataAccessException(String message, Throwable cause) {
        super(message, cause);   // Throwable's constructor accepts a CAUSE -- this is what CHAINS them
    }
}

void loadUser(String id) {
    try {
        connectToDatabase(id);   // throws a low-level SQLException
    } catch (SQLException e) {
        throw new DataAccessException("Failed to load user " + id, e);   // e becomes the CAUSE
    }
}
```

```java
try {
    loadUser("123");
} catch (DataAccessException e) {
    System.out.println(e.getMessage());              // "Failed to load user 123"
    System.out.println(e.getCause());                    // the ORIGINAL SQLException, fully preserved
    e.printStackTrace();                                     // prints BOTH stack traces, chained together,
}                                                              // labeled "Caused by:" for the original
```

**Why does this matter, concretely?** Without chaining, translating exceptions (a genuinely common, legitimate pattern — hiding low-level implementation details like "this uses SQL" behind a higher-level, more meaningful exception type) would **silently discard** the original exception's diagnostic information — exactly Topic 3's `finally`-swallowing danger, in a different disguise. `Throwable`'s built-in `cause` mechanism (`getCause()`, and the `(message, cause)` constructor overload) exists specifically to let you **translate exceptions across abstraction layers without ever losing the original root cause** — `printStackTrace()`'s output automatically shows the **full chain**, labeled with "Caused by:", precisely for this reason.

## Real, Common Anti-Patterns to Avoid

### 1. The Empty `catch` Block ("Exception Swallowing")

```java
try {
    riskyOperation();
} catch (Exception e) {
    // nothing here at all -- the SINGLE most damaging exception anti-pattern in all of Java
}
```
**This silently discards ALL information that anything ever went wrong** — directly, severely violating Module 01, Topic 3's "Robust" philosophy (fail loudly, predictably). A failure that should have been visible simply vanishes, often leaving a program in a subtly broken, hard-to-diagnose state, discovered only much later, far from the actual root cause.

### 2. Catching `Exception` (or `Throwable`) Too Broadly

```java
try {
    doSomethingSpecific();
} catch (Exception e) {   // catches EVERYTHING -- including bugs that should have crashed loudly!
    log.warn("Something went wrong");
}
```
**This can accidentally catch and "handle" genuine programming errors** (like a `NullPointerException` from an actual bug, Topic 2) that should have propagated and been fixed, not silently logged and papered over. Catch the **most specific** exception type your code can actually, meaningfully handle.

### 3. Using Exceptions for Ordinary Control Flow

```java
try {
    while (true) {
        list.get(index++);
    }
} catch (IndexOutOfBoundsException e) {
    // "using" the exception to detect the end of the list -- a real, genuine anti-pattern
}
```
**Exceptions are for genuinely exceptional, unexpected conditions — not routine control flow.** This example should simply check `index < list.size()` directly (Module 04's ordinary loop condition). Exception handling has real overhead (Topic 1) and, more importantly, this pattern is confusing and non-idiomatic — a reader has to work much harder to understand "this loop ends via a thrown exception" than "this loop ends via a normal condition check."

### 4. Losing the Original Exception (Not Chaining)

```java
catch (SQLException e) {
    throw new RuntimeException("Database error");   // ⚠️ the ORIGINAL SQLException 'e' is LOST --
}                                                       // never passed as a cause!
```
Always pass the original exception as the `cause` (as shown in the chaining example above) when translating between exception types — losing it makes debugging real production issues significantly harder.

### 5. Throwing Generic Exceptions

```java
throw new Exception("something went wrong");   // uninformative -- no specific TYPE for callers to
                                                   // catch selectively, no structured data, minimal
                                                   // information beyond a single string
```
Prefer specific, well-named exception types (built-in where a good match exists, custom where domain-specific meaning genuinely helps) over generic `Exception`/`RuntimeException` instances — this is precisely why this topic exists: to make throwing meaningful, well-designed exceptions as easy as throwing generic ones.

## Real-World Analogy

Think of a custom exception like a **specific, detailed incident report form**, rather than a blank sticky note reading "something happened." A well-designed incident report (a custom exception with meaningful fields) lets whoever reviews it later (a `catch` block, or a developer reading logs) understand exactly what occurred and respond appropriately, without needing to reconstruct the situation from vague, generic notes. Exception chaining is like **stapling the original incident report to a follow-up summary report** written for a different audience (translating a low-level database failure into a higher-level "couldn't load user" business error) — the summary is more appropriate for its intended reader, but the original, detailed report remains fully attached and accessible for anyone who needs to dig deeper.

## Advantages

- Well-designed custom exceptions communicate domain-specific failure conditions clearly, with structured, programmatically-usable data.
- Exception chaining preserves full diagnostic context across abstraction-layer translations, without sacrificing either layer's appropriate level of detail.
- Avoiding the common anti-patterns keeps failures visible and debuggable, directly supporting Module 01, Topic 3's "Robust" philosophy throughout a real codebase.

## Disadvantages / Trade-offs

- Designing a rich hierarchy of custom exception types takes real, deliberate effort compared to just throwing generic exceptions everywhere — a cost worth paying for any codebase of meaningful size or longevity.
- Overly fine-grained custom exception hierarchies (a distinct exception type for every conceivable failure) can become unwieldy — judgment is needed about where the value of specificity outweighs added complexity.

## Best Practices

- Extend `RuntimeException` (unchecked) unless there's a genuine, specific reason to require compile-time-enforced handling (Topic 2's guidance).
- Always chain exceptions when translating across abstraction layers — never silently discard the original cause.
- Catch the most specific exception type your code can meaningfully act on; never write an empty `catch` block.
- Never use exceptions for ordinary, expected control flow.

## Common Mistakes

- Writing empty `catch` blocks, silently discarding failure information.
- Catching `Exception`/`Throwable` broadly enough to accidentally swallow genuine programming errors.
- Forgetting to chain the original exception when translating between exception types.
- Using exceptions to control ordinary, expected program flow instead of genuinely exceptional conditions.

## Interview Questions

1. **Q: What is exception chaining, and why does it matter?**
   A: Passing an original, lower-level exception as the `cause` of a new, higher-level exception (via `Throwable`'s `(message, cause)` constructor), preserving full diagnostic information when translating exceptions across abstraction layers. Without it, the original root cause is silently lost, making real debugging significantly harder — `printStackTrace()` automatically shows the full "Caused by:" chain when chaining is used correctly.

2. **Q: Why is an empty `catch` block considered the most damaging exception anti-pattern?**
   A: It silently discards all information that a failure occurred, directly violating Java's "fail loudly, predictably" robustness philosophy — the program may continue in a subtly broken state, with the real problem discovered only much later, far removed from its actual root cause.

3. **Q: Why shouldn't exceptions be used for ordinary control flow?**
   A: Exceptions represent genuinely unexpected, exceptional conditions — using them for routine flow (like detecting a loop's natural end) has real performance overhead and produces confusing, non-idiomatic code compared to a simple, direct condition check.

## Summary

- Custom exceptions are ordinary classes extending `Exception`/`RuntimeException`, optionally carrying structured, domain-specific data beyond just a message.
- **Exception chaining** (`super(message, cause)`, `getCause()`) preserves the original failure's full diagnostic information when translating between exception types across abstraction layers.
- Real, common anti-patterns to always avoid: empty `catch` blocks, catching too broadly, using exceptions for ordinary control flow, losing the original cause, and throwing uninformative generic exceptions.

## Module-Wide Quick Revision

- Every exception extends `Throwable`; `Error` (uncatchable, JVM-level) vs `Exception` (application-handleable); `catch` blocks match by type/inheritance, most-specific-first (Topic 1).
- Checked (extends `Exception` directly, compiler-enforced) vs unchecked (extends `RuntimeException`, no enforcement) — checked exceptions remain genuinely controversial; use sparingly for truly recoverable, foreseeable conditions (Topic 2).
- Multi-catch for unrelated types; `finally` always runs, even with `try`-block `return`; NEVER put `return`/`throw` inside `finally` — it silently overrides/discards; stack unwinding walks up the call stack to find a matching `catch` (Topic 3).
- Try-with-resources guarantees deterministic `close()` for `AutoCloseable` types, closing in reverse declaration order, with suppressed exceptions preserving dual-failure information — the complete, satisfying resolution to `finalize()`'s Module 07 deprecation story (Topic 4).
- Custom exceptions should extend `RuntimeException` by default, chain their cause when translating across layers, and never be caught in silent, empty `catch` blocks (this topic).

## Common Pitfalls (Module-Wide)

- Ordering catch blocks general-to-specific.
- Putting `return`/`throw` inside `finally`.
- Manual `finally`-based resource cleanup instead of try-with-resources.
- Empty `catch` blocks and overly broad `catch (Exception e)`.
- Not chaining the original cause when translating exceptions.