# JDK vs JRE vs JVM

## Learning Objectives

- Precisely distinguish JDK, JRE, and JVM
- Know exactly what each one contains
- Never again confuse these three in conversation, documentation, or an interview

## Prerequisites

[01 — What Is Java?](01-what-is-java.md), [03 — Features of Java](03-features-of-java.md)

## Motivation

This is arguably **the single most commonly asked Java interview question**, and also one of the most commonly *mis*-answered — most people can say "JDK is bigger than JRE" but can't explain what's actually inside each layer, or why the layering exists at all. Get this rock solid now; it will make Module 02 (JVM) and Module 16 (JVM Internals) far easier to absorb.

## Problem Statement

Different people need different subsets of "Java" installed:

- A person who just wants to **run** a Java desktop app doesn't need a compiler — they never touch source code.
- A **developer** writing and building Java code needs the compiler, debugger, and doc-generation tools.
- Both of the above ultimately depend on something that can actually **execute** bytecode.

Bundling everything into a single, undifferentiated installer would waste space and blur responsibilities. Java's designers split this into three **layered** concepts.

## Concept

### JVM — Java Virtual Machine

**Definition:** The JVM is a **specification** (a document describing required behavior) with multiple **implementations** (actual software, e.g., HotSpot JVM, OpenJ9). Practically, the JVM is the engine that:
1. Loads `.class` files (bytecode) into memory
2. Verifies the bytecode is valid and safe
3. Executes it — via interpretation and/or JIT compilation
4. Manages memory (allocates objects on the heap, runs the Garbage Collector)
5. Manages threads

**Key insight:** the JVM is *platform-specific*. There is a distinct JVM binary for Windows x64, Linux ARM64, macOS Apple Silicon, etc. **Bytecode is what's portable — the JVM itself is not.** This is the resolution to a very common misconception (see Common Mistakes below).

### JRE — Java Runtime Environment

**Definition:** JRE = JVM + the **standard Java class libraries** (`java.lang`, `java.util`, `java.io`, and hundreds more packages) needed to actually run a typical Java application, + supporting files.

**Why it's a separate layer from the JVM:** the JVM alone can execute bytecode, but almost no real program is *pure* logic with zero use of standard classes — even `System.out.println` relies on the `java.lang.System` class from the standard library. The JRE bundles "the JVM + the minimum stuff basically every program needs."

**Who needs only a JRE:** historically, end users who just want to *run* Java desktop applications, with no intention of writing or compiling Java code.

> **Important 2024+ note:** Oracle stopped shipping standalone JRE-only downloads starting around Java 11. In modern Java, most people install a JDK even just to *run* things, or use `jlink` (Module 21) to build a minimal custom runtime image containing only the modules an application needs. The JDK-vs-JRE distinction is still conceptually essential (and still interview-relevant) even though separately-downloadable JRE installers are largely a thing of the past.

### JDK — Java Development Kit

**Definition:** JDK = JRE + **development tools**:
- `javac` — the compiler (source → bytecode)
- `javadoc` — generates HTML API documentation from source comments
- `jar` — packages `.class` files into a distributable `.jar` archive
- `jdb` — the command-line debugger
- `jshell` — the interactive Java REPL (since Java 9)
- `jlink` — builds custom, minimal runtime images (since Java 9)
- and more

**Who needs a JDK:** anyone *writing* Java code — every developer's machine, every CI/CD build server.

