# Exercises — Consolidated

All practice problems from every module, collected here in one place. Attempt each before checking the module's solution/discussion.

> This file grows with every module.

---

## Module 01 — Introduction

1. **Conceptual:** In your own words, explain why a `.class` file compiled on Windows can run unmodified on Linux. What component makes this possible, and what does *that* component do differently on each OS?
2. **Conceptual:** A colleague says "Java is an interpreted language." Another says "Java is a compiled language." Who is right? Justify your answer using the concepts of `javac`, bytecode, the interpreter, and the JIT compiler.
3. **Applied:** Without looking back at the notes, write out the exact `javac` and `java` commands you'd use to compile and run a file named `Calculator.java` containing a class `Calculator`.
4. **Applied:** Predict what error you'd get if you ran `java Calculator.java` (Java 11+ single-file mode) on a file where the class name does **not** match the filename. Then, if you have Java installed, verify it.
5. **Design reasoning:** Why do you think Java's designers made `main` require the exact signature `public static void main(String[] args)` instead of allowing any method name to serve as the entry point (like Python's implicit top-level script execution)?
6. **Research/Reflection:** List two situations where "write once, run anywhere" can break down in real-world Java applications, and explain why the JVM's portability guarantee doesn't cover them.

*(Solutions/discussion for these appear inline within [01-Introduction/09-Module-Summary-Exercises.md](01-Introduction/09-Module-Summary-Exercises.md).)*

---

## Module 02 — JVM

1. **Recall test:** Redraw the full JVM architecture diagram from memory (Class Loader Subsystem, Runtime Data Areas, Execution Engine, Native Method Interface), then annotate which specific error types or observable behaviors originate from each box.
2. **Hands-on:** Write a class with a `static` initializer block that prints a message, and a `main` method that first only *declares* a variable of that type, then later *actively uses* it. Run it and confirm the initializer runs only at the point of active use.
3. **Hands-on:** Write a deliberately infinite (no base case) recursive method, run it, and observe a real `StackOverflowError`. Relate the stack trace to the JVM Stack concept.
4. **Hands-on:** Run any small program with `java -XX:+PrintCompilation YourProgram` and identify at least one line showing a method being JIT-compiled.
5. **Conceptual:** Explain why reassigning an object parameter inside a method never affects the caller's own reference, grounding your answer in the Stack-vs-Heap model.
6. **Conceptual:** Would GraalVM Native Image likely help or hurt a long-running, CPU-intensive backend service that stays warm for weeks? Justify your answer.
7. **Synthesis:** Trace `System.out.println("Hello, World!")` from `HelloWorld.java` (Module 01) through class loading, the Method Area's constant pool, the Heap (where the String literal lives), the JVM Stack (the `main` frame), and the Execution Engine.

*(Solutions/discussion for these appear inline within [02-JVM/07-Module-Summary-Exercises.md](02-JVM/07-Module-Summary-Exercises.md).)*

---

## Module 03 — Java Basics

1. **Recall test:** List all 8 primitive types with sizes, then all 8 corresponding wrapper class names, from memory.
2. **Hands-on:** Write and run a program demonstrating the `Integer` cache boundary at `127`/`128` and `-128`/`-129`, confirming your `==` predictions against actual output.
3. **Hands-on:** Trigger silent `int` overflow (`Integer.MAX_VALUE + 1`), print the result, then rewrite using `Math.addExact()` and observe the thrown exception.
4. **Hands-on:** Write a method with a `final` parameter holding a mutable `List` — mutate its contents (legal) and attempt to reassign the parameter (illegal); confirm which the compiler rejects.
5. **Conceptual:** Explain why `ArrayList<int>` is illegal but `ArrayList<Integer>` works, referencing primitives' inline storage vs. generics' object requirement.
6. **Conceptual:** Explain precisely why `if (userInputFlag == 1)` fails to compile in Java when `userInputFlag` is an `int` used where a `boolean` condition is expected.
7. **Synthesis:** Write a method adding two `double` money values, and explain why this is inappropriate for a real financial application, and what type should be used instead.

*(Solutions/discussion for these appear inline within [03-Java-Basics/09-Module-Summary-Exercises.md](03-Java-Basics/09-Module-Summary-Exercises.md).)*

