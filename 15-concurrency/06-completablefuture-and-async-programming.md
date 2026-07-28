# `CompletableFuture` & Async Programming

## Learning Objectives

- Understand the real limitation of plain `Future` that `CompletableFuture` solves
- Chain and compose asynchronous operations using `thenApply`/`thenCompose`/`thenCombine`
- Handle exceptions in asynchronous chains correctly
- Combine multiple independent async operations

## Prerequisites

[05 — Executors & Thread Pools](05-executors-and-thread-pools.md)

## Motivation

Plain `Future` (Topic 5) has a real, practical limitation: once you call `.get()`, you're blocked waiting — there's no way to say "when this finishes, automatically do the next thing." `CompletableFuture` (Java 8+) solves this, and is the direct foundation of the reactive/async programming style you'll encounter throughout modern Java backend frameworks.

## The Limitation of Plain `Future`

```java
Future<Integer> future = executor.submit(() -> fetchDataFromApi());
Integer result = future.get();   // BLOCKS -- you can't say "and then process it automatically"
int processed = process(result);   // must happen AFTER the blocking .get() returns
```

**Plain `Future` has no way to attach a "when this completes, do this next" callback** — you can only synchronously block and wait. For chains of dependent asynchronous operations (fetch data, then process it, then save it, then notify someone), this forces you back into blocking, sequential-looking code even though the underlying work is asynchronous.

## `CompletableFuture` — Composable Asynchronous Chains

```java
import java.util.concurrent.CompletableFuture;

CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> fetchDataFromApi());

CompletableFuture<String> chained = future
    .thenApply(data -> process(data))          // runs AFTER 'future' completes, transforms the result
    .thenApply(processed -> "Result: " + processed);

chained.thenAccept(System.out::println);          // final step -- consumes the result, no further chaining
```

**Each `then...` call registers a callback that runs automatically once the previous stage completes** — no blocking `.get()` needed anywhere in the chain itself. This is a genuinely different **style** of programming: instead of "do this, wait, do the next thing," you're **describing a pipeline** of transformations that execute as data becomes available.

## The Core Chaining Methods

| Method | Purpose |
|---|---|
| `thenApply(Function)` | Transform the result — like `Stream.map` (Module 18 preview) |
| `thenAccept(Consumer)` | Consume the result, produce no further value (end of a chain) |
| `thenRun(Runnable)` | Run some code after completion, ignoring the result entirely |
| `thenCompose(Function returning another CompletableFuture)` | **Chain** dependent async operations — like `Stream.flatMap` |
| `thenCombine(otherFuture, BiFunction)` | Combine **two independent** async operations' results once **both** complete |

```java
// thenApply vs thenCompose -- a genuinely important distinction:

CompletableFuture<Integer> f1 = CompletableFuture.supplyAsync(() -> 5);

// thenApply: the function returns a PLAIN value
CompletableFuture<Integer> doubled = f1.thenApply(x -> x * 2);   // CompletableFuture<Integer>

// thenCompose: the function returns ANOTHER CompletableFuture (a DEPENDENT async operation)
CompletableFuture<Integer> chained2 = f1.thenCompose(x -> fetchRelatedData(x));   // still
                                                                                      // CompletableFuture<Integer>,
                                                                                      // NOT
                                                                                      // CompletableFuture<CompletableFuture<Integer>>!
```

**This `thenApply`/`thenCompose` distinction is precisely analogous to `Stream.map` vs. `Stream.flatMap`** (full depth Module 18) — `thenApply` wraps a plain transformation; `thenCompose` **flattens** a nested asynchronous dependency, avoiding a genuinely awkward `CompletableFuture<CompletableFuture<T>>` that `thenApply` would otherwise produce if used for a function that itself returns a `CompletableFuture`.

## Combining Independent Operations — `thenCombine` and `allOf`

```java
CompletableFuture<Integer> priceFuture = CompletableFuture.supplyAsync(() -> fetchPrice());
CompletableFuture<Integer> stockFuture = CompletableFuture.supplyAsync(() -> fetchStock());

CompletableFuture<String> combined = priceFuture.thenCombine(stockFuture,
    (price, stock) -> "Price: " + price + ", Stock: " + stock);
```

**`thenCombine` waits for BOTH independent futures to complete**, then combines their results — genuinely useful when you have two (or more) unrelated async operations that can run **concurrently**, and you need both results together before proceeding. For combining an arbitrary **number** of futures, `CompletableFuture.allOf(future1, future2, future3, ...)` waits for all of them to complete (though, notably, it returns `CompletableFuture<Void>` — you retrieve each individual result separately afterward).

## Exception Handling in Async Chains

