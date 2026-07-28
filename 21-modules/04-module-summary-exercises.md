# Module 21 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-module-overview.md) has been covered:

- [x] **Why JPMS — The Problems It Solves** — "JAR hell" and the classpath's fragility, `public`'s weak cross-JAR encapsulation, and why the JDK team specifically needed this to safely evolve JDK internals
- [x] **Module Descriptors & Directives** — `module-info.java`, `requires` (verified dependencies), `exports` (genuine structural encapsulation), `opens` (deep Reflective access, distinct from `exports`), and `provides`/`uses` (formalized SPI)
- [x] **Migration, `jlink` & Practical Adoption** — the unnamed module and automatic modules preserving backward compatibility, `jlink`'s minimal custom runtime images, and an honest assessment of real-world JPMS adoption

## Practical Connections

- **The JDK itself, since Java 9**, is the single largest, most successful real-world JPMS deployment — every module in this course has been running on a modularized JDK without you needing to think about it directly.
- **Container image optimization** (a genuine, everyday concern in modern cloud-native Java deployment) directly uses `jlink`-produced custom runtime images for meaningfully smaller, faster-starting containers.
- **Spring Boot applications**, even without adopting `module-info.java` themselves, still benefit from the JDK's own internal modularization (stronger security, cleaner internal boundaries) — a real, if invisible, benefit of this module's subject matter.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| Classpath vs. module path | Classpath: traditional, flat, unordered JAR list (unnamed module). Module path: JPMS-aware, with genuine or automatic modules. |
| `public` vs `exports` | `public`: visible within the same compilation context (Module 05, Topic 2). `exports`: visible to other *modules* — a package can have `public` classes yet remain completely inaccessible externally if not exported. |
| `exports` vs `opens` | `exports`: normal compile-time access to `public` members. `opens`: additionally permits deep Reflective access (e.g., `setAccessible(true)`), a separate, deliberate grant. |
| Automatic module vs. genuine module | Automatic: inferred from a plain JAR's filename, all packages implicitly exported — a migration bridge. Genuine: has its own real `module-info.java` with deliberate, precise directives. |

## Consolidated Interview Questions (Module 21)

1. What problem does "JAR hell" refer to?
2. Why didn't `public` alone provide real encapsulation across JAR boundaries before JPMS?
3. What does `exports` control, and how does it differ from `public`?
4. Why does `opens` exist as a directive separate from `exports`?
5. What do `provides`/`uses` formalize, and what's the connection to Module 20's JDBC drivers?
6. What is the unnamed module, and why does it exist?
7. What is an automatic module, and what problem does it solve?
8. What does `jlink` produce, and why does JPMS specifically make it possible?
9. How successful has JPMS adoption been across the broader Java ecosystem, honestly?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** From memory, explain the difference between `public` and `exports`, using a concrete example of a class that's `public` but not accessible externally.
2. **Hands-on:** Write a small two-module example: a `com.example.api` module exporting an interface, and a `com.example.impl` module requiring it and providing an implementation via `provides`/`uses`.
3. **Hands-on:** If you have a JDK installed, run `jlink --list-modules` to see the JDK's own full list of modules — pick a few and speculate on what they contain.
4. **Conceptual:** Explain why a Jackson-based application needs `opens`, not just `exports`, on packages containing its data model classes.
5. **Conceptual:** Explain why the JDK team's own internal modularization was a genuine, lasting success, even though broader ecosystem adoption of JPMS for application code has been more limited.
6. **Synthesis:** Design a `module-info.java` for a hypothetical `com.example.blog` application that requires `java.sql` and a third-party (automatic module) library, exports only its public API package, and opens its entity package for a JSON library's Reflection needs — explain each directive's purpose.

## What's Next

Module 21 completed your understanding of how Java itself is structured at the largest scale — the JDK's own module boundaries — closing the loop from Module 01's earliest JDK/JRE/JVM discussion. **Module 22 — Performance** now turns to making all of this run fast: JVM tuning, GC algorithm selection (building directly on Module 16), profiling, and the concrete, practical decision-making this entire course has been building toward.

---

**Previous:** [03 — Migration, `jlink` & Practical Adoption](03-migration-jlink-and-practical-adoption.md) · **Module Overview:** [00 — Module Overview](00-module-overview.md)

**Type "Continue" to begin Module 22 — Performance.**
