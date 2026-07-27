# Migration, `jlink` & Practical Adoption

## Learning Objectives

- Understand automatic modules and the unnamed module — how JPMS coexists with the traditional classpath
- Use `jlink` to build a minimal custom runtime image, closing the loop from Module 01
- Have an honest, practical understanding of JPMS's real-world adoption

## Prerequisites

[02 — Module Descriptors & Directives](02-Module-Descriptors-And-Directives.md), Module 01 Topic 4 (JDK/JRE — the `jlink` mention)

## Motivation

Recall Module 01, Topic 2's core philosophy: Java maintains extremely strong backward compatibility. JPMS, introduced in Java 9, had to coexist with 20+ years of existing, non-modular JARs and applications — this topic covers exactly how that coexistence works, and closes with an honest assessment of how JPMS adoption has actually gone in practice.

## The Unnamed Module — Where Traditional Classpath Code Lives

**Any code still loaded via the traditional classpath (not a genuine, `module-info.java`-declared module) is automatically placed into a special catch-all called the "unnamed module."** This is precisely what makes JPMS backward-compatible by design: **your existing, non-modularized applications continue working, completely unchanged**, exactly per Module 01, Topic 2's core promise — you are never *forced* to modularize existing code.

## Automatic Modules — A Bridge for Existing JARs

**When a traditional (non-modularized) JAR is placed on the *module path* (rather than the classpath) instead**, the JVM automatically treats it as an **"automatic module"** — inferring a module name from the JAR's filename, and implicitly `exports`-ing **every** package it contains (since the JAR itself has no `module-info.java` declaring anything more precise).

```
 Traditional classpath:              Module path, with an automatic module:

 my-library.jar (unnamed module,      my-library.jar (AUTOMATIC module --
 works exactly as before, no           inferred name, ALL packages
 module system involvement)             implicitly exported)
```

**This is a deliberate, practical bridge**: it lets a **genuine** module (with its own real `module-info.java`) `requires` an older, not-yet-modularized library, **without** that library's authors needing to have already adopted JPMS themselves — enabling gradual, incremental migration rather than requiring an all-or-nothing "the entire ecosystem must modularize simultaneously" transition, which would have been genuinely impractical.

## `jlink` — Building Minimal, Custom Runtime Images

Recall Module 01, Topic 4's JDK tools list, where `jlink` was mentioned but not explained. **Here is that explanation, in full**:

```bash
jlink --module-path $JAVA_HOME/jmods:mods --add-modules com.mycompany.inventory \
      --output my-custom-runtime
```

**`jlink` analyzes your application's actual module dependencies** (via its `requires` declarations, Topic 2) **and produces a genuinely minimal, custom Java runtime image** — containing **only** the specific JDK modules your application actually needs, rather than a full, general-purpose JDK/JRE installation with every standard module included, whether used or not.

