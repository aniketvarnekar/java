# Coding Problems & Live Coding

## Learning Objectives

- Solve the specific live-coding problems that recur constantly across Java interviews
- Explain the reasoning behind each solution out loud, not just produce working code
- Recognize the specific course concepts (immutability, the JMM, hashing) each problem is designed to test

## Prerequisites

Module 06 (Classes), Module 07 (Objects), Module 08 (Strings), Module 10 (Collections), Module 15 (Concurrency)

## Motivation

Live coding tests something conceptual questions (Topic 1) can't: can you actually **produce correct code**, under mild time pressure, while narrating your thinking? Most Java live-coding problems aren't algorithmically exotic — they're deliberately chosen because they probe a *specific* piece of course knowledge (does this candidate actually understand `String` immutability, the `equals`/`hashCode` contract, thread safety) through a small, concrete task. This topic walks through the problems that come up constantly, with the reasoning an interviewer is specifically listening for.

## Problem 1: Thread-Safe Singleton

**What it tests:** lazy initialization, the double-checked locking pattern, and the JMM (`volatile`)'s role in making it actually correct (Module 15, Topic 2).

```java
public class Singleton {
    private static volatile Singleton instance;   // volatile is NOT optional -- see below

    private Singleton() { }

    public static Singleton getInstance() {
        if (instance == null) {                     // 1st check -- avoids locking on the common path
            synchronized (Singleton.class) {
                if (instance == null) {                // 2nd check -- another thread may have
                    instance = new Singleton();          // won the race while we waited for the lock
                }
            }
        }
        return instance;
    }
}
```

**Say this out loud:** "The double check avoids paying the `synchronized` cost on every call once the instance exists. But `volatile` is essential, not optional: `new Singleton()` isn't a single atomic step — it involves allocating memory, running the constructor, then assigning the reference. Without `volatile`, the JMM (Module 15, Topic 2) permits those steps to be reordered from another thread's point of view, so a second thread could see a non-null `instance` reference that points to a **partially constructed** object. `volatile` forbids that reordering."

**The simpler, usually-better alternative — the initialization-on-demand holder:**

```java
public class Singleton {
    private Singleton() { }

    private static class Holder {                 // not loaded until first referenced
        static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;                     // class loading itself is thread-safe (JLS guarantee)
    }
}
```

**Say this out loud:** "This relies on the JVM's guarantee that a class is initialized lazily and thread-safely exactly once (Module 02, Topic 2's class-loading discussion) — no explicit locking or `volatile` needed at all. I'd reach for this over double-checked locking in real code; it's simpler and equally correct."

## Problem 2: Detect a Cycle in a Linked List

**What it tests:** the two-pointer technique, and — often as a targeted follow-up — the `equals`/`hashCode` alternative, to probe Module 07's understanding.

```java
boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;              // moves 1 step
        fast = fast.next.next;           // moves 2 steps
        if (slow == fast) return true;     // they WILL meet inside a cycle, eventually
    }
    return false;                            // fast reached the end -- no cycle
}
```

**Say this out loud:** "This is Floyd's cycle detection — O(1) space. If there's a cycle, the fast pointer eventually laps the slow one inside it; if there isn't, `fast` simply reaches `null`. An interviewer might ask for a `HashSet`-based alternative next — that's O(n) space but conceptually simpler: walk the list, and if you ever revisit a node already in the set (identity, `==`, not `.equals()` — Module 07, Topic 3 — since we want the exact same node object), you've found the cycle."

## Problem 3: LRU Cache

