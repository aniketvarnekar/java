# How Java Works Internally

## Learning Objectives

- Trace the complete journey of a Java program from source code to running process
- Understand compilation, bytecode, class loading, interpretation, and JIT compilation
- Understand why this hybrid model exists, and what it trades off against pure compilation or pure interpretation
- Build the mental model that Module 02 (JVM) and Module 16 (JVM Internals) will build directly on top of

## Prerequisites

[01 — What Is Java?](01-What-Is-Java.md), [04 — JDK vs JRE vs JVM](04-JDK-vs-JRE-vs-JVM.md)

## Motivation

This is the most important topic in the entire Introduction module. Nearly every "why does Java do X" question in later modules (why is String immutable, why does GC exist, why is Java "slow to start but fast once warmed up," why does reflection work) ultimately traces back to the execution model explained here. Read it slowly.

## Problem Statement

We need a single mechanism that satisfies **all** of these simultaneously:
1. Portability — one build artifact, runs anywhere a JVM exists.
2. Safety — untrusted code shouldn't be able to corrupt memory or bypass access rules.
3. Reasonable performance — competitive with natively compiled languages for long-running programs.

Pure interpretation (read source/bytecode line-by-line, execute immediately, every single time) satisfies portability and is simple to make safe, but is slow — no optimization opportunity. Pure ahead-of-time (AOT) native compilation (like C) is fast, but loses portability entirely. Java's answer: **compile to an intermediate, portable format (bytecode), then execute that bytecode using a smart runtime (the JVM) that can both interpret and dynamically recompile to native code.**

## The Full Journey: Source Code to Running Program

```
STAGE 1: WRITE              STAGE 2: COMPILE              STAGE 3: LOAD & RUN
────────────────────        ────────────────────          ─────────────────────────

HelloWorld.java   ───▶   javac (Compiler)   ───▶   HelloWorld.class   ───▶   java
(Source Code)            (Syntax Checking          (Platform-Independent       │
                           + Bytecode Generation)    Bytecode)                 │
                                                                               ▼
                 ┌────────────────────────────────────────────────────────────────────┐
                 │                                JVM                                 │
                 │                                                                    │
                 │  1. Class Loader                                                   │
                 │     • Finds and loads .class files into memory                     │
                 │                                                                    │
                 │  2. Bytecode Verifier                                              │
                 │     • Verifies safety, integrity, and correctness                  │
                 │                                                                    │
                 │  3. Execution Engine                                               │
                 │                                                                    │
                 │     ┌───────────────────────┐                                      │
                 │     │ Interpreter           │                                      │
                 │     │ • Executes bytecode   │                                      │
                 │     │   instruction-by-     │                                      │
                 │     │   instruction         │                                      │
                 │     │ • Fast startup        │                                      │
                 │     └───────────┬───────────┘                                      │
                 │                 │                                                  │
                 │                 │ Frequently Executed ("Hot") Code                 │
                 │                 ▼                                                  │
                 │     ┌───────────────────────┐                                      │
                 │     │ JIT Compiler          │                                      │
                 │     │ • Compiles hot        │                                      │
                 │     │   bytecode into       │                                      │
                 │     │   native machine code │                                      │
                 │     │ • Caches compiled     │                                      │
                 │     │   code for reuse      │                                      │
                 │     └───────────────────────┘                                      │
                 │                                                                    │
                 │  4. Runtime Memory Areas                                           │
                 │     • Heap                                                         │
                 │     • Java Stacks                                                  │
                 │     • PC Registers                                                 │
                 │     • Native Method Stacks                                         │
                 │     • Metaspace                                                    │
                 │                                                                    │
                 │  5. Garbage Collector                                              │
                 │     • Reclaims unused heap memory                                  │
                 └────────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼
                             Program Output / Behavior
```

Let's walk through each stage in detail.

## Stage 1: Compilation (`javac`)

You run:

```bash
javac HelloWorld.java
```

`javac` does two jobs:

