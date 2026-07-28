# Module 21 Summary

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

## What's Next

Module 21 completed your understanding of how Java itself is structured at the largest scale — the JDK's own module boundaries — closing the loop from Module 01's earliest JDK/JRE/JVM discussion. **Module 22 — Performance** now turns to making all of this run fast: JVM tuning, GC algorithm selection (building directly on Module 16), profiling, and the concrete, practical decision-making this entire course has been building toward.