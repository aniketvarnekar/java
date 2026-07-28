# System-Design-Adjacent Java Questions

## Learning Objectives

- Reason through open-ended Java design questions that have no single "correct" answer
- Structure a judgment-call answer the way an interviewer wants to see: state assumptions, propose a design, name the trade-offs, then defend the choice
- Apply Modules 10, 15, 16, 20, and 22's knowledge to realistic system components, not just isolated syntax

## Prerequisites

Module 10 (Collections), Module 15 (Concurrency), Module 16 (JVM Internals), Module 20 (JDBC), Module 22 (Performance)

## Motivation

Topics 1 and 2 tested knowledge and mechanical skill. This topic tests **judgment** — the thing senior-level Java interviews increasingly probe for. These questions are deliberately open-ended; there is rarely one right answer, and an interviewer is evaluating whether you can **reason explicitly about trade-offs**, not whether you land on a specific "correct" design. The single most important habit for this entire topic: **state your assumptions and constraints out loud before proposing a design** — jumping straight to a solution without doing so is the most common way strong technical knowledge fails to land well in this style of question.

## Question 1: Design a Thread-Safe, Expiring In-Memory Cache

**A weak opening:** immediately writing `ConcurrentHashMap<K, V>` and calling it done.

**A strong opening:** "First, a few questions: what's the expected read/write ratio? Do entries need a fixed TTL, or LRU-style eviction under memory pressure, or both? Is single-JVM-instance sufficient, or does this need to be distributed?" — **assuming reasonable answers**, then proceeding:

```java
class ExpiringCache<K, V> {
    private record Entry<V>(V value, long expiresAtMillis) { }

    private final ConcurrentHashMap<K, Entry<V>> store = new ConcurrentHashMap<>();
    private final long ttlMillis;

    ExpiringCache(long ttlMillis) {
        this.ttlMillis = ttlMillis;
        Executors.newSingleThreadScheduledExecutor()                 // Module 15, Topic 5
            .scheduleAtFixedRate(this::evictExpired, ttlMillis, ttlMillis, TimeUnit.MILLISECONDS);
    }

    V get(K key) {
        Entry<V> e = store.get(key);
        if (e == null || System.currentTimeMillis() > e.expiresAtMillis()) return null;
        return e.value();
    }

    void put(K key, V value) {
        store.put(key, new Entry<>(value, System.currentTimeMillis() + ttlMillis));
    }

    private void evictExpired() {
        long now = System.currentTimeMillis();
        store.entrySet().removeIf(en -> now > en.getValue().expiresAtMillis());   // Module 10
    }
}
```

**Say this out loud:** "`ConcurrentHashMap` (Module 15, Topic 7) gives thread-safe access without one big lock. I wrap each value in a small `record` (Module 23, Topic 1) carrying its expiry timestamp — records are a clean fit for this immutable internal detail. A background scheduled task does periodic sweeping rather than checking every entry on every read, trading a small memory overhead for expired-but-not-yet-swept entries against avoiding a per-`get()` scan cost. **The real trade-off to name explicitly**: this design is single-JVM-only; a genuinely distributed cache would need Redis or a similar external store, and I'd say so directly rather than let the interviewer assume otherwise."

## Question 2: Design a Simple Database Connection Pool

**Say this out loud, before coding:** "A connection pool solves a specific problem: opening a JDBC connection (Module 20) is expensive — TCP handshake, authentication — so we reuse a fixed set of pre-opened connections rather than opening one per request."

```java
class SimpleConnectionPool implements AutoCloseable {
    private final BlockingQueue<Connection> available;                  // Module 15, Topic 7

    SimpleConnectionPool(String url, String user, String pass, int size) throws SQLException {
        available = new LinkedBlockingQueue<>(size);
        for (int i = 0; i < size; i++) {
            available.add(DriverManager.getConnection(url, user, pass));   // Module 20
        }
    }

    Connection borrow() throws InterruptedException {
        return available.take();          // blocks if the pool is exhausted -- backpressure, by design
    }

    void release(Connection c) {
        available.offer(c);
    }

    @Override
    public void close() throws SQLException {
        for (Connection c : available) c.close();   // Module 12's try-with-resources discipline
    }
}
```

