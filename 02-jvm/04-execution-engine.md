# Execution Engine

## Learning Objectives

- Explain what the Execution Engine does, and its three major components
- Understand tiered JIT compilation (C1 and C2) in more depth than Module 01's preview
- Understand the Garbage Collector's role as part of the Execution Engine's responsibilities
- Know how to observe JIT behavior yourself

## Prerequisites

[01 — JVM Architecture Overview](01-jvm-architecture-overview.md), [03 — Runtime Data Areas](03-runtime-data-areas.md), [01-Introduction/05 — How Java Works Internally](../01-introduction/05-how-java-works.md)

## Motivation

Module 01 gave you the *concept* of interpretation and JIT compilation. This topic gives you the *mechanism* in more depth — specifically, how modern HotSpot actually layers **two different JIT compilers** to get both fast startup and excellent peak performance, and how the Garbage Collector fits into "execution" as an ongoing, continuous JVM responsibility rather than a one-time cleanup step.

## Problem Statement

The Execution Engine must satisfy several competing goals simultaneously:
- Start executing bytecode **immediately** — no one wants to wait seconds before a program does anything.
- Eventually run **as fast as possible** for code that executes repeatedly (loops, frequently-called methods).
- Continuously reclaim memory from objects that are no longer needed — **without** stopping the whole program for unacceptable lengths of time.

Recall from Module 01: a purely interpreted approach satisfies the first goal but fails the second; a purely ahead-of-time compiled approach satisfies the second but not portability, and doesn't adapt to real runtime behavior. Java's Execution Engine is built specifically to satisfy all of the above together.

## The Three Components

```
┌────────────────────────────────────────────────────────────────────────────────────┐
│                               EXECUTION ENGINE                                     │
│                                                                                    │
│  ┌────────────────────────────────┐      ┌──────────────────────────────────────┐  │
│  │ Interpreter                    │◀────▶│ JIT Compiler                         │  │
│  │                                │      │                                      │  │
│  │ • Executes bytecode            │      │ Tier 1: C1 Compiler                  │  │
│  │   instruction by instruction   │      │ • Fast compilation                   │  │
│  │ • Starts execution immediately │      │ • Basic optimizations                │  │
│  │ • Profiles methods, loops,     │      │                                      │  │
│  │   and branches                 │      │ Tier 4: C2 Compiler                  │  │
│  │                                │      │ • Slower compilation                 │  │
│  │                                │      │ • Aggressive optimizations           │  │
│  │                                │      │ • Used for the hottest code          │  │
│  └────────────────────────────────┘      └──────────────────────────────────────┘  │
│                                                                                    │
│                     Frequently Executed ("Hot") Methods                            │
│                Are Compiled to Native Machine Code by the JIT                      │
│                                                                                    │
│  ┌──────────────────────────────────────────────────────────────────────────────┐  │
│  │ Garbage Collector                                                            │  │
│  │                                                                              │  │
│  │ • Reclaims unreachable heap objects                                          │  │
│  │ • Runs concurrently or during GC pauses, depending on the collector          │  │
│  │ • Helps manage memory automatically                                          │  │
│  └──────────────────────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────────────────┘
```

## The Interpreter, Revisited

As covered in Module 01, the interpreter reads and executes bytecode instructions one at a time, with no upfront compilation delay. What Module 01 didn't emphasize: **the interpreter is also constantly profiling** — silently counting how many times each method is invoked, and which branches inside loops are actually taken. This profiling data is exactly what the JIT compiler uses to decide *what's worth compiling*, and even *how* to optimize it (e.g., which branch of an `if` is overwhelmingly the common case).

## Tiered JIT Compilation: C1 and C2

Modern HotSpot (the default JVM implementation — Topic 6) doesn't use just one JIT compiler — it layers **two**, in a strategy called **tiered compilation**:

