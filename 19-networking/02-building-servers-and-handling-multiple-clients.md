# Building Servers & Handling Multiple Clients

## Learning Objectives

- Understand why a single-threaded server cannot handle concurrent clients
- Build a thread-per-connection server using an `ExecutorService` (Module 15, Topic 5)
- Understand how Virtual Threads (Module 15, Topic 8) directly apply to this exact problem

## Prerequisites

[01 — Networking Fundamentals & Sockets](01-networking-fundamentals-and-sockets.md), Module 15 (Concurrency), Module 14 Topic 3 (C10K)

## Motivation

This topic is where Module 14's C10K problem and Module 15's entire concurrency toolkit stop being abstract and become directly, concretely applicable to a real, practical need: a server that can actually handle more than one client at a time. This is deliberately the payoff topic that ties three prior modules together.

## The Problem, Made Concrete

Recall Topic 1's basic server: it calls `accept()` once, handles that **one** client fully, and only then loops back to `accept()` again. **A second client attempting to connect while the first is still being served must simply wait** — completely unacceptable for any real, practical server.

```java
// The BROKEN, single-threaded approach:
try (ServerSocket serverSocket = new ServerSocket(8080)) {
    while (true) {
        Socket client = serverSocket.accept();   // blocks until a client connects
        handleClient(client);                       // handles this client COMPLETELY before
    }                                                   // looping back to accept() again --
}                                                          // NO OTHER CLIENT CAN CONNECT MEANWHILE!
```

## The Classic Fix: Thread-Per-Connection (Recall Module 15, Topic 5)

```java
try (ServerSocket serverSocket = new ServerSocket(8080);
     ExecutorService executor = Executors.newFixedThreadPool(50)) {   // Module 15, Topic 5's
                                                                          // EXACT thread-pool pattern
    while (true) {
        Socket client = serverSocket.accept();      // still blocks, but ONLY for accepting --
        executor.submit(() -> handleClient(client));   // handling itself happens on a POOL thread,
                                                            // freeing this loop to immediately accept
    }                                                        // the NEXT client right away
}
```

**This is a direct, literal application of Module 15, Topic 5's `ExecutorService` pattern to networking**: the main loop's only job is to `accept()` connections as fast as they arrive and immediately hand each one off to a pooled worker thread — precisely avoiding the "one client blocks everyone else" problem, using exactly the tools you already have.

**But recall Module 14, Topic 3's honest C10K assessment**: a `newFixedThreadPool(50)`-based server can genuinely, comfortably handle 50 concurrent clients — but scaling to thousands runs directly into the same platform-thread memory/scheduling ceiling Module 14 explained in depth.

## The Modern Fix: Virtual Threads (Recall Module 15, Topic 8)

```java
try (ServerSocket serverSocket = new ServerSocket(8080);
     ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {   // Module 15,
                                                                                     // Topic 8 --
    while (true) {                                                                  // ONE line changed!
        Socket client = serverSocket.accept();
        executor.submit(() -> handleClient(client));
    }
}
```

**This is genuinely the entire change required** — swapping `newFixedThreadPool(50)` for `newVirtualThreadPerTaskExecutor()`. **This single line directly, completely resolves Module 14's C10K problem for this exact server**, using precisely the mechanism Module 15, Topic 8 explained in depth: each client connection gets its own Virtual Thread, written using the same simple, readable, blocking-style `handleClient` method — but now scalable to hundreds of thousands of simultaneous connections, since Virtual Threads transparently unmount from their carrier thread whenever `handleClient`'s blocking I/O calls (reading from the socket) are waiting.

**This is the single most concrete, satisfying payoff of this entire course's concurrency arc** — Module 14 taught you the problem, Module 15 taught you the tools (culminating in Virtual Threads), and this exact server example is where you can see, directly, that a genuinely real-world networking problem is solved with a one-line change once you understand why.

## `handleClient` — Ordinary, Simple, Blocking-Style Code

