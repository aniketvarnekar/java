# Module 14 — NIO (New I/O)

## Module Goal

Module 13 covered `java.io`'s stream-based model — simple, but fundamentally **blocking**: a thread calling `read()` sits idle until data arrives. This module covers `java.nio` ("New I/O," introduced in Java 1.4, 2002) — Java's alternative, buffer-and-channel-based I/O model, purpose-built for a problem classic streams handle poorly: **scaling to many thousands of simultaneous connections**. This is genuinely more advanced material — the foundation for understanding how high-performance servers (and frameworks like Netty) actually work.

## Topics Covered in This Module

1. **[Buffers](01-buffers.md)** — `ByteBuffer` and the capacity/position/limit model that underlies all of NIO.
2. **[Channels & Memory-Mapped Files](02-channels-and-memory-mapped-files.md)** — `FileChannel`, how channels differ fundamentally from streams, and memory-mapped file access.
3. **[Selectors, Non-Blocking I/O & the C10K Problem](03-selectors-non-blocking-io-and-the-c10k-problem.md)** — blocking vs. non-blocking I/O, `Selector`, and the specific scalability problem NIO was built to solve.
4. **[`WatchService` & Modern NIO Recap](04-watchservice-and-modern-nio-recap.md)** — filesystem change notifications, and a clear decision guide for classic IO vs. NIO vs. modern alternatives.
5. **[Module Summary](05-module-summary.md)** — consolidated recap.

## Prerequisites

- Module 13 (IO) — this module is defined largely in contrast to it.
- Module 02 (JVM), especially Topic 3 (Runtime Data Areas — Stack per thread) and Topic 4 (Execution Engine).
- Module 15 (Concurrency) concepts are referenced at a preview level here; full depth comes next.

## How to Study This Module

This module is more specialized than most — you will use `java.io` far more often day-to-day than raw NIO. The value here is **conceptual**: understanding *why* blocking I/O doesn't scale to huge numbers of connections, and what non-blocking I/O actually does differently, is foundational knowledge for understanding modern server architecture, even if you rarely write raw `Selector` code yourself (most real applications use a framework — Netty, or Java 21's Virtual Threads, Module 15 — built on top of these exact concepts).