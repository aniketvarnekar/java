# Networking Fundamentals & Sockets

## Learning Objectives

- Understand the client-server model and the role of IP addresses and ports
- Use `Socket` and `ServerSocket` to establish a basic TCP connection
- Understand why sockets are read/written via ordinary I/O streams

## Prerequisites

Module 13 (IO), Module 12 Topic 4 (try-with-resources)

## Motivation

Every HTTP request, every database connection (Module 20), every message queue client is ultimately built on the same foundation: a TCP socket connection between two programs, possibly on entirely different machines. This topic gives you that foundation directly, before Topic 3 shows you the much higher-level `HttpClient` abstraction built on top of it.

## The Client-Server Model

> A **server** is a program that listens on a specific **port**, waiting for incoming connections. A **client** is a program that **initiates** a connection to a server's known **IP address and port**.

**An IP address identifies a specific machine on a network; a port (a number from 0–65535) identifies a specific application/service running on that machine** — a single machine can run many different network services simultaneously (a web server on port 80, a database on port 5432), and the port number is what distinguishes which specific service a connection is meant for.

```
                     CLIENT                                 SERVER (listening on port 8080)

┌────────────────────────────┐                  ┌────────────────────────────────────┐
│ initiates a                │                  │ ServerSocket.accept()              │
│ connection to              │── connection ──▶ │ (waiting/blocking here             │
│ IP:8080                    │     request      │ until a client connects)           │
│                            │◀─ connection ─── │                                    │
│                            │    accepted      │                                    │
└────────────────────────────┘                  └────────────────────────────────────┘
```

## `ServerSocket` and `Socket` — Raw TCP Communication

```java
// SERVER side:
try (ServerSocket serverSocket = new ServerSocket(8080)) {   // listen on port 8080
    System.out.println("Server listening...");
    Socket clientSocket = serverSocket.accept();                // BLOCKS until a client connects
                                                                   // (recall Module 14, Topic 3's
                                                                   //  "blocking I/O" discussion --
                                                                   //  THIS is exactly that scenario!)

    BufferedReader in = new BufferedReader(
        new InputStreamReader(clientSocket.getInputStream()));      // Module 13, Topic 4's EXACT bridge pattern!
    PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);

    String message = in.readLine();
    out.println("Server received: " + message);
}
```

```java
// CLIENT side:
try (Socket socket = new Socket("localhost", 8080)) {   // INITIATES the connection
    PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
    BufferedReader in = new BufferedReader(
        new InputStreamReader(socket.getInputStream()));

    out.println("Hello, server!");
    String response = in.readLine();
    System.out.println("Server said: " + response);
}
```

**Notice: both sides communicate via ordinary `InputStream`/`OutputStream` (Module 13), bridged into character streams via `InputStreamReader`/`PrintWriter`, exactly Module 13, Topic 4's pattern.** This is a genuinely important, unifying realization: **a network socket is, from your program's own perspective, just another I/O stream** — everything you learned in Module 13 about reading/writing streams applies directly, unchanged, to network communication. There's no separate "networking I/O" API to learn from scratch — it's the exact same `Reader`/`Writer`/`InputStream`/`OutputStream` machinery, simply connected to a different kind of underlying source/destination (a network connection instead of a file).

## Why `Socket`/`ServerSocket` Are `AutoCloseable`