| Compiler | Nickname | Compiles... | Optimization level | Compilation speed |
|---|---|---|---|---|
| **C1** | "Client compiler" | Warm methods (moderately hot) | Light — quick wins, minimal analysis | Fast to compile |
| **C2** | "Server compiler" | Very hot methods (the true bottlenecks) | Heavy — aggressive inlining, loop unrolling, escape analysis, etc. | Slower to compile, but produces much faster code |

**Why layer two compilers instead of just picking one?** This is a direct engineering answer to a real trade-off:
- If you *only* had C1-style fast, light compilation, you'd get quick warm-up but leave real performance on the table for genuinely hot, long-running code paths.
- If you *only* had C2-style slow, heavy compilation, you'd get excellent peak performance eventually, but pay a much larger compilation delay before *any* method gets optimized — hurting startup and warm-up time badly.
- **Tiered compilation gets both:** methods get quick, modest optimization from C1 first (fast wins early), and only the methods that turn out to be *truly* hot over time get escalated to the expensive, high-payoff C2 optimization pass. This mirrors a general engineering principle: **spend your most expensive effort only where the payoff is proven to be worth it, and profile before you optimize.**

```
Method call count over time  ────────────────────────────────▶

  Calls 1-N        : Interpreted (no compilation yet, fastest start)
  Calls N+1 onward  : C1-compiled (light optimization -- "this looks warm")
  Calls M+1 onward  : C2-compiled (heavy optimization -- "this is genuinely hot")
                       (M is a much higher threshold than N)
```

### Observing JIT Behavior Yourself

