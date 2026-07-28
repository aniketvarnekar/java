# Module 16 Summary, Interview Questions & Exercises

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

## Consolidated Interview Questions (Module 16)

1. What does the `invokevirtual` bytecode instruction do, and why is it significant?
2. What is the generational hypothesis, and how does it shape GC design?
3. What is a stop-the-world pause?
4. Why can a reader thread see a stale value even after observing a "ready" flag, without synchronization?
5. What is a memory barrier, and how does it relate to `synchronized`/`volatile`?
6. What special JMM guarantee do properly-initialized `final` fields receive?
7. What is Reflection, and how does it power framework "magic" like Jackson/Spring?
8. What are the real costs of using Reflection?
9. Why do framework annotations need `RUNTIME` retention?
10. What is a Dynamic Proxy, and how does Spring's `@Transactional` use it?
11. What is `MethodHandle`, and why is it generally faster than classic Reflection?
12. What does `invokedynamic` enable, and how does it relate to lambda expressions?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** From memory, explain what `invokevirtual` does and why it's the concrete bytecode confirmation of Module 05's polymorphism.
2. **Hands-on:** Compile a small class with an overridden method, a static method, and a `private` method, then use `javap -c` to find and compare `invokevirtual`, `invokestatic`, and `invokespecial` in the output.
3. **Hands-on:** Write a custom `@Retention(RUNTIME)` annotation, apply it to a method, and write Reflection-based code that discovers and invokes annotated methods, mimicking a minimal JUnit-style test runner.
4. **Hands-on:** Implement a Dynamic Proxy that logs every method call's name and timing around a real implementation.
5. **Conceptual:** For each scenario, choose the most appropriate GC and justify it: (a) a small CLI tool, (b) a latency-critical trading system with a 64GB heap, (c) a high-throughput nightly batch job.
6. **Synthesis:** Explain, end to end, referencing Topics 4–5 directly, how a minimal dependency-injection framework could use annotations (`@Inject`) plus Reflection to automatically construct and wire two classes together, without either class needing to know about the framework itself.

## What's Next

Module 16 completed the deepest, most mechanistic tier of this course's JVM coverage — you now understand not just what Java does, but precisely how, down to the bytecode and memory-model level. **Module 17 — Functional Programming** shifts to a completely different, higher-level lens: lambda expressions (now grounded in this module's `invokedynamic` mechanism), functional interfaces, method references, and the paradigm shift Java 8 introduced — one of the most significant changes in Java's entire history (Module 01, Topic 2's timeline flagged this repeatedly).

---

**Previous:** [06 — Method Handles & Modern Internals](06-method-handles-and-modern-internals.md) · **Module Overview:** [00 — Module Overview](00-module-overview.md)

**Type "Continue" to begin Module 17 — Functional Programming.**
