# Object Lifecycle & Garbage Collection

## Learning Objectives

- Trace an object's complete lifecycle, from creation to reclamation
- Understand "reachability" precisely, and what GC Roots are
- Understand why `finalize()` is deprecated, and what replaced it
- Preview `AutoCloseable`/try-with-resources as the modern resource-cleanup mechanism

## Prerequisites

Module 02 Topic 3 (Heap) and Topic 4 (Execution Engine — Garbage Collector), Module 06 Topic 5 (Object Creation Order)

## Motivation

This topic closes a loop that's been open since Module 02: you've known "the Garbage Collector reclaims unreachable objects" as a fact, but not precisely what "unreachable" means, or how an object's life actually ends. This is also where an important, historically significant deprecated feature (`finalize()`) gets its full, honest explanation — including *why* it failed and what real, modern Java code uses instead.

## The Object Lifecycle, End to End

```
1. CREATION                  2. IN USE                 3. UNREACHABLE              4. RECLAIMED

┌────────────────────────┐   ┌────────────────────────┐   ┌────────────────────────┐   ┌────────────────────────┐
│ new Foo()              │──▶│ Reachable via          │──▶│ No references          │──▶│ Memory reclaimed       │
│                        │   │ a reference chain      │   │ point to the           │   │ by the Garbage         │
│ (Module 06,            │   │                        │   │ object anymore         │   │ Collector              │
│  Topic 5's full        │   │                        │   │                        │   │                        │
│  initialization runs)  │   │                        │   │                        │   │ (Module 02, Topic 4)   │
└────────────────────────┘   └────────────────────────┘   └────────────────────────┘   └────────────────────────┘
```

1. **Creation**: `new` allocates memory on the Heap and runs the full, precise initialization order from Module 06, Topic 5.
2. **In use**: the object is **reachable** — some chain of references leads to it from an active part of the program.
3. **Unreachable**: the last reference to the object goes away (reassigned, went out of scope, or its containing object itself became unreachable).
4. **Reclaimed**: at some **later, unpredictable** time, the Garbage Collector identifies the object as unreachable and reclaims its memory.

## What "Reachability" Actually Means — GC Roots

An object is **reachable** if it can be reached by following a chain of references starting from a **GC Root** — a set of starting points the JVM always treats as inherently "alive":

```
GC ROOTS (always considered alive/reachable):
  - Local variables and parameters on any thread's current JVM Stack (Module 02, Topic 3)
  - Active static fields (Method Area/Metaspace, Module 02, Topic 3)
  - JNI references (Module 02, Topic 5)

               GC Root (a local variable on some thread's Stack)
                    │
                    ▼
                 Object A  ──▶  Object B  ──▶  Object C
                                                    │
                                                    ▼
                                                 Object D

   A, B, C, D are all REACHABLE (a chain exists from a GC Root all the way to each of them)
```

```java
void demo() {
    Point p = new Point(3, 4);   // p is a GC Root (local variable) -- the Point object is reachable
    // ... use p ...
}   // when demo() returns, 'p' goes out of scope (its Stack frame is popped -- Module 02, Topic 3) --
     // the Point object is now UNREACHABLE (assuming nothing else references it)
```

**Reachability, not "has anyone called delete/free," is the entire basis of Java's automatic memory management** — this is precisely the mechanism behind Module 01's "Robust" feature (no manual memory management, no dangling pointers) and directly explains why a **memory leak** in Java (Module 02, Topic 3) means "unintentionally keeping a reference alive," not "forgetting to free something" — there's no "free" to forget in the first place.

## When Does GC Actually Run?

**This is deliberately, specification-wise, left unpredictable.** The JVM specification does **not** guarantee exactly when garbage collection runs, or that an unreachable object is reclaimed at any particular, predictable moment — only that it eventually will be, if memory pressure requires it. `System.gc()` exists as a **hint/suggestion** to the JVM that garbage collection might be a good idea right now — it is explicitly **not a guarantee or a command**, and relying on it for correctness (rather than just as a debugging/testing aid) is considered poor practice.

