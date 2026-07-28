# Module 01 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-module-overview.md) has been covered:

- [x] **What Is Java?** — definition, classification (statically typed, OOP, compiled-to-bytecode, garbage-collected), Java vs. JavaScript, PDF/PDF-reader analogy
- [x] **History of Java** — Green Project (1991), rename to Java, WORA era, full version timeline through Java 25, LTS model origins
- [x] **Features of Java** — all 11 classic features (Platform Independent, OOP, Simple, Secure, Robust, Architecture-Neutral, Portable, High Performance, Multithreaded, Distributed, Dynamic), each with its underlying mechanism
- [x] **JDK vs JRE vs JVM** — full layered comparison, diagram, table, distribution vendors
- [x] **How Java Works Internally** — compilation, bytecode, class loading, verification, interpreter, JIT, warm-up behavior, memory preview
- [x] **Setting Up Java** — installation across OSes, PATH vs JAVA_HOME, distribution landscape, verification steps
- [x] **Your First Java Program** — `HelloWorld.java` dissected keyword-by-keyword, compilation/execution, common mistakes, modern compact-source variation
- [x] **Java Editions & Version Timeline** — SE vs EE/Jakarta EE vs ME, LTS model in depth, version-by-version feature map through Java 25

## Practical Connections

Even at this early stage, the concepts in this module underpin everything you'll do professionally with Java:

- **Spring Boot** applications are ordinary Java SE programs (with a `main` method just like `HelloWorld.java`!) that bootstrap a large dependency-injection framework — the JVM startup/class-loading process you learned in Topic 5 is exactly what happens (at a much larger scale) when a Spring Boot app boots up.
- **Docker/Kubernetes deployments** of Java services package a JDK (or a minimal `jlink`-built runtime — Module 21) directly into a container image — the JDK/JRE/JVM distinctions from Topic 4 directly inform how small/large your container images end up.
- **Cloud/serverless environments** (AWS Lambda, etc.) are acutely sensitive to the JVM's warm-up behavior from Topic 5 — "cold start" latency in Java-based Lambda functions is a direct, practical consequence of the interpreter-then-JIT model, and is a major reason technologies like GraalVM Native Image (Module 22) exist.
- **Enterprise job postings** referencing "Java 17" or "Java 21" are referencing LTS versions from Topic 8 — you can now read those postings with real understanding of what's implied.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| Java vs JavaScript | Unrelated languages beyond a 1995 marketing name choice; different type systems, runtimes, purposes |
| JVM vs JRE vs JDK | Execute vs run vs develop — each layer strictly contains the previous |
| Compiled vs Interpreted (as applied to Java) | Java is both — `javac` compiles to bytecode; the JVM interprets that bytecode and JIT-compiles hot paths to native code |
| Java SE vs Java EE/Jakarta EE | SE = core platform (this course); EE = enterprise APIs built on top of SE |
| Feature release vs LTS release | Feature releases ship every 6 months, short support window; LTS releases get years of support and anchor most production systems |

## Consolidated Interview Questions (Module 01)

1. Why is Java called "platform independent"? What specifically is portable, and what specifically is not?
2. What is the difference between JDK, JRE, and JVM?
3. Is Java a compiled language or an interpreted language? Justify your answer precisely.
4. Why must Java's `main` method be `public static void main(String[] args)` exactly?
5. What problem was Java originally designed to solve, and how did that shape its core architecture?
6. What is bytecode, and why does it enable platform independence in a way that native machine code cannot?
7. What is JIT compilation, and why does a Java program often get faster the longer it runs?
8. Why does the JVM verify bytecode even when it was compiled by a trusted `javac` on your own machine?
9. What is the difference between Java SE and Java EE/Jakarta EE?
10. What is an LTS release, and why do most enterprises track LTS versions rather than the newest release?

*(Full answers with reasoning for all of these are in the respective topic files — this list is for self-testing before checking back.)*

## Module Exercises

1. **Recall test:** Close this course and, from memory, write a one-paragraph explanation of "how Java works" covering compilation, bytecode, class loading, and JIT — then compare against [Topic 5](05-how-java-works.md) and note any gaps.
2. **Hands-on:** If you haven't already, install a JDK, verify it with both `java -version` and `javac -version`, then write, compile, and run `HelloWorld.java` yourself.
3. **Hands-on variation:** Modify your `HelloWorld.java` to accept a name via command-line arguments and print a personalized greeting (see the Variations section of [Topic 7](07-first-java-program.md) if you need a hint).
4. **Diagram redraw:** Redraw the JDK/JRE/JVM nested-box diagram from [Topic 4](04-jdk-vs-jre-vs-jvm.md) from memory, and separately redraw the full compile-to-execution pipeline from [Topic 5](05-how-java-works.md).
5. **Explain to a beginner:** Write 3–4 sentences explaining to someone with zero programming background why "Write Once, Run Anywhere" was such a big deal in the 1990s, using an analogy that isn't the PDF/PDF-reader one used in this module.
6. **Version mapping:** Without looking back, name which Java version introduced: (a) Generics, (b) Lambdas & Streams, (c) `var`, (d) Sealed classes & finalized pattern matching for `instanceof`, (e) Virtual Threads. Then check your answers against [Topic 8](08-java-editions-and-versions.md).
7. **Research/Reflection:** Find one real job posting (or recall one you've seen) that mentions a specific Java version. Identify whether it's an LTS version, and what that implies about the team's likely feature set and upgrade cadence.

## What's Next

Module 01 gave you the *conceptual* foundation: what Java is, why it exists, and how it fundamentally executes. **Module 02 — JVM** goes deep into the JVM's internal architecture itself — class loader subsystems, runtime memory areas (Method Area, Heap, Stack, PC Registers, Native Method Stacks), and the execution engine — building directly on the "preview" you got in Topic 5 of this module.

---

**Previous:** [08 — Java Editions & Version Timeline](08-java-editions-and-versions.md) · **Module Overview:** [00 — Module Overview](00-module-overview.md)

**Type "Continue" to begin Module 02 — JVM.**
