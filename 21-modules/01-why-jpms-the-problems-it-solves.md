# Why JPMS — The Problems It Solves

## Learning Objectives

- Understand the classpath's real, historical problems precisely
- Understand why `public` alone doesn't provide genuine encapsulation across JAR boundaries
- Understand why the JDK itself needed to be modularized

## Prerequisites

Module 05 Topic 2 (Encapsulation), Module 02 Topic 2 (Class Loader Subsystem)

## Motivation

JPMS's directives (Topic 2) will feel like unnecessary ceremony unless you first understand the genuine, real, historically significant problems they solve. This topic makes those problems concrete, using two specific, well-documented failure modes: the classpath's fragility, and `public`'s surprisingly weak encapsulation guarantee across library boundaries.

## Problem 1: The Classpath — A Flat, Unordered, Fragile List

**Before JPMS, every JAR your application depended on sat together on one single, flat "classpath"** — the Class Loader (Module 02, Topic 2) searched through this list, in whatever order it happened to be given, to find any requested class.

**Two genuine, real, historically common problems this caused:**

1. **"JAR hell" — silent version conflicts.** If your application depends on Library A, which itself depends on `commons-utility` version 1.0, and you also directly depend on `commons-utility` version 2.0 for your own code, **both versions end up on the same flat classpath** — whichever one the Class Loader happens to find **first** (an outcome depending on unpredictable classpath ordering) is the one that's actually used, **for everyone**, even code that specifically needed the other version. This could produce genuinely bizarre, hard-to-diagnose `NoSuchMethodError`s or `ClassNotFoundException`s at runtime, from code that looked, by inspection, like it should obviously work.

2. **No reliable way to know if your dependencies were even complete.** The classpath provided **no compile-time or startup-time verification** that every dependency your application actually needed was genuinely present — a missing dependency was often only discovered when the specific code path requiring it finally executed, potentially in production, long after deployment.

## Problem 2: `public` Doesn't Mean What You Think It Means, Across JAR Boundaries

