# Class Loader Subsystem

## Learning Objectives

- Explain the three phases of class loading: Loading, Linking, Initialization — and the sub-steps within Linking
- Describe the built-in class loader hierarchy and the parent-delegation model
- Explain why delegation exists, and what problem it prevents
- Understand when custom class loaders are used in real frameworks

## Prerequisites

[01 — JVM Architecture Overview](01-jvm-architecture-overview.md)

## Motivation

`NoClassDefFoundError`, `ClassNotFoundException`, and the "magic" way frameworks like Tomcat or Spring Boot isolate different applications' classes from each other — all of this is explained by the class loading process in this topic. It also explains a genuinely surprising fact: **static initializers don't run when a class is merely referenced — they run when it's first *actively used***, which trips up even experienced developers.

## Problem Statement

Before the JVM can execute a single instruction of your class, it needs an answer to several questions:
- Where is this class's bytecode? (It could be on the local disk, inside a `.jar`, or — historically for applets — over a network.)
- Is this bytecode safe and well-formed?
- Have all the other classes it depends on also been made ready?
- When exactly should static fields be given their real initial values, and static initializer blocks be run?

The Class Loader Subsystem answers all of these, in three well-defined phases.

## The Three Phases

```
           LOADING                           LINKING                           INITIALIZATION
┌────────────────────────┐   ┌────────────────────────────────────────────┐   ┌─────────────────────────────┐
│ Find and Read          │   │ Verification                               │   │ Run Static Initializers     │
│ .class File            │   │ • Bytecode Safety Checks                   │   │ and Assign Static Values    │
│                        │──▶│                                            │──▶│                             │
│ Create Class Object    │   │ Preparation                                │   │ Execute static blocks       │
│ in JVM Memory          │   │ • Allocate Memory for Static Fields        │   │ and static field            │
│                        │   │ • Assign Default Values                    │   │ initializations             │
│                        │   │                                            │   │                             │
│                        │   │ Resolution                                 │   │                             │
│                        │   │ • Symbolic References                      │   │                             │
│                        │   │   → Direct References                      │   │                             │
└────────────────────────┘   └────────────────────────────────────────────┘   └─────────────────────────────┘
```

### Phase 1: Loading

The JVM locates the `.class` file's bytecode (via a specific class loader — see hierarchy below), reads its binary contents, and constructs an in-memory `java.lang.Class` object representing that type in the **Method Area** (Topic 3). This `Class` object is what powers **Reflection** (Module 16) — it's the JVM's own internal metadata record for the type.

### Phase 2: Linking

Linking itself has three sub-steps:

1. **Verification** — the Bytecode Verifier (introduced in Module 01, Topic 5) checks the loaded bytecode for structural correctness and safety: valid jump targets, correct operand stack usage, legal type conversions, and enforced access rules. **This is where a corrupted or maliciously hand-crafted `.class` file gets rejected**, regardless of who produced it.
2. **Preparation** — the JVM allocates memory for the class's **static fields** and assigns them **default values** (`0` for numeric types, `false` for `boolean`, `null` for references) — **not** their real programmer-specified initial values yet. That happens later, in Initialization.
3. **Resolution** (can happen lazily) — symbolic references in the bytecode (e.g., a reference to another class or method by name) are resolved into direct, concrete references — actual memory addresses/offsets the Execution Engine can use directly, instead of looking the name up again on every use.

### Phase 3: Initialization

The JVM executes the class's **static initializer blocks** (`static { ... }`) and assigns **real values** to static fields, in the order they textually appear in the source. This is the phase where, for example, `static int counter = 5;` actually becomes `5` (Preparation had already set it to `0`).

> **This is precisely why `static` initialization order matters and can cause real bugs** — if `static` field `A` in one class depends on being initialized before `static` field `B` in another, and their initialization order isn't what you assumed, you can observe unexpected `0`/`null` values. Full depth on `static` behavior: Module 06.

## When Does Initialization Actually Happen?

This is one of the most commonly misunderstood details in all of Java: **a class is NOT initialized just because it's mentioned/referenced somewhere in your code.** Initialization is **lazy** — it happens only upon **active use**, which specifically includes:

