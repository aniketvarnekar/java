# Core Conceptual Questions — Ranked & Synthesized

## Learning Objectives

- Recall, from memory and in your own words, the highest-signal Java conceptual questions across the entire course
- Answer each in interview-appropriate form: a tight, complete answer in 20-45 seconds, not a five-minute lecture
- Know exactly where to go for full depth if an interviewer pushes further

## Prerequisites

Modules 01–23, in full — this topic is pure retrieval practice over material you've already learned.

## Motivation

`Interview-Questions.md` at the course root already gives you every Q&A this course has produced, in module order — comprehensive, but not **prioritized**. In a real interview, some questions come up constantly across companies and interview styles (JVM memory model, `equals`/`hashCode`, `HashMap` internals, checked-vs-unchecked exceptions), while others are rarer, deeper cuts. This topic re-organizes the course's highest-signal questions **by theme**, roughly in the order interviewers actually reach for them, so your review time goes to what matters most first.

**How to use this list:** cover the answer, say the question out loud, answer it out loud in your own words in under a minute, *then* check the model answer. Reading the answer first feels productive but builds almost no real interview recall.

## Theme: JVM, Memory & Execution

**Q1. What's the difference between JDK, JRE, and JVM?**
> JVM: the abstract execution engine that runs bytecode. JRE: JVM + the standard class libraries needed to *run* Java programs. JDK: JRE + development tools (`javac`, debugger, etc.) needed to *build* Java programs. (Module 01)

**Q2. What's the difference between Stack and Heap memory?**
> Stack: per-thread, stores method call frames and local primitive/reference variables, LIFO, automatically reclaimed on method return. Heap: shared across all threads, stores all objects, reclaimed only by the garbage collector. (Module 02, Topic 3)

**Q3. Explain the generational garbage collection hypothesis.**
> Most objects die young. The Heap is split into Young Generation (Eden + Survivor spaces, collected frequently and cheaply via Minor GC) and Old Generation (collected rarely, via a more expensive Major/Full GC), because scanning only the small, short-lived Young Generation most of the time is far cheaper than repeatedly scanning the whole Heap. (Module 16, Topic 2)

**Q4. What is the Java Memory Model (JMM), and why does it matter for concurrency?**
> A specification defining the rules for how and when changes made by one thread become visible to other threads, in the presence of compiler/CPU reordering and per-core caching — without it, multi-threaded code has no guaranteed visibility or ordering at all. `volatile`, `synchronized`, and `final` are the JMM's visibility/ordering tools. (Module 15, Topic 2)

**Q5. What's the difference between `==` and `.equals()`?**
> `==` compares references (are these the exact same object in memory) for objects, or values for primitives. `.equals()` compares logical/content equality, as defined by the class's override — `Object`'s default `.equals()` is `==` unless overridden. (Module 07, Topic 3)

## Theme: OOP & Object Behavior

**Q6. Explain the `equals()`/`hashCode()` contract, and why violating it is dangerous.**
> If `a.equals(b)` is true, `a.hashCode()` must equal `b.hashCode()`. Violating it breaks hash-based collections silently: a `HashSet`/`HashMap` may fail to find an object that is logically `.equals()` to a stored key, because it hashes to the wrong bucket first. (Module 07, Topic 3)

**Q7. What's the difference between overloading and overriding?**
> Overloading: same method name, different parameter list, resolved at **compile time** based on static argument types. Overriding: subclass redefines a superclass method with an identical signature, resolved at **runtime** via dynamic dispatch based on the actual object's type. (Module 05, Topic 5)

**Q8. What's the difference between an abstract class and an interface?**
> An abstract class can hold state (instance fields) and a constructor, and models an "is-a" relationship with single inheritance. An interface (pre-Java 8) could hold no implementation at all, and models a "can-do" capability with multiple inheritance of type. Since Java 8, interfaces can have default/static methods, narrowing but not eliminating the gap. (Module 05, Topic 6)

