# Module 02 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-module-overview.md) has been covered:

- [x] **JVM Architecture Overview** — the four-part map (Class Loader Subsystem, Runtime Data Areas, Execution Engine, Native Method Interface), why this specific separation of concerns exists
- [x] **Class Loader Subsystem** — Loading/Linking(Verification, Preparation, Resolution)/Initialization phases, lazy initialization rules, the Bootstrap/Platform/Application loader hierarchy, the parent delegation model and why it exists, custom class loaders and class identity
- [x] **Runtime Data Areas** — Method Area/Metaspace (and the PermGen→Metaspace history), Heap, JVM Stack, PC Register, Native Method Stack; full Stack vs Heap comparison with diagrams; where references vs. objects actually live
- [x] **Execution Engine** — Interpreter + profiling, tiered JIT compilation (C1/C2), JIT de-optimization, the Garbage Collector as an integrated Execution Engine component
- [x] **Native Method Interface (JNI)** — the `native` keyword, why the bridge exists, how it maps onto the Native Method Stack, trade-offs (lost portability, lost safety)
- [x] **JVM Implementations** — specification vs. implementation, HotSpot vs. Eclipse OpenJ9 vs. GraalVM, GraalVM Native Image as a concrete trade-off instance of the interpret-vs-compile theme from Module 01

## Practical Connections

- **Spring Boot startup time** and the well-known "fat JAR cold start" discussion in the Java ecosystem is a direct, practical consequence of Class Loading (Topic 2) + Execution Engine warm-up (Topic 4) — this is exactly why Spring Boot 3+ has invested heavily in GraalVM Native Image support (Topic 6).
- **Docker container sizing** for Java services is directly informed by Method Area/Metaspace (Topic 3) and the JDK/JRE choice (Module 01) — minimal container images often use `jlink` (Module 21) or Native Image (Topic 6) specifically to shrink what needs to be shipped.
- **Debugging production incidents**: an `OutOfMemoryError: Java heap space` (Topic 3), a `StackOverflowError` from a runaway recursive call in business logic (Topic 3), or a `NoClassDefFoundError` from a missing dependency `.jar` at runtime (Topic 2) are all everyday, real production issues you can now diagnose with a precise mental model instead of guesswork.
- **AWS Lambda / serverless Java functions**: cold-start latency complaints are a direct, practical consequence of Topic 4's warm-up behavior — and GraalVM Native Image (Topic 6) is the industry's direct engineering answer to exactly this problem.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| `ClassNotFoundException` vs `NoClassDefFoundError` | The former: explicit runtime lookup by name fails (e.g. `Class.forName`). The latter: a class was present at compile time but missing at runtime when actively needed. |
| `StackOverflowError` vs `OutOfMemoryError: Java heap space` | Former: per-thread JVM Stack exhausted (usually runaway recursion). Latter: shared Heap exhausted by too many live, reachable objects. |
| PermGen vs Metaspace | PermGen (pre-Java 8): fixed-size, carved out of the Heap, leak-prone. Metaspace (Java 8+): native/off-heap, grows dynamically by default. |
| Object vs Reference | The object's data always lives on the Heap; a variable of an object type holds only a reference (pointer) to it, and that reference itself lives on the Stack (or as a field inside another Heap object). |
| C1 vs C2 JIT compiler | C1: fast to compile, modest optimization, for warm code. C2: slow to compile, heavy optimization, reserved for the hottest code — used together via tiered compilation. |
| Standard JVM execution vs GraalVM Native Image | Standard: bytecode + runtime interpret/JIT, adaptive, portable, has warm-up cost. Native Image: ahead-of-time compiled to a platform-specific native executable, near-instant startup, no runtime adaptivity, loses bytecode portability. |

## Consolidated Interview Questions (Module 02)

1. What are the major subsystems of the JVM, and how do they relate to each other?
2. What are the three phases of class loading? What happens in each?
3. When exactly is a class initialized in Java — is it eager or lazy, and what specifically triggers it?
4. What is the parent delegation model, and what security problem does it solve?
5. Can two classes with the same fully-qualified name coexist in one JVM? Under what condition?
6. What's the difference between the Stack and the Heap? Where does an object variable's *reference* live versus the actual *object data*?
7. What replaced PermGen, when, and why?
8. What causes a `StackOverflowError` versus an `OutOfMemoryError: Java heap space`?
9. What is tiered JIT compilation, and why does HotSpot use two compilers (C1/C2) instead of one?
10. What is JIT de-optimization, and why is it possible only because Java compiles at runtime rather than purely ahead-of-time?
11. What is JNI, and what does Java give up (safety, portability) at that boundary?
12. Is "the JVM" one piece of software? Name two real, independent implementations and one key way they differ.
13. What is GraalVM Native Image, and what does it trade away to achieve near-instant startup?

*(Full reasoning for every answer is in the respective topic file — use this list to self-test before checking back.)*

## Module Exercises

1. **Recall test:** From memory, redraw the full JVM architecture diagram from [Topic 1](01-jvm-architecture-overview.md), then annotate each box with which specific error types or observable behaviors originate from it (e.g., `StackOverflowError` → JVM Stack).
2. **Hands-on:** Write a small Java program with a `static` initializer block that prints a message, and a `main` method that first merely *declares* a variable of that type, then later *actively uses* it (e.g., accesses a static field). Run it and confirm the initializer only runs at the point of active use, as explained in Topic 2.
3. **Hands-on:** Write a deliberately infinite (no base case) recursive method and run it to observe a real `StackOverflowError`. Read the stack trace and identify how it reflects the JVM Stack concept from Topic 3.
4. **Hands-on:** If you have a JDK installed, run any small program with `java -XX:+PrintCompilation YourProgram` (Topic 4) and observe real compilation tier output. Identify at least one line showing a method being compiled.
5. **Conceptual:** Explain, step by step, why a variable reassignment inside a method never affects the caller's own reference to an object — ground your answer specifically in the Stack-vs-Heap model from Topic 3.
6. **Conceptual:** A colleague proposes using GraalVM Native Image for a long-running, CPU-intensive backend service that stays warm for weeks at a time. Based on Topic 6, is this likely to help or hurt peak performance, and why?
7. **Synthesis:** Explain, using concepts from this entire module, why `System.out.println` inside your `HelloWorld.java` from Module 01 actually manages to print anything at all — trace it through class loading, the Method Area's constant pool, the Heap (where the `String` literal object lives), the JVM Stack (the `main` frame), and the Execution Engine.

## What's Next

Module 02 gave you the JVM's internal architecture: how classes get loaded, where everything lives in memory, how execution actually happens, and how different implementations approach these problems differently. **Module 03 — Java Basics** shifts from "how the JVM works" to "how to actually write Java code" — variables, primitive data types (and exactly how/where they're stored, building directly on this module's Stack/Heap model), operators, and type conversion rules.

---

**Previous:** [06 — JVM Implementations](06-jvm-implementations.md) · **Module Overview:** [00 — Module Overview](00-module-overview.md)

**Type "Continue" to begin Module 03 — Java Basics.**