**Say this out loud:** "`BlockingQueue` (Module 15, Topic 7) is a natural fit — `take()` gives me blocking, thread-safe borrow-with-backpressure for free: if every connection is in use, a caller simply waits rather than the pool trying to open unbounded extra connections, which would defeat its purpose. **A real trade-off to raise**: a fixed pool size caps concurrency — under load, callers queue rather than fail immediately; a production system (I'd mention HikariCB in practice) adds connection validation, leak detection, and a borrow timeout, which I've deliberately left out of this simplified version."

## Question 3: Design a Rate Limiter

**Say this out loud:** "Which algorithm fits depends on the requirement: a hard cap on burst traffic, or a smoothed, steady rate? I'll implement the token bucket algorithm — it allows controlled bursts up to the bucket size while enforcing a steady long-term rate, which is usually what's actually wanted."

```java
class TokenBucketRateLimiter {
    private final long capacity;
    private final long refillPerNano;                        // tokens added per nanosecond
    private final AtomicLong tokens;                            // Module 15, Topic 3
    private volatile long lastRefillNanos = System.nanoTime();    // Module 15, Topic 4

    TokenBucketRateLimiter(long capacity, long ratePerSecond) {
        this.capacity = capacity;
        this.tokens = new AtomicLong(capacity);
        this.refillPerNano = ratePerSecond;   // simplified -- real math scales by 1e9 nanos/sec
    }

    boolean tryAcquire() {
        refill();
        long current;
        do {
            current = tokens.get();
            if (current <= 0) return false;
        } while (!tokens.compareAndSet(current, current - 1));    // CAS retry loop -- Module 15, Topic 4
        return true;
    }

    private void refill() {
        long now = System.nanoTime();
        long elapsed = now - lastRefillNanos;
        long newTokens = elapsed * refillPerNano / 1_000_000_000L;
        if (newTokens > 0) {
            tokens.updateAndGet(t -> Math.min(capacity, t + newTokens));   // Module 15, Topic 3
            lastRefillNanos = now;
        }
    }
}
```

**Say this out loud:** "I used `AtomicLong` with a CAS retry loop instead of `synchronized` (Module 15, Topic 3-4) because token acquisition is a short, high-contention operation — lock-free CAS typically scales better than blocking synchronization under contention like this. **A trade-off worth naming**: this is per-JVM-instance state; a rate limiter shared across multiple service instances needs a centralized store (Redis with a Lua script for atomicity is the common real-world choice), which I'd flag as the next question to resolve before shipping this."

## Question 4: How Would You Diagnose and Fix "Our Service Has High GC Pause Times in Production"?

This is a pure judgment/process question — no code expected, just structured reasoning applying Module 16 (GC algorithms) and Module 22 (profiling/tuning) directly.

**A strong answer, structured:**

1. **"First, I'd measure, not guess"** (Module 22, Topic 2): enable JFR (`-XX:StartFlightRecording=...`) or check existing GC logs (`-Xlog:gc*`) to see actual pause frequency, duration, and which generation/collector phase is responsible — never tune blind.
2. **"Then I'd ask what's actually being allocated"**: high allocation rate driving frequent Young GC? Or long-lived objects being promoted and driving expensive Old Generation collections? These point to very different fixes (Module 16, Topic 2).
3. **"Then I'd consider the collector choice itself"** (Module 22, Topic 1): is this workload latency-sensitive enough to justify ZGC/Shenandoah over G1's already-reasonable default behavior? What's the actual heap size, and would `-Xms`=`-Xmx` remove resizing-related variability?
4. **"Only then would I change flags"** — and I'd change one variable at a time, re-measuring after each, rather than adjusting several flags simultaneously and losing the ability to attribute the effect.