**Q9. Why is Java "pass-by-value" even for objects, and why does this confuse people?**
> Java always copies the value being passed. For a reference type, the *value being copied is the reference itself* (the address) — so the copy points to the same object, letting you mutate that object's state through it, but reassigning the parameter itself never affects the caller's original reference. This looks like pass-by-reference because mutation is visible, but reassignment proves it isn't. (Module 06, Topic 2 / Module 02, Topic 3)

**Q10. Why is `String` immutable, and what are the real benefits?**
> Once created, a `String`'s character data can never change. Benefits: safe sharing via the String Pool (Module 08) without defensive copying, inherent thread safety with no synchronization needed, and safety as a `HashMap` key (its hash code can be cached once, safely). (Module 08, Topic 1)

## Theme: Collections

**Q11. How does `HashMap` work internally?**
> Keys are hashed via `hashCode()`, mapped to a bucket index (`hash & (capacity - 1)`), and stored there; collisions within a bucket form a linked list (or a red-black tree, since Java 8, once a bucket exceeds a threshold — Module 10). Lookup re-hashes the key, jumps to the bucket, then uses `equals()` to find the exact match.

**Q12. `ArrayList` vs `LinkedList` — when would you choose each?**
> `ArrayList`: contiguous array-backed storage, O(1) random access, O(n) insert/delete in the middle (shifting). `LinkedList`: doubly-linked nodes, O(1) insert/delete at a known position, O(n) random access. Default to `ArrayList` — it's cache-friendlier and covers the vast majority of real use cases; `LinkedList` wins only for genuinely frequent middle-insertion workloads. (Module 10)

**Q13. `HashMap` vs `TreeMap` vs `LinkedHashMap`?**
> `HashMap`: no ordering guarantee, O(1) average operations. `TreeMap`: sorted key order (Red-Black tree), O(log n) operations. `LinkedHashMap`: insertion (or access) order preserved, O(1) average operations with a small extra linked-list overhead. Choose based on whether you need ordering at all, and if so, which kind. (Module 10)

**Q14. Why is `ConcurrentHashMap` preferred over a `synchronized HashMap` for concurrent access?**
> A `synchronized` wrapper locks the *entire* map for every operation — one big lock, serializing all access. `ConcurrentHashMap` uses fine-grained internal locking/CAS, allowing many threads to operate on different parts of the map concurrently — far higher throughput under contention. (Module 15, Topic 7)

## Theme: Exceptions & Robustness

**Q15. Checked vs. unchecked exceptions — what's the actual design intent?**
> Checked exceptions (extend `Exception`, not `RuntimeException`) represent recoverable conditions the caller is *forced* by the compiler to handle or declare — used for expected failure modes like file-not-found. Unchecked exceptions (extend `RuntimeException`) represent programming errors that shouldn't normally be caught and handled locally, only logged/propagated. (Module 12, Topic 2)

**Q16. What does `try`-with-resources do, and why is it better than manual `finally` cleanup?**
> Automatically calls `.close()` on any `AutoCloseable` resource, in reverse declaration order, even if an exception occurs — and correctly handles the case where both the try-block *and* `.close()` throw, via suppressed exceptions, which manual `finally` blocks get subtly wrong by default (a later exception silently replacing an earlier, often more important one). (Module 12, Topic 5)

## Theme: Concurrency

**Q17. What's the difference between a `Thread` and a `Runnable`?**
> `Runnable` is a functional interface describing "a task to run"; `Thread` is the actual OS-backed (or since Java 21, potentially virtual) execution vehicle that runs it. Implementing `Runnable` (composition) is preferred over extending `Thread` (inheritance) because it doesn't burn Java's single-inheritance slot and cleanly separates "what to do" from "how it's executed." (Module 15, Topic 1)

**Q18. What is a race condition, and how do you prevent one?**
> When multiple threads access shared mutable state without proper synchronization, and the outcome depends on unpredictable timing/interleaving. Prevented via `synchronized`, explicit locks (`ReentrantLock`), atomic classes (`AtomicInteger`), or by avoiding shared mutable state entirely (immutability, thread confinement). (Module 15, Topic 2)

