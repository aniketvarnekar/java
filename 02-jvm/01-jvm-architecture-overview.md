# JVM Architecture Overview

## Learning Objectives

- Name and locate the three major JVM subsystems and how they connect
- Understand the JVM as a specification with a defined architecture, not a monolithic black box
- Build the map you'll use for the rest of this module

## Prerequisites

[01-Introduction/04 — JDK vs JRE vs JVM](../01-introduction/04-jdk-vs-jre-vs-jvm.md), [01-Introduction/05 — How Java Works Internally](../01-introduction/05-how-java-works.md)

## Motivation

In Module 01 you learned the JVM "loads bytecode, verifies it, and runs it." That's true, but it's a black-box description. This module opens the box. Once you know the JVM's actual internal architecture, error messages that used to feel mysterious (`OutOfMemoryError: Metaspace`, `StackOverflowError`, `NoClassDefFoundError`) become perfectly logical — each one is a specific subsystem telling you specifically what went wrong.

## Problem Statement

The JVM has to do several genuinely different jobs:
- Find and load class definitions from disk/network into memory
- Store class metadata somewhere, and store running objects somewhere else, with different lifetimes
- Actually execute instructions
- Occasionally, call out to non-Java (native) code
- Manage memory automatically

Cramming all of this into one undifferentiated blob would be unmanageable to specify, implement, or reason about. So the **JVM Specification** defines it as a set of cooperating **subsystems**, each with one clear job — the same separation-of-concerns principle you saw with JDK/JRE/JVM in Module 01.

## The Three Major Subsystems (Plus One)

```
┌────────────────────────────────────────────────────────────────────────────────────┐
│                                      JVM                                           │
│                                                                                    │
│  ┌──────────────────────────────────────────────────────────────────────────────┐  │
│  │  1. CLASS LOADER SUBSYSTEM                                                   │  │
│  │                                                                              │  │
│  │  Finds .class files → Loads → Links → Initializes                            │  │
│  │                                                                              │  │
│  │  • Bootstrap Class Loader                                                    │  │
│  │  • Platform Class Loader                                                     │  │
│  │  • Application Class Loader                                                  │  │
│  │  • Parent Delegation Model                                                   │  │
│  └────────────────────────────────────────────┬─────────────────────────────────┘  │
│                                               │                                    │
│                                               │ Produces in-memory Class objects   │
│                                               ▼                                    │
│  ┌──────────────────────────────────────────────────────────────────────────────┐  │
│  │  2. RUNTIME DATA AREAS                                                       │  │
│  │                                                                              │  │
│  │  Shared Across All Threads              Per Thread                           │  │
│  │                                                                              │  │
│  │  ┌───────────────┐  ┌───────────────┐   ┌──────────────┐  ┌───────────────┐  │  │
│  │  │ Method Area   │  │ Heap          │   │ JVM Stack    │  │ PC Register   │  │  │
│  │  │ (Metaspace)   │  │ (Objects)     │   │ (Frames)     │  │               │  │  │
│  │  │ Class Metadata│  │               │   │              │  │               │  │  │
│  │  └───────────────┘  └───────────────┘   └──────────────┘  └───────────────┘  │  │
│  │                                                                              │  │
│  │                                           ┌───────────────────────────────┐  │  │
│  │                                           │ Native Method Stack           │  │  │
│  │                                           │ (For JNI Native Calls)        │  │  │
│  │                                           └───────────────────────────────┘  │  │
│  └────────────────────────────────────────────┬─────────────────────────────────┘  │
│                                               │                                    │
│                                               │ Execution Engine Reads/Writes      │
│                                               ▼                                    │
│  ┌──────────────────────────────────────────────────────────────────────────────┐  │
│  │  3. EXECUTION ENGINE                                                         │  │
│  │                                                                              │  │
│  │  ┌────────────────┐  ┌──────────────────┐  ┌────────────────────────────┐    │  │
│  │  │ Interpreter    │  │ JIT Compiler     │  │ Garbage Collector          │    │  │
│  │  │                │  │ (C1 / C2 / JVMCI)│  │                            │    │  │
│  │  └────────────────┘  └──────────────────┘  └────────────────────────────┘    │  │
│  └────────────────────────────────────────────┬─────────────────────────────────┘  │
│                                               │                                    │
│                                               │ Calls Native Code When Needed      │
│                                               ▼                                    │
│  ┌──────────────────────────────────────────────────────────────────────────────┐  │
│  │  4. JNI (Java Native Interface) & Native Libraries                           │  │
│  │                                                                              │  │
│  │  Bridge Between Java and Platform-Specific Native Code (C/C++)               │  │
│  └──────────────────────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────────────────┘
```

