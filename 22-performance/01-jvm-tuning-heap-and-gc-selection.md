# JVM Tuning: Heap & GC Selection

## Learning Objectives

- Configure heap size correctly using `-Xms`/`-Xmx`
- Choose and configure a garbage collector, applying Module 16, Topic 2's algorithm knowledge directly
- Understand the practical trade-offs behind these choices, not just the flag syntax

## Prerequisites

Module 16 Topic 2 (Garbage Collection Algorithms), Module 02 Topic 3 (Heap)

## Motivation

Module 16, Topic 2 gave you deep knowledge of *how* Serial, Parallel, G1, ZGC, and Shenandoah actually work. This topic makes that knowledge immediately actionable — the specific flags you'd actually set, and the specific reasoning for choosing one collector over another for a real, given workload.

## Heap Sizing — `-Xms` and `-Xmx`

```bash
java -Xms512m -Xmx2g -jar myapp.jar
```

- **`-Xms`** (initial heap size): how much heap the JVM allocates **upfront**, at startup.
- **`-Xmx`** (maximum heap size): the **ceiling** the heap (Module 02, Topic 3) is allowed to grow to before `OutOfMemoryError: Java heap space` (Module 02, Topic 3's famous error) becomes possible.

**Why set `-Xms` and `-Xmx` to the SAME value in many production deployments?** If they differ, the JVM must **dynamically resize** the heap as demand grows — a real, if usually modest, performance cost (and a source of some GC pause-time variability, Module 16, Topic 2). Setting them equal (`-Xms2g -Xmx2g`) **pre-allocates the full heap upfront**, trading a slightly larger initial memory footprint for more predictable, consistent runtime performance — a common, deliberate choice for latency-sensitive production services.

**Why not simply set `-Xmx` as high as possible?** Recall Module 16, Topic 2's generational hypothesis and stop-the-world discussion: a **larger heap generally means longer GC pauses when a full/major collection does eventually occur** (more memory to scan/compact) — bigger isn't unconditionally better; it's a genuine trade-off between "how often GC runs" and "how long each GC pause takes," directly informed by which collector (below) you're using.

## Choosing a Garbage Collector — Directly Applying Module 16, Topic 2

```bash
java -XX:+UseG1GC ...          # G1 -- the modern DEFAULT since Java 9
java -XX:+UseZGC ...             # ZGC -- ultra-low-latency, large heaps
java -XX:+UseShenandoahGC ...      # Shenandoah -- ultra-low-latency (an alternative to ZGC)
java -XX:+UseParallelGC ...          # Parallel -- maximum throughput, pause time less critical
java -XX:+UseSerialGC ...              # Serial -- small heaps/single-core environments
```

**This is a direct, practical application of Module 16, Topic 2's complete comparison table** — the flag you choose here is precisely the algorithmic trade-off decision that topic explained mechanistically. **A concrete, worked decision process:**

```
 Question: What's this application's actual workload shape?

 - Small heap, minimal resources (e.g., a simple CLI tool)?         -> SerialGC
 - Batch/offline processing, pause time doesn't matter?               -> ParallelGC (max throughput)
 - Typical web service/API, balanced needs?                             -> G1GC (the sensible DEFAULT)
 - Large heap (16GB+) AND strict, consistent low-latency requirements?    -> ZGC or Shenandoah
```

**G1's target pause time can itself be configured**, directly applying Module 16, Topic 2's "G1 adapts its strategy to try to meet a target pause time" description:

```bash
java -XX:+UseG1GC -XX:MaxGCPauseMillis=200 ...   # "try to keep GC pauses under 200ms"
```

## Metaspace Tuning — Directly Applying Module 02, Topic 3

Recall Module 02, Topic 3's PermGen → Metaspace history: Metaspace grows dynamically by default, bounded only by available native memory unless explicitly constrained:

```bash
java -XX:MaxMetaspaceSize=256m ...   # cap Metaspace growth explicitly
```

**Why would you deliberately cap it, given unbounded growth sounds convenient?** In a genuinely memory-constrained environment (a container with a strict memory limit, Module 13's practical connection), **unbounded** Metaspace growth could consume memory the rest of the system/container needs — an explicit cap trades "Metaspace can never artificially run out" for "predictable, bounded total memory usage," a real, practical production concern, especially relevant for applications doing heavy dynamic class generation/loading (some Reflection-heavy frameworks, Module 16, Topic 4).

## Stack Size — `-Xss`, Directly Applying Module 02, Topic 3

Recall Module 02, Topic 3's `StackOverflowError` discussion:

```bash
java -Xss512k ...   # per-thread Stack size (default varies by platform, often ~512KB-1MB)
```

**Why adjust this?** A genuinely deep, legitimate recursive algorithm (not a bug — a real, deliberate design, Module 04's recursion discussion) might need more Stack space than the default provides; conversely, in a platform-thread-heavy application (pre-Virtual-Threads, Module 15, Topic 8) with **many** threads, a **smaller** per-thread Stack size can meaningfully reduce total memory overhead across all those threads — directly connecting to Module 14, Topic 3's C10K memory-cost discussion.

## Real-World Analogy

Think of `-Xms`/`-Xmx` like **deciding how large a warehouse to lease upfront, and its maximum possible expansion size** — leasing the full maximum size immediately (`-Xms` = `-Xmx`) avoids the disruption of expanding mid-operation, at the cost of paying for unused space from day one. Think of GC algorithm selection like **choosing between different warehouse cleanup crews** (exactly Module 16, Topic 2's analogy, now with a concrete decision process attached) — a small shop needs only an occasional simple sweep (Serial); a huge, always-open, latency-sensitive operation needs a crew that tidies continuously in the background, almost invisibly (ZGC/Shenandoah), even if that crew costs more in ongoing overhead.

## Advantages

- Explicit, deliberate heap/GC configuration lets you match the JVM's behavior to your actual application's real workload shape and constraints.
- Understanding Module 16, Topic 2's algorithms deeply (not just flag syntax) lets you make genuinely informed, reasoned choices rather than cargo-culting flags from unrelated projects.

## Disadvantages / Trade-offs

- Over-tuning without measurement (Topic 2 of this module) can waste real engineering time chasing configuration changes that don't actually matter for your specific workload.
- JVM defaults have genuinely improved substantially over the years (G1's adaptive behavior, in particular) — manual tuning is increasingly needed only for genuinely unusual or demanding workloads, not routine applications.

## Best Practices

- Start with sensible modern defaults (G1, JVM-chosen default heap sizing) and only tune further based on **measured**, real evidence of a genuine problem (Topic 2) — never tune speculatively.
- Match GC choice to your actual workload's latency vs. throughput priorities, using Module 16, Topic 2's comparison table as your decision framework.
- Consider `-Xms` = `-Xmx` for latency-sensitive production services specifically to avoid dynamic heap-resizing overhead.

## Common Mistakes

- Tuning JVM flags speculatively, without first measuring (Topic 2) whether a genuine problem exists at all.
- Assuming "bigger heap is always better" — ignoring the real trade-off with longer potential GC pause durations.
- Never adjusting defaults at all for a genuinely unusual, demanding workload where the defaults are provably a poor fit — measured evidence should drive tuning either way.

## Interview Questions

1. **Q: What's the practical difference between setting `-Xms` and `-Xmx` to the same value versus different values?**
   A: Equal values pre-allocate the full heap upfront, avoiding dynamic resizing overhead and improving consistency — common for latency-sensitive production services. Different values let the heap start smaller and grow on demand, reducing initial memory footprint at the cost of some resize-related overhead/variability.

2. **Q: How would you choose between G1, ZGC, and Parallel GC for a given application?**
   A: Based on the workload's actual priorities (Module 16, Topic 2): G1 as a sensible general-purpose default balancing throughput and pause time; ZGC/Shenandoah for large heaps with strict, consistent low-latency requirements; Parallel GC for throughput-maximizing batch/offline workloads where pause time is less critical.

3. **Q: Why might you explicitly cap Metaspace size with `-XX:MaxMetaspaceSize`, given it grows dynamically by default?**
   A: In memory-constrained environments (like containers with strict limits), unbounded Metaspace growth could consume memory needed elsewhere — an explicit cap trades away "never artificially limited" for predictable, bounded total memory usage.

## Summary

- **`-Xms`/`-Xmx`** control initial and maximum heap size; setting them equal avoids dynamic resizing overhead, common for latency-sensitive services.
- **GC selection flags** (`-XX:+UseG1GC`, `-XX:+UseZGC`, etc.) directly apply Module 16, Topic 2's algorithm trade-offs to a real deployment decision.
- **`-XX:MaxMetaspaceSize`** and **`-Xss`** provide further, situational tuning, directly extending Module 02, Topic 3's memory model knowledge.
- Tuning should always be driven by **measured evidence** (Topic 2), never speculation.