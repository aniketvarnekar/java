# Synchronization & Locks

## Learning Objectives

- Use the `synchronized` keyword correctly, on both methods and blocks
- Understand the intrinsic lock/monitor mechanism precisely
- Use `wait()`/`notify()`/`notifyAll()` correctly — finally delivering Module 07, Topic 1's preview in full
- Compare `synchronized` with `java.util.concurrent.locks.Lock`/`ReentrantLock`
- Understand deadlock and how to avoid it

## Prerequisites

[02 — Race Conditions & the Java Memory Model](02-race-conditions-and-the-java-memory-model.md), Module 07 Topic 1 (`wait`/`notify` preview)

## Motivation

Topic 2 established two hazards; this topic delivers Java's classical, foundational solution to both simultaneously. This is also where Module 07, Topic 1's promise — "every object has a built-in monitor lock, which is why `wait`/`notify` live on `Object`" — finally gets its complete explanation.

## `synchronized` — Mutual Exclusion, Fixing Both Hazards at Once

```java
class Counter {
    private int count = 0;

    public synchronized void increment() {   // SYNCHRONIZED method
        count++;    // now genuinely thread-safe -- no interleaving possible
    }

    public synchronized int getCount() {
        return count;
    }
}
```

**`synchronized` ensures only ONE thread at a time can execute a synchronized method (or block) on the same object** — any other thread attempting to enter is **blocked**, waiting, until the first thread exits. This directly, completely fixes Topic 2's race condition: `count++`'s four bytecode instructions can no longer interleave with another thread's, since no other thread can even **begin** executing `increment()` until the current one finishes entirely.

**It also fixes the visibility problem simultaneously**, because — recall Topic 2's `happens-before` list — **unlocking** a `synchronized` block happens-before a subsequent **locking** of that same lock. This is precisely why `synchronized` is so often the default, safe choice: **one mechanism addressing both of Topic 2's hazards**, rather than needing `volatile` plus something else entirely for atomicity.

## The Intrinsic Lock (Monitor) — What's Actually Happening

**Every single Java object has a built-in, hidden lock associated with it — called its "intrinsic lock" or "monitor."** This directly, finally explains Module 07, Topic 1's flag: `wait()`/`notify()`/`notifyAll()` exist on `Object` **precisely because every object has this monitor**, and those methods operate on it.

```
 synchronized void increment() {  ...  }
       is EXACTLY equivalent to:
 void increment() {
     synchronized (this) {   // acquires THIS object's intrinsic lock
         ...
     }
 }                             // lock automatically released here, even if an exception is thrown
```

```java
class BankAccount {
    private double balance;
    private final Object lock = new Object();   // a DEDICATED lock object (a common, deliberate pattern)

    void withdraw(double amount) {
        synchronized (lock) {   // SYNCHRONIZED BLOCK -- locks on 'lock', not 'this'
            balance -= amount;
        }
    }
}
```

**Why sometimes lock on a dedicated, private object rather than `this`?** Locking on `this` exposes your object's lock to **any** external code holding a reference to it — external code could (accidentally or maliciously) `synchronized(myBankAccount) { ... }` on your object too, creating unexpected contention or even deadlock risk (below) that your class has no control over. A **private, dedicated lock object** keeps the locking entirely internal and under your class's own control — a genuine, real best practice for library/API code.