**What it tests:** knowing the right built-in tool (`LinkedHashMap`'s access-order mode) rather than reinventing a doubly-linked list + hash map by hand — and, as a follow-up, being able to explain how you *would* build it by hand.

```java
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    LRUCache(int capacity) {
        super(16, 0.75f, true);        // accessOrder=true -- Module 10's LinkedHashMap ordering mode
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;        // hook LinkedHashMap calls after every put()
    }
}
```

**Say this out loud:** "`LinkedHashMap` with `accessOrder=true` (Module 10) already maintains entries in **access order**, not just insertion order — every `get()` moves an entry to the most-recently-used end. `removeEldestEntry` is a protected hook it calls after every insertion, letting me evict the least-recently-used entry in O(1) with almost no code. If asked to implement this *without* `LinkedHashMap`, I'd use a `HashMap<K, Node>` for O(1) lookup combined with a manual doubly-linked list for O(1) move-to-front/eviction — exactly what `LinkedHashMap` does internally."

## Problem 4: Producer-Consumer

**What it tests:** classic `wait()`/`notify()` coordination (Module 15) *and* recognizing that a `BlockingQueue` almost always makes it unnecessary in real code.

```java
class BoundedBuffer<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;

    BoundedBuffer(int capacity) { this.capacity = capacity; }

    synchronized void put(T item) throws InterruptedException {
        while (queue.size() == capacity) wait();   // while, not if -- guards against spurious wakeup
        queue.add(item);
        notifyAll();                                  // wake any waiting consumers
    }

    synchronized T take() throws InterruptedException {
        while (queue.isEmpty()) wait();               // while, not if -- same reason
        T item = queue.poll();
        notifyAll();                                    // wake any waiting producers
        return item;
    }
}
```

**Say this out loud:** "The `while` loop instead of `if` around `wait()` is deliberate — `wait()` can return without actually being notified (a spurious wakeup, a JLS-permitted possibility), so the condition must always be re-checked, not assumed true after waking. In real production code, though, I'd almost always just use `java.util.concurrent.BlockingQueue` (Module 15, Topic 7) — `put()`/`take()` block automatically and correctly, with none of this hand-written coordination logic or its risk of subtle bugs."

## Problem 5: Implement a Correct `equals()`/`hashCode()` Pair

**What it tests:** the exact contract from Module 07, Topic 3 — this comes up constantly, precisely because it's easy to get subtly wrong.

```java
public final class Point {
    private final int x, y;

    public Point(int x, int y) { this.x = x; this.y = y; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;               // fast path
        if (!(o instanceof Point p)) return false;  // handles null AND wrong type in one check
        return x == p.x && y == p.y;
    }

    @Override
    public int hashCode() {
        return Objects.hash(x, y);                  // combines both fields consistently with equals()
    }
}
```

**Say this out loud:** "`instanceof` returns `false` for `null` automatically, so this one check safely handles both the null case and the wrong-type case without a separate `null` check — a common shortcut interviewers like seeing. The critical rule (Module 07, Topic 3): `hashCode()` must use **exactly the same fields** `equals()` compares — if I add a field to one, I must add it to the other, or I risk violating the contract and silently breaking `HashMap`/`HashSet` behavior."

## Problem 6: An Immutable Class With a Mutable Field

**What it tests:** whether "immutable" was actually understood, not just memorized as "add `final`" (Module 08's String immutability, applied to a custom class).

```java
public final class Event {                              // final -- no subclass can add mutability
    private final String name;
    private final Date timestamp;                          // Date is MUTABLE -- the trap

    public Event(String name, Date timestamp) {
        this.name = name;
        this.timestamp = new Date(timestamp.getTime());       // DEFENSIVE COPY on the way in
    }

    public String getName() { return name; }
    public Date getTimestamp() {
        return new Date(timestamp.getTime());                   // DEFENSIVE COPY on the way out
    }
}
```

**Say this out loud:** "Marking a field `final` only prevents *reassigning the reference* — it does nothing to stop the object it points to from being mutated internally. `Date` is mutable, so without defensive copies both in the constructor and in the getter, a caller could grab the `Date` reference and call `.setTime()` on it, silently mutating supposedly-immutable state from outside the class. Both copies are required — one isn't enough." *(If asked for the modern fix: "In new code, I'd just use `java.time.Instant`, an actually-immutable type, and this entire problem disappears.")*

## Real-World Analogy

A conceptual question (Topic 1) is like being asked to describe how a car engine works. A live-coding problem is like being handed a wrench and asked to actually fix one, while someone watches and asks why you're using that wrench — it tests the same knowledge, but through demonstrated, narrated action instead of description alone.

## Best Practices

- Narrate your thinking *before* and *while* you type — silence during live coding reads as uncertainty even when the final code is correct.
- State the time/space complexity of your solution unprompted once finished — it signals you're thinking beyond "does it merely compile."
- If you know a simpler built-in alternative exists (like `LinkedHashMap` for Problem 3, or `BlockingQueue` for Problem 4), mention it even after hand-rolling the manual version — it shows you know both the mechanism *and* the pragmatic real-world choice.

## Common Mistakes

- Writing code silently for several minutes without any narration, then presenting a finished block — interviewers lose the ability to follow your reasoning or intervene if you're heading down a wrong path.
- Forgetting the `while` (not `if`) around `wait()` in producer-consumer code — a textbook detail interviewers specifically probe for.
- Marking a class `final` with `final` fields and calling it "fully immutable" without checking whether any field's *type* is itself mutable (Problem 6) — one of the most common subtle mistakes in this exact family of question.

## Interview Questions

1. **Q: Why is `volatile` required, not optional, in double-checked-locking singleton implementations?**
   A: Object construction isn't a single atomic step; without `volatile`'s JMM ordering guarantee, another thread could observe a non-null but only partially-constructed instance reference due to reordering.

2. **Q: Why must `wait()` always be called inside a `while` loop checking the condition, not an `if`?**
   A: `wait()` can return due to a spurious wakeup without an actual `notify()` having occurred (a JLS-permitted possibility) — the condition must be re-checked after waking, not assumed true.

3. **Q: Why does marking every field `final` not guarantee a class is truly immutable?**
   A: `final` only prevents reassigning a field's reference — if a field's type is itself mutable (like `Date`), the referenced object can still be mutated internally unless defensive copies are made on both construction and access.

## Summary

- Live-coding problems are chosen to probe **specific** course concepts through small, concrete tasks — recognizing which concept is being tested is often more valuable than the algorithm itself.
- Narrating your reasoning throughout, not just producing correct final code, is what live coding actually evaluates.
- Six recurring problems covered here: thread-safe Singleton, cycle detection, LRU cache, producer-consumer, a correct `equals()`/`hashCode()` pair, and a genuinely immutable class with a mutable-typed field.