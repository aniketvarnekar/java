# Java Editions & Version Timeline

## Learning Objectives

- Distinguish Java SE, Java EE (Jakarta EE), and Java ME
- Understand the LTS release model in practical depth
- Have a working map of what each major Java version (5 through 25) introduced, to contextualize "since Java X" references throughout the rest of this course

## Prerequisites

[02 — History of Java](02-history-of-java.md)

## Motivation

This course teaches **Core Java**, which is really **Java SE** (Standard Edition). But you will constantly encounter the terms "Java EE," "Jakarta EE," and occasionally "Java ME" in job descriptions, documentation, and codebases — and you need to know precisely how they relate to what you're learning, and what they are not.

## Concept: The Three (Historical Four) Editions

### Java SE — Standard Edition

**What it is:** The core Java platform — the language itself, the JVM, and the foundational standard library (collections, I/O, concurrency, networking basics, etc.). **This is what this entire course covers.** Every other edition is built *on top of* Java SE.

### Java EE — Enterprise Edition (now Jakarta EE)

**What it is:** A set of **additional specifications and APIs**, built on top of Java SE, for building large-scale enterprise server applications — things like Servlets (handling HTTP requests), JPA (database object-relational mapping), JMS (messaging), EJB (distributed business components), and more.

**Important history:** Oracle transferred stewardship of Java EE to the Eclipse Foundation in 2017, and for trademark reasons it was renamed **Jakarta EE** (package names eventually changed from `javax.*` to `jakarta.*` in newer versions — a real, practical migration pain point many enterprise codebases dealt with).

**Relationship to frameworks you may have heard of:** Frameworks like **Spring** and **Spring Boot** are *not* Java EE/Jakarta EE themselves, but they exist in the same problem space (building enterprise/web applications) and often implement or interoperate with pieces of the Jakarta EE specification (e.g., the Servlet API) under the hood. **Spring Boot runs on top of Java SE** — everything you learn in this course is the direct foundation for using it effectively (see Module 24 / practical connections sections throughout for how core concepts map onto Spring Boot, Hibernate, Kafka, etc.).

### Java ME — Micro Edition

**What it is:** A stripped-down subset of Java SE, designed for resource-constrained embedded devices (older feature phones, embedded controllers, some IoT devices). Historically significant (it's the closest living descendant of Java's *original* embedded-device goal from the Green Project — Topic 2!) but far less relevant to most developers' day-to-day work today, as mobile development shifted to Android (which uses Java/Kotlin syntax but a fundamentally different runtime — the Android Runtime/ART, *not* a standard JVM) and modern embedded work often uses other stacks entirely.

### Visual Summary

```
                           ┌──────────────────────────────────────────────┐
                           │          Java ME (Micro Edition)             │
                           │     Embedded / Constrained Devices           │
                           └──────────────────────────────────────────────┘


┌──────────────────────────────────────────────────────────────────────────────────┐
│                 Jakarta EE (Formerly Java EE)                                    │
│                                                                                  │
│  Enterprise APIs: Servlets, JPA, JMS, CDI, EJB, WebSocket, etc.                  │
│                 (Built on Top of Java SE)                                        │
│                                                                                  │
│  ┌────────────────────────────────────────────────────────────────────────────┐  │
│  │                                Java SE                                     │  │
│  │                            ── THIS COURSE ──                               │  │
│  │                                                                            │  │
│  │  • Java Language                                                           │  │
│  │  • JVM                                                                     │  │
│  │  • Core Standard Library                                                   │  │
│  │  • Collections, I/O, Networking                                            │  │
│  │  • Concurrency, Strings, Generics, etc.                                    │  │
│  └────────────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────────────┘
```

## The LTS Model, Revisited With Practical Detail

Recall from Topic 2: since 2017, Java ships a new feature release every 6 months, with select versions designated **LTS (Long-Term Support)**.

