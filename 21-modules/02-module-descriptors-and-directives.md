# Module Descriptors & Directives

## Learning Objectives

- Write a `module-info.java` file with `requires` and `exports`
- Understand `opens` and why Reflection needs it specifically
- Understand `provides`/`uses` as a formalized version of Module 20's Service Provider Interface mechanism

## Prerequisites

[01 — Why JPMS — The Problems It Solves](01-why-jpms-the-problems-it-solves.md), Module 16 Topic 4 (Reflection)

## Motivation

This topic delivers the actual mechanics — a `module-info.java` file, and the small set of directives that let you declare, precisely and enforceably, what a module needs from others and what it genuinely offers in return.

## `module-info.java` — The Module Descriptor

**A module is simply a JAR file with one additional file at its root: `module-info.java`** (compiled to `module-info.class`), declaring the module's name and its relationships to other modules:

```java
// module-info.java
module com.mycompany.inventory {
    requires java.sql;                    // this module NEEDS the java.sql module
    requires com.mycompany.shared;           // ...and this OTHER, custom module too

    exports com.mycompany.inventory.api;       // packages EXTERNALLY VISIBLE to other modules
    // com.mycompany.inventory.internal is NOT listed -- therefore genuinely,
    // STRUCTURALLY inaccessible from outside this module, EVEN IF its classes are public!
}
```

## `requires` — Declaring Dependencies Explicitly

```java
module com.mycompany.inventory {
    requires java.sql;
}
```

**This directly, completely solves Topic 1's "no reliable way to verify dependencies are complete" problem**: if `com.mycompany.inventory`'s code actually uses classes from `java.sql` but the module descriptor doesn't declare `requires java.sql`, **the module simply fails to compile or launch at all** — not "maybe fails at runtime, whenever that specific code path happens to execute," but an immediate, guaranteed, upfront failure. This is Module 01, Topic 3's "Robust" philosophy (fail loudly, predictably, as early as possible) applied at the dependency-management level.

## `exports` — Genuine, Structural Encapsulation

```java
module com.mycompany.inventory {
    exports com.mycompany.inventory.api;   // ONLY this package is visible externally
}
```

**This is the direct, complete resolution of Topic 1's "public doesn't mean what you think" problem**: any package **not** explicitly listed in an `exports` directive is **completely, structurally inaccessible** from outside the module — **regardless of whether its classes are marked `public`**. A class in `com.mycompany.inventory.internal` can be `public` (necessary for other packages *within* the same module to use it) while remaining **genuinely, enforceably hidden** from every other module — precisely the capability Topic 1 established was missing before JPMS.

```java
// From a DIFFERENT module, attempting to use a non-exported internal class:
import com.mycompany.inventory.internal.InternalHelper;   // ⚠️ COMPILE ERROR:
                                                              // package is not visible/does not exist
                                                              // to this module -- EVEN THOUGH the
                                                              // class itself is technically 'public'!
```

**This is genuinely, structurally stronger encapsulation than `public`/`private`/`protected` (Module 05, Topic 2) alone ever provided** — those modifiers control visibility *within* a single compilation context; `exports` controls visibility **between entire modules**, a fundamentally different, additional layer of encapsulation.

## `opens` — Allowing Deep Reflective Access (Recall Module 16, Topic 4)

**Recall Module 16, Topic 4**: frameworks like Jackson/Spring rely on Reflection, sometimes including `setAccessible(true)` to bypass normal access control entirely. **JPMS's strong encapsulation, by default, blocks this too** — `exports` only permits **normal, compile-time** access to a package's `public` members; it does **not**, by default, permit deep Reflective access (like reading/writing `private` fields via `setAccessible(true)`) from outside the module.

```java
module com.mycompany.inventory {
    exports com.mycompany.inventory.api;         // normal compile-time access
    opens com.mycompany.inventory.entities;         // ADDITIONALLY permits deep REFLECTIVE access
}                                                       // (needed for frameworks like Jackson/Hibernate
                                                          // to populate private fields via Reflection!)
```

**Why does `opens` need to exist as a separate directive from `exports`?** This is a genuinely important, deliberate design distinction: **ordinary compile-time code access** and **framework-style deep Reflective access** are different capabilities with different risk profiles, and JPMS lets you grant them **independently** — a package might be safe to use normally (`exports`) while its internal field structure shouldn't be reflectively manipulated by arbitrary external code, or vice versa. This directly, precisely closes the loop on Module 16, Topic 4's discussion of `setAccessible(true)`'s real power and risk — JPMS gives module authors explicit, fine-grained control over exactly this capability, rather than an all-or-nothing choice.

## `provides`/`uses` — A Formalized Service Provider Interface (Recall Module 20, Topic 1)

Recall Module 20, Topic 1's JDBC driver discovery mechanism — the Service Provider Interface (SPI), a Reflection-based dynamic discovery pattern. **JPMS formalizes this exact pattern with dedicated directives:**

```java
// The module DEFINING the service interface:
module com.mycompany.paymentapi {
    exports com.mycompany.paymentapi;
    uses com.mycompany.paymentapi.PaymentProcessor;   // "I will USE some implementation of this,
}                                                        // discovered dynamically, without knowing
                                                          // which one at compile time"

// A module PROVIDING a specific implementation:
module com.mycompany.stripeprovider {
    requires com.mycompany.paymentapi;
    provides com.mycompany.paymentapi.PaymentProcessor
        with com.mycompany.stripeprovider.StripePaymentProcessor;   // "I PROVIDE this implementation"
}
```

