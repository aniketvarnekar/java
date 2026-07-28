# Built-In Functional Interfaces

## Learning Objectives

- Use the four core `java.util.function` interfaces correctly
- Use their primitive-specialized variants to avoid unnecessary autoboxing
- Compose functional interfaces together (`andThen`, `compose`, `and`, `or`, `negate`)

## Prerequisites

[01 — Functional Interfaces](01-functional-interfaces.md), [02 — Lambda Expressions](02-lambda-expressions.md), Module 03 Topic 6 (autoboxing)

## Motivation

You don't need to write a custom functional interface (Topic 1) for most everyday needs — Java 8 shipped `java.util.function`, a standard library of common functional shapes. This topic covers the small handful you'll use constantly, especially once you reach Module 18's Streams API, which is built almost entirely on top of these exact interfaces.

## The Four Core Shapes

| Interface | Abstract method | Shape | Purpose |
|---|---|---|---|
| **`Function<T, R>`** | `R apply(T t)` | Takes one, returns one (possibly different type) | Transform a value |
| **`Supplier<T>`** | `T get()` | Takes nothing, returns one | Produce/generate a value |
| **`Consumer<T>`** | `void accept(T t)` | Takes one, returns nothing | Do something with a value (a side effect) |
| **`Predicate<T>`** | `boolean test(T t)` | Takes one, returns boolean | Test a condition |

```java
Function<String, Integer> length = String::length;      // "hello" -> 5
Supplier<LocalDate> today = LocalDate::now;                 // () -> today's date
Consumer<String> printer = System.out::println;               // "hello" -> (prints it, returns nothing)
Predicate<Integer> isEven = n -> n % 2 == 0;                     // 4 -> true
```

**These four shapes cover the overwhelming majority of everyday functional needs** — genuinely, most custom functional interfaces you might be tempted to write (Topic 1) already have a standard equivalent among these four (or their variants below), and reusing the standard ones is strongly preferred over reinventing them.

## Two-Argument Variants

```java
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
BiConsumer<String, Integer> printPair = (name, age) -> System.out.println(name + ": " + age);
BiPredicate<String, Integer> isLongerThan = (s, n) -> s.length() > n;
```

**`Bi`-prefixed variants exist for the common two-argument case** — there's no general `TriFunction`/etc. in the standard library, since beyond two parameters, a custom class/record (Module 23) genuinely communicates intent more clearly than an increasingly unwieldy generic functional interface.

## Primitive Specializations — Avoiding Autoboxing (Recall Module 03, Topic 6)

```java
Function<Integer, Integer> square = x -> x * x;          // BOXES every int argument/result!
IntUnaryOperator squareInt = x -> x * x;                    // NO boxing -- works directly with int

Predicate<Integer> isPositive = n -> n > 0;                  // boxes
IntPredicate isPositiveInt = n -> n > 0;                       // no boxing

Function<Integer, String> toStr = Object::toString;             // generic Function ALWAYS uses
                                                                    // reference types -- T and R must
                                                                    // be object types (Module 11's
                                                                    // generics/erasure discussion)
```

