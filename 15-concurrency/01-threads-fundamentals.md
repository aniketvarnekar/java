# Threads Fundamentals

## Learning Objectives

- Distinguish a process from a thread precisely
- Create and start threads using both approaches Java provides
- Understand the complete thread lifecycle

## Prerequisites

Module 02 Topic 3 (Runtime Data Areas — per-thread JVM Stack), Module 05 Topic 6 (Interfaces)

## Motivation

Every concurrency tool in this module ultimately manages threads underneath. This topic builds the foundational vocabulary and mechanics — what a thread actually is, precisely how it relates to (and differs from) a process, and how the JVM Stack model from Module 02 was quietly describing per-thread behavior all along.

## Process vs. Thread

> A **process** is an independent, running instance of a program, with its **own** private memory space — one process cannot directly access another's memory.
>
> A **thread** is a unit of execution **within** a process — multiple threads in the same process **share** that process's memory (the Heap, Module 02, Topic 3), but each has its **own** private JVM Stack, PC Register, and Native Method Stack.

```
                         ONE PROCESS (one JVM instance)

┌──────────────────────────────────────────────────────────────────────────────┐
│ SHARED: Heap, Method Area / Metaspace (Module 02)                            │
│                                                                              │
│   Thread A               Thread B               Thread C                     │
│  ┌──────────────┐       ┌──────────────┐       ┌──────────────┐              │
│  │ own Stack    │       │ own Stack    │       │ own Stack    │              │
│  │ own PC Reg   │       │ own PC Reg   │       │ own PC Reg   │              │
│  └──────────────┘       └──────────────┘       └──────────────┘              │
│                                                                              │
└──────────────────────────────────────────────────────────────────────────────┘
```

**This diagram is exactly Module 02, Topic 3's Runtime Data Areas picture** — that entire "per-thread vs. shared" split was built specifically to support multithreading, and you've had the complete memory-model foundation for this module since Module 02.

**Why does sharing the Heap matter so much?** It's simultaneously multithreading's greatest strength and its greatest danger: threads can **cooperate efficiently** by directly sharing objects (no need to copy data between them, unlike separate processes) — but that same shared, mutable state is exactly what creates the genuine hazards this module spends most of its time addressing (Topic 2).

## Creating Threads — Two Approaches

### Approach 1: Extending `Thread`

```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Running in: " + Thread.currentThread().getName());
    }
}

MyThread t = new MyThread();
t.start();   // NOT t.run()!! -- see below
```

### Approach 2: Implementing `Runnable` (Preferred)

```java
class MyTask implements Runnable {
    @Override
    public void run() {
        System.out.println("Running in: " + Thread.currentThread().getName());
    }
}

Thread t = new Thread(new MyTask());
t.start();

// Or, since Runnable is a functional interface (Module 10, Topic 7's pattern; full depth Module 17):
Thread t2 = new Thread(() -> System.out.println("Running in a lambda!"));
t2.start();
```

**Why is `Runnable` generally preferred over extending `Thread`?** This is a direct, concrete application of Module 05, Topic 7's "favor composition over inheritance" principle: extending `Thread` consumes your class's single-inheritance slot (Module 05, Topic 4) purely to gain threading behavior, even though your class **IS-A** task to run, not genuinely **IS-A** thread. `Runnable` lets you **compose** a thread with any task (including a task that also needs to extend some other class, which `Thread`-extension would preclude entirely, since Java only allows single inheritance).

## `start()` vs. `run()` — A Critical, Commonly-Tested Distinction

```java
Thread t = new Thread(() -> System.out.println("Hello from: " + Thread.currentThread().getName()));

t.run();      // ⚠️ runs the code, but on the CURRENT thread -- NO new thread is created at all!
t.start();      // CORRECT -- creates a genuine NEW thread, which THEN calls run() on itself
```

**Calling `run()` directly is just an ordinary method call** — it executes the `run()` method's code synchronously, on whichever thread called it, with **zero** actual concurrency involved. **`start()` is what actually asks the JVM (and ultimately the OS) to create a new thread of execution**, which then, on its own, independently invokes `run()`. This is a genuinely common, real beginner mistake — code that calls `run()` instead of `start()` will often "work" in simple tests (since it still executes), while providing **none** of the actual concurrency benefit, silently.

## The Thread Lifecycle

```
                             Thread Lifecycle

        NEW                  RUNNABLE               BLOCKED / WAITING /          TERMINATED
   (created, not         (eligible to run,            TIMED_WAITING             (run() has
    yet started)          OS scheduler decides       (waiting for a lock,       completed, or
                           WHEN it actually runs)     or wait()/join()/          an uncaught
                                                      sleep())                  exception
                                                                                 propagated out)

          │
          │ start()
          ▼
    ┌────────────┐
    │    NEW     │
    └────────────┘
          │
          ▼
    ┌────────────┐      waiting / blocked      ┌────────────────────────┐
    │ RUNNABLE   │ ◀────────────────────────▶ │ BLOCKED / WAITING /     │
    └────────────┘                             │ TIMED_WAITING           │
          │                                   └────────────────────────┘
          │ run() completes /
          │ uncaught exception
          ▼
    ┌──────────────┐
    │ TERMINATED   │
    └──────────────┘
```