1. **Syntax & semantic checking** — Is every statement grammatically valid Java? Do the types match (`int x = "hello"` is rejected *here*, at compile time — this is what "statically typed" buys you in practice)? Are all referenced classes/methods actually accessible?
2. **Bytecode generation** — If everything checks out, `javac` emits a `.class` file: not your source text, and not native CPU instructions, but **bytecode** — a compact, well-defined instruction set for a *hypothetical* stack-based computer (the "Java Virtual Machine" as defined by its specification).

**What bytecode actually looks like:** each `.class` file contains a sequence of numeric opcodes. For example, the tiny expression `1 + 2` might compile to bytecode instructions conceptually like:

```
iconst_1        // push int constant 1 onto the operand stack
iconst_2        // push int constant 2 onto the operand stack
iadd             // pop top two ints, add them, push result
istore_1         // pop result, store into local variable slot 1
```

You can inspect real bytecode yourself with `javap -c HelloWorld.class` (the disassembler tool, included in the JDK) — we'll use this hands-on in Module 16 (JVM Internals) once you're comfortable with more Java syntax.

> **Key idea:** bytecode is *not* tied to Windows, Linux, macOS, x86, or ARM. It's tied only to the **JVM specification** — an abstract machine definition. This is the actual mechanism behind platform independence, made concrete.

## Stage 2: Class Loading

When you run:

```bash
java HelloWorld
```

The JVM starts up as an OS process, and its **Class Loader** subsystem locates `HelloWorld.class` and reads it into memory. Class loading itself has three sub-phases (full detail in Module 02):

1. **Loading** — find the bytecode (from the file system, a `.jar`, or the network) and bring it into the JVM's memory as a `Class` object.
2. **Linking** — verify the bytecode is well-formed and safe (**Bytecode Verification** — see below), prepare default values for static fields, and resolve symbolic references between classes.
3. **Initialization** — run static initializers and assign real initial values to static fields.

## Stage 3: Bytecode Verification