**This is precisely, formally, the same "discover an implementation without compile-time knowledge of it" pattern from Module 20's JDBC drivers and Module 16's Reflection-based framework discovery** — JPMS simply gives it dedicated, module-descriptor-level syntax, letting the module system itself verify and wire up service implementations, rather than relying purely on file-based SPI metadata conventions.

## Real-World Analogy

Think of `requires` like a **building's explicit, verified utility hookups list** ("this building requires water, electricity, and gas connections") — if a hookup is missing, the building inspector catches it **before** anyone moves in, rather than someone discovering "oh, there's no water" only when they first try to use a faucet. Think of `exports` like **genuine, structural walls with real locked doors between departments** — not just a "staff only" sign taped to an always-open doorway (`public` alone) — someone from outside the department **cannot** physically get in at all, regardless of whether they know the way. `opens` is like a **specific, additional permission slip granting a particular inspector (a framework) authority to open normally-locked internal cabinets for a legitimate audit purpose**, separate from the general "staff only" door policy. `provides`/`uses` is like a **standardized supplier registry** — a building declares "I need a caterer" (`uses`), and any registered catering company can formally declare "I provide catering services" (`provides`), letting the building find and use a suitable supplier without needing to know which specific company in advance.

## Advantages

- `requires` provides genuine, upfront, guaranteed dependency verification — no more "missing dependency only discovered at runtime, in production."
- `exports` provides genuinely stronger, structural encapsulation than access modifiers alone, finally letting library/JDK authors truly hide internal implementation details.
- `opens`/`provides`/`uses` give fine-grained, deliberate control over Reflection access and service discovery, formalizing patterns (Modules 16, 20) that previously relied on looser, convention-based mechanisms.

## Disadvantages / Trade-offs

- Writing and maintaining `module-info.java` files adds real, additional structure/ceremony compared to the traditional, simpler classpath model.
- Migrating a large, existing, non-modular codebase to genuine JPMS modules can be a genuinely significant, real undertaking (Topic 3 covers this honestly).

## Best Practices

- Declare `requires` for every module dependency your code genuinely uses — let the module system catch missing dependencies at compile/launch time, not runtime.
- Only `exports` packages that are genuinely part of your module's intended public API; keep implementation-detail packages unexported.
- Use `opens` deliberately and specifically, only for packages that genuinely need deep Reflective access from frameworks — not as a blanket default.

## Common Mistakes

- Assuming `exports` alone is sufficient for framework Reflection needs, forgetting `opens` is required separately for deep Reflective access.
- Exporting every package "just in case," undermining JPMS's core encapsulation benefit.
- Forgetting `requires` declarations, encountering module-system errors that (correctly, per Module 01, Topic 3's philosophy) surface early rather than silently allowing missing-dependency issues to reach runtime.

## Interview Questions

1. **Q: What does `exports` in a `module-info.java` file actually control, and how is it different from `public`?**
   A: It controls which packages are genuinely, structurally visible to code in *other* modules — a package not exported is completely inaccessible externally, regardless of whether its classes are marked `public`. This is a stronger, module-level encapsulation layer, distinct from and additional to `public`/`private`/`protected`'s within-compilation-context visibility control (Module 05, Topic 2).

2. **Q: Why does `opens` need to exist as a directive separate from `exports`?**
   A: Ordinary compile-time code access and deep Reflective access (like `setAccessible(true)`-based field manipulation, Module 16, Topic 4) are different capabilities with different risk profiles — `opens` lets a module author grant the latter specifically and deliberately, independent of (and often needed by frameworks in addition to) `exports`'s normal access grant.

3. **Q: How do `provides`/`uses` relate to Module 20's JDBC driver discovery mechanism?**
   A: They formalize the same underlying Service Provider Interface (SPI) pattern — a module declares it `uses` some service interface without knowing the implementation at compile time, and other modules `provide` specific implementations, letting the module system discover and wire them together, directly analogous to how JDBC drivers are discovered.

## Summary

- **`module-info.java`** declares a module's name and its relationships to other modules.
- **`requires`** declares dependencies explicitly, letting missing dependencies be caught at compile/launch time rather than discovered at runtime.
- **`exports`** provides genuine, structural encapsulation — a non-exported package is inaccessible from other modules regardless of `public`, directly resolving Topic 1's "public doesn't mean what you think" problem.
- **`opens`** separately grants deep Reflective access (needed by frameworks, Module 16, Topic 4), independent of `exports`'s normal compile-time access grant.
- **`provides`/`uses`** formalize the Service Provider Interface pattern (Module 20, Topic 1) with dedicated module-descriptor syntax.

## Exercises

1. Write a `module-info.java` for a hypothetical `com.example.orders` module requiring `java.sql` and exporting only a `com.example.orders.api` package.
2. Explain why a Jackson-based JSON library would need `opens`, not just `exports`, on the package containing your data classes.
3. Explain, using the JDBC driver analogy, what `uses PaymentProcessor` and `provides PaymentProcessor with StripePaymentProcessor` each declare.

---

**Previous:** [01 — Why JPMS — The Problems It Solves](01-why-jpms-the-problems-it-solves.md) · **Next:** [03 — Migration, `jlink` & Practical Adoption](03-migration-jlink-and-practical-adoption.md)