## Walking Through the Diagram

1. **Class Loader Subsystem** (Topic 2 of this module) — this is always the *first* thing that touches your `.class` file. It finds it (on disk, in a `.jar`, over a network), reads its bytecode, verifies it's safe, and produces an in-memory representation the rest of the JVM can use. **Nothing executes before this stage completes for a given class.**

2. **Runtime Data Areas** (Topic 3) — once a class is loaded, its metadata and any objects created from it need somewhere to actually *live in memory* while the program runs. This is not one undifferentiated memory pool — it's split into distinct regions with different lifetimes and different sharing rules (some shared across all threads, some private per thread). This split is precisely why Java has both a `StackOverflowError` and an `OutOfMemoryError: Java heap space` as **two different, specific exceptions** — they come from two different memory regions failing in two different ways.

3. **Execution Engine** (Topic 4) — this is the subsystem that actually *runs* the bytecode instructions, reading and writing to the Runtime Data Areas as it goes. It contains the Interpreter and JIT compiler(s) you previewed in Module 01, plus the Garbage Collector, which continuously reclaims Heap memory that's no longer reachable.

4. **Native Method Interface** (Topic 5) — sometimes Java code needs to call code that *isn't* Java at all — existing C libraries, OS-specific system calls not exposed any other way. JNI is the sanctioned bridge for this, and it's why the Runtime Data Areas diagram above shows a "Native Method Stack" living alongside the regular Java Stack per thread.

## Why This Architecture, Specifically?

This is another direct application of **separation of concerns**, but let's be precise about *why this particular split*, not some other one:

- Loading/verifying bytecode is a **one-time, per-class** concern — it happens once when a class is first used, then never again for that class. It naturally forms its own subsystem, distinct from the "runs continuously" concerns.
- Memory storage needs differ wildly by lifetime and sharing scope: class metadata (loaded once, shared by everyone) is fundamentally different from a method's local variables (created and destroyed constantly, private to one thread's current call), which is again different from heap objects (created unpredictably, potentially shared across threads, reclaimed only when truly unreachable). One-size-fits-all memory would be both slower and much harder to reason about — hence multiple, purpose-built Runtime Data Areas (full detail in Topic 3).
- Execution (actually running instructions) is conceptually separate from *where things are stored* — the Execution Engine's job is behavior, not storage, so it's its own subsystem that *uses* the Runtime Data Areas rather than being merged with them.

## Real-World Analogy

Think of the JVM like a **restaurant kitchen**:

- The **Class Loader** is like receiving and inspecting a delivery of ingredients (your bytecode) — checking it's not spoiled/tampered with (verification) before it's allowed into the kitchen.
- The **Runtime Data Areas** are like different storage zones in the kitchen: a shared pantry everyone draws from (Heap/Method Area), and each chef's own personal prep station that only they use, cleared after each dish (per-thread Stack).
- The **Execution Engine** is the chefs themselves actually cooking (executing instructions) — some following the recipe step-by-step every time (Interpreter), one veteran chef who's cooked a dish so many times they've streamlined it into muscle memory (JIT) — plus a cleanup crew continuously clearing finished plates and unused ingredients (Garbage Collector).
- The **Native Method Interface** is like calling an outside specialist supplier for something the kitchen itself can't produce in-house.

