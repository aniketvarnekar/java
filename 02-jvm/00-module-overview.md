# Module 02 — The JVM (Java Virtual Machine)

## Module Goal

Module 01 gave you a **preview** of the JVM in [05-How-Java-Works.md](../01-introduction/05-how-java-works.md): "bytecode goes in, the class loader loads it, the verifier checks it, the execution engine runs it." This module takes that preview and turns it into a full architectural understanding — **every subsystem inside the JVM, what it does, and why it's designed that way.**

This is the module that separates "someone who uses Java" from "someone who understands Java." Concepts here directly explain behavior you'll see constantly: `NoClassDefFoundError` vs `ClassNotFoundException`, `OutOfMemoryError: Java heap space` vs `StackOverflowError`, why static fields behave the way they do, why some frameworks (like Spring) can do "magic" class loading, and why performance tuning flags exist at all.

## Topics Covered in This Module

1. **[JVM Architecture Overview](01-jvm-architecture-overview.md)** — The big picture: the three major subsystems (Class Loader, Runtime Data Areas, Execution Engine) and how they fit together, plus the Native Interface.
2. **[Class Loader Subsystem](02-class-loader-subsystem.md)** — The three phases (Loading, Linking, Initialization) in full detail, the class loader hierarchy (Bootstrap, Platform, Application), the parent-delegation model, and why it exists.
3. **[Runtime Data Areas](03-runtime-data-areas.md)** — Method Area/Metaspace, Heap, Java Stack, PC Register, Native Method Stack — what lives where, per-thread vs shared, and a full Stack vs Heap comparison.
4. **[Execution Engine](04-execution-engine.md)** — The Interpreter, the tiered JIT compilers (C1/C2), and the Garbage Collector's role, building on the Module 01 preview.
5. **[Native Method Interface (JNI)](05-native-method-interface.md)** — How Java code calls native (C/C++) code, and why this capability exists at all.
6. **[JVM Implementations](06-jvm-implementations.md)** — HotSpot vs Eclipse OpenJ9 vs GraalVM — the JVM is a specification, not one program; here's what differs between real implementations.
7. **[Module Summary, Interview Questions & Exercises](07-module-summary-exercises.md)** — Consolidated recap, quiz, and practice problems.

## Prerequisites

- Module 01 (Introduction), especially Topic 4 (JDK vs JRE vs JVM) and Topic 5 (How Java Works Internally) — this module directly extends both.

## How to Study This Module

Topic 1 gives you the map; keep referring back to it as you go through Topics 2–5, each of which zooms into one region of that map. Topic 3 (Runtime Data Areas) is the most important for day-to-day debugging — `OutOfMemoryError` and `StackOverflowError` both originate from concepts taught there, and it's foundational for Modules 06–07 (Classes/Objects) and Module 16 (JVM Internals, which goes even deeper into bytecode and GC algorithms specifically).

---

**Previous module:** [01 — Introduction](../01-introduction/00-module-overview.md) · **Next:** [01 — JVM Architecture Overview](01-jvm-architecture-overview.md)