**Q19. What's the difference between a platform thread and a virtual thread?**
> A platform thread maps 1:1 to an OS thread — expensive (megabytes of stack, OS-scheduled), so thread pools cap concurrency far below real demand for I/O-bound work. A virtual thread is a lightweight, JVM-scheduled thread multiplexed M:N onto a small pool of platform "carrier" threads — cheap enough to create millions, solving the C10K-style scalability problem for blocking, I/O-bound code without rewriting it as reactive/async. (Module 15, Topic 8)

**Q20. What does `volatile` actually guarantee, and what does it *not* guarantee?**
> Guarantees visibility (a write is immediately visible to other threads, no stale caching) and ordering (via the JMM's happens-before rules) for that specific variable. Does **not** guarantee atomicity for compound operations like `count++` (read-modify-write) — that still needs `synchronized` or an atomic class. (Module 15, Topic 4)

## Theme: Modern Java

**Q21. What problem do Streams solve compared to classic loops?**
> Streams express **what** transformation to perform (declarative) rather than **how** to iterate (imperative) — often more concise and readable, and can be trivially parallelized (`.parallelStream()`) without rewriting the logic. The trade-off: harder to debug step-by-step than an equivalent loop, and not always faster for small collections due to overhead. (Module 18)

**Q22. What is type erasure, and what's its most common practical consequence?**
> Generic type information exists only at compile time; the compiler enforces it there via checks and inserted casts, then erases it, so at runtime a `List<String>` and a `List<Integer>` are both just `List`. Practical consequence: you cannot do `new T()`, `new T[]`, or `instanceof T` with a generic type parameter, and can't overload methods that would erase to the same signature. (Module 11, Topic 2)

**Q23. How do sealed classes and pattern-matching `switch` combine to guarantee exhaustiveness?**
> A `sealed` type declares its complete, closed set of permitted subtypes to the compiler. A `switch` over that type can then be checked at compile time for covering every permitted case — a missing case is a compile error, and adding a new subtype later breaks every relevant `switch` until updated, rather than leaving a silent runtime gap. (Module 23, Topics 2-3)

**Q24. What specifically does a Java `record` generate for you?**
> A canonical constructor, private final fields, accessor methods (`x()`, not `getX()`), and correct `equals()`/`hashCode()`/`toString()` implementations — all derived from the record header, always immutable, cannot extend another class. (Module 23, Topic 1)

## Best Practices for Using This List

- Time yourself: a strong 30-45 second answer beats a rambling 3-minute one — interviewers read excessive length as uncertainty, not thoroughness.
- Always be ready to go **one level deeper** than the model answer if asked "why" again — every question here links back to a full module topic for exactly that purpose.
- Practice explaining these **out loud**, not just recalling them silently — the muscle interviews actually test is verbal, real-time explanation, not passive recognition.

## Common Mistakes

- Reciting a memorized definition without connecting it to *why* it matters — interviewers consistently rate "explains the reasoning" answers higher than "recites the fact" answers, even when both are technically correct.
- Answering with excessive scope-creep — e.g., turning "what's the difference between `==` and `.equals()`" into an unprompted five-minute tour of the entire Object class. Answer what was asked; let the interviewer ask a follow-up if they want more.
- Freezing on a question you genuinely don't remember instead of reasoning toward it out loud — interviewers routinely value visible, structured reasoning under uncertainty over a memorized-but-silent struggle.

## Summary

- This topic re-organized the course's highest-signal conceptual questions **by cross-cutting theme** (JVM/Memory, OOP, Collections, Exceptions, Concurrency, Modern Java) rather than by module order, prioritizing what interviewers ask most.
- Every answer is deliberately interview-length (20-45 seconds spoken), with a pointer back to the full module topic for further depth.
- The full, unranked, module-ordered Q&A set remains available in the course root's `Interview-Questions.md` for exhaustive review.