## Technical Explanation, Restated Precisely

Formally, per the JVM Specification, a running JVM instance consists of:
- **One** Class Loader subsystem (though it delegates across multiple individual loader objects — Topic 2)
- **One** shared Method Area and **one** shared Heap, both created at JVM startup and shared by every thread in the process
- **Per-thread** JVM Stacks, PC Registers, and Native Method Stacks — created when a thread starts, destroyed when it ends
- **One** Execution Engine driving instruction execution across all threads (conceptually — a real implementation like HotSpot maps this onto actual OS threads and CPU cores)

## Advantages of This Architecture

- Clear separation lets each subsystem be optimized, debugged, and even swapped (different GC algorithms, different JIT tiers) independently.
- Precise, specific error types (`StackOverflowError`, `OutOfMemoryError` with different messages) are possible *because* the memory model is subdivided this way — a huge debugging advantage over an undifferentiated memory model.
- Multiple vendors can build genuinely different, competing JVM implementations (Topic 6) against the same specification, exactly like multiple browsers implement the same HTML/CSS/JS specs.

## Disadvantages / Trade-offs

- More subsystems mean more configuration surface area (heap size, stack size, metaspace size, GC algorithm choice — all separately tunable, covered practically in Module 22).
- Understanding JVM behavior deeply requires understanding several interacting subsystems, not just one — there's a real learning curve, which is precisely why this module exists as a dedicated module rather than a single paragraph.

## Best Practices

- When debugging any JVM-related error, first ask: "which subsystem is this error coming from?" — this immediately narrows your investigation (e.g., `OutOfMemoryError` → Runtime Data Areas; `NoClassDefFoundError` → Class Loader Subsystem).
- Keep this diagram in your head as a map for the rest of this module — every subsequent topic zooms into exactly one box from this picture.

## Common Mistakes

- Treating "the JVM" as one undifferentiated black box that "just runs Java" — this makes debugging feel like guesswork instead of targeted investigation.
- Assuming all memory in the JVM is "the heap" — the Method Area/Metaspace, JVM Stacks, and PC Registers are distinct regions with different rules (full detail, Topic 3).

## Interview Questions

1. **Q: What are the major subsystems of the JVM?**
   A: The Class Loader Subsystem (finds/loads/links/initializes classes), the Runtime Data Areas (Method Area, Heap, JVM Stacks, PC Registers, Native Method Stacks — where everything lives in memory), the Execution Engine (Interpreter, JIT compiler(s), Garbage Collector — what actually runs the code and manages memory reclamation), and the Native Method Interface (bridges to non-Java native code).

2. **Q: Why does Java have both `StackOverflowError` and `OutOfMemoryError` as separate exception types?**
   A: Because the JVM's memory is subdivided into distinct Runtime Data Areas with different purposes — the (per-thread) JVM Stack overflowing (e.g., from unbounded recursion) is a specifically different failure from the (shared) Heap running out of space for new objects, so the JVM reports them as distinct, specifically diagnosable errors rather than one generic "out of memory" failure.

3. **Q: Is the JVM a single, standard piece of software?**
   A: No — the JVM is a **specification**. Multiple vendors ship different **implementations** of it (HotSpot, Eclipse OpenJ9, GraalVM, etc. — Topic 6), each free to implement the internal architecture differently (e.g., different GC algorithms, different JIT strategies) as long as they honor the same externally observable behavior defined by the spec.

## Summary

- The JVM is built from four cooperating parts: **Class Loader Subsystem → Runtime Data Areas → Execution Engine**, plus the **Native Method Interface** as a side bridge to native code.
- Each part exists because it has a genuinely distinct job: one-time class loading vs. ongoing memory storage vs. ongoing execution vs. occasional native interop.
- This architecture is a specification — different vendors implement it differently (Topic 6), which is exactly analogous to how JDK/JRE/JVM worked as a layered specification-driven model in Module 01.
- Every subsequent topic in this module zooms into exactly one box in the diagram above.