| Term | Meaning |
|---|---|
| **Feature release** | Ships every 6 months (March/September), contains whatever's ready. Supported only until the *next* release ships (~6 months). |
| **LTS release** | A designated feature release (8, 11, 17, 21, 25, ...) that vendors commit to patching/supporting for **years** (varies by vendor — often 5–8+ years). |
| **Current LTS cadence** | Every LTS was 3 years apart initially (8→11→17), then Oracle moved to a 2-year LTS cadence starting with 17→21→25. |

**Practical implication:** if a job posting says "Java 17" or "Java 21," they mean an LTS version their production systems are pinned to — not necessarily "the newest thing available." Always check what the *newest LTS* is when this matters (at the time of this course, Java 25 is the latest LTS).

## Version-by-Version Feature Highlights (5 through 25)

This table is a **map**, not a teaching moment — every feature listed here gets fully taught in its dedicated module later in this course. Use this now just to build the timeline in your head; you are not expected to understand these features yet.

| Version | Year | Type | Headline features |
|---|---|---|---|
| **Java 5** | 2004 | Major | Generics, Enums, Annotations, Autoboxing/unboxing, Enhanced for-loop, Varargs, Static imports |
| Java 6 | 2006 | Major | Performance/JVM improvements, scripting engine support, renamed "Java SE" |
| Java 7 | 2011 | Major | try-with-resources, diamond operator (`<>`), switch on Strings, multi-catch |
| **Java 8** | 2014 | Major (LTS) | **Lambdas, Streams API, `java.time`, default/static interface methods, `Optional`** — the biggest philosophical shift in Java's history (Modules 17–18) |
| Java 9 | 2017 | Major | Java Platform Module System (JPMS — Module 21), JShell, private interface methods |
| Java 10 | 2018 | Feature | `var` — local variable type inference |
| **Java 11** | 2018 | Feature (LTS) | New `HttpClient`, single-file source launch, `String` new methods |
| Java 12–16 | 2019–2021 | Feature | Switch expressions (14), Text blocks (15), Records preview (14), Sealed classes preview (15/16), Pattern matching for `instanceof` (16) |
| **Java 17** | 2021 | Feature (LTS) | **Sealed classes (final), Pattern matching for `instanceof` (final)**, strong encapsulation of JDK internals |
| Java 18–20 | 2022–2023 | Feature | UTF-8 by default, simple web server, pattern matching for `switch` (preview), virtual threads (preview), structured concurrency (incubator) |
| **Java 21** | 2023 | Feature (LTS) | **Virtual Threads (final), Record Patterns (final), Pattern Matching for `switch` (final), Sequenced Collections, Structured Concurrency (preview)** — a landmark modern release |
| Java 22–24 | 2024–2025 | Feature | Further structured concurrency/scoped values refinement, foreign function & memory API maturing, stream gatherers, further pattern-matching/unnamed-variable ergonomics |
| **Java 25** | 2025 | Feature (LTS) | Latest LTS at time of writing — continues maturing compact source files, structured concurrency, pattern matching, and modern collections/streams ergonomics |

> **Module 23 (Modern Java)** revisits every one of these modern features (records, sealed classes, pattern matching, virtual threads, structured concurrency, scoped values, text blocks, `var`) in full teaching depth, always comparing "old way" vs. "new way" code side by side. This table is purely a map for now.

## Why This History Matters Practically

- **Reading job postings accurately:** "5+ years Java experience, Java 17+" tells you the *minimum* modern feature set expected (sealed classes, pattern matching, records are all fair game).
- **Reading legacy code accurately:** if you encounter a codebase using anonymous inner classes instead of lambdas, or verbose `Date`/`Calendar` instead of `java.time`, you can infer it likely predates Java 8, or the team simply hasn't modernized — both real, common situations.
- **Interview credibility:** being able to say "that's been possible since Java 8" or "that's a Java 21 feature" signals genuine depth, not just syntax familiarity.

## Common Mistakes

- Saying "Java EE" when you mean "Java SE" (or vice versa) — remember: SE is the core language/platform (this course); EE/Jakarta EE is enterprise APIs built on top of it.
- Assuming Android development uses "Java SE" the same way this course teaches it — Android uses Java-like syntax but runs on a different runtime (ART, not a standard JVM) with a different (though overlapping) standard library subset. Core language concepts transfer; JVM internals (Module 16) largely do not.
- Assuming the newest Java version is automatically what's "used in industry" — most production systems run on an LTS, often not even the newest one.

