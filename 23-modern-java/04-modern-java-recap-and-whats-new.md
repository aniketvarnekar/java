# Modern Java Recap & What's New Through Java 25

## Learning Objectives

- Recite a complete, dated timeline of Java's major features from Java 8 (2014) through Java 25 (2025)
- Know exactly where in this course each already-covered feature was taught, in full
- Learn, for the first time, the remaining Java 21-25 features that don't warrant a standalone topic: Sequenced Collections, Stream Gatherers, unnamed variables/patterns, Scoped Values, the Foreign Function & Memory API, and the status of String Templates

## Prerequisites

Every prior module of this course — this topic is a deliberate synthesis, not a new subject.

## Motivation

You have now learned Java's features **in place**, at the module where each conceptually belonged. That was the right way to *learn* them — but it means no single place in this course shows the whole timeline at once. Interviewers, tech leads doing version-upgrade planning, and your own mental model all benefit from one more thing: a **map of the whole territory**, in chronological order, with pointers back to full coverage. That's this topic's first half. Its second half gives full, first-time treatment to the handful of genuinely new Java 21-25 features that don't need (and didn't get) a dedicated topic of their own.

## The Complete Timeline: Java 8 → Java 25

| Version | Year | Feature | Where taught |
|---|---|---|---|
| **8** | 2014 | Lambda expressions | Module 17, Topic 2 |
| **8** | 2014 | Functional interfaces (`Function`, `Predicate`, etc.) | Module 17, Topic 3 |
| **8** | 2014 | Method references | Module 17, Topic 4 |
| **8** | 2014 | Streams API | Module 18 (entire module) |
| **8** | 2014 | `Optional` | Module 17, Topic 6 |
| **8** | 2014 | Default & static interface methods | Module 05, Topic 6 |
| **8** | 2014 | New Date/Time API (`java.time`) | Module 10 (IO), practical connection |
| **9** | 2017 | Java Platform Module System (JPMS) | Module 21 (entire module) |
| **9** | 2017 | `try`-with-resources multi-variable improvement | Module 12, Topic 5 |
| **10** | 2018 | Local variable type inference (`var`) | Module 03, Topic 1 |
| **11** | 2018 | LTS release; `HttpClient` standardized | Module 19, Topic 4 |
| **12-13** | 2019 | Switch expressions (preview) | Module 04, Topic 2 |
| **14** | 2020 | Switch expressions (finalized); Records (preview) | Module 04, Topic 2; Module 23, Topic 1 |
| **15** | 2020 | Text blocks (finalized) | Module 08, Topic 5 |
| **16** | 2021 | **Records** (finalized); `instanceof` pattern matching | Module 23, Topic 1; Module 23, Topic 3 |
| **17** | 2021 | LTS release; **Sealed classes** (finalized) | Module 23, Topic 2 |
| **19-20** | 2022-23 | Virtual Threads (preview); Structured Concurrency (incubator) | Module 15, Topic 8 |
| **21** | 2023 | **LTS release**; Virtual Threads (finalized); `switch` pattern matching & record patterns (finalized); Sequenced Collections | Module 15, Topic 8; Module 23, Topic 3; **below** |
| **22-23** | 2024 | Structured Concurrency (preview, continued); Scoped Values (preview); Stream Gatherers (preview) | **below** |
| **24-25** | 2025 | **LTS release (25)**; Structured Concurrency & Scoped Values (finalizing); Stream Gatherers (finalized); FFM API (finalized) | **below** |

**How to read the "preview" progression:** Java's modern release cadence (a new version every six months, with an LTS — Long-Term Support — release roughly every two years) deliberately ships large features as **preview** first (usable, but requiring an explicit `--enable-preview` flag, and still subject to change), often across multiple releases, before **finalizing** them as permanent, stable language features. This is *why* you'll see some features (Virtual Threads, Structured Concurrency) listed across several versions — the finalized version is the one that matters for production use, but the preview history explains why online resources sometimes describe seemingly conflicting syntax for the "same" feature.

## Feature Coverage Already Complete Elsewhere — Quick Recap

You do not need to re-read these — they are complete. This is purely a locator:

- **`var`** (Java 10) — full coverage: Module 03, Topic 1. Recall the core rule: `var` infers the **compile-time static type** from the initializer; it is not dynamic typing (Module 03, Topic 1's explicit myth-busting), and requires an initializer, so `var x;` alone is a compile error.
- **Switch expressions** (Java 14) — full coverage: Module 04, Topic 2. Recall `->` arrow syntax (no fall-through), `yield` for multi-statement branches, and that switch expressions **must** be exhaustive.
- **Text blocks** (Java 15) — full coverage: Module 08, Topic 5. Recall `"""` delimiters, automatic incidental-whitespace stripping, and why they solved the "SQL/JSON string readability" problem specifically.
- **Virtual Threads & Structured Concurrency** (finalized Java 21) — full coverage: Module 15, Topic 8. Recall the M:N scheduling model solving Module 14's C10K problem, and Structured Concurrency's "a task's child threads' lifetime is scoped to the parent" discipline.
- **Records** (Java 16) — full coverage: Module 23, Topic 1.
- **Sealed classes** (Java 17) — full coverage: Module 23, Topic 2.
- **Pattern matching (`instanceof`, `switch`, record patterns)** (Java 16-21) — full coverage: Module 23, Topic 3.

## First Coverage Here: Sequenced Collections (Java 21)

Recall Module 09's Collections Framework: `List` had a clear first/last-element notion, but `Set` and `Map` historically didn't guarantee one uniformly — `LinkedHashSet` happened to have a predictable order, but there was no **common interface** exposing "get the first element" or "get the last element" across `List`, `Set`, and `Map` implementations that do have a defined order. **`SequencedCollection`, `SequencedSet`, and `SequencedMap`** (Java 21) are new interfaces filling that gap:

```java
SequencedCollection<String> list = new ArrayList<>(List.of("a", "b", "c"));
list.getFirst();     // "a" -- no more list.get(0)
list.getLast();      // "c" -- no more list.get(list.size() - 1)
list.reversed();      // a VIEW of the collection in reverse order -- no manual Collections.reverse()

SequencedMap<String, Integer> map = new LinkedHashMap<>();
map.putFirst("x", 1);
map.putLast("z", 3);
map.firstEntry();     // the first Map.Entry, by encounter order
```

**Why did this need a dedicated interface, instead of just adding these methods to `List`/`Set`/`Map` directly?** Because `Set` and `Map` are not *inherently* ordered (Module 09) — `HashSet`/`HashMap` have no defined "first" element. `SequencedCollection`/`SequencedSet`/`SequencedMap` are implemented **only** by collections that genuinely have a defined encounter order (`ArrayList`, `LinkedHashSet`, `LinkedHashMap`, `TreeMap`, etc.) — `HashMap` itself does not implement `SequencedMap`, correctly reflecting that it has no meaningful "first entry."

## First Coverage Here: Stream Gatherers (Java 24, finalized Java 25)

Recall Module 18's Streams: the standard intermediate operations (`map`, `filter`, `flatMap`, etc.) cover most needs, but some genuinely useful transformations — like "group consecutive equal elements" or "take a sliding window of size N" — have **no** built-in Stream operation, historically forcing an awkward drop into `collect()` with a custom `Collector`, or abandoning the Stream API for a manual loop. **`Stream.gather()`** (finalized Java 25) introduces a new, custom, **composable intermediate operation** for exactly this gap:

```java
List<List<Integer>> windows = Stream.of(1, 2, 3, 4, 5)
    .gather(Gatherers.windowSliding(2))     // built-in gatherer: sliding windows of size 2
    .toList();
// [[1, 2], [2, 3], [3, 4], [4, 5]]

List<Integer> fixedGroups = Stream.of(1, 2, 3, 4, 5)
    .gather(Gatherers.windowFixed(2))       // fixed, non-overlapping windows of size 2
    .toList();
```

**Why is this different from `collect()`?** `collect()` (Module 18) is a **terminal** operation — it ends the stream and produces one final result. `gather()` is an **intermediate** operation — like `map()` or `filter()`, it can be chained with further stream operations afterward, and custom gathers can be composed together, which a `Collector` cannot.

## First Coverage Here: Unnamed Variables & Patterns (Java 21 for catch, Java 22 for general use)

Recall Module 12's `catch` blocks and Module 23, Topic 3's pattern variables: sometimes you're required to write a variable name that you will **never actually use** — e.g., a `catch` block that only cares *that* an exception occurred, not its details, or a pattern match where you only care about a record's type, not one of its components. The **unnamed variable**, `_`, states that explicitly:

```java
try {
    riskyOperation();
} catch (IOException _) {          // "_" -- we don't need the exception object itself
    log.error("operation failed");
}

record Point(int x, int y) { }
switch (obj) {
    case Point(int x, int _) -> System.out.println("x is " + x);   // don't care about y
    default -> {}
}
```

**Why does this matter, beyond saving a few characters?** It's a **readability and intent signal**: `catch (IOException _)` tells the next reader, unambiguously, "this variable is never used, on purpose" — whereas `catch (IOException e)` with an unused `e` leaves a reader wondering whether that was an oversight (and typically triggers an "unused variable" lint warning that `_` deliberately silences, because it isn't a variable to use at all).

## First Coverage Here: Scoped Values (finalizing Java 21-25)

Recall Module 15's `ThreadLocal` (a Module 15, Topic 1-adjacent concept): a way to give each thread its own private copy of a value, commonly used for things like a request ID or a security context that needs to be accessible anywhere in a thread's call stack without passing it as an explicit parameter everywhere. `ThreadLocal` has two real problems in the Virtual Threads era (Module 15, Topic 8): it is **mutable** (any code, anywhere in the call stack, can call `.set()` and change it — a genuine bug source), and it does not **automatically clean up**, which can cause memory leaks in long-lived thread pools. **`ScopedValue`** is the modern replacement, designed specifically to pair well with the potentially **millions** of Virtual Threads a Structured Concurrency-based application might create:

```java
static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();

void handleRequest(String id) {
    ScopedValue.where(REQUEST_ID, id).run(() -> {
        processOrder();   // REQUEST_ID.get() is valid ANYWHERE inside this call tree
    });
    // once .run() returns, REQUEST_ID is automatically, deterministically unbound -- no leak possible
}

void processOrder() {
    log.info("Processing for request " + REQUEST_ID.get());   // accessible without being passed as a parameter
}
```

**Why is this a meaningfully better fit than `ThreadLocal` for Virtual Threads specifically?** A `ScopedValue` is **immutable for the duration of its binding** (set once, for the exact scope of that `.run()` call, and automatically and deterministically un-bound the instant it returns) — no `.set()` method exists to mutate it mid-flight, and no risk of a value silently "leaking" into unrelated work on a reused thread, which matters enormously when Virtual Threads (Module 15, Topic 8) mean a single logical task's thread is genuinely cheap and short-lived, potentially one of millions.

## First Coverage Here: The Foreign Function & Memory API (finalized Java 22)

Recall Module 16, Topic 4's JNI mention in passing, and Module 02's Heap/native-memory distinction: calling native (C/C++) code or accessing memory outside the JVM's managed Heap historically required **JNI (Java Native Interface)** — notoriously difficult, unsafe, and verbose, requiring native compiled glue code on both sides. The **Foreign Function & Memory API** (`java.lang.foreign`, finalized in Java 22) is a modern, pure-Java replacement:

```java
// Calling a native C library function directly from Java -- no JNI glue code required
Linker linker = Linker.nativeLinker();
SymbolLookup stdlib = linker.defaultLookup();
MethodHandle strlen = linker.downcallHandle(
    stdlib.find("strlen").orElseThrow(),
    FunctionDescriptor.of(ValueLayout.JAVA_LONG, ValueLayout.ADDRESS)
);
```

**Why does this matter, practically?** It gives Java safe(r), directly-in-Java access to native libraries and off-heap memory — genuinely relevant for high-performance computing, interfacing with existing native libraries (image processing, ML runtimes), and workloads wanting explicit, GC-independent memory management (Module 16's GC discussion) — all without JNI's fragile native glue-code layer. This is a specialist API most application developers will never touch directly, but its existence explains how modern high-performance Java libraries increasingly avoid JNI.

## String Templates — Status Note

Java 21 and 22 previewed **String Templates** (`STR."Hello \{name}"` syntax, intended as a safer, more powerful successor to text blocks and string concatenation for interpolation). As of Java 23, this feature was **withdrawn from preview** for redesign, based on preview feedback — it is not part of Java 25. **Why mention a withdrawn feature at all?** Because you will encounter code samples and older articles online referencing `STR."..."` syntax, and should recognize it as a preview feature that did **not** ship, rather than assuming your JDK is simply out of date. Java's preview mechanism (see the timeline note above) exists precisely to allow this kind of course-correction before a feature becomes permanent.

## Real-World Analogy

Learning each feature in its home module was like learning a new tool exactly when you first needed it, on the job. This topic's timeline is like **finally seeing the entire toolbox laid out on a bench at once**, labeled with the year each tool was added — useful not for learning to use any individual tool (you already can), but for understanding the whole collection's history and reasoning about *why* the toolkit evolved the way it did.

## Advantages

- A dated, complete timeline is directly useful for real engineering decisions: choosing a minimum supported Java version, planning an upgrade, or explaining "why does our codebase use X but not Y" in a design review.
- Sequenced Collections, Stream Gatherers, unnamed variables, Scoped Values, and the FFM API each solve a genuine, specific gap — not novelty for its own sake.

## Disadvantages / Trade-offs

- Several of these features (FFM API, Stream Gatherers, Scoped Values) are specialist tools most application developers will rarely touch directly — worth recognizing, not necessarily worth reaching for by default.
- Preview-feature churn (String Templates being withdrawn) means "the newest thing you read about online" is not always representative of what actually shipped — always check a feature's finalization status before relying on it in production code.

## Best Practices

- Before adopting any Java 21-25 feature in production, verify its status (preview vs. finalized) for your specific target Java version — preview features require explicit flags and are not guaranteed stable.
- Prefer `SequencedCollection` methods (`getFirst()`/`getLast()`/`reversed()`) over the old index-arithmetic idioms (`list.get(list.size() - 1)`) in any codebase targeting Java 21+ — clearer and less error-prone.
- Prefer `ScopedValue` over `ThreadLocal` for new code specifically in Virtual-Thread-heavy (Module 15, Topic 8) codebases targeting Java 21+.

## Common Mistakes

- Writing `STR."..."` string template syntax expecting it to compile on a current JDK — it was withdrawn and is not present in Java 25.
- Assuming `HashMap`/`HashSet` implement `SequencedMap`/`SequencedSet` — they don't, because they have no defined encounter order to sequence.
- Reaching for the Foreign Function & Memory API or JNI-replacement tooling for ordinary application code where no genuine native-interop need exists — added complexity with no corresponding benefit.

## Interview Questions

1. **Q: Walk through Java's major features from Java 8 to Java 21 in order.**
   A: Lambdas/Streams/Optional/default methods (8) → JPMS (9) → `var` (10) → switch expressions (14) → text blocks (15) → records (16) → sealed classes (17) → Virtual Threads & switch pattern matching/record patterns finalized (21).

2. **Q: What problem does `SequencedCollection` solve that `List`/`Set`/`Map` didn't already?**
   A: It provides a common interface for "get first/last element" and "get a reversed view" across ordered collection types (`List`, `LinkedHashSet`, `LinkedHashMap`, etc.) that previously had no shared contract for that, despite genuinely having a defined encounter order.

3. **Q: Why is `ScopedValue` considered a better fit than `ThreadLocal` for Virtual-Thread-heavy code?**
   A: `ScopedValue` is immutable for the duration of a single, well-defined binding scope and is automatically, deterministically unbound when that scope ends — eliminating both the accidental-mutation risk and the memory-leak risk that `ThreadLocal`'s mutable, manually-managed lifecycle carries, which matters more at the potentially millions-of-threads scale Virtual Threads enable.

4. **Q: What happened to Java's String Templates feature?**
   A: It was previewed in Java 21-22, then withdrawn from preview in Java 23 for redesign based on feedback — it is not part of Java 25, despite still appearing in some older articles and tutorials.

## Summary

- Java's full feature timeline from **Java 8 (2014) to Java 25 (2025)** is now mapped in one place, with pointers back to every feature's full, in-place coverage earlier in this course.
- **First covered here:** Sequenced Collections (Java 21) for uniform first/last/reversed access; Stream Gatherers (Java 25) for custom composable intermediate stream operations; unnamed variables `_` (Java 21-22) for explicit "intentionally unused" signaling; Scoped Values (finalizing Java 21-25) as `ThreadLocal`'s safer, Virtual-Thread-friendly successor; and the Foreign Function & Memory API (Java 22) as JNI's modern, pure-Java replacement.
- **String Templates were withdrawn** from preview in Java 23 and are not part of the language as of Java 25 — a useful reminder that preview status is not a guarantee of eventual finalization.

## Exercises

1. From memory, list five Java features in chronological order, stating the version each was introduced or finalized in.
2. Rewrite `list.get(list.size() - 1)` and a manual `Collections.reverse(list)` copy using `SequencedCollection` methods instead.
3. Explain, in your own words, why `ScopedValue` is a better fit than `ThreadLocal` specifically in a codebase using Virtual Threads and Structured Concurrency (Module 15, Topic 8).
4. Explain why encountering `STR."Hello \{name}"` syntax in an online code sample should make you check which Java version and feature status it refers to, rather than assuming it will compile.

---

**Previous:** [03 — Pattern Matching](03-pattern-matching.md) · **Next:** [05 — Module Summary, Interview Questions & Exercises](05-module-summary-exercises.md)
