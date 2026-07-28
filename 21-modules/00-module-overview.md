# Module 21 — The Java Platform Module System (JPMS)

## Module Goal

Module 02, Topic 2 mentioned the Platform ClassLoader loading "JDK platform modules" in passing, without explaining what a "module" actually is. This module delivers that explanation: the Java Platform Module System (JPMS), introduced in Java 9 (2017) — one of the largest structural changes to the JDK itself in Java's history, restructuring the entire JDK into modules and giving you the tools to modularize your own applications for genuinely stronger encapsulation than `public`/`private` alone can offer.

## Topics Covered in This Module

1. **[Why JPMS — The Problems It Solves](01-why-jpms-the-problems-it-solves.md)** — the classpath/"JAR hell" problem, and the weak encapsulation of pre-modular Java.
2. **[Module Descriptors & Directives](02-module-descriptors-and-directives.md)** — `module-info.java`, and the `requires`/`exports`/`opens`/`provides`/`uses` directives.
3. **[Migration, `jlink` & Practical Adoption](03-migration-jlink-and-practical-adoption.md)** — automatic modules, the unnamed module, `jlink` custom runtime images (closing the loop from Module 01), and the honest state of JPMS adoption in the real world.
4. **[Module Summary](04-module-summary.md)** — consolidated recap.

## Prerequisites

- Module 02 (JVM), Topic 2 (Class Loader Subsystem — the Platform ClassLoader preview this module completes).
- Module 05 (OOP), Topic 2 (Encapsulation — JPMS extends this concept beyond a single class).
- Module 01 (Introduction), Topic 6 (`jlink` mentioned in the JDK tools list).

## How to Study This Module

Topic 1 is essential — JPMS's directives (Topic 2) will feel like arbitrary syntax if you don't first understand the genuine, real problems (fragile classpath ordering, no true encapsulation across JARs) they were specifically designed to solve. Topic 3 closes with an honest, practical note: JPMS adoption in the wider Java ecosystem has been slower and more partial than originally anticipated — worth knowing as context, not just the mechanics.