---

## Module 04 — Control Flow

1. **Recall test:** Draw the dangling-else example from Topic 1 and explain, without re-reading, exactly which `if` the `else` binds to and why.
2. **Hands-on:** Write a modern `switch` expression mapping a `char` grade to a GPA `double`, including a `default` that throws `IllegalArgumentException`.
3. **Hands-on:** Write a `do-while` loop implementing a number-guessing game's loop structure, explaining why `do-while` fits better than `while` here.
4. **Hands-on:** Write nested loops printing a multiplication table (1-10), then add a labeled `break` to stop generation early if any product exceeds 50.
5. **Conceptual:** Explain, referencing Module 03's compile-time constants, why `case someVariable:` is illegal in a `switch` but `case SOME_FINAL_CONSTANT:` is legal.
6. **Conceptual:** Explain why `for (float i = 0.0f; i != 1.0f; i += 0.1f)` may never terminate as expected, referencing Module 03's floating-point precision discussion.
7. **Synthesis:** Write a method that searches a 2D `int[][]` grid for a target value using nested loops and a labeled `break`, returning the position found or `{-1, -1}`.

*(Solutions/discussion for these appear inline within [04-Control-Flow/06-Module-Summary-Exercises.md](04-Control-Flow/06-Module-Summary-Exercises.md).)*

---

## Module 05 — OOP

1. **Recall test:** List the four pillars of OOP and write a one-sentence, precise definition for each, from memory.
2. **Hands-on:** Implement an abstract `Shape` class (with a `color` field and abstract `area()`/`perimeter()`), two concrete subclasses, and a `Drawable` interface both implement with a `default describe()` method.
3. **Hands-on:** Write a `Garage` class that HAS-A `List<Car>` via composition, with methods to add and start all cars — explain why this is composition, not aggregation.
4. **Conceptual:** Explain, using the Diamond Problem, why Java disallows `class Employee extends Person, Company`, and propose the correct alternative design.
5. **Conceptual:** Explain why `List<String> list = new ArrayList<>();` is generally preferred over `ArrayList<String> list = new ArrayList<>();`.
6. **Synthesis:** Design a `PaymentMethod` interface implemented by unrelated `CreditCard`, `PayPal`, and `BankTransfer` classes, and a `Checkout` class that calls `processPayment` polymorphically — explain how the JVM invokes the correct implementation for each.

*(Solutions/discussion for these appear inline within [05-OOP/08-Module-Summary-Exercises.md](05-OOP/08-Module-Summary-Exercises.md).)*

---

## Module 06 — Classes

1. **Recall test:** Write out the complete six-step object creation order for a two-level inheritance hierarchy, from memory.
2. **Hands-on:** Write a method taking a `List<Integer>` that both mutates it and reassigns the local parameter — verify experimentally which change the caller observes.
3. **Hands-on:** Implement a fluent `Pizza` builder (`addTopping`, `setSize`, each returning `this`) and chain three calls in one expression.
4. **Hands-on:** Implement a thread-unsafe Singleton `ConfigManager`, and explain one risk this simple implementation has in a multithreaded context.
5. **Conceptual:** Explain why `Parent p = new Child(); p.staticMethod();` calls `Parent`'s static method, contrasting with what would happen if it were an instance method instead.
6. **Synthesis:** Design a `BinaryTree` class using a `static` nested `Node` class with an `insert(int value)` method, explaining why `Node` should be static.

*(Solutions/discussion for these appear inline within [06-Classes/07-Module-Summary-Exercises.md](06-Classes/07-Module-Summary-Exercises.md).)*

---

## Module 07 — Objects

1. **Recall test:** List all five properties of the `equals()` contract, plus the separate equals/hashCode contract rule, from memory.
2. **Hands-on:** Write a `Money` class with correct `equals()`, `hashCode()`, and `toString()`, and demonstrate it working correctly inside a `HashSet`.
3. **Hands-on:** Deliberately break the equals/hashCode contract (equals only, no hashCode), demonstrate the `HashSet.contains()` failure, then fix it.
4. **Hands-on:** Implement a class with a mutable `List` field using a copy constructor, demonstrating the copy and original don't share state.
5. **Conceptual:** Explain, referencing GC Roots, why a local variable going out of scope can make a Heap object eligible for garbage collection.
6. **Synthesis:** Explain why relying on `finalize()` to close a database connection is unsafe, and write the correct try-with-resources replacement.