```java
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    if (Math.random() < 0.5) throw new RuntimeException("failed!");
    return 42;
});

CompletableFuture<Integer> handled = future
    .exceptionally(ex -> {                 // catches ANY exception from earlier in the chain
        System.out.println("Recovered from: " + ex.getMessage());
        return -1;                              // a FALLBACK value
    });

// or, to handle BOTH success and failure in one place:
future.handle((result, ex) -> {
    if (ex != null) {
        return -1;    // fallback
    }
    return result;
});
```

**An exception thrown anywhere earlier in a `CompletableFuture` chain automatically propagates through subsequent `thenApply`/`thenCompose` stages, skipping them entirely, until an `exceptionally`/`handle` stage catches it** — directly echoing Module 12's exception-propagation model (stack unwinding, Module 12 Topic 3), but adapted specifically for asynchronous chains, where there's no traditional call stack to unwind through in the same way.

## Real-World Analogy

Think of plain `Future` like **ordering food and being forced to stand at the counter, staring at the kitchen, until your order is ready** — you can't do anything else productive in the meantime except wait. Think of `CompletableFuture` like **ordering food and being handed a pager that buzzes when it's ready, with a note attached saying "when this buzzes, automatically start step 2" ** — you're free to walk away and do other things, and the *next* step happens automatically, triggered by completion, without you needing to stand there blocking on anything. `thenCombine` is like **two separate pagers for two separate orders, with a rule saying "combine both orders into one bag only once BOTH have buzzed."**

## Advantages

- Enables genuinely non-blocking, composable asynchronous pipelines — no thread sits idle waiting on `.get()` between chained steps.
- `thenCompose` correctly handles dependent asynchronous operations without awkward nested-future types.
- Built-in exception handling (`exceptionally`/`handle`) integrates cleanly into the async chain, rather than requiring separate, disconnected try/catch blocks around blocking calls.

## Disadvantages / Trade-offs

- Async chain-based code can be genuinely harder to read and debug than straightforward sequential/blocking code — stack traces in deeply chained async code can be less immediately intuitive than synchronous call stacks.
- Choosing between `thenApply`/`thenCompose` correctly requires understanding the distinction precisely — using the wrong one produces either a compile error or an awkward nested-future type.

## Best Practices

- Use `thenCompose` (not `thenApply`) whenever your transformation function itself returns a `CompletableFuture` — avoiding nested future types.
- Always include exception handling (`exceptionally`/`handle`) in real production async chains — an unhandled exception in a chain fails silently unless explicitly retrieved via `.get()`/`.join()`.
- Use `thenCombine` for genuinely independent operations that can run concurrently; use `thenCompose` for genuinely dependent, sequential async steps.

## Common Mistakes

- Using `thenApply` where `thenCompose` was needed, producing an awkward, doubly-wrapped `CompletableFuture<CompletableFuture<T>>`.
- Forgetting to handle exceptions in an async chain, silently losing failure information unless something later explicitly calls `.get()`/`.join()` and observes the exception.
- Treating `CompletableFuture` chains as a drop-in replacement for all blocking code without considering whether the added complexity is actually justified for the specific use case.

## Interview Questions

1. **Q: What limitation of plain `Future` does `CompletableFuture` address?**
   A: Plain `Future` offers no way to attach a "when this completes, automatically do the next thing" callback — you can only synchronously block via `.get()`. `CompletableFuture` lets you chain dependent transformations (`thenApply`/`thenCompose`) that run automatically upon completion, without blocking in between.

2. **Q: What's the difference between `thenApply` and `thenCompose`?**
   A: `thenApply` transforms a result with a function returning a plain value. `thenCompose` is used when the transformation function itself returns another `CompletableFuture` (a dependent async operation) — it flattens the result, avoiding a nested `CompletableFuture<CompletableFuture<T>>`, analogous to `Stream.map` vs. `Stream.flatMap`.

3. **Q: How do you handle an exception that occurs partway through a `CompletableFuture` chain?**
   A: `exceptionally(...)` catches any exception from earlier in the chain and supplies a fallback value; `handle(...)` receives both the result and the exception (one will be `null`), letting you handle both success and failure paths in one place.

## Summary

- **`CompletableFuture`** (Java 8+) solves plain `Future`'s lack of composability — you can chain dependent transformations without blocking, using `thenApply` (plain transformation), `thenCompose` (dependent async operation, flattened), and `thenCombine` (combine two independent async results).
- Exceptions propagate automatically through a chain, skipping subsequent stages, until caught by `exceptionally`/`handle`.
- This composable, chain-based style is foundational to modern reactive/async Java backend programming.