**Static synchronized methods lock on the `Class` object itself** (recall Module 07, Topic 1's `getClass()` — the same `Class` object), not on any particular instance — since `static` methods aren't tied to any specific object (Module 06, Topic 4).

## `wait()`/`notify()`/`notifyAll()` — Full Depth, At Last

These methods let a thread holding a monitor **temporarily release it and pause**, until another thread signals it to wake up — used for coordinating threads around a shared condition (a classic "producer-consumer" pattern):

```java
class Buffer {
    private final Object lock = new Object();
    private int data;
    private boolean available = false;

    void produce(int value) {
        synchronized (lock) {
            while (available) {         // ⚠️ ALWAYS use a while loop, never an if -- see below
                lock.wait();               // RELEASES the lock, pauses THIS thread, until notified
            }
            data = value;
            available = true;
            lock.notifyAll();               // wakes up any thread(s) waiting on THIS SAME lock
        }
    }

    int consume() {
        synchronized (lock) {
            while (!available) {
                lock.wait();
            }
            available = false;
            lock.notifyAll();
            return data;
        }
    }
}
```

**Critical rule: `wait()` must ALWAYS be called inside a `while` loop re-checking its condition, never a plain `if`.** Why? When a waiting thread is woken by `notify()`/`notifyAll()`, it doesn't **immediately** resume — it must **re-acquire the lock** first, and by the time it actually does, **some other thread might have already changed the condition again** (a classic real race, since `notifyAll()` can wake multiple waiters, only one of which should actually proceed). Re-checking the condition in a `while` loop after waking up is what makes this pattern genuinely correct — this is a well-known, real, important idiom, not a stylistic preference.

**`wait()`/`notify()`/`notifyAll()` must be called from within a `synchronized` block holding the relevant lock** — calling them without holding the lock throws `IllegalMonitorStateException` at runtime.

## `Lock`/`ReentrantLock` — A More Flexible Alternative

```java
import java.util.concurrent.locks.ReentrantLock;

class Counter {
    private int count = 0;
    private final ReentrantLock lock = new ReentrantLock();

    void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();   // MUST be in finally -- Module 12, Topic 3's guarantee, applied here!
        }
    }
}
```

**`ReentrantLock` (from `java.util.concurrent.locks`, Java 5+) offers capabilities `synchronized` fundamentally cannot**:

| Capability | `synchronized` | `ReentrantLock` |
|---|---|---|
| Basic mutual exclusion | Yes | Yes |
| Automatically released (even on exception) | **Yes**, automatically | **No** — must manually `unlock()` in `finally` |
| Can attempt to acquire without blocking forever | No | **Yes** — `tryLock()` |
| Can be interrupted while waiting | No | **Yes** — `lockInterruptibly()` |
| Supports fairness (first-come-first-served waiting) | No | **Yes** — optional fair-ordering constructor |
| Multiple independent wait conditions per lock | No (only one implicit condition) | **Yes** — `newCondition()` for multiple, separate conditions |

**Why does `synchronized` remain the common default despite `ReentrantLock`'s greater flexibility?** `synchronized`'s automatic lock release (even on an exception propagating out) is a genuine, real safety advantage — recall Module 12's `finally` guarantee — `synchronized` gets that guarantee **built in**, automatically, while `ReentrantLock` requires you to **remember** the `try`/`finally` pattern yourself, every single time, and forgetting `unlock()` in a `finally` block is a genuine, real bug risk. **Reach for `ReentrantLock` specifically when you need one of its extra capabilities** (`tryLock`, fairness, multiple conditions) — otherwise, `synchronized`'s simplicity and automatic safety make it the sensible default.

## Deadlock — A Genuine, Real Danger of Locking

```java
// Thread 1:                          // Thread 2:
synchronized (lockA) {                 synchronized (lockB) {
    synchronized (lockB) {                 synchronized (lockA) {   // ⚠️ OPPOSITE ORDER!
        // ...                                 // ...
    }                                       }
}                                       }
```

**If Thread 1 acquires `lockA` and Thread 2 acquires `lockB` at nearly the same moment, each then tries to acquire the *other* lock — and neither can ever proceed, since each is waiting for a lock the other thread is holding, forever.** This is called **deadlock** — a genuine, real, and historically common concurrency bug.

**The standard, reliable prevention technique: always acquire multiple locks in a single, globally consistent order**, throughout your entire codebase (e.g., always lock `lockA` before `lockB`, everywhere, never the reverse) — this makes the circular-waiting scenario shown above structurally impossible, since no thread can ever be holding `lockB` while waiting for `lockA` if every thread always acquires them in the same order.

## Real-World Analogy

Think of `synchronized` like a **single-occupancy restroom with a lock on the door** — only one person can be inside at a time; anyone else who arrives simply waits outside until the door is unlocked (Topic 2's `happens-before` guarantee: whatever the previous occupant did inside is now safely "flushed" and visible to the next person entering). Think of `wait()`/`notify()` like **a barista calling out "order #42 ready!"** — customers (`wait()`-ing threads) who don't hear their specific number called simply keep waiting, but importantly, they don't just trust the first call blindly — they check their own ticket number again (the `while` loop re-check) before actually stepping up, in case multiple calls happened and someone else's order was actually the one just announced. **Deadlock** is like **two people, each holding one door of a two-door turnstile the other person needs to pass, each waiting for the other to move first** — neither can ever proceed, forever, unless there's an agreed-upon rule (a consistent lock-acquisition order) preventing this standoff from arising in the first place.

## Advantages

- `synchronized` provides a single, simple mechanism fixing both Topic 2 hazards (race conditions and visibility) at once, with automatic, exception-safe lock release.
- `ReentrantLock` provides genuine additional flexibility (`tryLock`, fairness, multiple conditions) for scenarios `synchronized` structurally cannot handle.
- Consistent lock-ordering discipline makes deadlock structurally preventable, not just "hopefully avoided."

## Disadvantages / Trade-offs

- Locking (either mechanism) introduces genuine contention — threads waiting for a lock make no progress, a real performance cost under high concurrency (motivating Topic 4's lock-free alternatives).
- `ReentrantLock`'s manual `unlock()` requirement is a genuine, real bug risk if the `try`/`finally` discipline isn't followed rigorously.
- Deadlock remains a real, serious risk whenever multiple locks are acquired without consistent ordering discipline.

## Best Practices

- Default to `synchronized` for straightforward mutual exclusion; reach for `ReentrantLock` specifically when you need its extra capabilities.
- Always call `lock.unlock()` inside a `finally` block when using `ReentrantLock`.
- Always call `wait()` inside a `while` loop re-checking the condition, never a plain `if`.
- Maintain a consistent, global lock-acquisition ordering across your codebase to prevent deadlock structurally.
- Prefer a small, private, dedicated lock object over locking on `this`, for library/API code specifically.

## Common Mistakes

- Calling `wait()` inside an `if` instead of a `while`, missing the required re-check after waking up.
- Forgetting to `unlock()` a `ReentrantLock` in a `finally` block, leaving it permanently held if an exception occurs.
- Acquiring multiple locks in inconsistent order across different code paths, risking deadlock.
- Calling `wait()`/`notify()` without holding the relevant lock, triggering `IllegalMonitorStateException`.

## Interview Questions

1. **Q: What does `synchronized` actually do, mechanically?**
   A: It acquires the target object's (or, for static methods, the `Class` object's) intrinsic lock/monitor before entering the block/method, ensuring only one thread can hold that lock at a time, and automatically releases it on exit (even via an exception) — fixing both the race-condition hazard (mutual exclusion) and the visibility hazard (via the happens-before guarantee on unlock/lock).

2. **Q: Why must `wait()` always be called inside a `while` loop, not an `if`?**
   A: A woken thread must re-acquire the lock before resuming, and by that time another thread may have already changed the condition again (especially relevant since `notifyAll()` can wake multiple waiters). Re-checking the condition after waking, in a loop, is what makes the pattern correct.

3. **Q: What can `ReentrantLock` do that `synchronized` cannot?**
   A: Attempt lock acquisition without blocking forever (`tryLock`), support interruptible waiting, provide fairness ordering, and support multiple independent wait conditions per lock (`newCondition()`) — capabilities `synchronized`'s simpler, built-in model doesn't offer.

4. **Q: What causes deadlock, and how is it prevented?**
   A: Two (or more) threads each holding a lock the other needs, waiting on each other indefinitely — typically from acquiring multiple locks in inconsistent order across different code paths. Prevented by always acquiring locks in a single, globally consistent order throughout the codebase.

## Summary

- **`synchronized`** provides mutual exclusion via an object's intrinsic lock/monitor, automatically released even on exception, fixing both race conditions and visibility problems (Topic 2) simultaneously.
- **`wait()`/`notify()`/`notifyAll()`** (on `Object`, Module 07, Topic 1) coordinate threads around a shared condition — `wait()` must always be re-checked in a `while` loop after waking.
- **`ReentrantLock`** offers greater flexibility (`tryLock`, fairness, multiple conditions) at the cost of manual, `finally`-block-required unlocking.
- **Deadlock** arises from inconsistent multi-lock acquisition order; prevented by enforcing a single, consistent ordering throughout the codebase.