- **NEW**: a `Thread` object has been created (`new Thread(...)`) but `start()` hasn't been called yet.
- **RUNNABLE**: the thread is eligible to execute — it might be **actually running** on a CPU core right now, or just **waiting for the OS scheduler's turn** to give it CPU time (Java's `RUNNABLE` state doesn't distinguish between these two — both count as "runnable").
- **BLOCKED/WAITING/TIMED_WAITING**: the thread is paused, waiting for something specific — a lock to become available (`BLOCKED`, Topic 3), an explicit `wait()`/`join()` call with no timeout (`WAITING`), or a timed wait like `sleep(ms)` or `wait(ms)` (`TIMED_WAITING`).
- **TERMINATED**: the thread's `run()` method has completed (normally or via an uncaught exception) — a terminated thread can **never** be restarted; calling `start()` again on it throws `IllegalThreadStateException`.

```java
Thread t = new Thread(() -> System.out.println("running"));
System.out.println(t.getState());   // NEW
t.start();
System.out.println(t.getState());   // RUNNABLE (likely, though timing-dependent)
t.join();                             // WAITS for t to finish (full depth below)
System.out.println(t.getState());   // TERMINATED
```

## `join()` — Waiting for a Thread to Finish

```java
Thread worker = new Thread(() -> {
    // ... some long-running work ...
});
worker.start();
worker.join();   // the CALLING thread BLOCKS here until 'worker' finishes (TERMINATED)
System.out.println("Worker is done, safe to continue");
```

**`join()` is how one thread waits for another to complete** — without it, the calling thread would continue immediately after `start()`, with no guarantee the spawned thread has finished (or even started meaningfully executing) yet. This is a genuinely common, real need: "do this in parallel, but don't proceed with the rest of my logic until it's actually done."

## `Thread.sleep()` — Pausing Execution

```java
Thread.sleep(1000);   // pauses the CURRENT thread for (at least) 1000 milliseconds
```

`sleep()` is a `static` method (Module 06, Topic 4) — it **always** pauses whichever thread called it, never some other thread you might specify — a common beginner assumption to double-check. It throws a **checked** `InterruptedException` (Module 12, Topic 2), since a sleeping thread can be deliberately interrupted by another thread.

## Real-World Analogy

Think of a **process** like an **entire, self-contained restaurant** — its own building, its own kitchen, its own staff, its own inventory, completely separate from any other restaurant. Think of **threads** like **multiple chefs working in that one restaurant's shared kitchen** — they share the same ingredients, the same stoves, the same walk-in fridge (the Heap), but each chef has their own personal workspace/cutting board (their own Stack) that no other chef touches. This shared kitchen is exactly why chefs can collaborate efficiently on the same dish — and exactly why two chefs reaching for the same knife at the same moment (shared, mutable state) is where real problems (Topic 2) can arise.

## Advantages

- Threads enable genuine parallelism (on multi-core hardware) and concurrency (interleaved progress on a single core), letting a program make progress on multiple tasks without needing separate, heavyweight processes.
- Sharing memory directly (the Heap) between threads is far more efficient than the inter-process communication mechanisms separate processes would require.

## Disadvantages / Trade-offs

- Shared, mutable state between threads is exactly what creates race conditions and the entire class of concurrency bugs this module addresses (Topic 2 onward).
- Threads have real, non-trivial creation cost (memory for the Stack, OS scheduling overhead — Module 14, Topic 3's C10K discussion) — a cost Topics 5 and 8 both address, from different angles.

## Best Practices

- Prefer implementing `Runnable` (or a lambda) over extending `Thread`, applying Module 05, Topic 7's composition-over-inheritance principle.
- Always call `start()`, never `run()` directly, when you actually want concurrent execution.
- Use `join()` whenever your program's correctness depends on a spawned thread having genuinely finished before proceeding.

## Common Mistakes

- Calling `run()` instead of `start()`, silently losing all actual concurrency while the code still appears to "work."
- Extending `Thread` unnecessarily, consuming the class's single-inheritance slot for no genuine benefit over composing with `Runnable`.
- Forgetting that a `TERMINATED` thread can never be restarted.

## Interview Questions

1. **Q: What's the difference between a process and a thread?**
   A: A process is an independent program instance with its own private memory space. A thread is a unit of execution within a process, sharing that process's memory (Heap) with other threads in the same process, while maintaining its own private Stack, PC Register, and Native Method Stack (Module 02, Topic 3).

2. **Q: What actually happens if you call `thread.run()` instead of `thread.start()`?**
   A: `run()` executes as an ordinary, synchronous method call on the calling thread — no new thread is created at all, and none of `start()`'s actual concurrency behavior occurs, even though the code still runs and may appear to "work" in casual testing.

3. **Q: Why is implementing `Runnable` generally preferred over extending `Thread`?**
   A: It follows the "favor composition over inheritance" principle (Module 05, Topic 7) — a class implementing `Runnable` genuinely represents a task to be run, not a thread itself, and doesn't consume the class's single-inheritance slot the way extending `Thread` would.

4. **Q: What does `join()` do?**
   A: Blocks the calling thread until the thread it was called on reaches the `TERMINATED` state — used whenever a program's correctness depends on a spawned thread having genuinely completed before proceeding.

## Summary

- A **thread** shares its process's Heap with other threads, while maintaining its own private Stack/PC Register/Native Method Stack — directly extending Module 02's Runtime Data Areas model.
- Threads are created by implementing `Runnable` (preferred, composition-based) or extending `Thread` (consumes single inheritance); always call `start()`, never `run()` directly, for genuine concurrency.
- The thread lifecycle: `NEW` → `RUNNABLE` ⇄ `BLOCKED`/`WAITING`/`TIMED_WAITING` → `TERMINATED` (final, non-restartable).
- `join()` blocks the calling thread until another thread terminates; `sleep()` pauses the calling thread for a specified duration.