Recall Module 05, Topic 2's encapsulation principle: `public` means "accessible from anywhere." **This has a genuinely surprising, real consequence at the JAR/library level**: if a library's internal implementation classes are marked `public` (often simply because they need to be `public` for the library's *own* different packages to use them internally — a real, common, hard-to-avoid situation pre-JPMS), **any external application depending on that library could also directly access those "internal" classes**, even though the library's authors never intended external code to use them at all.

```java
// A library's INTERNAL helper class, public only because OTHER packages
// within the SAME library need to use it:
package com.somelibrary.internal;
public class InternalHelper { ... }   // never MEANT for external use...

// ...but ANY external application depending on this library CAN still do:
import com.somelibrary.internal.InternalHelper;   // ⚠️ COMPILES FINE, even though this was
InternalHelper.doSomething();                        // never meant to be a PUBLIC API at all!
```

**This is a genuine, real problem**: library authors had **no way**, pre-JPMS, to say "this class must be `public` for my own library's internal packages to cooperate, but genuinely NOT part of my public API, and NOT something external code should ever depend on." Any external code depending on such an "accidentally public" internal class would then **break** the moment the library's authors refactored that internal implementation detail in a later version — precisely the fragile coupling good encapsulation (Module 05, Topic 2) is supposed to prevent, but couldn't, at this larger, cross-JAR scale.

## Why the JDK Itself Needed This

**The JDK's own internal implementation classes suffered from exactly this same problem, for decades** — some application code, over the JDK's long history, came to depend directly on **internal**, never-officially-supported JDK implementation classes (in packages like `sun.misc.*`), simply because they happened to be `public` and reachable. **This severely constrained the JDK team's own ability to refactor and improve the JDK's internals** — changing an internal implementation detail could break real, existing applications that had (often unknowingly) taken a dependency on something never meant to be a stable, public contract.

**JPMS was, in significant part, specifically designed to let the JDK team finally, genuinely hide these internal implementation details** — restructuring the entire JDK into modules, each explicitly declaring exactly which packages are genuinely, deliberately part of its public API (`exports`, Topic 2), with everything else made **truly, structurally inaccessible** from outside that module, regardless of whether individual classes happen to be marked `public`.

## Real-World Analogy

Think of the pre-JPMS classpath like a **single, shared warehouse shelf where every vendor's boxes are simply piled together, unlabeled, in whatever order they happened to arrive** — if two vendors' boxes happen to have the same label but different contents, whichever box you grab first (unpredictable) is what you get, with no reliable guarantee, and no way to verify beforehand that everything you actually need is even present on the shelf. Think of pre-JPMS `public` classes like a **company office with no genuine "employees only" doors at all** — some areas were only meant to be walked through by staff moving between departments internally, but since there was never a real door with a lock, any visitor who happened to know the way could wander in and start relying on furniture and equipment that was only ever meant as internal, subject-to-change office infrastructure, not a stable, publicly-provided service.

## Advantages of Understanding This History

- Makes JPMS's directives (Topic 2) feel like deliberate, well-motivated solutions to real, documented problems, rather than arbitrary new syntax to memorize.
- Explains genuinely confusing, real historical Java errors (`NoClassDefFoundError` from classpath ordering issues, code breaking after a JDK upgrade due to relying on internal `sun.*` classes) that you may encounter in legacy codebases.

## Disadvantages / Trade-offs

- These problems, while real and significant, are somewhat less visible to newer developers who started with build tools (Maven/Gradle) that partially mitigate classpath issues through dependency management — the *underlying* JVM-level classpath mechanism remains unchanged by these tools, though.

## Best Practices

- Understand that Maven/Gradle's dependency management (version resolution, conflict detection) operates at the *build tool* level, mitigating but not eliminating the underlying JVM classpath's own lack of built-in versioning/conflict awareness.
- Never depend directly on packages/classes clearly intended as internal implementation details (even if technically `public`), in either the JDK or third-party libraries — precisely the practice JPMS was designed to make impossible going forward.

## Common Mistakes

- Assuming `public` alone was ever a sufficient encapsulation guarantee across library/JAR boundaries — it wasn't, prior to JPMS's stronger, module-level enforcement (Topic 2).
- Underestimating how disruptive and real "JAR hell" version conflicts were in large, real-world dependency trees before better tooling and JPMS existed.

## Interview Questions

1. **Q: What problem does "JAR hell" refer to, and why couldn't the classpath prevent it?**
   A: Different libraries in the same application depending on different, conflicting versions of the same shared dependency — since the classpath is a single, flat, unordered list with no built-in version awareness, whichever version the Class Loader happens to find first (unpredictably) is used for everyone, potentially breaking code that needed the other version.

2. **Q: Why didn't marking a class `public` reliably provide real encapsulation across JAR boundaries before JPMS?**
   A: A class often needed to be `public` simply so other packages *within the same library* could use it internally — but this also made it fully accessible to any *external* code depending on that library, even though it was never intended as part of that library's actual public API, creating fragile, unintended coupling.

3. **Q: Why did the JDK team specifically need JPMS to modularize the JDK itself?**
   A: Decades of application code had come to depend on internal, never-officially-supported JDK implementation classes (like `sun.misc.*`) simply because they happened to be reachable/public — severely constraining the JDK team's ability to refactor internals without breaking real applications. JPMS let them finally, structurally hide these internals behind explicit module boundaries.

## Summary

- The pre-JPMS **classpath** was a flat, unordered list with no built-in version conflict detection ("JAR hell") and no reliable way to verify dependency completeness ahead of runtime.
- **`public`** alone never provided genuine encapsulation across JAR/library boundaries — classes needed `public` for internal cross-package cooperation were also, unintentionally, fully exposed to any external dependent code.
- The **JDK itself** suffered from exactly this problem for decades, motivating JPMS's core design: genuinely, structurally hiding internal implementation details behind explicit module boundaries.