You can actually watch this happen with a JVM flag:
```bash
java -XX:+PrintCompilation YourProgram
```
This prints a line every time a method gets compiled (by C1 or C2), including which tier compiled it — a genuinely eye-opening way to *see* the warm-up process happening live, rather than just trusting it exists. (We'll use this hands-on again in Module 22 — Performance.)

## De-optimization: The JIT Can Change Its Mind

An advanced but important fact: the JIT compiler sometimes makes **speculative optimizations** based on the specific behavior observed *so far* — for example, assuming a particular `if` branch is always taken, or that a method call always resolves to one specific implementation (relevant once you learn polymorphism — Module 05). If that assumption is later violated (a previously-untaken branch finally gets hit, or a different subclass implementation is called for the first time), the JVM can **de-optimize**: discard the compiled native code for that method and fall back to the interpreter temporarily, re-profiling before potentially re-compiling with corrected assumptions.

**Why this matters conceptually:** it's a vivid demonstration of something a purely static, ahead-of-time compiler fundamentally cannot do — make optimization decisions based on *actual observed runtime behavior*, and gracefully revise those decisions if reality changes. This is a genuine, structural advantage of Java's execution model, not just a performance detail.

## The Garbage Collector's Role in the Execution Engine

The Garbage Collector (GC) isn't a separate, occasional cleanup pass bolted onto the side — it's an integral, continuously (or periodically) running part of the Execution Engine's responsibilities, working in tandem with the Heap (Topic 3) to:
1. Identify which Heap objects are still **reachable** (accessible via some chain of references from an active thread's Stack, static fields, etc.)
2. Reclaim the memory of everything **unreachable**
3. Do this with minimal disruption to your running program (modern collectors aim for very short "stop-the-world" pauses, or avoid full pauses altogether)

**Full depth on GC algorithms (Serial, Parallel, G1, ZGC, Shenandoah) is deliberately deferred to Module 16 (JVM Internals)** — for this module, the key takeaway is architectural: **the GC is a first-class member of the Execution Engine**, not an afterthought, and it runs *concurrently* with your program's own execution, competing for the same CPU resources.

## Real-World Analogy

Think of the Interpreter as a **new employee following a written manual exactly, step by step, every single time** — reliable, and productive on day one, but slow because they never build muscle memory. C1 is like that employee, after doing a task a moderate number of times, **starting to streamline the obvious repeated steps** — a quick, low-effort speedup. C2 is like an employee who's done a task *so* many times that management invests serious effort into **redesigning their entire workflow from scratch**, worth doing only because the payoff (from doing this task thousands more times) clearly justifies the redesign cost. The Garbage Collector is like a **facilities team continuously tidying the shared workspace in the background**, reclaiming supplies (memory) that were used and then abandoned, without stopping everyone else's work for long.

## Advantages

- Tiered compilation gets the best of both worlds: fast startup (interpreter/C1) and excellent peak throughput (C2) for genuinely hot code.
- Runtime-informed optimization (and de-optimization) lets the JVM make smarter choices than a static compiler ever could, adapting to actual program behavior.
- The GC running as an integrated part of execution means memory safety doesn't require any manual work from you at all, ever.

## Disadvantages / Trade-offs

- Warm-up time is real and matters in latency-sensitive or very short-lived processes (CLI tools, serverless cold starts) — a recurring theme that motivates technologies like GraalVM Native Image (Module 22).
- JIT compilation itself consumes CPU cycles and memory (for the compiled code cache) — it's not free, just usually a good trade.
- GC pauses, however short in modern collectors, are still a real source of latency variance that performance-sensitive systems must account for (Module 15/16/22).

## Best Practices

- Never benchmark Java code without accounting for warm-up — a "cold" measurement (first few calls) measures the interpreter, not your code's true steady-state performance; proper microbenchmarking tools (like JMH, covered conceptually in Module 22) exist specifically to handle this correctly.
- When investigating a "why is this Java service slow to start" complaint, separate "class loading + JIT warm-up" as a hypothesis from "the code itself is inefficient" — they require completely different fixes.

## Common Mistakes

- Assuming there's only "one JIT compiler" — modern HotSpot layers C1 and C2 specifically to balance warm-up speed against peak performance.
- Assuming the GC is a rare, occasional event — in most Java programs, minor GC cycles happen quite frequently and briefly; it's a continuous background responsibility, not a rare emergency measure.
- Believing JIT-compiled code is permanent/fixed once compiled — de-optimization is a real, designed-for mechanism the JVM uses when its speculative assumptions turn out to be wrong.

## Interview Questions

1. **Q: What is tiered compilation, and why does HotSpot use two JIT compilers instead of one?**
   A: Tiered compilation layers a fast, lightly-optimizing compiler (C1) for warm methods and a slower, heavily-optimizing compiler (C2) reserved for genuinely hot methods. This balances fast program startup/warm-up (C1 kicks in quickly) against excellent peak throughput for the code that actually matters most (C2's expensive optimization is only spent where it's proven worthwhile).

2. **Q: What is JIT de-optimization, and why is it a meaningful capability rather than a bug?**
   A: The JVM sometimes compiles code based on speculative assumptions about observed runtime behavior (e.g., "this branch is always taken"). If that assumption is later violated, the JVM discards the compiled native code, falls back to interpretation, and can re-profile/re-compile with corrected assumptions. This is a deliberate capability that lets Java adapt to genuinely changing runtime behavior — something a static, ahead-of-time compiler cannot do at all.

3. **Q: Is the Garbage Collector part of the Execution Engine?**
   A: Yes — architecturally, the GC is one of the Execution Engine's core components, running continuously/periodically alongside the Interpreter and JIT compilers, not a separate, occasional subsystem.

## Summary

- The Execution Engine has three components: the **Interpreter** (immediate execution + profiling), the **JIT Compiler(s)** (C1 for fast/light compilation, C2 for slow/heavy compilation of the hottest code — "tiered compilation"), and the **Garbage Collector** (continuous memory reclamation).
- Tiered compilation exists to balance fast startup against excellent peak performance — a direct engineering trade-off resolution.
- The JIT can **de-optimize** speculative compilations when runtime reality contradicts its assumptions — a capability unique to runtime-informed compilation.
- The GC is a first-class, continuously-running member of the Execution Engine, not a bolted-on afterthought — full algorithm depth comes in Module 16.