- Creating an instance of the class (`new SomeClass()`)
- Calling a static method on it
- Accessing (reading or writing) a static field on it (that isn't a `final` compile-time constant)
- Reflectively invoking it in certain ways
- Initializing a subclass (which requires its superclass to be initialized first)

**Merely declaring a variable of that type**, or accessing a `static final` field that's a compile-time constant (the compiler inlines these directly), does **not** trigger initialization.

```java
class Config {
    static { System.out.println("Config class initialized!"); }
    static int value = 42;
}

class Demo {
    public static void main(String[] args) {
        Config c;                    // no initialization -- this is just a reference declaration
        System.out.println("Before use");
        int v = Config.value;        // <-- THIS triggers initialization (active use)
    }
}
```
**Output:**
```
Before use
Config class initialized!
```
Notice `"Config class initialized!"` prints *after* `"Before use"` — proving initialization was deferred until the actual field access, not eagerly done just because `Config` was mentioned earlier as a declared type.

## The Class Loader Hierarchy

The JVM doesn't use one single class loader — it uses a **hierarchy** of loaders, each responsible for a different source of classes:

```
                           ┌──────────────────────────────────────────────┐
                           │        Bootstrap Class Loader                │
                           │                                              │
                           │ • Implemented in native code                 │
                           │ • Loads core Java modules                    │
                           │   (java.base: java.lang, java.util, etc.)    │
                           └──────────────────────┬───────────────────────┘
                                                  │
                                                  │ Parent of
                                                  ▼
                           ┌──────────────────────────────────────────────┐
                           │         Platform Class Loader                │
                           │                                              │
                           │ • Loads JDK platform modules                 │
                           │ • Examples: java.sql, java.xml, java.desktop │
                           └──────────────────────┬───────────────────────┘
                                                  │
                                                  │ Parent of
                                                  ▼
                           ┌──────────────────────────────────────────────┐
                           │      Application (System) Class Loader       │
                           │                                              │
                           │ • Loads application classes                  │
                           │ • Searches the classpath/modulepath          │
                           └──────────────────────┬───────────────────────┘
                                                  │
                                                  │ Parent of (Optional)
                                                  ▼
                           ┌──────────────────────────────────────────────┐
                           │          Custom Class Loader(s)              │
                           │                                              │
                           │ • Created by applications/frameworks         │
                           │ • Used for plugins, hot reloading,           │
                           │   web application isolation, etc.            │
                           └──────────────────────────────────────────────┘
```

> **Naming note:** older material (pre-Java 9) calls the middle loader the "Extension ClassLoader" (`ext` directory) — this was renamed conceptually to the "Platform ClassLoader" after the Java Platform Module System (JPMS, Module 21) restructured the JDK itself into modules in Java 9. You may still see "Extension ClassLoader" in older books/answers — know both names.

## The Parent Delegation Model

**The rule:** when a class loader is asked to load a class, it does **not** try to load it itself first. Instead, it **delegates upward** to its parent first, and only tries to load it itself if every ancestor, all the way up to the Bootstrap loader, fails to find it.

```
Request to load "com.myapp.MyClass"
              │
              ▼
    Application ClassLoader
              │  "let me ask my parent first..."
              ▼
    Platform ClassLoader
              │  "let me ask MY parent first..."
              ▼
    Bootstrap ClassLoader
              │
              ▼
    "Not a core JDK class -- I can't find it. Failing back down."
              │
              ▼
    Platform ClassLoader: "Not a platform module class either -- failing back down."
              │
              ▼
    Application ClassLoader: "OK, *I'll* look for it on the classpath now." -- FOUND, loads it.
```

### Why Delegation Exists (The "Why")

**Problem it solves: preventing core class spoofing.** Imagine you (accidentally or maliciously) create your own class named `java.lang.String` in your application code. Without delegation, your Application ClassLoader might load *your* fake `String` class instead of the JDK's real one — a catastrophic security and correctness hole, since so much of the platform assumes `String` behaves exactly as specified.

**With delegation:** any request for `java.lang.String` always gets delegated all the way up to the Bootstrap ClassLoader first, which finds and loads the *real*, trusted core class before your Application ClassLoader ever gets a chance to load a conflicting one. This guarantees **core Java classes are always the authentic, trusted ones**, no matter what your application code defines.

> This is called the **"sandbox" of core classes**, and is a direct extension of the security model discussed in Module 01, Topic 3.

## Why Multiple Loaders (Not Just One)?

Beyond just security, having distinct loader *instances* enables a powerful capability: **the same class name can be loaded independently, multiple times, as genuinely different types**, if loaded by different class loader instances. This is precisely how:
- **Application servers** (Tomcat, etc.) run multiple independent web applications in one JVM process, each with its own class loader, so App A's version of a library doesn't conflict with App B's different version of the same library.
- **Plugin systems** (IDEs, build tools) load plugin code in isolated class loaders, so plugins can be loaded/unloaded/updated independently.
- **Hot-reloading frameworks** (some dev-tool "live reload" features) replace a class loader entirely to load fresh bytecode for changed classes.

> **Deep fact:** in the JVM, a class's true runtime identity is not just its fully-qualified name — it's the **pair** (class name, defining class loader). Two classes with the identical name, loaded by two different class loader instances, are treated as **completely different, incompatible types** by the JVM, even though they look identical in source code. This is an advanced but real and interview-relevant fact.

## Custom Class Loaders

You can write your own class loader (by extending `java.lang.ClassLoader`) to load bytecode from unconventional sources — decrypting encrypted class files, loading classes generated dynamically at runtime, loading from a database or network location, or providing plugin isolation as described above. This is an advanced technique (used heavily by application servers and frameworks), not something typical application code needs to do directly — but recognizing *why* it's possible, and that it underlies tools you may already use, is valuable.

## Advantages

- Delegation model provides a strong, simple security guarantee (core classes can't be spoofed) with very little runtime overhead.
- Multiple loader instances enable powerful isolation capabilities (multi-app servers, plugin systems) without needing multiple JVM processes.
- Lazy initialization avoids doing unnecessary work for classes that are referenced but never actually used.

## Disadvantages / Trade-offs

- `ClassLoader`-related bugs (e.g., `ClassCastException: com.foo.Bar cannot be cast to com.foo.Bar` — yes, that's the same name twice!) are notoriously confusing precisely *because* of the "same name, different loader = different type" rule — this is a real, if advanced, category of production bugs in complex systems (like app servers) with multiple loaders.
- Lazy initialization means static initializer side effects (like logging, or eager caching) can happen at surprising, hard-to-predict times if you don't understand the "active use" trigger rules above.

## Best Practices

- Don't rely on static initializer side effects for anything time-sensitive unless you fully understand exactly what "active use" means for your specific access pattern.
- If you ever see a `ClassCastException` between what looks like the *same* class name, suspect a class-loader identity mismatch (multiple loaders having independently loaded "the same" class) before assuming it's a simple logic bug.

## Common Mistakes

| Mistake | Correction |
|---|---|
| "A class is initialized as soon as the JVM starts, if it's on the classpath." | False — initialization is **lazy**, triggered only by active use (instantiation, static method/field access, subclass initialization). |
| "Class loaders load classes independently, whoever gets asked first." | False — the parent-delegation model means a loader always asks its parent first, and only loads it itself if every ancestor fails. |
| Confusing `ClassNotFoundException` and `NoClassDefFoundError` | `ClassNotFoundException` is a checked exception thrown when code explicitly asks to load a class by name (e.g., `Class.forName(...)`) and it can't be found. `NoClassDefFoundError` is thrown when a class **was** present and successfully compiled against, but is missing at runtime when actually needed (e.g., a `.jar` dependency wasn't included when running) — a different failure mode at a different point in the lifecycle. (Full exception-type distinctions: Module 12.) |

## Interview Questions

1. **Q: What are the three phases of class loading, and what happens in each?**
   A: Loading (find and read bytecode, create the in-memory `Class` object), Linking (Verification — safety checks; Preparation — allocate static fields with default values; Resolution — turn symbolic references into direct ones), and Initialization (run static initializer blocks, assign real static field values).

2. **Q: What is the parent delegation model, and why does it exist?**
   A: Each class loader asks its parent to try loading a requested class before attempting to load it itself, all the way up to the Bootstrap loader. This exists primarily for security: it guarantees core JDK classes (like `java.lang.String`) are always loaded by the trusted Bootstrap loader first, preventing application code from "spoofing"/shadowing core classes with a same-named malicious or accidental replacement.

3. **Q: When exactly is a class initialized in Java?**
   A: Lazily, upon "active use" — the first time it's instantiated, a static method/non-constant static field on it is accessed, or a subclass of it is being initialized — not merely when it's referenced as a declared type or imported.

4. **Q: Can two classes with the exact same fully-qualified name coexist in one running JVM?**
   A: Yes — if they're loaded by two *different* class loader instances, the JVM treats them as entirely distinct, incompatible types, because a class's true runtime identity is the pair (name, defining loader), not just the name alone. This is what enables app servers to isolate multiple applications' classes within one JVM process.

## Summary

- Class loading has three phases: **Loading → Linking (Verify, Prepare, Resolve) → Initialization**.
- Initialization is **lazy** — triggered by active use, not by mere reference.
- The JVM uses a **hierarchy** of class loaders (Bootstrap → Platform → Application → optional custom loaders), connected by the **parent delegation model**, which exists primarily to protect core classes from being spoofed by application code.
- A class's true identity is (name + defining loader) — the same name loaded by different loaders is treated as different, incompatible types, which is what enables app-server-style class isolation.

## Exercises

1. Predict the output of the `Config`/`Demo` example in this topic, then explain in your own words *why* initialization happens when it does, referencing the specific rule that triggers it.
2. Explain, step by step, what happens (in terms of the delegation model) if your own application code defines a class named `java.lang.String`. Does your version ever get used for core JDK operations? Why or why not?
3. Without looking back, list the three sub-steps of the Linking phase and one sentence on what each one checks/does.
4. Research prompt: name one real framework or tool you've heard of (or look one up) that uses custom class loaders, and briefly explain why it needs that capability rather than using the default Application ClassLoader alone.

---

**Previous:** [01 — JVM Architecture Overview](01-jvm-architecture-overview.md) · **Next:** [03 — Runtime Data Areas](03-runtime-data-areas.md)