```java
void handleClient(Socket client) {
    try (client;   // Socket is AutoCloseable -- Java 9+ allows existing variables in try-with-resources
         BufferedReader in = new BufferedReader(new InputStreamReader(client.getInputStream()));
         PrintWriter out = new PrintWriter(client.getOutputStream(), true)) {

        String line;
        while ((line = in.readLine()) != null) {   // BLOCKS waiting for client data -- exactly
            out.println("Echo: " + line);              // the moment a Virtual Thread transparently
        }                                                  // unmounts from its carrier (Module 15, Topic 8)
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

**Notice: this method contains zero concurrency-specific code at all** — no `Selector`, no manual non-blocking logic, no callbacks. It's exactly the same style of simple, sequential, blocking code you'd write for a single-client toy example — and, wrapped in a Virtual Thread (via the executor), it genuinely scales.

## Real-World Analogy

Think of the single-threaded server like a **single bank teller who fully completes one customer's entire transaction — deposits, questions, everything — before even acknowledging the next person in line exists.** Think of the thread-pool version like **hiring 50 tellers** — a real, meaningful improvement, but eventually the building itself (platform thread memory/OS scheduling) runs out of physical teller windows to add. Think of the Virtual Thread version like **each customer getting their own personal, extremely lightweight assistant who steps aside the instant the customer needs to fill out a form (blocking I/O) — freeing the small number of actual bank windows (carrier threads) to serve someone else in the meantime**, and stepping back in the instant that customer is ready to continue — genuinely, comfortably serving thousands of customers "simultaneously," each experiencing simple, uninterrupted service.

## Advantages

- Thread-per-connection (whether platform or virtual) lets you write simple, readable, sequential `handleClient` logic — no manual event-loop complexity.
- Virtual Threads specifically deliver Module 14's NIO-level scalability with Module 15, Topic 1's original simple, blocking coding style — the best of both worlds this course has been building toward since Module 14.

## Disadvantages / Trade-offs

- Platform-thread-pool-based servers remain genuinely limited in maximum concurrent connections, exactly per Module 14's C10K analysis.
- Even with Virtual Threads, genuinely CPU-bound per-request work (not I/O-bound) won't see the same scalability benefit (Module 15, Topic 8's honest caveat) — Virtual Threads specifically help because `handleClient` is dominated by blocking I/O waits.

## Best Practices

- Never handle client connections directly on the `accept()`-calling thread — always hand off to a pool (platform or, preferably, virtual) immediately.
- Default to `Executors.newVirtualThreadPerTaskExecutor()` for new, I/O-bound server code in Java 21+, applying Module 15, Topic 8's guidance directly.
- Keep `handleClient`-style methods simple and blocking-style — this is precisely the code shape Virtual Threads are designed to make scalable.

## Common Mistakes

- Handling each client fully on the single `accept()`-loop thread, allowing only one client to be served at a time.
- Assuming a fixed platform-thread pool scales indefinitely, without acknowledging Module 14's real C10K ceiling.
- Manually attempting complex NIO `Selector`-based server code (Module 14, Topic 3) when Virtual Threads now provide the same scalability with dramatically simpler code for most typical use cases.

## Interview Questions

1. **Q: Why can't a single-threaded server (calling `accept()` then fully handling each client before looping back) serve multiple clients concurrently?**
   A: The main thread is fully occupied handling one client (including blocking on reads from that client) before it can call `accept()` again — any other client attempting to connect in the meantime simply has to wait, since there's no other thread available to serve them.

2. **Q: How does swapping a platform thread pool for `Executors.newVirtualThreadPerTaskExecutor()` change a server's scalability, with what code changes required?**
   A: It resolves Module 14's C10K problem for that server — Virtual Threads transparently unmount from their carrier thread whenever `handleClient`'s blocking I/O calls are waiting, letting a small number of carrier threads serve a huge number of concurrent Virtual Thread-backed connections. The code change required is minimal — typically just the executor factory method call itself, with `handleClient`'s simple, blocking-style logic completely unchanged.

## Summary

- A single-threaded `accept()`-then-fully-handle loop can serve only one client at a time — genuinely unusable for real servers.
- **Thread-per-connection via `ExecutorService`** (Module 15, Topic 5) fixes this for moderate concurrency, but hits Module 14's C10K-style platform-thread ceiling at scale.
- **Virtual Threads** (Module 15, Topic 8) resolve this completely for I/O-bound server workloads, letting simple, blocking-style `handleClient` code scale to massive concurrent connection counts with essentially a one-line executor change — the concrete, practical payoff of this course's entire Modules 14–15 concurrency arc.

## Exercises

1. Extend Topic 1's basic echo server to handle multiple simultaneous clients using `Executors.newFixedThreadPool(...)`, and test it with several clients connecting at once.
2. Change the executor to `Executors.newVirtualThreadPerTaskExecutor()`, and explain, referencing Module 15, Topic 8, why `handleClient`'s own code required no changes at all.
3. Explain, in your own words, why this server's use case (client connections dominated by waiting for I/O) is exactly the scenario Virtual Threads are specifically optimized for, referencing Module 15, Topic 8's CPU-bound-vs-I/O-bound distinction.

---

**Previous:** [01 — Networking Fundamentals & Sockets](01-networking-fundamentals-and-sockets.md) · **Next:** [03 — The Modern `HttpClient`](03-the-modern-httpclient.md)