## Internal Working / Structure Diagram

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                                   JDK                                        │
│                 (Java Development Kit — For Developers)                      │
│                                                                              │
│  Dev Tools: javac · javadoc · jar · jdb · jshell · jlink                     │
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐  │
│  │                                JRE                                     │  │
│  │             (Java Runtime Environment — To Run Applications)           │  │
│  │                                                                        │  │
│  │  Standard Class Libraries:                                             │  │
│  │  java.lang · java.util · java.io · java.net · ...                      │  │
│  │                                                                        │  │
│  │  ┌──────────────────────────────────────────────────────────────────┐  │  │
│  │  │                              JVM                                 │  │  │
│  │  │          (Executes Bytecode; Platform-Specific)                  │  │  │
│  │  │                                                                  │  │  │
│  │  │  Class Loader                                                    │  │  │
│  │  │        │                                                         │  │  │
│  │  │        ▼                                                         │  │  │
│  │  │  Bytecode Verifier                                               │  │  │
│  │  │        │                                                         │  │  │
│  │  │        ▼                                                         │  │  │
│  │  │  Execution Engine (Interpreter + JIT)                            │  │  │
│  │  │        │                                                         │  │  │
│  │  │        ▼                                                         │  │  │
│  │  │  Garbage Collector                                               │  │  │
│  │  │        │                                                         │  │  │
│  │  │        ▼                                                         │  │  │
│  │  │  Runtime Memory Areas                                            │  │  │
│  │  └──────────────────────────────────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────────┘
```

Each outer layer **contains** the inner layer entirely, plus adds its own responsibilities. This is a classic "layered system" design — the same principle you'll see again in OS/network stacks.

## Comparison Table

| Aspect | JVM | JRE | JDK |
|---|---|---|---|
| Full name | Java Virtual Machine | Java Runtime Environment | Java Development Kit |
| Purpose | Execute bytecode | Run Java applications | Develop + build + run Java applications |
| Contains | Class loader, bytecode verifier, execution engine, GC | JVM + standard class libraries | JRE + compiler + dev tools |
| Includes `javac`? | No | No | **Yes** |
| Includes standard libraries? | No | **Yes** | Yes (via JRE) |
| Platform-specific? | **Yes** (different binary per OS/CPU) | Yes (bundles a specific JVM) | Yes |
| Typical installer today | N/A — always bundled | Rare as standalone (post Java 11) | **Standard install for both dev and prod today** |
| Who installs this | Nobody directly (it's embedded) | End users historically | Developers, build servers; commonly also production servers today |

## Why the Layering Exists (The "Why")

This is a textbook case of the **separation of concerns** principle:

- The JVM's job is narrowly "execute verified bytecode correctly and safely, on this specific platform." It shouldn't need to know anything about `String` or `ArrayList`.
- The JRE's job is "provide everything a compiled Java program needs to *run*." It shouldn't need to know how to compile anything.
- The JDK's job is "provide everything a human needs to *produce* a compiled Java program." It's the outermost, most feature-complete layer.

This mirrors a real-world analogy: think of a **car engine (JVM)**, the **whole drivable car including fuel, wheels, dashboard (JRE)**, and a **full mechanic's garage with diagnostic tools and a car-building assembly line (JDK)**. You can *drive* with just the car; you only need the garage if you're *building or fixing* cars.

## Advantages of This Separation

- Historically allowed lightweight deployment (ship just a JRE to end users, keep the heavier JDK on developer machines only).
- Conceptually clean: each layer has one clear job, which makes reasoning about "what does my server actually need installed" easier even today.

## Disadvantages / Modern Nuance

- The clean 3-way split has blurred in practice since Java 11 — most people now install "a JDK" everywhere, including production, since standalone JRE downloads are largely gone.
- `jlink` (Module 21) has partially replaced the old JRE concept with an even more precise idea: a **custom runtime image** containing only the modules your specific application needs (smaller than even a full JRE).

## Best Practices

- On a developer machine or CI server: install a **JDK**.
- On a minimal production container: consider a JDK still (for simplicity) or a `jlink`-produced custom runtime image (for minimal size) — covered later in Module 21.
- When someone says "install Java," clarify whether they mean "run Java apps" (JRE-equivalent needs) or "develop Java apps" (JDK) — in 2024+ practice, just install a JDK; it's the standard choice either way.

## Common Mistakes

| Mistake | Correction |
|---|---|
| "The JVM is platform-independent." | The JVM is platform-*specific* (a different binary per OS/CPU). It's the **bytecode** it runs that is platform-independent. |
| "JRE and JDK are basically the same thing." | JDK strictly contains everything in JRE, plus development tools like `javac`. You cannot compile source code with just a JRE. |
| "I need the JDK just to run a `.jar` file." | Not strictly true — historically a JRE was sufficient to *run* a compiled `.jar`. (Though in modern practice, most environments just install a JDK regardless, per the note above.) |
| Thinking JVM is a single universal program | It's a **specification**; multiple vendors ship different **implementations** (HotSpot — the default, used by Oracle/OpenJDK; Eclipse OpenJ9; GraalVM, etc.), each with different performance characteristics, covered in Module 22. |

## Interview Questions

1. **Q: What's the difference between JDK, JRE, and JVM?**
   A: JVM executes bytecode and is platform-specific. JRE bundles a JVM with the standard class libraries needed to run programs. JDK bundles a JRE with development tools (`javac`, `javadoc`, `jar`, etc.) needed to write and build programs. Each layer strictly contains the one before it.

2. **Q: If I only want to run a compiled `.jar` file someone gave me, what's the minimum I need installed?**
   A: Historically, just a JRE. In modern Java (11+), since standalone JRE installers are largely discontinued, practically you'd install a full JDK (which includes everything a JRE would have) or use `jlink` to produce a minimal custom runtime.

3. **Q: Is the JVM platform-independent?**
   A: No — this is a common trap. The JVM itself is platform-*specific* software (different binaries for Windows/Linux/macOS/different CPU architectures). What's platform-independent is the *bytecode* it executes — the same `.class` file runs correctly on any of these different JVM implementations, which is what creates the illusion that "the JVM is everywhere."

4. **Q: Name two development tools included in the JDK but not the JRE.**
   A: `javac` (compiler) and `javadoc` (documentation generator) — also acceptable: `jar`, `jdb`, `jshell`, `jlink`.

## Summary

- **JVM** = the execution engine (platform-specific).
- **JRE** = JVM + standard libraries (enough to run programs).
- **JDK** = JRE + dev tools like `javac` (enough to build and run programs).
- The layering follows separation-of-concerns: execute vs. run vs. develop.
- Modern practice (Java 11+) has mostly collapsed the "just install a JRE" use case — install a JDK, or use `jlink` for a minimal custom image.

## Mini Quiz

1. True or False: The JVM is platform-independent. *(False — bytecode is; the JVM is platform-specific.)*
2. Which of the three (JVM/JRE/JDK) contains `javac`? *(Only the JDK.)*
3. Which of the three is a specification with multiple different vendor implementations? *(The JVM.)*