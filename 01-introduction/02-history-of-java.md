# History of Java

## Learning Objectives

- Understand the original problem Java was built to solve, and how that shaped its core design
- Know the major milestones in Java's evolution
- Understand the release cadence Java uses today, and why it changed

## Prerequisites

[01 — What Is Java?](01-what-is-java.md)

## Motivation

Version numbers ("added in Java 8," "since Java 17," "preview in Java 21") appear constantly throughout this course and in real Java codebases. Without a timeline in your head, these numbers are meaningless trivia. With one, they tell a story — you'll understand *why* a feature was needed at that point in Java's life, which makes it far easier to remember.

## The Origin Story: The Green Project (1991)

Java did not start as an internet language — that's a common misconception. It began in 1991 at **Sun Microsystems**, as part of an internal effort called the **Green Project**, led by **James Gosling**, along with Mike Sheridan and Patrick Naughton.

**The actual goal:** build software for **consumer electronics** — set-top boxes, interactive TVs, handheld devices. The team quickly ran into the exact problem described in Topic 1: these devices used many *different* CPU chips from different manufacturers. Writing and recompiling C/C++ code separately for every chip was slow, expensive, and error-prone.

Their solution: design a language that compiles to an intermediate form (bytecode) that runs on a small, portable virtual machine — one virtual machine implementation per chip, instead of recompiling the whole application per chip.

