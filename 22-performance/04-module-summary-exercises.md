# Module 22 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-module-overview.md) has been covered:

- [x] **JVM Tuning: Heap & GC Selection** — `-Xms`/`-Xmx`, GC selection flags directly applying Module 16, Topic 2's algorithm comparison to real deployment decisions, Metaspace and Stack size tuning
- [x] **Profiling & Benchmarking** — precisely why naive `System.nanoTime()` benchmarking is invalidated by Module 02, Topic 4's JIT warm-up behavior, JMH as the correct tool, and JFR for profiling real applications
- [x] **Common Performance Pitfalls & Optimization** — a consolidated, actionable checklist of pitfalls already explained mechanistically earlier in this course, plus escape analysis as a genuine new JIT optimization, and a final, practical GraalVM Native Image judgment

## Practical Connections

- **Every production incident involving "the application is slow" or "we're getting OOM errors"** is directly addressed by this module's tools — JFR profiling (Topic 2) to find the actual cause, and Topic 1's heap/GC tuning to address it once identified.
- **Cloud cost optimization** (right-sizing container memory limits, choosing between always-on services and serverless functions) directly applies this module's heap sizing and GraalVM Native Image guidance.
- **Code review culture** at any serious Java shop will flag Topic 3's pitfalls (string concatenation in loops, un-sized collections) as a matter of course — you're now equipped to both flag and explain precisely why.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| `-Xms` vs `-Xmx` | `-Xms`: initial heap size. `-Xmx`: maximum heap size — equal values avoid dynamic resizing overhead. |
| Benchmarking vs profiling | Benchmarking (JMH): measures a small, isolated piece of code precisely. Profiling (JFR): observes an entire, real, running application's actual behavior. |
| Escape analysis vs. "objects live on the Heap" | Escape analysis is a specific, provable-circumstance JIT optimization refining physical storage; the Heap-allocation model (Module 02, Topic 3) remains the correct general mental model. |
| Standard JVM vs GraalVM Native Image | Standard: adaptive, runtime-optimized, slower startup. Native Image: near-instant startup, no runtime adaptation — a deliberate trade-off, not a strict upgrade (Module 02, Topic 6). |

## Consolidated Interview Questions (Module 22)

1. What's the practical difference between setting `-Xms` equal to `-Xmx` versus leaving them different?
2. How would you choose between G1, ZGC, and Parallel GC for a given application?
3. Why is a naive `System.nanoTime()`-based loop benchmark usually misleading?
4. What does JMH do differently from a naive benchmark?
5. What is Java Flight Recorder (JFR), and how does it differ from a microbenchmark?
6. Name three common Java performance pitfalls and their fixes.
7. What is escape analysis, and does it contradict "objects always live on the Heap"?
8. When would you choose GraalVM Native Image over a standard JVM deployment?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** From memory, explain why a naive `System.nanoTime()` benchmark is invalid, referencing Module 02, Topic 4's JIT warm-up model precisely.
2. **Hands-on:** If you have a JDK installed, run a small program with `-XX:+PrintCompilation` (Module 02, Topic 4) alongside a naive timing loop, and observe the JIT compilation events happening mid-loop.
3. **Hands-on:** Rewrite a loop-based String-concatenation method using `StringBuilder`, and explain the expected performance difference.
4. **Conceptual:** For each scenario, choose an appropriate GC and justify it: a nightly batch job, a real-time bidding service, a small CLI tool.
5. **Conceptual:** Explain escape analysis in your own words, and why it doesn't contradict this course's foundational Stack/Heap model from Module 02.
6. **Synthesis:** Design a performance investigation plan for a "the API is slow" production report: what would you check first (Topic 2's profiling), what tuning options you'd consider (Topic 1), and what code-level pitfalls (Topic 3) you'd look for — in what order, and why.

## What's Next

Module 22 completed the practical, applied culmination of this course's deep JVM knowledge — you can now not just explain how Java works, but actively measure and tune it. **Module 23 — Modern Java** now turns to the language's most recent evolution: Records, Sealed Classes, Pattern Matching, Virtual Threads (revisited in full context), and every other feature through Java 25 — comparing each against the "old way" you've learned throughout this course, completing your journey from first principles to the cutting edge of the language.

---

**Previous:** [03 — Common Performance Pitfalls & Optimization](03-common-performance-pitfalls-and-optimization.md) · **Module Overview:** [00 — Module Overview](00-module-overview.md)

**Type "Continue" to begin Module 23 — Modern Java.**
