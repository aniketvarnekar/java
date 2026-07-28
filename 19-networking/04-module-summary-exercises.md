# Module 19 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-module-overview.md) has been covered:

- [x] **Networking Fundamentals & Sockets** — the client-server model, IP addresses/ports, `Socket`/`ServerSocket`, and the direct realization that socket I/O reuses Module 13's exact stream API
- [x] **Building Servers & Handling Multiple Clients** — the single-threaded server's fundamental limitation, the `ExecutorService`-based fix (Module 15, Topic 5), and Virtual Threads (Module 15, Topic 8) fully resolving Module 14's C10K problem with minimal code change
- [x] **The Modern `HttpClient`** — synchronous and asynchronous requests, and direct, deliberate integration with `CompletableFuture` (Module 15, Topic 6)

## Practical Connections

- **Every microservice architecture** is fundamentally built on this module's concepts — services communicating over sockets (usually via HTTP, Topic 3), servers handling many concurrent client connections (Topic 2).
- **Spring's embedded Tomcat/Netty server**, handling thousands of concurrent HTTP requests, is a production-grade, highly-engineered version of exactly Topic 2's thread-per-connection (increasingly Virtual-Thread-based) model.
- **API Gateway / BFF (Backend-for-Frontend) patterns**, fetching from multiple downstream services concurrently and combining results, are a direct, everyday application of Topic 3's `thenCombine`-based async request pattern.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| IP address vs. port | IP address identifies the machine; port identifies the specific service/application on that machine. |
| `Socket` I/O vs. "networking-specific" I/O | There is no separate networking I/O API — sockets are read/written via the exact same `InputStream`/`Reader`/`Writer` machinery as files (Module 13). |
| Thread-per-connection (platform) vs. Virtual-Thread-per-connection | Platform: genuine C10K ceiling from real OS thread overhead. Virtual: same simple code, scales to hundreds of thousands of connections (Module 15, Topic 8). |
| `HttpClient.send` vs `.sendAsync` | `send`: blocks the calling thread until the response arrives. `sendAsync`: returns immediately with a `CompletableFuture`, integrating with Module 15, Topic 6's chaining. |

## Consolidated Interview Questions (Module 19)

1. What's the difference between an IP address and a port number?
2. How do you read and write data over a `Socket` in Java?
3. Why can't a single-threaded server serve multiple clients concurrently?
4. How does swapping to `Executors.newVirtualThreadPerTaskExecutor()` change a server's scalability?
5. Why did Java introduce a new `HttpClient` in Java 11?
6. What does `HttpClient.sendAsync(...)` return, and why is that a deliberate design choice?
7. How would you fetch data from two independent services concurrently and combine their results?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** Explain, from memory, why socket I/O requires no new API knowledge beyond what Module 13 already taught.
2. **Hands-on:** Build a multi-client echo server using `Executors.newVirtualThreadPerTaskExecutor()`, and test it by connecting with several simultaneous client instances.
3. **Hands-on:** Write a program using `HttpClient` to fetch a public API endpoint synchronously, printing the status code and body.
4. **Hands-on:** Rewrite the same request using `sendAsync(...)`, chaining `.thenApply(HttpResponse::body).thenAccept(...)`.
5. **Conceptual:** Explain why `HttpClient.sendAsync` returning `CompletableFuture` specifically (rather than some other async abstraction) was a deliberate design choice, referencing Module 15, Topic 6.
6. **Synthesis:** Design a small service method fetching two independent pieces of data concurrently via `HttpClient.sendAsync` + `thenCombine`, and explain how this differs in total latency from making the two requests sequentially with `send(...)`.

## What's Next

Module 19 completed Java's networking story, from raw sockets through the modern, `CompletableFuture`-integrated `HttpClient` — and gave you a genuinely concrete, practical payoff for Modules 14–15's concurrency arc. **Module 20 — JDBC** now covers database connectivity: `Connection`, `PreparedStatement` (and why it prevents SQL injection), `ResultSet`, and connection pooling — the final piece of foundational infrastructure knowledge before this course turns to Modules (21) and Performance (22).

---

**Previous:** [03 — The Modern `HttpClient`](03-the-modern-httpclient.md) · **Module Overview:** [00 — Module Overview](00-module-overview.md)

**Type "Continue" to begin Module 20 — JDBC.**
