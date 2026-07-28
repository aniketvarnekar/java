# Module 16 Summary

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-module-overview.md) has been covered:

- [x] **Bytecode Deep Dive** — the `.class` file format, hands-on `javap` disassembly, the `invoke*` family as the concrete bytecode-level implementation of polymorphism (Module 05) and static method hiding (Module 06)
- [x] **Garbage Collection Algorithms** — the generational hypothesis, stop-the-world pauses, and the real trade-offs across Serial/Parallel/G1/ZGC/Shenandoah
- [x] **Java Memory Model Deep Dive** — concrete reordering examples, memory barriers as the mechanism behind `synchronized`/`volatile`, and the `final` field safe-publication guarantee
- [x] **Reflection** — inspecting/invoking code at runtime, and the complete, demystified explanation of how frameworks (Jackson, Spring) actually work
- [x] **Annotations & Dynamic Proxies** — custom annotations, retention/target, and Dynamic Proxies as the concrete mechanism behind Spring's AOP features
- [x] **Method Handles & Modern Internals** — `MethodHandle` as a faster Reflection alternative, and `invokedynamic` as the mechanism enabling efficient lambda expressions

## Practical Connections

- **Every Spring Boot application's dependency injection, `@Transactional`, and `@RequestMapping` routing** is now fully demystified — Reflection (Topic 4) + Annotations (Topic 5) + Dynamic Proxies (Topic 5), applied systematically.
- **Production JVM tuning** (choosing between G1/ZGC, sizing heap regions) directly applies Topic 2's algorithm knowledge — full practical, hands-on tuning guidance comes in Module 22.
- **Debugging genuinely subtle concurrency bugs** that survive Module 15's tools often requires Topic 3's reordering/memory-barrier understanding — this is where "it works on my machine but fails intermittently in production" concurrency bugs get their deepest explanation.
- **Understanding why lambdas (Module 17, next) are lightweight** is now grounded in Topic 6's `invokedynamic` mechanism, not just an assertion to trust.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| `invokestatic`/`invokespecial` vs `invokevirtual`/`invokeinterface` | The former are resolved at compile time (no dynamic dispatch); the latter implement runtime polymorphic dispatch (Module 05, Topic 5). |
| Stop-the-world vs. concurrent GC phases | STW pauses halt application threads; concurrent phases (ZGC/Shenandoah) run alongside the still-executing application. |
| `volatile`/`synchronized` vs. memory barriers | The former are the Java-level tools; memory barriers are the actual, low-level mechanism those tools are implemented with. |
| Classic Reflection vs `MethodHandle` | Both achieve dynamic invocation; `MethodHandle` is lower-level and generally faster/more JIT-friendly. |
| Annotation retention `SOURCE` vs `RUNTIME` | `SOURCE`: compiler-checked only, discarded after compilation (like `@Override`). `RUNTIME`: retained and discoverable via Reflection while the program runs (required for framework annotations). |

## What's Next

Module 16 completed the deepest, most mechanistic tier of this course's JVM coverage — you now understand not just what Java does, but precisely how, down to the bytecode and memory-model level. **Module 17 — Functional Programming** shifts to a completely different, higher-level lens: lambda expressions (now grounded in this module's `invokedynamic` mechanism), functional interfaces, method references, and the paradigm shift Java 8 introduced — one of the most significant changes in Java's entire history (Module 01, Topic 2's timeline flagged this repeatedly).