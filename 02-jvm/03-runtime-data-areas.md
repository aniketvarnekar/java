# Runtime Data Areas (JVM Memory Structure)

## Learning Objectives

- Name every JVM runtime data area and what specifically lives in each
- Distinguish shared (per-JVM) areas from per-thread areas, and explain why that split exists
- Fully understand Stack vs Heap — the single most interview-tested memory concept in Java
- Trace exactly how a method call and object creation affect memory, step by step

## Prerequisites

[01 — JVM Architecture Overview](01-jvm-architecture-overview.md), [02 — Class Loader Subsystem](02-class-loader-subsystem.md)

## Motivation

Nearly every "why did my program crash" question in real Java development traces back to this topic: `StackOverflowError`, `OutOfMemoryError: Java heap space`, `OutOfMemoryError: Metaspace`, memory leaks despite "having a garbage collector," and why passing an object to a method lets you mutate it but reassigning it inside the method doesn't affect the caller — all of it is explained by precisely understanding where things live in memory and how they're referenced.

## Problem Statement

A running program needs to store many *kinds* of things, with very different lifetimes and sharing requirements:
- Class definitions (loaded once, needed by every thread, for the entire program's life)
- Objects you create with `new` (created and destroyed unpredictably, potentially shared between threads)
- A method's local variables and the fact that it was called from another method (created when a method starts, destroyed the instant it returns, and — critically — completely private to the one thread executing it)

Using one undifferentiated memory pool for all of this would be both slow (no simple bulk-cleanup strategy) and dangerous (one thread's local variables could accidentally leak into another thread's view). The JVM Specification solves this with distinct **Runtime Data Areas**.

## The Full Picture

```
┌────────────────────────────────────────────────────────────────────────────────────┐
│                             SHARED ACROSS ALL THREADS                              │
│                         (Created Once During JVM Startup)                          │
│                                                                                    │
│  ┌──────────────────────────────────┐    ┌──────────────────────────────────────┐  │
│  │ Method Area (Metaspace)          │    │ Heap                                 │  │
│  │                                  │    │                                      │  │
│  │ • Class metadata                 │    │ • All objects created with `new`     │  │
│  │ • Method bytecode                │    │   are allocated here                 │  │
│  │ • Field information              │    │ • Instance fields are stored as      │  │
│  │ • Runtime constant pool          │    │   part of their objects              │  │
│  │ • Static variables               │    │ • Shared by all threads              │  │
│  │                                  │    │ • Managed by the Garbage Collector   │  │
│  └──────────────────────────────────┘    └──────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────────────────┘


┌────────────────────────────────────────────────────────────────────────────────────┐
│                              PER THREAD (One Set Per Thread)                       │
│               (Created When a Thread Starts, Destroyed When It Ends)               │
│                                                                                    │
│      Thread A                 Thread B                 Thread C                    │
│                                                                                    │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐              │
│  │ JVM Stack        │    │ JVM Stack        │    │ JVM Stack        │              │
│  │ • Stack Frames   │    │ • Stack Frames   │    │ • Stack Frames   │              │
│  ├──────────────────┤    ├──────────────────┤    ├──────────────────┤              │
│  │ PC Register      │    │ PC Register      │    │ PC Register      │              │
│  ├──────────────────┤    ├──────────────────┤    ├──────────────────┤              │
│  │ Native Method    │    │ Native Method    │    │ Native Method    │              │
│  │ Stack            │    │ Stack            │    │ Stack            │              │
│  └──────────────────┘    └──────────────────┘    └──────────────────┘              │
└────────────────────────────────────────────────────────────────────────────────────┘
```

## Each Area, In Detail

### Method Area (Metaspace)

**What lives here:** the class-level information every instance of a class shares — the class's structure, its method bytecode, `static` fields, and the **runtime constant pool** (a per-class table of literals like string constants and symbolic references, resolved during Linking — Topic 2).

**Shared or per-thread?** Shared — one copy per loaded class, for the whole JVM, no matter how many objects or threads use it.

**Historical name change — a genuinely important interview fact:** before Java 8, this area was called **"PermGen" (Permanent Generation)** and was a fixed-size region carved out of the heap, notorious for a very common production error: **`OutOfMemoryError: PermGen space`**, especially in application servers that reloaded web apps repeatedly (each reload could leave old class metadata that couldn't be reclaimed, slowly filling PermGen). **Since Java 8**, PermGen was **removed entirely** and replaced with **Metaspace**, which lives in **native (off-heap) memory**, not the Java heap, and **grows dynamically by default** (bounded only by available native memory, or an explicit `-XX:MaxMetaspaceSize` if you set one). This directly solved the class-metadata-leak problem that plagued PermGen. You'll still commonly hear "Method Area," "PermGen," and "Metaspace" used near-interchangeably in casual conversation — know that PermGen is the *pre-Java-8* implementation and Metaspace is the *modern* one.

### Heap

**What lives here:** every single object you create with `new` (or via other object-creation mechanisms like reflection, deserialization, autoboxing, etc.) — including all of that object's **instance fields**. Arrays (Module 09) also live on the Heap, since arrays are objects in Java.

**Shared or per-thread?** Shared — a single Heap per JVM process, accessible (and potentially mutable) by every thread, which is exactly why concurrent access to shared objects needs coordination (synchronization — Module 15).

**Reclaimed by:** the Garbage Collector, which periodically identifies objects no longer *reachable* from any active reference (Topic 4 of this module gives the Execution Engine's role; full GC algorithm depth is Module 16).

**The famous error:** `OutOfMemoryError: Java heap space` — you've created (and are still holding live references to) more objects than the configured heap can hold. This is the classic symptom of a **memory leak** in a garbage-collected language: not "forgetting to free memory" (impossible to even attempt in Java) but **unintentionally holding a reference to objects you no longer actually need**, which prevents the GC from ever reclaiming them (deep dive in Module 16).

### JVM Stack (per thread)

**What lives here:** a stack of **frames**, one frame per currently-active (not-yet-returned) method call on that thread. Each frame holds:
- **Local variables** for that method call (including its parameters)
- The **operand stack** used for intermediate calculation results within that method (recall the `iconst_1 / iconst_2 / iadd` bytecode example from Module 01, Topic 5 — that operand stack lives here)
- A reference back to the calling frame, so execution can resume correctly after this method returns

**Shared or per-thread?** **Strictly per-thread** — this is critical. Thread A's local variables are never visible to Thread B, because they live in entirely separate JVM Stacks. This is a foundational reason why local variables never need synchronization in concurrent code (Module 15), while shared Heap objects often do.

```
JVM Stack for One Thread
Mid-execution of: main() → calculateTotal() → add()

                Top of Stack
                     │
                     ▼
┌──────────────────────────────────────────┐
│ Frame: add(int a, int b)                 │
│ ──────────────────────────────────────── │
│ Local Variables:                         │
│   a = 3                                  │
│   b = 4                                  │
│ Operand Stack:                           │
│   (currently executing)                  │
└──────────────────────────────────────────┘
┌──────────────────────────────────────────┐
│ Frame: calculateTotal()                  │
│ ──────────────────────────────────────── │
│ Local Variables:                         │
│   x = 3                                  │
│   y = 4                                  │
│   result = <pending>                     │
└──────────────────────────────────────────┘
┌──────────────────────────────────────────┐
│ Frame: main(String[] args)               │
│ ──────────────────────────────────────── │
│ Local Variables:                         │
│   args                                   │
└──────────────────────────────────────────┘
                     ▲
                     │
               Bottom of Stack
```

When `add()` returns, its frame is **popped off** the stack entirely — its local variables cease to exist immediately. This is exactly why a method's local variables are never accessible after the method returns.

**The famous error:** `StackOverflowError` — pushing more frames than the configured stack size allows, almost always from **runaway or excessively deep recursion** (a recursive method that never reaches its base case, or legitimately needs more recursion depth than the default stack size supports).

### PC (Program Counter) Register (per thread)

**What lives here:** a small piece of data recording **which bytecode instruction this thread is currently executing** within its current method's frame. When a thread is paused and later resumed (e.g., after being time-sliced by the OS scheduler), the PC Register is exactly how the JVM knows precisely where to resume execution.

**Shared or per-thread?** Per-thread — each thread needs to track its own independent execution position.

### Native Method Stack (per thread)

**What lives here:** essentially the JVM Stack's counterpart for **native (non-Java) method calls** — when Java code calls into native C/C++ code via JNI (Topic 5), that native call gets its own stack frame area, separate from the regular JVM Stack used for pure-Java method calls.

## Stack vs Heap — The Full Comparison

This is asked in nearly every Java interview, at every level. Know it cold.

| Aspect | Stack | Heap |
|---|---|---|
| Stores | Local variables, method parameters, operand stack, call frames | Objects (and their instance fields), arrays |
| Scope | Per-thread — private to the thread that owns it | Shared — accessible by all threads |
| Lifetime | Tied to the method call — created when the method is entered, destroyed the instant it returns | Tied to object reachability — lives until the Garbage Collector determines it's unreachable |
| Allocation/deallocation speed | Very fast (simple pointer bump up/down, LIFO order) | Slower, more complex (must find free space, track reachability, run GC algorithms) |
| Size | Small, fixed per thread (configurable via `-Xss`) | Large, shared, configurable (`-Xms` initial, `-Xmx` maximum) |
| Failure mode | `StackOverflowError` (too many nested frames, e.g. runaway recursion) | `OutOfMemoryError: Java heap space` (too many live, reachable objects) |
| Thread safety | Inherently thread-safe (no other thread can ever see it) | Requires explicit synchronization for safe concurrent mutation (Module 15) |
| What a variable actually holds | For primitives: the value itself, directly. For objects: a **reference** (essentially a pointer) to the real object data, which lives on the Heap | The actual object data itself |

### The Critical Nuance: Where Does an Object *Variable* Live vs. the Object *Itself*?

This is the single most common source of confusion, so let's be extremely precise:

```java
void demo() {
    int x = 42;                 // 'x' AND its value 42 both live on the STACK (primitive)
    Car myCar = new Car("Red"); // 'myCar' (the REFERENCE/pointer) lives on the STACK
                                 // the actual Car OBJECT (with field color="Red") lives on the HEAP
}
```

```
                    STACK (Current Thread)                           HEAP (Shared by All Threads)

┌──────────────────────────────────────────┐          ┌──────────────────────────────────────┐
│ Frame: demo()                            │          │ Car Object                           │
│ ──────────────────────────────────────── │          │ ──────────────────────────────────── │
│ Local Variables:                         │          │ color = "Red"                        │
│                                          │          │                                      │
│ x      = 42                              │          │ (Object stored in heap memory)       │
│                                          │          │                                      │
│ myCar  = 0xF3A2 ─────────────────────────┼─────────▶│ Address: 0xF3A2                      │
│         (Reference to the object)        │          │                                      │
└──────────────────────────────────────────┘          └──────────────────────────────────────┘
```

**Primitive variables** (`int`, `double`, `boolean`, etc.) hold their actual value directly on the Stack. **Object-typed variables** never hold the object itself on the Stack — they hold a **reference** (an address pointing into the Heap) on the Stack, while the real object data lives on the Heap. This single fact explains a huge amount of Java's pass-by-value behavior (fully covered with worked examples in Module 06/07), because **Java is always pass-by-value — but for object types, the "value" being passed is the reference itself**, not the object.

## Real-World Analogy

Think of the **Heap like a big shared warehouse** — anyone (any thread) with the right address (reference) can go find and use an item stored there. Think of the **Stack like a personal sticky-note pad on each worker's own desk** — each worker (thread) has their own pad, no one else can read or write on it, and when they finish a task (a method returns), they tear off and throw away that page immediately. A sticky note might have a warehouse aisle-and-shelf number written on it (a reference/pointer) — the note itself is private and disposable, but it can point to something in the shared warehouse that outlives the note.

## Advantages of This Split

- Stack allocation/deallocation is extremely fast (no GC involvement at all) — this is precisely why Java strongly prefers primitives and short-lived local variables on the Stack wherever possible for performance.
- Thread-private Stacks eliminate an entire category of concurrency bugs for local variables, with zero synchronization cost.
- Precise, distinct error types (`StackOverflowError` vs `OutOfMemoryError`) make diagnosing memory problems far more tractable than a single undifferentiated "out of memory" failure would.

## Disadvantages / Trade-offs

- The Stack's small, fixed size means deep recursion is a real, hard limit in Java — some recursive algorithms genuinely need to be rewritten iteratively for large inputs (practical concern revisited in later modules).
- Heap allocation and GC introduce real runtime overhead compared to purely stack-based (or manually-managed) memory in languages like C/C++ or Rust — the trade-off Java makes deliberately for safety and productivity (as discussed in Module 01, Topic 1).

## Best Practices

- When you see `StackOverflowError`, immediately suspect uncontrolled/infinite recursion — check the base case of your recursive method first.
- When you see `OutOfMemoryError: Java heap space`, suspect objects being unintentionally retained (held by a reference somewhere they shouldn't be) rather than "the heap being too small" as your first hypothesis — Module 16 covers real memory-leak patterns in depth.
- Understand that reassigning an object parameter inside a method never affects the caller's reference — because the reference itself was passed *by value* (copied) onto the callee's Stack frame; more on this with full worked examples in Module 06/07.

## Common Mistakes

- Believing "objects live on the Stack" — objects **always** live on the Heap; only primitive values and references (pointers to Heap objects) live on the Stack.
- Believing Metaspace is "part of the Heap" — since Java 8, Metaspace is explicitly **native (off-heap)** memory, a deliberate change from the old PermGen design.
- Assuming all threads share one Stack — each thread has its own **entirely separate** JVM Stack; this is what makes local variables thread-safe by default.

## Performance Considerations

- Stack allocation is essentially free (a pointer bump); Heap allocation is comparatively expensive and adds GC pressure — this is one motivation behind features like **Records** (Module 23) and JVM-level "escape analysis" optimizations, where the JIT compiler (Topic 4) can sometimes determine an object never "escapes" a method and allocate it on the Stack instead of the Heap, entirely transparently to you.
- Very deep call chains (recursion, or simply very long call stacks) increase memory pressure on the Stack; the default Stack size can be increased with the `-Xss` JVM flag if a legitimate use case needs deeper recursion than the default allows.

## Interview Questions

1. **Q: What's the difference between Stack and Heap in Java?**
   A: The Stack (one per thread) holds local variables, method parameters, and call frames — fast, private per-thread, LIFO, destroyed the instant a method returns. The Heap (one shared per JVM) holds all actual object data created with `new` — slower to allocate, shared across all threads, and reclaimed only by the Garbage Collector when objects become unreachable.

2. **Q: If I pass an object to a method and reassign the parameter inside that method, does the caller's original reference change?**
   A: No. Java is always pass-by-value; for object types, the *value* being copied is the reference (the Heap address) itself. Reassigning the local parameter variable inside the method only changes that method's own Stack-local copy of the reference — the caller's original variable, on the caller's own Stack frame, still points at the original object.

3. **Q: What replaced PermGen, and why?**
   A: **Metaspace**, since Java 8. PermGen was a fixed-size region carved out of the Heap for class metadata, and was a notorious source of `OutOfMemoryError: PermGen space`, especially under repeated class loading/unloading (e.g., app server redeploys). Metaspace moved this data to native (off-heap) memory and grows dynamically by default, directly addressing that failure mode.

4. **Q: What causes a `StackOverflowError`, and what's the most common real-world cause?**
   A: Pushing more frames onto a thread's JVM Stack than its configured size allows — overwhelmingly, in practice, this is caused by a recursive method that never reaches its base case (or needs a legitimately larger recursion depth than the default stack size supports).

## Summary

- The JVM's memory is split into **shared** areas (Method Area/Metaspace, Heap) and **per-thread** areas (JVM Stack, PC Register, Native Method Stack).
- The **Method Area/Metaspace** holds class metadata and static fields; since Java 8, it's native (off-heap) memory, replacing the old, leak-prone PermGen.
- The **Heap** holds every object ever created with `new`; it's reclaimed only by the Garbage Collector.
- The **JVM Stack** holds per-method local variables, parameters, and the operand stack, in per-thread call frames — extremely fast, but limited in size (`StackOverflowError` on overflow).
- **Object variables hold references (pointers) on the Stack; the actual object data always lives on the Heap** — this single fact underlies Java's pass-by-value semantics for object references.

## Exercises

1. Draw the Stack-vs-Heap picture for this code (label every Stack frame and every Heap object, with references drawn as arrows):
   ```java
   class Point { int x, y; }
   void demo() {
       int a = 10;
       Point p1 = new Point();
       Point p2 = p1;
   }
   ```
   (Hint: how many `Point` objects exist on the Heap after this code runs — one or two?)
2. Without looking back, explain why local variables never require synchronization for thread safety, referencing the specific Runtime Data Area responsible.
3. Explain the practical difference between PermGen and Metaspace, and why the change was made.
4. A recursive method to compute a Fibonacci number crashes with `StackOverflowError` for large inputs. Explain, in terms of the JVM Stack, exactly why this happens, and suggest (conceptually, not necessarily in code yet) one way to avoid it.

---

**Previous:** [02 — Class Loader Subsystem](02-class-loader-subsystem.md) · **Next:** [04 — Execution Engine](04-execution-engine.md)
