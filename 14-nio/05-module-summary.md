# Module 14 Summary

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

## What's Next

Module 14 completed your understanding of Java's I/O models, from simple streams through scalable non-blocking channels — including *why* the C10K problem exists and how modern Java addresses it. **Module 15 — Concurrency** now delivers the full payoff: threads, synchronization, the Java Memory Model, `java.util.concurrent`'s executor framework, and — directly building on this module's Topic 3 preview — Java 21's Virtual Threads and Structured Concurrency in complete depth.