## Interview Questions

1. **Q: What's the difference between Java SE and Java EE/Jakarta EE?**
   A: Java SE is the core language, JVM, and standard library — the foundation. Java EE (now Jakarta EE) is a separate, additional set of specifications for enterprise server-side development (Servlets, JPA, messaging, etc.), built on top of Java SE, not a replacement for it.

2. **Q: What is an LTS release and why would a company deliberately stay on Java 17 instead of moving to a newer feature release?**
   A: LTS (Long-Term Support) releases receive extended vendor support/patches over years, unlike regular feature releases which are supported only until the next 6-month release. Companies stay on LTS versions for production stability and predictable, well-tested support windows, rather than chasing every 6-month release.

3. **Q: Name the Java version that introduced Lambdas and Streams, and explain why that release is considered a major turning point.**
   A: Java 8 (2014). It's considered a turning point because it introduced functional-programming-style constructs into what had been a purely object-oriented language for its first ~18 years — a genuine paradigm expansion, not just an incremental API addition (fully covered in Modules 17–18).

## Summary

- **Java SE** = the core platform (this entire course). **Jakarta EE** (formerly Java EE) = enterprise APIs built on top of SE. **Java ME** = embedded/constrained-device subset, now niche.
- Since 2017: a new feature release every 6 months; select releases (8, 11, 17, 21, 25...) are designated **LTS** and get long-term vendor support — most production systems track an LTS.
- Java's version history has two eras: the slow, big-bang releases era (1996–2017) and the fast, predictable 6-month-train era (2017–present).
- Java 8 (2014, Lambdas/Streams) and Java 21 (2023, Virtual Threads/Pattern Matching/Records) are the two biggest inflection points in the modern era — both fully covered later in this course (Modules 17–18 and 23 respectively).

## Module-Wide Quick Revision

- Java = statically-typed, OOP, bytecode-compiled, garbage-collected language, designed for portability (Topic 1).
- Born 1991 (Green Project) for embedded devices, pivoted to the web in 1995, WORA is its founding principle (Topic 2).
- Its "features" (secure, robust, portable, etc.) each map to a concrete mechanism, not just marketing words (Topic 3).
- JVM executes bytecode (platform-specific); JRE = JVM + libraries; JDK = JRE + dev tools (Topic 4).
- Source → `javac` → bytecode → Class Loader → Verifier → Interpreter/JIT → running program, with GC managing memory throughout (Topic 5).
- Install a JDK, understand `PATH` vs `JAVA_HOME` (Topic 6).
- `HelloWorld.java`: file name must match public class name; `main` must be `public static void main(String[] args)` exactly (Topic 7).
- Java SE (core) vs Jakarta EE (enterprise APIs) vs Java ME (embedded); LTS releases anchor production usage (this topic).

## Common Pitfalls (Module-Wide)

- Confusing Java with JavaScript.
- Believing the JVM itself is platform-independent (it's the bytecode that is).
- Believing Java is "just interpreted" and therefore inherently slow (ignores JIT).
- Forgetting `main` must be `static`, or mismatching the file name to the public class name.
- Confusing Java SE with Java EE/Jakarta EE.

## Mini Quiz (Module-Wide)

1. What does WORA stand for, and what mechanism makes it possible?
2. Name the three layers: which one contains `javac`? Which one is platform-specific?
3. What are the two phases bytecode goes through in the JVM's execution engine, and what triggers the transition between them?
4. What must be true about a method for the JVM to treat it as the program's entry point?
5. What edition of Java does this course teach — SE, EE, or ME?

*(Answers are derivable from Topics 1, 4, 5, 7, and this topic, respectively — re-read the relevant topic if you're unsure rather than guessing.)*

---

**Previous:** [07 — Your First Java Program](07-first-java-program.md) · **Next:** [09 — Module Summary, Interview Questions & Exercises](09-module-summary-exercises.md)