**Why leave this unpredictable, deliberately?** This gives GC algorithm implementations (Module 16 covers several: Serial, Parallel, G1, ZGC, Shenandoah) maximum freedom to optimize *when* and *how* they reclaim memory — batching work efficiently, minimizing pause times, adapting to actual runtime memory pressure — rather than being forced into a rigid, specification-mandated timing that might not suit every workload or every GC algorithm's internal strategy.

## `finalize()` — Deprecated, and Why

`Object` historically provided a `finalize()` method, intended to run **once**, sometime before the GC actually reclaimed an object's memory — conceptually, "a last chance to clean up":

```java
class ResourceHolder {
    @Override
    protected void finalize() throws Throwable {
        // historically: "clean up resources here, right before GC reclaims this object"
        super.finalize();
    }
}
```

**`finalize()` was formally deprecated in Java 9, and its use is strongly discouraged (removal-track in later JDK evolution).** The reasons are numerous and genuinely instructive about broader Java design philosophy:

- **No guaranteed timing** — since GC timing itself is unpredictable (as established above), `finalize()` might run seconds, minutes, or (in some circumstances) **never** before the JVM exits — utterly unsuitable for anything time-sensitive, like promptly closing a file handle or database connection.
- **No guaranteed execution at all** — the JVM makes no absolute promise every unreachable object's `finalize()` will run before shutdown.
- **Real performance cost** — objects with a `finalize()` method require extra GC bookkeeping (they can't be reclaimed in a single, simple pass; the GC must first queue them for finalization, then re-check reachability afterward), slowing collection down measurably.
- **Can "resurrect" objects** — a poorly-written `finalize()` could theoretically make an unreachable object reachable again (by storing `this` into a live reference), creating deeply confusing lifecycle semantics.
- **Exceptions thrown inside `finalize()` are silently swallowed** — a serious anti-pattern for reliable error handling, essentially hiding real failures.

## The Modern Replacement: `AutoCloseable` and Try-With-Resources (Preview)

Since GC-timing-dependent cleanup (`finalize()`) is fundamentally unreliable for anything that needs **prompt, deterministic** cleanup (closing files, releasing database connections, releasing locks), modern Java's answer is **not** to fix `finalize()` — it's to **not depend on GC timing for cleanup at all**. Instead:

```java
try (FileReader reader = new FileReader("data.txt")) {   // try-WITH-RESOURCES
    // ... use reader ...
}   // reader.close() is called AUTOMATICALLY here, DETERMINISTICALLY,
     // the INSTANT this block ends -- NOT whenever GC eventually happens to run
```

Any class implementing the `AutoCloseable` interface (a single `close()` method) can be used in a **try-with-resources** statement, guaranteeing its `close()` method runs **deterministically**, at a precise, predictable point in the code — the moment the `try` block ends, regardless of whether it ended normally or via an exception. **This is a genuinely important design lesson**: rather than trying to make an inherently unpredictable mechanism (GC timing) work for a use case that needs predictability (resource cleanup), Java's designers introduced a **completely separate, deterministic mechanism** for that specific need. **Full depth on `AutoCloseable` and try-with-resources is in Module 12 (Exceptions) and Module 13 (IO)** — this is intentionally just a preview, enough to recognize the pattern and understand why it exists in relation to `finalize()`'s failure.

## A Brief Preview: Weak References

For advanced scenarios (like implementing caches that should let entries be garbage-collected under memory pressure), Java's `java.lang.ref` package provides `WeakReference` and related types — references that **do not, by themselves, keep an object reachable/alive**. An object referenced only by `WeakReference`s can still be garbage collected as if no reference existed at all. This is a genuinely advanced, specialized tool (used internally by things like `WeakHashMap` and some caching frameworks) — flagged here for completeness, with full treatment appropriately deferred to Module 16 (JVM Internals) once you have the GC algorithm depth to fully appreciate it.

## Real-World Analogy

Think of reachability like a **family tree search starting from a known living person** (a GC Root) — anyone connected to them through a chain of relationships is considered "part of the family" (reachable). Someone with **zero connecting path** back to any known living person (unreachable) is, for the purposes of this specific family record, effectively lost track of — and eventually the record-keeper (the GC) cleans their entry out of the archive, at some point they choose, not necessarily the instant the last connection was severed. `finalize()`'s failure is like relying on **that same record-keeper to also personally hand-deliver a specific, time-sensitive letter** exactly when a connection is severed — but the record-keeper only promises to *eventually* clean up the archive, on their own schedule, with no commitment to delivering anything on time (or even at all, if the archive itself closes first) — a fundamentally wrong tool for a job that actually needs a firm, predictable deadline.

## Advantages of Java's Reachability-Based GC Model

- Eliminates manual memory management entirely — no `free()`, no dangling pointers, no double-frees (Module 01's "Robust" feature, made fully concrete).
- GC algorithm implementations have maximum freedom to optimize timing/strategy, since the spec deliberately doesn't mandate exact timing.
- Try-with-resources provides a clean, deterministic, purpose-built solution for the specific cases that genuinely need prompt cleanup — without compromising GC's own flexibility.

## Disadvantages / Trade-offs

- Unpredictable GC timing means Java is generally unsuitable, without careful tuning (Module 22), for extremely hard real-time systems requiring guaranteed, bounded pause times.
- `finalize()`'s historical existence means you'll still encounter it in legacy code — understanding why it's deprecated is necessary for correctly evaluating and modernizing such code.

## Best Practices

- Never rely on `finalize()` for resource cleanup in new code — it's deprecated for good, well-demonstrated reasons.
- Always use try-with-resources (`AutoCloseable`) for anything requiring deterministic cleanup (files, connections, locks) — full syntax and depth: Modules 12–13.
- Never call `System.gc()` expecting deterministic behavior — treat it purely as an optional hint, never a guarantee, and avoid relying on it in production code.

## Common Mistakes

- Assuming an object is immediately reclaimed the instant its last reference disappears — reclamation timing is unpredictable and generally happens later, not immediately.
- Relying on `finalize()` for anything time-sensitive, given no guarantee of prompt (or even eventual) execution.
- Confusing "unreachable" with "deleted" — an object becoming unreachable just makes it *eligible* for collection; the actual reclamation is a separate, later event.

## Interview Questions

1. **Q: What does it mean for an object to be "reachable," and what are GC Roots?**
   A: An object is reachable if there's a chain of references leading to it starting from a GC Root — active local variables/parameters on any thread's Stack, active static fields, or JNI references. Objects unreachable from every GC Root are eligible for garbage collection.

2. **Q: Why is `finalize()` deprecated?**
   A: It offers no guaranteed timing (might run long after an object becomes unreachable, or never before JVM shutdown), imposes real GC performance overhead, can "resurrect" unreachable objects into reachability, and silently swallows exceptions — making it fundamentally unsuitable for reliable, prompt resource cleanup.

3. **Q: What replaced `finalize()` as the recommended way to clean up resources like files or database connections?**
   A: `AutoCloseable` combined with try-with-resources — a deterministic mechanism guaranteeing `close()` runs at a precise, predictable point (the end of the `try` block), rather than relying on unpredictable GC timing.

4. **Q: Does calling `System.gc()` guarantee garbage collection runs immediately?**
   A: No — it's only a hint/suggestion to the JVM; the specification makes no guarantee it will run garbage collection at all, or at any particular time, in response to this call.

## Summary

- An object's lifecycle: creation (Module 06's full init order) → reachable/in-use → unreachable (no path from any GC Root) → eventually reclaimed by the GC, at an unpredictable time.
- **Reachability**, traced from **GC Roots** (Stack locals/parameters, static fields, JNI references), is the entire basis of Java's automatic memory management.
- `finalize()` is deprecated due to unreliable timing, performance cost, resurrection risk, and swallowed exceptions.
- **`AutoCloseable`/try-with-resources** is the modern, deterministic replacement for anything needing prompt, reliable cleanup — full depth in Modules 12–13.

## Exercises

1. Draw a small reference graph (GC Root → several objects, with one object having no path back to any GC Root) and identify which objects are reachable vs. eligible for garbage collection.
2. Explain why `System.gc()` should never be relied upon for correctness in production code, referencing what it actually guarantees (or doesn't).
3. Explain, in your own words, why `finalize()`'s unreliable timing makes it fundamentally unsuitable for closing a file handle promptly — and what modern Java feature should be used instead.

---

**Previous:** [04 — Object Cloning](04-Object-Cloning.md) · **Next:** [06 — Module Summary, Interview Questions & Exercises](06-Module-Summary-Exercises.md)