**Say this out loud, as the closing line:** "The single biggest mistake I'd avoid here is tuning speculatively — Module 22 was explicit that unmeasured tuning wastes real engineering time and can easily make things worse, not better."

## Real-World Analogy

A conceptual question (Topic 1) tests whether you know what a bridge is made of. A coding problem (Topic 2) tests whether you can build a small one. A system-design-adjacent question tests whether you can be handed a vague brief — "people need to cross this river" — and produce a reasoned proposal: what kind of bridge, why, what it trades away, and what you'd ask the client before finalizing anything.

## Advantages of Structuring Answers This Way

- Demonstrates the exact skill senior roles are hiring for: reasoning under ambiguity, not just recalling facts.
- Naming trade-offs explicitly, unprompted, consistently reads to interviewers as more experienced than a confident answer that pretends no trade-off exists.

## Disadvantages / Trade-offs

- Over-qualifying every answer with excessive caveats can read as indecisiveness — state assumptions once, briefly, then commit to a concrete design; don't hedge every sentence.

## Best Practices

- Always state assumptions and clarifying questions **before** designing, even if the interviewer doesn't explicitly invite them.
- Name at least one genuine trade-off or limitation of your own design, unprompted — it's one of the strongest, most consistent signals of seniority in this style of question.
- Reference the specific course concept powering each design decision (e.g., "`BlockingQueue` gives me backpressure for free") — it shows the design isn't arbitrary.

## Common Mistakes

- Jumping directly to code without stating any assumptions or asking clarifying questions first.
- Proposing a design with no acknowledged weaknesses — every real design has trade-offs, and claiming otherwise reads as inexperience, not confidence.
- Getting lost in premature micro-optimization (e.g., agonizing over the exact CAS retry strategy in Question 3) instead of first establishing the overall architecture is sound.

## Interview Questions

1. **Q: Why is stating your assumptions before designing a system considered more important than the specific design you land on?**
   A: These questions are open-ended by design — interviewers are primarily evaluating structured reasoning under ambiguity, not verifying one "correct" answer; skipping assumptions signals jumping to a solution without first understanding the actual problem.

2. **Q: Why might `BlockingQueue` be preferred over a manually synchronized collection when designing a connection pool?**
   A: It provides thread-safe, blocking borrow-with-backpressure semantics natively — a caller waits when the pool is exhausted rather than the pool needing custom logic to handle contention safely.

3. **Q: What's the correct first step when asked to fix high GC pause times in production, and why?**
   A: Measure first (JFR/GC logs), never tune speculatively — unmeasured changes waste engineering effort and can make the actual problem worse without evidence showing what's really happening.

## Summary

- System-design-adjacent Java questions test judgment and trade-off reasoning, not single correct answers — always state assumptions before designing.
- Four worked judgment questions: an expiring cache (`ConcurrentHashMap` + a scheduled sweep), a connection pool (`BlockingQueue` for backpressure), a rate limiter (token bucket via `AtomicLong`/CAS), and a pure-reasoning GC diagnosis process (measure, then diagnose, then tune, one variable at a time).
- Naming your own design's limitations unprompted is one of the strongest, most consistent seniority signals in this interview style.

## Exercises

1. Design a simple write-through cache (writes go to both the cache and a backing store) and explicitly name the consistency trade-off if the backing-store write fails after the cache write succeeds.
2. Extend the token bucket rate limiter to support per-user limits (a `Map<UserId, TokenBucketRateLimiter>`), and discuss the memory growth trade-off of unboundedly many users.
3. Practice the GC diagnosis question's four-step structure out loud, from memory, without referring back to this topic.

---

**Previous:** [02 — Coding Problems & Live Coding](02-coding-problems-and-live-coding.md) · **Next:** [04 — Mock Interview Walkthrough & Presentation Guidance](04-mock-interview-walkthrough-and-presentation-guidance.md)
