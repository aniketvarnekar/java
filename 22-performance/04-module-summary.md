# Module 22 Summary

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

## What's Next

Module 22 completed the practical, applied culmination of this course's deep JVM knowledge — you can now not just explain how Java works, but actively measure and tune it. **Module 23 — Modern Java** now turns to the language's most recent evolution: Records, Sealed Classes, Pattern Matching, Virtual Threads (revisited in full context), and every other feature through Java 25 — comparing each against the "old way" you've learned throughout this course, completing your journey from first principles to the cutting edge of the language.