Before executing *any* bytecode — even bytecode from a completely trusted, self-compiled `.class` file — the JVM runs it through the **Bytecode Verifier**. This checks things like:
- No operand stack underflows/overflows (never trying to pop more values than were pushed)
- No illegal type conversions (never treating an `int` slot as if it were an object reference)
- No access-control violations (a piece of code doesn't call a `private` method it has no right to call)

**Why verify even trusted, self-compiled code?** Because `.class` files can come from anywhere — a downloaded library, a network source, a hand-modified file — and the JVM cannot assume every bytecode file it's asked to run was honestly produced by a well-behaved `javac`. This is the concrete mechanism behind the "Secure" feature from Topic 3.

## Stage 4: Execution — Interpreter and JIT Compiler

This is the heart of Java's performance story, and the most commonly misunderstood part of "how Java works."

### The Interpreter

The simplest way to execute bytecode: read one instruction, execute it, move to the next — repeat. This is what happens **first**, for every method, the very first time it's called. It's fast to *start* (no compilation delay) but slow to *run repeatedly*, since every single execution re-does the work of decoding and dispatching each instruction.

### The JIT (Just-In-Time) Compiler

The JVM continuously **profiles** your running program: it counts how often each method is called, and which branches are actually taken. When a method is called enough times to be considered "**hot**" (a JVM-tunable threshold), the **JIT compiler** kicks in and compiles that specific method's bytecode into **real, native machine code** for the current CPU — and caches it. All future calls to that method run the compiled native version directly, skipping the interpreter entirely — at (or near) the speed of a natively compiled C program.

```
 Call count for method foo()
 ────────────────────────────
   1st call   ─▶ interpreted (slow path, but starts immediately)
   2nd call   ─▶ interpreted
   ...
   nth call   ─▶ JVM notices foo() is "hot"
   n+1 call   ─▶ JIT compiles foo() to native machine code (happens in background)
   n+2 call   ─▶ runs the CACHED NATIVE VERSION directly (fast!)
```

**This is why Java has "warm-up" behavior:** a Java program is often slower in its first few seconds (everything is freshly interpreted) and gets faster as the JVM identifies and compiles hot paths — which is the opposite of a natively compiled C program, which runs at full speed immediately but never "learns" from actual runtime behavior the way the JIT does.

> **Modern nuance:** Production JVMs (like HotSpot) actually run **two** JIT compilers in tiers — **C1 (client compiler)**, which compiles quickly with modest optimization, and **C2 (server compiler)**, which takes longer to compile but produces much more optimized code, for the *very* hottest methods. This "tiered compilation" gets you fast warm-up *and* excellent peak performance. Full depth in Module 16 and Module 22.

### Why This Hybrid (Interpret + JIT) Beats Either Extreme

| Approach | Startup speed | Peak throughput | Portability |
|---|---|---|---|
| Pure interpretation (no JIT) | Fast | Poor (repeats decode/dispatch work forever) | Excellent |
| Pure AOT native compilation (like C) | Instant at "full speed" | Excellent | None (recompile per platform) |
| **Java's hybrid (interpret, then JIT hot paths)** | Fast (interpreter starts immediately) | **Approaches native, adapts to actual runtime behavior** | **Excellent (bytecode is portable)** |

The JIT can even make decisions a static AOT compiler fundamentally *cannot* — like **inlining** a method based on what implementation is *actually* being called at runtime (relevant for polymorphism, Module 05), because it has live information a compiler working purely on source code ahead of time never has access to.

## Stage 5: Memory Management (Preview)

While your bytecode executes, the JVM is also managing memory automatically:
- Objects you create (`new SomeClass()`) are allocated on the **Heap** — a large, shared memory region.
- Method calls, local variables, and intermediate calculation results live on a per-thread **Stack**.
- The **Garbage Collector** periodically identifies heap objects no longer reachable by your program and reclaims their memory — you never call `free()`.

This is intentionally just a preview — full depth (Stack vs Heap diagrams, GC algorithms) is in Module 16. For now, just know: **execution and memory management happen together, continuously, inside the running JVM process** — they are not separate stages that happen once.

## Full Execution Flow, End to End

```
 [You write HelloWorld.java]
              │
              ▼  javac HelloWorld.java
 [HelloWorld.class -- portable bytecode -- created on disk]
              │
              ▼  java HelloWorld
 [OS starts a JVM process]
              │
              ▼
 [Class Loader reads HelloWorld.class into memory]
              │
              ▼
 [Bytecode Verifier checks it's safe & well-formed]
              │
              ▼
 [Execution Engine begins running main()]
              │
      ┌───────┴────────┐
      ▼                ▼
 [Interpreter runs   [JIT compiles "hot"
  bytecode line       methods to native
  by line, at first]  code over time]
              │
              ▼
 [Heap/Stack memory allocated as objects/variables created]
              │
              ▼
 [Garbage Collector reclaims unreachable objects, continuously]
              │
              ▼
 [Program produces output, eventually finishes / JVM exits]
```

## Real-World Analogy (Restated Technically Here)

Think of the interpreter as **reading a recipe out loud, step by step, every single time you cook the dish** — correct, but slow if you cook that same dish 10,000 times. The JIT compiler is like **noticing you've cooked this exact dish so often that you memorize the whole sequence as one fluid motion** — you stop consciously reading each step, and just *do it*, much faster, but it took you a few repetitions to reach that point. Bytecode itself is like a **recipe written in a universal, standardized format** that any trained chef (any JVM) can follow — regardless of what kitchen (OS) they're in.

## Advantages of This Model

- Genuine platform independence, with real performance headroom via JIT.
- Runtime adaptivity — optimization decisions based on real, observed execution behavior, not just static guesses.
- Strong, enforced safety net (verification) regardless of where the bytecode came from.

## Disadvantages / Trade-offs

- Startup/warm-up latency — a real, practical pain point (this is precisely why technologies like **GraalVM Native Image** and **Ahead-of-Time (AOT) compilation** exist for Java today, to trade some of the JIT's adaptivity for instant startup in contexts like serverless functions — covered in Module 22).
- Higher baseline memory footprint than a minimal native binary (JVM metadata, JIT-compiled code cache, GC bookkeeping all cost memory).

## Best Practices

- When reasoning about Java performance, always ask "has this code path warmed up yet?" — a microbenchmark that doesn't account for JIT warm-up will produce misleading numbers (this becomes very concrete in Module 22).
- Don't assume "compiled" and "interpreted" are mutually exclusive labels for a language — Java is a genuine hybrid, and explaining *why* is a strong interview signal.

## Common Mistakes

- "Java is interpreted, so it's always slow." — Wrong; ignores the JIT compiler entirely.
- "Java compiles directly to machine code, like C." — Wrong; it compiles to bytecode first, an intermediate portable format, only *later* (at runtime, selectively) compiled to native code by the JIT.
- Believing bytecode verification only matters for "malicious" code — it also catches bugs in tooling that generates bytecode directly (some frameworks/libraries generate bytecode dynamically at runtime — Module 16), not just hand-tampered files.

## Performance Considerations

- JIT compilation happens on background threads in modern JVMs — it doesn't halt your program to "pause and compile," though it does consume CPU resources while doing so.
- Long-running server applications benefit the most from JIT (more time to warm up); short-lived CLI scripts benefit the least (they may never get past the interpreter for most of their code) — this is a real, practical reason some teams choose other languages or Java + Native Image for short-lived processes like CLI tools or serverless functions.

## Interview Questions

1. **Q: Walk me through what happens when I run `java HelloWorld` after compiling it.**
   A: The JVM starts as an OS process → the Class Loader locates and reads `HelloWorld.class` → the Bytecode Verifier checks it's safe/well-formed → the Execution Engine begins interpreting bytecode starting at `main` → the JVM profiles method calls and JIT-compiles frequently executed methods into native code over time → memory is allocated on the heap/stack as needed and reclaimed by the GC → the program runs to completion and the JVM exits.

2. **Q: Is Java compiled or interpreted?**
   A: Both — genuinely a hybrid. `javac` performs real compilation (source → bytecode). The JVM then interprets that bytecode directly, and simultaneously profiles execution to JIT-compile hot methods into native machine code at runtime, which then run without further interpretation.

3. **Q: Why might a Java program be slow for the first few seconds but fast afterward?**
   A: Because execution starts via the interpreter (no compilation delay, but slower per-instruction), and the JIT compiler only kicks in for a method once it's been observed as "hot" (called frequently enough) — at which point it's compiled to native code and runs much faster on subsequent calls. This is called the "warm-up" period.

4. **Q: What does the Bytecode Verifier check for, and why does it run even on code you compiled yourself?**
   A: It checks for stack over/underflows, illegal type conversions, and illegal access before allowing any bytecode to execute — because the JVM cannot assume every `.class` file it's asked to run was produced by a trustworthy, unmodified `javac` output; `.class` files can come from anywhere.

## Summary

- The journey: **source (`.java`) → `javac` → bytecode (`.class`) → Class Loader → Bytecode Verifier → Execution Engine (Interpreter + JIT) → running program**, with the Garbage Collector managing memory throughout.
- Bytecode is the portable artifact; the JVM (and its interpreter/JIT) is the platform-specific engine.
- The Interpreter gives fast startup; the JIT gives strong peak performance by compiling "hot" methods to native code based on real runtime profiling — something a purely ahead-of-time compiler can't do as effectively.
- This hybrid model is *the* mechanism behind "write once, run anywhere" *and* "fast enough for production," simultaneously.

## Exercises

1. Draw the full "source to running program" pipeline from memory, labeling every stage (compilation, class loading, verification, interpretation, JIT).
2. Explain, in your own words, why a program's *first* run tends to be slower than later runs of the *same* long-running process.
3. A friend claims: "Since Java has a compiler, it must produce a native executable, just like C." Correct this misconception precisely, referencing what `javac` actually outputs.
4. Why does the JVM verify bytecode even for `.class` files you personally compiled with your own trusted `javac`?

---

**Previous:** [04 — JDK vs JRE vs JVM](04-JDK-vs-JRE-vs-JVM.md) · **Next:** [06 — Setting Up Java](06-Setting-Up-Java.md)
