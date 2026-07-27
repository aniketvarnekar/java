# Module 19 — Networking

## Module Goal

Module 01, Topic 3 flagged "Distributed" as one of Java's defining features — built-in networking support, out of the box. This module makes that concrete: the client-server model, raw TCP sockets, and the modern `java.net.http.HttpClient`, directly applying Module 14's NIO/C10K knowledge and Module 15's concurrency/`CompletableFuture` toolkit to real network communication.

## Topics Covered in This Module

1. **[Networking Fundamentals & Sockets](01-Networking-Fundamentals-And-Sockets.md)** — the client-server model, and `Socket`/`ServerSocket` for raw TCP communication.
2. **[Building Servers & Handling Multiple Clients](02-Building-Servers-And-Handling-Multiple-Clients.md)** — directly applying Module 14's C10K problem and Module 15's thread-pool/Virtual Thread solutions to real network servers.
3. **[The Modern `HttpClient`](03-The-Modern-HttpClient.md)** — synchronous and asynchronous HTTP requests, and `CompletableFuture`-based response handling (Module 15, Topic 6).
4. **[Module Summary, Interview Questions & Exercises](04-Module-Summary-Exercises.md)** — consolidated recap, quiz, and practice problems.

## Prerequisites

- Module 14 (NIO), Topic 3 (the C10K problem).
- Module 15 (Concurrency), especially Topic 5 (Executors) and Topic 8 (Virtual Threads).
- Module 13 (IO), especially Topic 4 (Readers/Writers — sockets are read/written via I/O streams).
- Module 12 (Exceptions), Topic 4 (try-with-resources — sockets are `AutoCloseable`).

## How to Study This Module

Topic 1 gives you the low-level foundation; Topic 2 is deliberately positioned to make you apply Modules 14–15's hard-won concurrency knowledge to a genuinely concrete, practical problem — "how do I actually handle many simultaneous client connections?" — rather than learning it in the abstract. Topic 3 is the part you'll use constantly in real backend development: nearly every microservice calls another service via `HttpClient` (or a framework built on the same principles).

---

**Previous module:** [18 — Streams](../18-Streams/00-Module-Overview.md) · **Next:** [01 — Networking Fundamentals & Sockets](01-Networking-Fundamentals-And-Sockets.md)
