# Module 17 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-module-overview.md) has been covered:

- [x] **Functional Interfaces** — the "exactly one abstract method" rule, `@FunctionalInterface`, and retroactively recognizing `Runnable`/`Comparator` as functional interfaces used since Modules 10/15
- [x] **Lambda Expressions** — every syntactic form, the direct anonymous-class comparison (grounded in Module 16's `invokedynamic`), and the precise "effectively final" closure rule tied to Module 02's Stack model
- [x] **Method References** — all four kinds, and the clear "pure pass-through" decision rule
- [x] **Built-In Functional Interfaces** — `Function`/`Supplier`/`Consumer`/`Predicate`, `Bi`-variants, primitive specializations (avoiding Module 03's autoboxing), and composition via `default` methods (Module 05, Topic 6)
- [x] **`Optional`** — the problem it solves, idiomatic functional-chaining usage, and the real, common anti-patterns

## Practical Connections

- **Module 18 (Streams), immediately next, is built almost entirely on this module's vocabulary** — every `.map(...)`, `.filter(...)`, `.forEach(...)` call in a stream pipeline is a direct application of `Function`/`Predicate`/`Consumer` (Topic 4), passed as lambdas or method references (Topics 2–3).
- **Every Spring `@Bean` method reference, every `List.forEach(System.out::println)`, every `Comparator.comparing(...)` chain** in real code is direct, everyday application of this module.
- **REST API response handling** commonly returns `Optional<T>` from repository lookups (`findById` in Spring Data), directly applying Topic 5's idiomatic pattern.
- **Event-driven and reactive frameworks** (message listeners, `CompletableFuture` callbacks from Module 15) are built entirely on functional interfaces as their core abstraction.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| Functional interface vs. ordinary interface | Functional: exactly one abstract method, usable with lambdas. Ordinary: any number of abstract methods, requires a full implementing/anonymous class. |
| Lambda capturing a local variable vs. an instance field | Local variable: captured as a value snapshot, must be effectively final. Instance field: captured via the enclosing object reference, freely mutable. |
| `andThen` vs `compose` | `f.andThen(g)`: f first, then g. `f.compose(g)`: g first, then f (reversed). |
| `Optional.get()` vs `.map()`/`.orElse()` | `.get()` without checking reproduces the exact bug `Optional` prevents; the functional-chaining style handles absence safely and idiomatically. |
| `Function<Integer,Integer>` vs `IntUnaryOperator` | The former autoboxes every call; the latter avoids boxing entirely — relevant for high-iteration-count code (Module 18). |

## Consolidated Interview Questions (Module 17)

1. What is a functional interface, precisely?
2. Is `@FunctionalInterface` required for an interface to be usable with a lambda?
3. Why must local variables captured by a lambda be effectively final?
4. Does the effectively-final restriction apply to instance fields captured by a lambda?
5. What's the structural difference between a lambda and an equivalent anonymous class?
6. What are the four kinds of method references?
7. What does `String::length` mean, given there's no specific object mentioned?
8. What are the four core `java.util.function` interfaces?
9. Why do primitive-specialized functional interfaces exist?
10. What's the difference between `andThen` and `compose`?
11. What problem does `Optional` solve?
12. Why is calling `.get()` on an `Optional` without checking presence an anti-pattern?
13. Why is using `Optional` as a field type discouraged?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** From memory, state the "exactly one abstract method" rule and explain why it's what makes lambdas possible within Java's static type system.
2. **Hands-on:** Write a custom `@FunctionalInterface` and implement it three ways: an anonymous class, a lambda, and (if applicable) a method reference — compare all three.
3. **Hands-on:** Write a method returning `Optional<User>`, and a caller using `.map(...).filter(...).orElseThrow(...)` to process the result safely.
4. **Hands-on:** Benchmark (conceptually or with `System.nanoTime()`) a tight loop using `Predicate<Integer>` versus `IntPredicate`, and explain the expected difference referencing autoboxing.
5. **Conceptual:** Explain why a lambda capturing a local variable that's later reassigned fails to compile, tracing the explanation back to Module 02's Stack model.
6. **Synthesis:** Design a small validation pipeline using `Predicate<String>.and(...)`/`.or(...)`/`.negate()` combining three named predicates (non-blank, minimum length, no whitespace), and apply it to filter a `List<String>`.

## What's Next

Module 17 gave you the complete functional programming toolkit — lambdas, method references, and the standard functional interface library. **Module 18 — Streams** now puts every piece of this module to direct, constant use: the Stream API's `map`/`filter`/`reduce`/`collect` pipeline, parallel streams, and the `Collectors` utility class — the modern, idiomatic way to process collections in Java, replacing much of the explicit loop-writing you've done since Module 04.

---

**Previous:** [05 — `Optional`](05-optional.md) · **Module Overview:** [00 — Module Overview](00-module-overview.md)

**Type "Continue" to begin Module 18 — Streams.**