**Why do these primitive-specialized variants exist at all, given `Function<Integer, Integer>` "already works"?** Recall Module 03, Topic 6 and Module 11, Topic 4 (type erasure) directly: **generics require object type parameters** — a `Function<Integer, Integer>` genuinely autoboxes every single `int` argument into an `Integer` object and back, on every single call. For code executed **millions of times** (exactly Module 18's Streams API use case, processing large datasets), this autoboxing overhead is genuinely measurable. `IntUnaryOperator`, `IntPredicate`, `IntFunction<R>`, `ToIntFunction<T>`, and similar `int`/`long`/`double`-specialized interfaces exist **specifically** to avoid this — the standard library's own Streams implementation (Module 18) uses these specialized interfaces internally for exactly this performance reason.

## Composing Functional Interfaces — `default` Methods in Action

Recall Module 05, Topic 6's explanation of *why* `default` methods were added to interfaces in Java 8: **this is their single most important real-world application** — letting `java.util.function` interfaces provide composition helpers without breaking the "exactly one abstract method" functional-interface rule (Topic 1):

```java
Function<Integer, Integer> addTen = x -> x + 10;
Function<Integer, Integer> square = x -> x * x;

Function<Integer, Integer> combined = addTen.andThen(square);   // addTen FIRST, then square
combined.apply(5);   // (5 + 10) squared = 225

Function<Integer, Integer> combined2 = addTen.compose(square);   // square FIRST, then addTen
combined2.apply(5);    // (5 squared) + 10 = 35
```

**`andThen` vs. `compose` — the precise distinction**: `f.andThen(g)` means "run `f` first, then feed its result into `g`" (left to right, matching how you likely read the chain). `f.compose(g)` means "run `g` first, then feed its result into `f`" (the composed function runs **before** the one you called `compose` on) — mathematically, `f.compose(g)` is the traditional `f(g(x))` notation.

```java
Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> isEven = n -> n % 2 == 0;

Predicate<Integer> isPositiveAndEven = isPositive.and(isEven);    // BOTH must be true
Predicate<Integer> isPositiveOrEven = isPositive.or(isEven);        // EITHER must be true
Predicate<Integer> isNotPositive = isPositive.negate();                // INVERTS the predicate
```

**`Predicate`'s `and`/`or`/`negate` are directly analogous to Module 03, Topic 5's `&&`/`||`/`!` operators**, but operating on **entire predicates as composable values**, rather than on individual `boolean` expressions — a genuinely powerful pattern for building up complex filtering/validation logic from small, independently-named, reusable pieces.

## Real-World Analogy

Think of `Function`, `Supplier`, `Consumer`, and `Predicate` like **four standard-shaped power tool attachments** — a "transformer" bit, a "generator" bit, a "consumer" bit, a "tester" bit — each fitting a wide range of specific tasks without needing a custom-machined attachment for every single job (Topic 1's custom functional interfaces remain available, but are rarely needed given these four cover so much ground). `andThen`/`compose` are like **snapping two attachments together in a specific order** — the order you snap them together in genuinely changes which one processes the material first.

## Advantages

- The four core shapes (plus `Bi`-variants and primitive specializations) cover the overwhelming majority of everyday functional programming needs without requiring custom functional interfaces.
- Primitive specializations (`IntPredicate`, etc.) avoid real, measurable autoboxing overhead in performance-sensitive, high-iteration-count code — directly relevant to Module 18's Streams.
- `default` method composition (`andThen`/`compose`/`and`/`or`/`negate`) enables building complex behavior from small, independently reusable pieces.

## Disadvantages / Trade-offs

- The sheer number of variants (`Function`, `BiFunction`, `IntFunction`, `ToIntFunction`, `IntUnaryOperator`, ...) has a real learning curve before the naming conventions feel intuitive.
- `andThen` vs. `compose`'s reversed execution order is a genuinely common, real source of confusion until deliberately memorized.

## Best Practices

- Reach for a standard `java.util.function` interface before writing a custom one (Topic 1) — the four core shapes cover most needs.
- Use primitive-specialized variants in performance-sensitive, high-iteration code, especially once you reach Module 18's Streams.
- Use `Predicate`'s composition methods (`and`/`or`/`negate`) to build complex filtering logic from small, well-named, independently testable predicates.

## Common Mistakes

- Using `Function<Integer, Integer>` in a tight loop instead of `IntUnaryOperator`, incurring unnecessary autoboxing overhead.
- Confusing `andThen`'s left-to-right order with `compose`'s right-to-left order.
- Writing a custom functional interface (Topic 1) for a shape that `java.util.function` already provides.

## Interview Questions

1. **Q: What are the four core `java.util.function` interfaces, and what does each do?**
   A: `Function<T,R>` (transforms a value), `Supplier<T>` (produces a value, no input), `Consumer<T>` (consumes a value, no output — a side effect), `Predicate<T>` (tests a condition, returns boolean).

2. **Q: Why do primitive-specialized functional interfaces like `IntPredicate` exist, given `Predicate<Integer>` already works?**
   A: Generic interfaces require object type parameters, so `Predicate<Integer>` autoboxes every `int` into an `Integer` on every call (Module 03, Topic 6) — a real, measurable overhead in high-iteration-count code. Primitive-specialized variants avoid this entirely, which is why the Streams API (Module 18) uses them internally.

3. **Q: What's the difference between `Function.andThen` and `Function.compose`?**
   A: `f.andThen(g)` runs `f` first, then feeds its result into `g` (left to right). `f.compose(g)` runs `g` first, then feeds its result into `f` — mathematically `f(g(x))`, the reverse order from `andThen`.

## Summary

- **`Function<T,R>`**, **`Supplier<T>`**, **`Consumer<T>`**, and **`Predicate<T>`** are the four core `java.util.function` shapes, covering most everyday functional programming needs; `Bi`-prefixed variants handle the two-argument case.
- **Primitive-specialized variants** (`IntPredicate`, `IntUnaryOperator`, etc.) avoid autoboxing overhead — directly relevant to Module 18's Streams internals.
- **Composition** (`andThen`, `compose`, `and`, `or`, `negate`) — enabled by Module 05, Topic 6's `default` methods — lets you build complex behavior from small, independently reusable functional pieces.