*(Solutions/discussion for these appear inline within [07-Objects/06-Module-Summary-Exercises.md](07-Objects/06-Module-Summary-Exercises.md).)*

---

## Module 08 — Strings

1. **Recall test:** List all four concrete benefits String's immutability enables, with a one-sentence mechanism for each.
2. **Hands-on:** Write a program demonstrating all five rows of the `==` decision table, printing actual results to confirm predictions.
3. **Hands-on:** Join a `List<String>` with `StringBuilder`, then rewrite using `String.join(...)` and compare.
4. **Hands-on:** Write a text block for a multi-line SQL query with intentional source indentation, and verify the stripped output.
5. **Conceptual:** Explain why String's immutability makes it an especially safe and efficient `HashMap` key type, referencing Module 07's equals/hashCode contract.
6. **Synthesis:** Build a formatted multi-line report using `StringBuilder` and `String.format` together.

*(Solutions/discussion for these appear inline within [08-Strings/06-Module-Summary-Exercises.md](08-Strings/06-Module-Summary-Exercises.md).)*

---

## Module 09 — Arrays

1. **Recall test:** Draw the Stack/Heap memory diagram for an `int[]` array with a few assigned values.
2. **Hands-on:** Write a method summing all elements of a jagged `int[][]` using correctly-bounded nested loops.
3. **Hands-on:** Demonstrate `Arrays.asList()`'s fixed-size quirk — attempt `.add()`, observe the exception, then use `.set()` and confirm the original array changed.
4. **Conceptual:** Explain step by step what happens internally when an `ArrayList` at backing-array capacity 4 receives a 5th `add()` call.
5. **Conceptual:** Explain why a plain `int[]` likely beats `ArrayList<Integer>` for a 10-million-element raw numeric dataset.
6. **Synthesis:** Write a method comparing two `int[]` arrays for equal content (using `Arrays.equals`) and, if equal, returning a new combined copy.

*(Solutions/discussion for these appear inline within [09-Arrays/05-Module-Summary-Exercises.md](09-Arrays/05-Module-Summary-Exercises.md).)*

---

## Module 10 — Collections

1. **Recall test:** Redraw the full Collections Framework hierarchy, then write out `HashMap.put`'s complete algorithm step by step.
2. **Hands-on:** Implement a `Book` class with correct `equals()`/`hashCode()` and `Comparable`, then use it in a `HashSet`, `Collections.sort`, and a `TreeSet`.
3. **Hands-on:** Build a word-frequency counter with `HashMap` and `computeIfAbsent`/`merge`, printing results sorted by frequency descending.
4. **Hands-on:** Reproduce and fix `ConcurrentModificationException` using both `Iterator.remove()` and `removeIf(...)`.
5. **Conceptual:** Explain why `ArrayDeque` and `ArrayList` share a cache-locality advantage over `LinkedList`.
6. **Synthesis:** Design a task-scheduling system using `PriorityQueue<Task>` ordered by urgency via `Comparable`.

*(Solutions/discussion for these appear inline within [10-Collections/09-Module-Summary-Exercises.md](10-Collections/09-Module-Summary-Exercises.md).)*

---

## Module 11 — Generics

1. **Recall test:** Explain type erasure's complete mechanism from memory, then list the three concrete restrictions it directly causes.
2. **Hands-on:** Write a generic `Stack<T>` (using an internal `ArrayList<T>`) with `push`/`pop`/`peek`, and use it with two type arguments.
3. **Hands-on:** Write a PECS-correct method `static <T> void copyAll(List<? super T> dest, List<? extends T> src)`.
4. **Conceptual:** Explain why generics were designed invariant, referencing array covariance's `ArrayStoreException`.
5. **Conceptual:** Explain why `Box<String>` and `Box<Integer>` report the same `.getClass()` at runtime.
6. **Synthesis:** Design a generic `Result<T>` class with a bounded method requiring `T extends Comparable<T>`, and explain the bound's necessity.