The language was originally called **"Oak"** (reportedly named after an oak tree outside Gosling's office). It was later renamed **"Java"** — the popular story is that it was named after Java coffee, chosen during a brainstorming session, partly because "Oak" was already trademarked by another company.

> **Ironic twist:** the consumer-electronics market Java was originally built for didn't take off the way Sun hoped. But right as that project was struggling, the World Wide Web was exploding in popularity (1994–1995) — and it turned out Java's "runs anywhere without recompiling" property was *exactly* what the web needed for interactive content (Java applets running inside browsers, on any visitor's OS). Java pivoted to the web and became a phenomenon.

## Public Release (1995–1996)

- **1995:** Java publicly announced at SunWorld, alongside the HotJava browser demonstrating Java applets running inside a web page.
- **January 1996:** **JDK 1.0** released. Sun's tagline: **"Write Once, Run Anywhere" (WORA)**.

## Major Milestones Timeline

```
1991 ─ Green Project begins (Gosling et al.), language named "Oak"
1995 ─ Renamed "Java", publicly announced
1996 ─ JDK 1.0 released — WORA era begins
1997 ─ JDK 1.1 — inner classes, JavaBeans, JDBC, RMI
1998 ─ J2SE 1.2 ("Java 2") — Collections Framework, Swing, JIT compiler standard
2000 ─ J2SE 1.3
2002 ─ J2SE 1.4 — assert keyword, NIO, regex, exception chaining
2004 ─ J2SE 5.0 — MASSIVE release: Generics, Enums, Autoboxing,
        Annotations, Varargs, Enhanced for-loop, Static imports
2006 ─ Java SE 6 — renamed from "J2SE" to "Java SE"
2011 ─ Java SE 7 — try-with-resources, diamond operator, switch on Strings
2014 ─ Java SE 8 — HUGE release: Lambdas, Streams API, java.time,
        Functional Interfaces, default methods in interfaces
2017 ─ Java SE 9 — Java Platform Module System (JPMS), JShell
2018 ─ Java SE 10 — 'var' (local variable type inference)
        ── Release cadence changes to every 6 months from here ──
2018 ─ Java SE 11 (LTS) — new HTTP Client, single-file source launch
2021 ─ Java SE 17 (LTS) — Sealed classes, pattern matching for switch (preview)
2023 ─ Java SE 21 (LTS) — Virtual Threads, Record Patterns,
        Pattern Matching for switch (final), Sequenced Collections
2025 ─ Java SE 25 (LTS) — Latest LTS at time of writing; continues the
        modern-Java direction: compact source files, further pattern
        matching maturity, structured concurrency refinements
```

> **Note:** We'll revisit this timeline with concrete before/after code in Module 23 (Modern Java). For now, just anchor the *shape* of the timeline: slow-moving major releases (1996–2017), then a switch to a fast, predictable 6-month cadence (2017–present).

## Why the Release Model Changed (2017 onward)

Before 2017, major Java versions took **years** to ship (Java 6 → 7 was 5 years; Java 7 → 8 was 3 years) because every release bundled up *whatever big features happened to be ready*, and shipped when the biggest feature was ready — which was unpredictable.

**Problem this caused:** developers waited years for small, uncontroversial improvements just because they were bundled with one slow, ambitious feature (this literally happened — Java 8's release was delayed because of the Lambda/Streams work).

**Solution:** starting with Java 10, Oracle moved to a strict **6-month release train**: a new version ships every March and September, containing *whatever features are done by the deadline* — nothing waits. Half-finished features simply roll to the next train.

To balance "fast releases" with "enterprises need long-term stability," Java also introduced the **LTS (Long-Term Support)** designation:

| Release type | Cadence | Who uses it |
|---|---|---|
| Feature release | Every 6 months | Enthusiasts, early adopters, teams wanting the newest features immediately |
| **LTS release** | Roughly every 2 years (8, 11, 17, 21, 25, ...) | Most production enterprise systems — these get extended support/patches for years |

> **Tip:** When someone says "we're a Java 17 shop" or "we're upgrading to Java 21," they almost always mean an LTS version. Non-LTS versions (9, 10, 12, 13, 14, 15, 16, 18, 19, 20, ...) are largely stepping stones — most production teams skip straight from one LTS to the next.

## Why Study the History at All?

This isn't trivia for its own sake — it explains three things you'll otherwise find confusing:

1. **Why Java looks "old-fashioned" in some corners.** Features designed in 1995 (like `main`'s verbose signature) coexist with features from 2023 (like records and pattern matching) because Java has a very strong **backward compatibility** promise — old code must keep compiling and running on new JVMs. This is a deliberate trade-off: stability for enterprises over aggressive language cleanup.
2. **Why some "obviously good" features took decades to arrive.** Lambdas (functional-style code) didn't arrive until 2014 (Java 8) — 18 years after Java 1.0 — because retrofitting them safely into an already-massive, backward-compatible language ecosystem is enormously hard. This is a recurring theme: Java moves deliberately, not fast.
3. **Why version numbers matter when reading job postings, docs, or Stack Overflow answers.** "Since Java 8" or "requires Java 17+" tells you immediately whether a technique is available to you.

## Advantages of Java's Evolution Model

- Strong backward compatibility — code from the 1990s largely still compiles today.
- Predictable release cadence (post-2017) — teams can plan upgrades.
- LTS releases give enterprises stability without freezing the language forever.

## Disadvantages

- Backward compatibility means some early design mistakes (e.g., checked exceptions being arguably over-used, or verbose boilerplate before `var`/records) can't simply be removed — only supplemented with better alternatives.
- The fast 6-month cadence means many teams are perpetually "behind," running an LTS from years ago while new features accumulate elsewhere.

## Best Practices

- When learning a feature, always note which Java version introduced it — this course will consistently call this out.
- For production code, target the most recent LTS your infrastructure supports, not necessarily the newest release.

## Common Mistakes

- Assuming "the latest Java feature" is available in a codebase without checking its actual configured Java version (`pom.xml` / `build.gradle` source/target level).
- Confusing "Java 2" (marketing name for 1.2–1.4 era) with "Java SE 2" as a real version number — it doesn't exist as such; it was a branding phase.

## Interview Questions

1. **Q: Why was Java originally created, and for what kind of devices?**
   A: Created in 1991 under Sun's "Green Project" for consumer electronics (set-top boxes, embedded devices) with varying hardware, to avoid recompiling per device — hence the bytecode + JVM design.

2. **Q: What is an LTS release, and why does it exist?**
   A: A Long-Term Support version (8, 11, 17, 21, 25, ...) that receives extended patches/support over years, giving enterprises stability against Java's fast 6-month feature-release cadence.

3. **Q: Name two features introduced in Java 8, and why that release is considered a turning point.**
   A: Lambda expressions and the Streams API (also: the new `java.time` date/time API, default methods on interfaces). Java 8 is considered a turning point because it introduced functional-programming style into a previously purely object-oriented language — a huge philosophical and practical shift covered in Modules 17–18.

## Summary

- Java began in 1991 as an embedded-systems language ("Oak"), pivoted to the web in 1995 as "Java."
- Its foundational goal — "write once, run anywhere" — came directly from the pain of recompiling for many different hardware targets.
- Major eras: 1996 (1.0, WORA), 2004 (5.0, generics/enums/annotations), 2014 (8, lambdas/streams), 2017+ (6-month cadence + LTS model), 2023–2025 (21, 25 — virtual threads, pattern matching, records maturing).
- Backward compatibility is a core, deliberate Java value — it explains both its strengths (stability) and its quirks (verbosity that newer features now address).