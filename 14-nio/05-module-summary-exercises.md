# Module 14 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-module-overview.md) has been covered:

- [x] **Buffers** — the capacity/position/limit model, `flip()` (and why it's essential), `clear()`/`rewind()`/`compact()`
- [x] **Channels & Memory-Mapped Files** — how `Channel` differs from `Stream`, `transferTo`/`transferFrom`'s zero-copy potential, and memory-mapped file access via `FileChannel.map`
- [x] **Selectors, Non-Blocking I/O & the C10K Problem** — precisely why thread-per-connection doesn't scale, what non-blocking I/O actually changes, `Selector`'s one-thread-many-channels model, and a preview of how Virtual Threads change this calculus
- [x] **`WatchService` & Modern NIO Recap** — filesystem change notification, and a complete, practical decision framework across this module and Module 13

## Practical Connections

- **Netty** (the dominant high-performance Java networking framework, underlying gRPC, many reactive frameworks, and parts of Spring WebFlux) is built directly on exactly this module's `Buffer`/`Channel`/`Selector` concepts — you now understand the foundation these tools are built on, even without writing raw NIO code yourself.
- **Java 21's Virtual Threads** (Module 15) exist specifically to address the C10K-style scalability problem this module explained in depth — understanding the *problem* deeply (Topic 3) is what makes Virtual Threads' *solution* genuinely meaningful, rather than just "another concurrency feature."
- **Large-scale data processing tools** (databases, search indices like Lucene/Elasticsearch) rely heavily on memory-mapped files (Topic 2) for efficient random access to huge datasets.
- **IDE "file watcher" features** (auto-reload on save, live-reload dev servers) are built on exactly `WatchService` (Topic 4).

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| `Stream` (java.io) vs `Channel` (java.nio) | Streams are unidirectional and read/write bytes directly; channels are bidirectional, buffer-oriented, and support non-blocking operation. |
| Blocking vs non-blocking I/O | Blocking `read()` waits until data arrives; non-blocking `read()` returns immediately, reporting what's currently available (possibly nothing). |
| Non-blocking I/O vs `Selector` | Non-blocking channels avoid waiting on a single read; `Selector` avoids wasteful busy-waiting across many non-blocking channels by efficiently blocking until at least one is ready. |
| `clear()` vs `rewind()` | `clear()` prepares for fresh writing (old data logically discarded); `rewind()` re-reads the same already-written data from the start. |
| Ordinary file reading vs memory-mapped files | Ordinary reading explicitly copies bytes into a buffer; memory-mapping makes the file directly accessible as if it were in-memory, with the OS lazily loading pages on demand. |

## Consolidated Interview Questions (Module 14)

1. What does `Buffer.flip()` do, and why is it necessary?
2. What's the difference between `clear()` and `rewind()`?
3. What's the fundamental difference between a `Channel` and a `Stream`?
4. What is a memory-mapped file, and why can it be significantly faster for certain workloads?
5. What is the C10K problem?
6. What does "non-blocking" mean for a channel's `read()` call?
7. What does `Selector` do, and why is it needed even with non-blocking channels?
8. How do Virtual Threads change the practical need for hand-written NIO Selector code?
9. What does `WatchService` provide, and why is `key.reset()` important?
10. For typical application file reading, should you use raw NIO, or something simpler?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** From memory, explain what `flip()`, `clear()`, `rewind()`, and `compact()` each do to a buffer's position/limit.
2. **Hands-on:** Write code using `FileChannel` and `ByteBuffer` to read a small file's contents, correctly using `flip()`.
3. **Hands-on:** Set up a `WatchService` monitoring a directory, and create/modify/delete a test file to observe the events it reports — remember `key.reset()`.
4. **Conceptual:** Calculate, with real numbers, roughly how much memory 10,000 thread-per-connection threads would consume in stack space alone, and explain why this motivated NIO's development.
5. **Conceptual:** Explain, in your own words, why memory-mapped files are well-suited to a large database index but poorly suited to a small, frequently-rewritten configuration file.
6. **Synthesis:** Using this module's decision framework (Topic 4), for each of these scenarios, state which specific tool (from Modules 13–14) you'd use and why: (a) reading a 2KB config file at startup, (b) copying a 10GB video file, (c) building a chat server handling 50,000 simultaneous connections, (d) randomly accessing records in a 20GB data file.

## What's Next

Module 14 completed your understanding of Java's I/O models, from simple streams through scalable non-blocking channels — including *why* the C10K problem exists and how modern Java addresses it. **Module 15 — Concurrency** now delivers the full payoff: threads, synchronization, the Java Memory Model, `java.util.concurrent`'s executor framework, and — directly building on this module's Topic 3 preview — Java 21's Virtual Threads and Structured Concurrency in complete depth.

---

**Previous:** [04 — `WatchService` & Modern NIO Recap](04-watchservice-and-modern-nio-recap.md) · **Module Overview:** [00 — Module Overview](00-module-overview.md)

**Type "Continue" to begin Module 15 — Concurrency.**