*(Solutions/discussion for these appear inline within [11-Generics/05-Module-Summary-Exercises.md](11-Generics/05-Module-Summary-Exercises.md).)*

---

## Module 12 — Exceptions

1. **Recall test:** Draw the `Throwable` hierarchy, then state the mechanical and philosophical differences between checked and unchecked exceptions.
2. **Hands-on:** Compare `try { return 1; } finally { println("x"); }` vs `try { return 1; } finally { return 2; }` — predict and verify both outputs.
3. **Hands-on:** Implement an `AutoCloseable` whose `close()` throws, combined with a `try` block that also throws — print the primary message and every suppressed exception.
4. **Hands-on:** Design a custom `RuntimeException` with a structured data field, chained from a lower-level exception.
5. **Conceptual:** Explain what could go wrong with `catch (Exception e) { }` "just to be safe."
6. **Synthesis:** Write a file-processing method using try-with-resources that validates content and chains any `IOException` into a custom exception.

*(Solutions/discussion for these appear inline within [12-Exceptions/06-Module-Summary-Exercises.md](12-Exceptions/06-Module-Summary-Exercises.md).)*

---

## Module 13 — IO

1. **Recall test:** Draw the byte-stream and character-stream hierarchies, and explain why they're kept separate.
2. **Hands-on:** Copy a text file using `Files.exists`, `BufferedReader`/`BufferedWriter`, explicit UTF-8, reading/writing line by line.
3. **Hands-on:** Benchmark copying a large file with unbuffered vs. buffered streams and observe the difference.
4. **Hands-on:** Serialize/deserialize a class with one `transient` field, verifying it's default/null after deserialization.
5. **Conceptual:** Explain why accepting user-uploaded serialized Java objects for deserialization is a serious security concern.
6. **Synthesis:** Use `Files.walk` to find `.log` files in a directory tree, then print each file's first line via buffered character streams.

*(Solutions/discussion for these appear inline within [13-IO/06-Module-Summary-Exercises.md](13-IO/06-Module-Summary-Exercises.md).)*

---

## Module 14 — NIO

1. **Recall test:** Explain what `flip()`, `clear()`, `rewind()`, and `compact()` each do to a buffer's position/limit.
2. **Hands-on:** Read a small file's contents using `FileChannel`/`ByteBuffer`, correctly using `flip()`.
3. **Hands-on:** Set up a `WatchService` on a directory and observe create/modify/delete events, remembering `key.reset()`.
4. **Conceptual:** Calculate roughly how much stack memory 10,000 thread-per-connection threads would consume, and explain how this motivated NIO.
5. **Conceptual:** Explain why memory-mapped files suit a large database index but not a small, frequently-rewritten config file.
6. **Synthesis:** For each of: a 2KB config file, a 10GB video copy, a 50,000-connection chat server, and random access into a 20GB data file — state which specific tool you'd use and why.

*(Solutions/discussion for these appear inline within [14-NIO/05-Module-Summary-Exercises.md](14-NIO/05-Module-Summary-Exercises.md).)*

---

## Module 15 — Concurrency

1. **Recall test:** Explain the difference between a race condition and a visibility problem, and which tools fix each.
2. **Hands-on:** Reproduce the `count++` race condition with two threads, then fix it three ways: `synchronized`, `ReentrantLock`, `AtomicInteger`.
3. **Hands-on:** Implement producer-consumer both manually (`wait`/`notifyAll`) and with `BlockingQueue`; compare.
4. **Hands-on:** Submit 100,000 short blocking tasks via `Executors.newVirtualThreadPerTaskExecutor()` and observe it completes without issue.
5. **Conceptual:** Explain what the JVM does transparently underneath Virtual Threads to let simple blocking code scale.
6. **Synthesis:** Fetch data from two simulated APIs concurrently via `CompletableFuture.thenCombine`, then rewrite using Structured Concurrency.

*(Solutions/discussion for these appear inline within [15-Concurrency/09-Module-Summary-Exercises.md](15-Concurrency/09-Module-Summary-Exercises.md).)*

---

## Module 16 — JVM Internals

