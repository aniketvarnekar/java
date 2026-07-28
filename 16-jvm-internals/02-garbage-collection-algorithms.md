# Garbage Collection Algorithms

## Learning Objectives

- Understand the generational hypothesis and why the Heap is subdivided accordingly
- Understand the major GC algorithms (Serial, Parallel, G1, ZGC, Shenandoah) and their real trade-offs
- Understand "stop-the-world" pauses and why minimizing them is the central GC design challenge

## Prerequisites

Module 02 Topic 3 (Heap), Module 02 Topic 4 (Garbage Collector preview), Module 07 Topic 5 (reachability, GC Roots)

## Motivation

Module 02 and Module 07 established *what* the GC does (reclaim unreachable objects, Module 07, Topic 5's reachability model) — this topic covers *how*, with enough real algorithmic detail to make informed decisions in Module 22 (Performance) and to answer genuinely deep interview questions confidently, not just "the GC cleans up memory."

## The Generational Hypothesis — The Foundational Insight

**Empirical observation, true across the overwhelming majority of real Java programs**: **most objects die young.** A huge fraction of allocated objects (temporary calculation results, short-lived loop variables boxed into objects, request-scoped objects in a web server) become unreachable very shortly after creation, while a much smaller fraction survive for a long time (caches, connection pools, long-lived application state).

**This single observation drives the entire structure of modern generational garbage collectors** — rather than scanning the **entire** Heap every single time, the Heap is subdivided into **generations**, and the GC focuses its effort disproportionately on the region where most garbage actually accumulates:

```
┌──────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                            HEAP                                                      │
│                                                                                                      │
│  ┌──────────────────────────────────────────────┐      ┌──────────────────────────────────────────┐  │
│  │               YOUNG GENERATION               │      │              OLD GENERATION              │  │
│  │        (new objects allocated HERE)          │      │      (long-surviving objects             │  │
│  │                                              │      │       PROMOTED here)                     │  │
│  │  ┌────────────┐ ┌────────────┐ ┌────────────┐│      │                                          │  │
│  │  │    Eden    │ │ Survivor 0 │ │ Survivor 1 ││      │ Collected LESS often                     │  │
│  │  │ (most      │ │            │ │            ││      │ (objects here have already proven        │  │
│  │  │ objects    │ │            │ │            ││      │ they're long-lived, so scanning          │  │
│  │  │ allocated  │ │            │ │            ││      │ them constantly would be WASTED          │  │
│  │  │ here first)│ │            │ │            ││      │ effort)                                  │  │
│  │  └────────────┘ └────────────┘ └────────────┘│      │                                          │  │
│  │                                              │      │ A "Major/Full GC" collects this          │  │
│  │ Collected FREQUENTLY, cheaply                │      │ region (and often everything)            │  │
│  │ ("Minor GC")                                 │      │                                          │  │
│  └──────────────────────────────────────────────┘      └──────────────────────────────────────────┘  │
│                                                                                                      │
└──────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

**How promotion works**: new objects are allocated in **Eden**. A **Minor GC** (frequent, cheap, since Eden is typically small and most objects there are already garbage by the time it runs) copies surviving objects into a **Survivor** space. Objects that **survive multiple** Minor GC cycles (proving they're not short-lived after all) get **promoted** to the **Old Generation**, which is collected far less frequently, since (per the generational hypothesis) most objects that make it there tend to stay reachable for a long time — scanning them on every Minor GC would be wasted effort.

## "Stop-the-World" Pauses — The Central Challenge

**Historically, and still true to varying degrees for different modern collectors, garbage collection requires pausing some or all application threads** — you genuinely cannot safely reclaim/move objects while application code might be actively reading or modifying references to them at the exact same instant (recall Module 15's entire race-condition/visibility discussion — GC has to solve an analogous coordination problem, at a much larger scale). **This pause is called a "stop-the-world" (STW) event.**

**The entire, decades-long story of GC algorithm evolution is fundamentally about minimizing STW pause duration** — from early collectors that paused the entire application for the full duration of any collection, to modern collectors achieving sub-millisecond pauses even on multi-gigabyte heaps, through progressively more sophisticated concurrent/incremental techniques.

## The Major Collectors — A Comparative Tour

### Serial GC

**The simplest collector**: single-threaded, stops the entire application for the full duration of every collection. **Appropriate specifically for small applications with small heaps** (or single-core environments) where pause time genuinely doesn't matter — the simplicity has essentially zero coordination overhead.

### Parallel GC (historically the default for a long time)

**Uses multiple threads to perform garbage collection itself faster** (still stop-the-world, but the pause itself completes quicker by parallelizing the collection work across several CPU cores simultaneously). **Optimizes for maximum application **throughput** (total useful work done over time), accepting individually longer, though less frequent, pauses as an acceptable trade-off.**

### G1 (Garbage-First) GC — The Modern Default (Since Java 9)

**A significant architectural shift**: instead of two large, contiguous generational regions, G1 divides the Heap into **many small, equal-sized regions**, each independently classified as Eden, Survivor, or Old as needed. **G1's key innovation, per its name**: it prioritizes collecting the regions containing the **most garbage first** ("garbage-first"), since those regions offer the best "reclaimed memory per unit of pause time" ratio — a direct application of getting the most value for a bounded, targeted pause budget you configure (G1 lets you specify a **target maximum pause time**, and it adapts its collection strategy to try to meet that target).

```
 G1's Heap layout (many small regions, NOT two large fixed generations):

 [E][E][O][S][E][O][O][E][S][O][E][O] ...  (E=Eden, S=Survivor, O=Old -- regions
                                              dynamically assigned roles, unlike the
                                              old fixed-size generational layout)
```

### ZGC and Shenandoah — Ultra-Low-Latency, Mostly-Concurrent Collectors

**Both represent the cutting edge of GC engineering**: designed to keep STW pauses **extremely short (sub-millisecond) regardless of heap size** — even for multi-terabyte heaps — by performing the overwhelming majority of collection work **concurrently**, while the application continues running, using sophisticated techniques (colored pointers/load barriers for ZGC, Brooks pointers for Shenandoah — genuinely advanced implementation details beyond this course's scope, but worth knowing these names exist) to safely coordinate concurrent collection with an actively-running, actively-mutating application.

**Why does ultra-low latency matter enough to justify this engineering effort?** For latency-sensitive applications (real-time trading systems, large in-memory caches serving live traffic, systems with strict SLA response-time guarantees), even a Parallel GC's occasional longer pause can be genuinely unacceptable — ZGC/Shenandoah trade some raw throughput (they do more background bookkeeping work overall) for dramatically more **predictable, consistently short** pause times.

## Full Comparison

| Collector | Pause style | Best suited for |
|---|---|---|
| Serial | Full STW, single-threaded | Small apps, small heaps, single-core environments |
| Parallel | Full STW, multi-threaded (faster completion) | Maximum throughput, pause time less critical |
| **G1** (modern default) | Mostly STW, but targeted/bounded, region-based | General-purpose default — balances throughput and pause time well |
| ZGC / Shenandoah | Mostly concurrent, sub-millisecond STW | Latency-critical applications, very large heaps |

## Real-World Analogy

Think of **Serial GC** like **closing an entire small shop to do a full inventory count** — simple, and fine for a small shop where closing briefly doesn't hurt much. Think of **Parallel GC** like **closing the shop but bringing in a large team to finish the inventory count as fast as possible** — still fully closed, but for a shorter total duration. Think of **G1** like **a large warehouse doing inventory one small, high-priority section at a time**, always tackling whichever section has accumulated the most clutter first, closing only that small section briefly while the rest of the warehouse stays open. Think of **ZGC/Shenandoah** like **a warehouse that has figured out how to do nearly all of its inventory counting while customers are still actively shopping**, only very briefly, almost imperceptibly, pausing customer activity for the tiny handful of moments genuinely requiring it.

## Advantages

- Generational collection focuses effort where garbage actually accumulates, dramatically more efficient than scanning the entire Heap on every collection.
- Modern collectors (G1, ZGC, Shenandoah) offer genuinely configurable, predictable pause-time targets, letting applications tune GC behavior to their specific latency/throughput needs.

## Disadvantages / Trade-offs

- Ultra-low-latency collectors (ZGC/Shenandoah) trade some raw throughput/CPU overhead for their dramatically better pause-time consistency — not automatically the "best" choice for every workload.
- GC tuning remains a genuinely deep, specialized skill — this topic provides the conceptual foundation, with practical tuning guidance deferred appropriately to Module 22.

## Best Practices

- Use G1 (the modern default) unless you have a specific, measured reason to choose otherwise.
- Consider ZGC/Shenandoah specifically for genuinely latency-sensitive applications with large heaps.
- Understand the generational hypothesis when reasoning about object lifetime patterns in your own application design — creating unnecessary long-lived references to what should be short-lived objects undermines the GC's core optimization assumption.

## Common Mistakes

- Assuming all garbage collectors work identically — they have genuinely different algorithms and trade-offs, chosen deliberately for different workload shapes.
- Assuming "concurrent" collectors (ZGC/Shenandoah) have zero pause time at all — they minimize STW pauses dramatically, but don't eliminate every brief coordination pause entirely.
- Ignoring GC choice entirely for a latency-sensitive production application, when a deliberate collector choice (Module 22) could meaningfully improve real user-facing performance.

## Interview Questions

1. **Q: What is the generational hypothesis, and how does it shape GC design?**
   A: The empirical observation that most objects die young, while a smaller fraction live much longer. This motivates dividing the Heap into a Young Generation (collected frequently, cheaply, via Minor GC) and an Old Generation (collected less often, via Major/Full GC), focusing collection effort where garbage actually accumulates most.

2. **Q: What is a "stop-the-world" pause, and why has minimizing it been the central focus of GC algorithm evolution?**
   A: A pause where application threads are halted so the GC can safely reclaim/move objects without interference. Long STW pauses directly hurt application responsiveness/latency, so decades of GC engineering (Parallel → G1 → ZGC/Shenandoah) have progressively reduced pause duration, culminating in modern collectors achieving sub-millisecond pauses even on very large heaps.

3. **Q: What makes G1 different from older generational collectors, and why is it named "Garbage-First"?**
   A: It divides the Heap into many small, equal-sized regions (dynamically assigned Eden/Survivor/Old roles) rather than two large fixed generational regions, and prioritizes collecting the regions with the most reclaimable garbage first — maximizing memory reclaimed per unit of pause time, within a configurable target pause-time budget.

4. **Q: When would you choose ZGC or Shenandoah over G1?**
   A: For genuinely latency-sensitive applications requiring very short, predictable pause times (even on very large heaps), accepting some additional throughput/CPU overhead in exchange for that consistency — G1 remains an excellent general-purpose default otherwise.

## Summary

- The **generational hypothesis** (most objects die young) motivates dividing the Heap into Young (Eden + Survivor, collected frequently) and Old (collected less often) generations.
- **Stop-the-world (STW) pauses** are the central GC design challenge; algorithm evolution has progressively minimized their duration.
- **Serial** (simple, single-threaded), **Parallel** (multi-threaded, throughput-optimized), **G1** (region-based, targeted pause times, modern default), and **ZGC/Shenandoah** (mostly concurrent, sub-millisecond pauses, latency-optimized) represent progressively more sophisticated points on the throughput-vs-latency trade-off curve.