**Recall Module 12, Topic 4 directly**: both classes implement `AutoCloseable`, which is exactly why the examples above use try-with-resources — a network connection is precisely the kind of external, limited resource (Module 12, Topic 4's motivating examples: files, database connections) that needs deterministic, guaranteed cleanup, not GC-timing-dependent cleanup. Leaving sockets unclosed is a genuine, real resource leak (exhausting available ports/file descriptors under sustained load) — exactly the class of problem try-with-resources was designed to prevent.

## `accept()` Is a Blocking Call — Directly Connecting to Module 14

**`serverSocket.accept()` blocks the calling thread until a client actually connects** — this is **precisely** Module 14, Topic 3's "blocking I/O" concept, made completely concrete: a thread calling `accept()` (or, afterward, `in.readLine()` waiting for client data) sits idle, doing nothing, until data/a connection actually arrives. **A server handling only this one client, on this one thread, cannot serve a second client at the same time** — the exact problem Topic 2 of this module addresses directly, applying Module 14 and Module 15's full toolkit to solve it properly.

## Real-World Analogy

Think of an IP address like a **building's street address**, and a port number like a **specific apartment/suite number within that building** — the address alone gets you to the right building, but you need the suite number to reach the specific occupant (service) you actually want to talk to. `ServerSocket.accept()` is like a **receptionist sitting at the front desk, doing nothing else, until a visitor actually walks in** — genuinely idle, blocked, waiting, exactly Module 14's blocking I/O concept in physical form.

## Advantages

- Sockets provide a genuinely low-level, direct, and fully general mechanism for network communication — the foundation everything else (HTTP, databases, message queues) is ultimately built on.
- Reusing the exact same `InputStream`/`Reader`/`Writer` API from Module 13 means no new I/O vocabulary is needed to work with network data.

## Disadvantages / Trade-offs

- Raw sockets require you to handle a genuinely large amount of low-level detail yourself (message framing, protocol design, error handling) that higher-level abstractions (like `HttpClient`, Topic 3) handle for you.
- A single-threaded server using raw blocking sockets can only serve one client at a time — directly motivating Topic 2's concurrency-based solutions.

## Best Practices

- Always use try-with-resources for `Socket`/`ServerSocket`, exactly like any other `AutoCloseable` resource (Module 12, Topic 4).
- Recognize socket I/O as "just I/O" — apply Module 13's buffering and encoding-awareness guidance directly.
- Reach for a higher-level abstraction (`HttpClient`, Topic 3, or a full framework) for anything beyond simple, educational, or genuinely custom-protocol network communication.

## Common Mistakes

- Forgetting sockets are `AutoCloseable` resources requiring proper cleanup, risking resource exhaustion under sustained use.
- Assuming a basic single-threaded server (as shown in this topic) can handle multiple simultaneous clients — it fundamentally cannot, without the concurrency techniques covered in Topic 2.
- Not recognizing that socket I/O uses the exact same stream API as file I/O (Module 13), and re-deriving I/O handling from scratch instead of applying existing knowledge.

## Interview Questions

1. **Q: What's the difference between an IP address and a port number?**
   A: An IP address identifies a specific machine on a network; a port number identifies a specific application/service running on that machine, allowing one machine to run multiple distinct network services simultaneously.

2. **Q: How do you read and write data over a `Socket` in Java?**
   A: Via ordinary `InputStream`/`OutputStream` (obtained from the socket), typically bridged into character streams using `InputStreamReader`/`PrintWriter` — exactly Module 13's standard I/O stream API, applied to a network connection instead of a file.

3. **Q: Why does `ServerSocket.accept()` block, and what problem does this create for handling multiple clients?**
   A: It waits until a client actually connects, exactly Module 14's blocking I/O behavior — a thread calling `accept()` (or subsequently reading from the accepted connection) is idle until something happens. A server using only one thread this way can serve only one client at a time, motivating the concurrency-based server designs covered in Topic 2.

## Summary

- The **client-server model**: a server listens on an IP address + port; a client initiates a connection to that specific address/port combination.
- **`ServerSocket`**/**`Socket`** provide raw TCP communication, read/written via the exact same `InputStream`/`OutputStream`/`Reader`/`Writer` API from Module 13.
- Both are `AutoCloseable` (Module 12, Topic 4) — always use try-with-resources.
- `accept()` (and subsequent reads) are blocking calls — directly, concretely illustrating Module 14, Topic 3's blocking I/O concept, and motivating Topic 2's concurrency-based solutions.