**Why does this matter, concretely?** Recall Module 02, Topic 6's GraalVM Native Image discussion, and Module 13's containerization concerns: a `jlink`-produced custom runtime can be **dramatically smaller** than a full JDK — directly relevant for minimal container images (Module 13's practical connection), faster application startup (less to load), and reduced attack surface (genuinely fewer JDK internals present at all, not just inaccessible via `exports`). **This is the direct, concrete payoff of everything this module has built toward**: because JPMS makes a module's true dependencies explicit and verifiable (`requires`, Topic 2), `jlink` can reliably determine the *minimum* necessary runtime — something impossible to do reliably with the traditional, unstructured classpath model.

## The Honest, Practical State of JPMS Adoption

**This is worth stating directly and honestly, not just optimistically**: JPMS adoption across the broader Java ecosystem has been **significantly slower and more partial** than originally anticipated when it shipped in Java 9 (2017). Several genuine, real reasons:

- **Migrating a large, existing codebase to genuine modules is real, non-trivial work** — resolving split packages (the same package spread across multiple JARs, which JPMS explicitly disallows), untangling implicit dependencies that the classpath never forced you to declare explicitly, and coordinating the migration across an entire dependency tree.
- **Many popular libraries and frameworks remain only partially modularized**, or rely on automatic modules rather than genuine `module-info.java` declarations — meaning full, end-to-end JPMS benefits are often not realized even in projects that attempt adoption.
- **The classpath/unnamed-module path remains fully supported and, in practice, remains extremely common** — many real, modern Java applications (including large numbers of Spring Boot applications) still run primarily on the traditional classpath, never adopting `module-info.java` for their own application code at all.

**What has stuck, and matters regardless of your own application's modularization status**: the JDK's own internal modularization (Topic 1's core motivating problem) is a permanent, real, and successful outcome — internal JDK classes are now genuinely, structurally inaccessible in ways they never were before Java 9, a real security and maintainability win for the platform itself, independent of whether *your* application ever adopts `module-info.java`.

## Real-World Analogy

Think of the unnamed module and automatic modules like a **city's transition to a new building-permit system** — existing buildings, built under the old rules, are **grandfathered in** and continue operating exactly as before (the unnamed module), while new construction adjacent to them can still connect to their utilities via a **temporary, provisional bridge permit** (an automatic module) even before the old building formally upgrades to the new permit system itself. `jlink` is like being able to **construct a brand-new, minimal building containing only the exact rooms your specific business actually needs**, precisely because the new permit system (JPMS) requires every room's purpose and connections to be explicitly, verifiably declared upfront — something impossible to do reliably under the old system, where buildings could have hidden, undeclared connections to anything, anywhere.

## Advantages

- The unnamed module and automatic modules provide genuine, real backward compatibility, letting adoption be gradual and incremental rather than forced and immediate.
- `jlink` provides a genuine, real, practical payoff for full module adoption — dramatically smaller, faster-starting custom runtime images.
- The JDK's own successful internal modularization is a real, permanent security and maintainability improvement, benefiting every Java application regardless of its own modularization status.

## Disadvantages / Trade-offs

- Full JPMS adoption across a large, real codebase remains genuinely significant work, and the broader ecosystem's partial adoption limits the realized benefit for many real projects.
- Split packages and other migration obstacles can make modularizing legacy code genuinely difficult in practice.

## Best Practices

- Understand that not modularizing your own application code is a completely normal, common, supported choice — the unnamed module ensures this remains fully functional.
- Consider `jlink` specifically when minimal container image size or startup time genuinely matters for your deployment — a real, practical motivation independent of broader philosophical JPMS adoption.
- When you do need to depend on a non-modularized library from a genuine module, rely on the automatic-module bridge mechanism rather than treating it as a blocker.

## Common Mistakes

- Assuming JPMS is mandatory for modern Java development — it remains fully optional; the classpath and unnamed module continue to work exactly as before.
- Assuming every real-world library is fully, properly modularized — many remain automatic modules or classpath-only, a genuine, real practical limitation to be aware of.
- Overlooking `jlink`'s real, practical value for container/deployment size, assuming JPMS's benefits are purely about compile-time encapsulation.

## Interview Questions

1. **Q: What is the unnamed module, and why does it exist?**
   A: The catch-all module that traditional, non-`module-info.java` classpath code is automatically placed into — it exists specifically to preserve full backward compatibility (Module 01, Topic 2's core philosophy), ensuring existing, non-modularized applications continue working entirely unchanged under JPMS.

2. **Q: What is an automatic module, and what problem does it solve?**
   A: A non-modularized JAR placed on the module path, automatically treated as a module with an inferred name and all packages implicitly exported — it lets genuine modules depend on libraries that haven't yet adopted JPMS themselves, enabling gradual, incremental migration rather than requiring simultaneous, ecosystem-wide adoption.

3. **Q: What does `jlink` do, and why does JPMS specifically make it possible?**
   A: It builds a minimal, custom Java runtime image containing only the specific JDK modules an application actually needs, based on its declared `requires` dependencies — this is only reliably possible because JPMS makes true module dependencies explicit and verifiable, something the traditional, unstructured classpath model could never provide.

## Summary

- The **unnamed module** preserves full backward compatibility for traditional classpath code; **automatic modules** bridge non-modularized JARs into module-based dependency graphs, enabling gradual migration.
- **`jlink`** builds minimal, custom runtime images based on a module's actual declared dependencies — a genuine, practical payoff for full adoption, directly relevant to container image size and startup time.
- JPMS adoption across the broader ecosystem has been genuinely slower and more partial than originally anticipated — modularizing your own application remains optional — but the JDK's own internal modularization is a real, permanent, successful outcome benefiting every Java application regardless.

## Module-Wide Quick Revision

- Pre-JPMS: the classpath was flat/unordered (JAR hell, no dependency verification), and `public` provided no genuine cross-JAR encapsulation, severely limiting the JDK team's own ability to hide internals (Topic 1).
- `module-info.java` declares `requires` (explicit, verified dependencies) and `exports` (genuine, structural package-level encapsulation, stronger than access modifiers alone); `opens` separately grants deep Reflective access; `provides`/`uses` formalize the Service Provider Interface pattern (Topic 2).
- The unnamed module and automatic modules preserve backward compatibility and enable gradual migration; `jlink` builds minimal custom runtime images; ecosystem-wide adoption remains genuinely partial, though the JDK's own modularization is a real, lasting success (this topic).

## Common Pitfalls (Module-Wide)

- Assuming `public` alone ever provided real cross-JAR encapsulation.
- Forgetting `opens` is required separately from `exports` for Reflection-heavy frameworks.
- Assuming JPMS adoption is mandatory for modern Java applications.
- Overlooking `jlink`'s practical container/startup-time benefits.

## Mini Quiz (Module-Wide)

1. What problem does "JAR hell" refer to?
2. Why doesn't `public` alone provide real encapsulation across JAR boundaries, pre-JPMS?
3. What does `exports` control, precisely?
4. Why does `opens` exist as a directive separate from `exports`?
5. What does `jlink` produce, and why does JPMS make it possible?

*(Answers are derivable from Topics 1, 1, 2, 2, and this topic, respectively.)*

---

**Previous:** [02 — Module Descriptors & Directives](02-Module-Descriptors-And-Directives.md) · **Next:** [04 — Module Summary, Interview Questions & Exercises](04-Module-Summary-Exercises.md)