1. **Recall test:** Explain what `invokevirtual` does and why it confirms Module 05's polymorphism at the bytecode level.
2. **Hands-on:** Compile a class with an overridden method, a static method, and a private method; use `javap -c` to compare `invokevirtual`/`invokestatic`/`invokespecial`.
3. **Hands-on:** Write a custom `@Retention(RUNTIME)` annotation and Reflection-based code that discovers and invokes annotated methods (a mini JUnit).
4. **Hands-on:** Implement a Dynamic Proxy logging every method call's name and timing.
5. **Conceptual:** Choose and justify a GC for: a small CLI tool, a latency-critical trading system, a high-throughput batch job.
6. **Synthesis:** Explain how a minimal DI framework could use `@Inject` + Reflection to wire two classes together automatically.

*(Solutions/discussion for these appear inline within [16-JVM-Internals/07-Module-Summary-Exercises.md](16-JVM-Internals/07-Module-Summary-Exercises.md).)*

---

## Module 17 — Functional Programming

1. **Recall test:** State the "exactly one abstract method" rule and explain why it makes lambdas possible within Java's static type system.
2. **Hands-on:** Implement a custom `@FunctionalInterface` three ways: anonymous class, lambda, method reference (if applicable) — compare.
3. **Hands-on:** Write a method returning `Optional<User>` and a caller using `.map(...).filter(...).orElseThrow(...)`.
4. **Hands-on:** Compare `Predicate<Integer>` vs `IntPredicate` in a tight loop, explaining the autoboxing difference.
5. **Conceptual:** Explain why a lambda capturing a later-reassigned local variable fails to compile, tracing back to Module 02's Stack model.
6. **Synthesis:** Build a validation pipeline combining three named `Predicate<String>`s with `.and()`/`.or()`/`.negate()`.

*(Solutions/discussion for these appear inline within [17-Functional-Programming/06-Module-Summary-Exercises.md](17-Functional-Programming/06-Module-Summary-Exercises.md).)*

---

## Module 18 — Streams

1. **Recall test:** Draw the source → intermediate → terminal pipeline model, marking what's lazy vs. what triggers execution.
2. **Hands-on:** Filter orders over $100, group by customer, and sum each customer's total using `groupingBy` + a downstream collector.
3. **Hands-on:** Use `flatMap` to flatten a `List<List<Integer>>` into a single sorted, deduplicated list.
4. **Hands-on:** Trigger `Collectors.toMap`'s duplicate-key exception, then fix it with a merge function.
5. **Conceptual:** Explain why a parallel stream over 10 elements with cheap work is likely slower than sequential.
6. **Synthesis:** Process a `List<Employee>`: filter active, group by department, compute count and average salary per group.

*(Solutions/discussion for these appear inline within [18-Streams/06-Module-Summary-Exercises.md](18-Streams/06-Module-Summary-Exercises.md).)*

---

## Module 19 — Networking

1. **Recall test:** Explain why socket I/O requires no new API knowledge beyond Module 13.
2. **Hands-on:** Build a multi-client echo server using `Executors.newVirtualThreadPerTaskExecutor()` and test with several simultaneous clients.
3. **Hands-on:** Fetch a public API endpoint synchronously with `HttpClient`, printing status code and body.
4. **Hands-on:** Rewrite the same request using `sendAsync(...)` with `.thenApply(...).thenAccept(...)`.
5. **Conceptual:** Explain why `HttpClient.sendAsync` returning `CompletableFuture` specifically was a deliberate design choice.
6. **Synthesis:** Fetch two independent pieces of data concurrently via `sendAsync` + `thenCombine`, comparing latency to sequential `send(...)` calls.

*(Solutions/discussion for these appear inline within [19-Networking/04-Module-Summary-Exercises.md](19-Networking/04-Module-Summary-Exercises.md).)*

---

## Module 20 — JDBC

1. **Recall test:** Explain why `PreparedStatement` structurally prevents SQL injection, not just "escapes" input.
2. **Hands-on:** Write vulnerable `Statement`-based login code, then rewrite it correctly with `PreparedStatement`.
3. **Hands-on:** Write a fund-transfer transaction using `commit`/`rollback` around a try/catch.
4. **Hands-on:** Iterate a `ResultSet` from a `SELECT`, printing each row.
5. **Conceptual:** Explain why connection pooling exists, paralleling it to Module 15's thread-pool reasoning.
6. **Synthesis:** Design a `UserRepository` with a `findById` (PreparedStatement) and `transferFunds` (transaction) method.

*(Solutions/discussion for these appear inline within [20-JDBC/04-Module-Summary-Exercises.md](20-JDBC/04-Module-Summary-Exercises.md).)*

---

## Module 21 — Modules (JPMS)

1. **Recall test:** Explain the difference between `public` and `exports`, with a concrete example of a `public` class that's still externally inaccessible.
2. **Hands-on:** Write two modules: an API module exporting an interface, and an implementation module providing it via `provides`/`uses`.
3. **Hands-on:** Run `jlink --list-modules` and speculate on what a few of the listed modules contain.
4. **Conceptual:** Explain why a Jackson-based application needs `opens`, not just `exports`, on its data model package.
5. **Conceptual:** Explain why the JDK's own modularization succeeded even though broader ecosystem adoption has been more limited.
6. **Synthesis:** Design a `module-info.java` for a hypothetical blog app requiring `java.sql` + a third-party automatic module, exporting its API, and opening its entity package.

*(Solutions/discussion for these appear inline within [21-Modules/04-Module-Summary-Exercises.md](21-Modules/04-Module-Summary-Exercises.md).)*

---

## Module 22 — Performance

1. **Recall test:** Explain why a naive `System.nanoTime()` benchmark is invalid, referencing JIT warm-up precisely.
2. **Hands-on:** Run a small program with `-XX:+PrintCompilation` alongside a naive timing loop, observing JIT compilation mid-loop.
3. **Hands-on:** Rewrite a loop-based String-concatenation method using `StringBuilder`.
4. **Conceptual:** Choose an appropriate GC for: a nightly batch job, a real-time bidding service, a small CLI tool.
5. **Conceptual:** Explain escape analysis and why it doesn't contradict the Stack/Heap model from Module 02.
6. **Synthesis:** Design a performance investigation plan for a "the API is slow" production report.

*(Solutions/discussion for these appear inline within [22-Performance/04-Module-Summary-Exercises.md](22-Performance/04-Module-Summary-Exercises.md).)*

---

## Module 23 — Modern Java

1. **Recall test:** Explain why records auto-generating `equals()`/`hashCode()` eliminates an entire category of bug from hand-written implementations.
2. **Hands-on:** Model a `PaymentMethod` sealed interface with `CreditCard`, `BankTransfer`, and `Cash` records, then write an exhaustive `switch` using record patterns.
3. **Hands-on:** Rewrite a classic `instanceof`-then-cast chain as a `switch` with type patterns and a `when` guard.
4. **Conceptual:** Explain why sealed classes combined with exhaustive `switch` turn a silent runtime risk into a compile-time guarantee.
5. **Conceptual:** Explain why `ScopedValue` is a better fit than `ThreadLocal` for a Structured-Concurrency-based service handling millions of Virtual Threads.
6. **Synthesis:** Design an order lifecycle using records for data and a sealed hierarchy for its states, then write an exhaustive `switch` that fails to compile if a new state is added without updating it.

*(Solutions/discussion for these appear inline within [23-Modern-Java/05-Module-Summary-Exercises.md](23-Modern-Java/05-Module-Summary-Exercises.md).)*

---

## Module 24 — Interview Preparation

1. **Recall test:** List all six live-coding problems from Topic 2 and, for each, state the Java concept it's designed to probe.
2. **Timed practice:** Answer ten questions from Topic 1, chosen at random, each in under 45 seconds, out loud.
3. **Full mock:** Recreate Topic 4's three-question mock transcript from a cold start, narrating the entire time.
4. **Synthesis:** Pick two of the hardest topics from across the whole course and prepare a 60-second, interview-ready explanation of each from memory.

*(Solutions/discussion for these appear inline within [24-Interview-Preparation/05-Module-Summary-And-Course-Conclusion.md](24-Interview-Preparation/05-Module-Summary-And-Course-Conclusion.md).)*

---

*(This is the final module. The course is complete.)*
