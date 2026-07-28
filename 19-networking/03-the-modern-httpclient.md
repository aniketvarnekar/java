# The Modern `HttpClient`

## Learning Objectives

- Use `java.net.http.HttpClient` for both synchronous and asynchronous HTTP requests
- Understand how async requests integrate directly with `CompletableFuture` (Module 15, Topic 6)
- Know why this modern API replaced the old `HttpURLConnection`

## Prerequisites

Module 15 Topic 6 (`CompletableFuture`), [01 — Networking Fundamentals & Sockets](01-networking-fundamentals-and-sockets.md)

## Motivation

Raw sockets (Topics 1–2) are the foundation, but almost no real application code talks to another web service using raw TCP directly — HTTP is the near-universal protocol for service-to-service and client-server communication, and `java.net.http.HttpClient` (finalized in Java 11) is Java's modern, well-designed answer for speaking it.

## Why a New HTTP Client Was Needed

Java had an HTTP client since 1.0 (`HttpURLConnection`), but it was, by wide consensus, a genuinely awkward, dated API — no built-in HTTP/2 support, no genuine asynchronous request model, and a clunky, hard-to-use programming interface. `HttpClient` (incubated in Java 9, finalized in Java 11) was a ground-up redesign addressing all of these, and is now the standard, recommended choice for HTTP communication in modern Java, without needing a third-party library for basic needs.

## Synchronous Requests — The Simple Case

```java
HttpClient client = HttpClient.newHttpClient();

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users/42"))
    .GET()
    .build();

HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

System.out.println(response.statusCode());   // 200
System.out.println(response.body());           // the response body, as a String
```

**Notice the builder pattern (`HttpRequest.newBuilder()...build()`) — directly Module 06, Topic 3's fluent-chaining pattern applied here**, letting you configure a request declaratively before sending it. **`client.send(...)` blocks the calling thread until the full response arrives** — exactly Module 14/19, Topic 1's blocking I/O behavior, now applied to an HTTP call specifically.

## Asynchronous Requests — Direct Integration With `CompletableFuture`

```java
CompletableFuture<HttpResponse<String>> futureResponse =
    client.sendAsync(request, HttpResponse.BodyHandlers.ofString());

futureResponse
    .thenApply(HttpResponse::body)              // Module 15, Topic 6's EXACT chaining pattern
    .thenAccept(body -> System.out.println("Got: " + body))
    .exceptionally(ex -> {
        System.out.println("Request failed: " + ex.getMessage());
        return null;
    });

System.out.println("Request sent, not blocking here...");   // this line runs IMMEDIATELY,
                                                                 // not waiting for the response at all
```

**`sendAsync(...)` returns a `CompletableFuture<HttpResponse<T>>` directly** — this is not a coincidental similarity to Module 15, Topic 6; `HttpClient`'s async API was **deliberately designed** around `CompletableFuture` specifically so that everything you already know about chaining (`thenApply`/`thenCompose`/`exceptionally`) applies **immediately and directly** to handling HTTP responses, with zero new concepts required.

```java
// Making MULTIPLE requests concurrently, combining their results -- Module 15, Topic 6's
// thenCombine, applied directly to real network calls:
CompletableFuture<HttpResponse<String>> userRequest = client.sendAsync(userReq, BodyHandlers.ofString());
CompletableFuture<HttpResponse<String>> ordersRequest = client.sendAsync(ordersReq, BodyHandlers.ofString());

userRequest.thenCombine(ordersRequest, (userResp, ordersResp) ->
    "User: " + userResp.body() + ", Orders: " + ordersResp.body()
).thenAccept(System.out::println);
```

**This is a genuinely common, real, practical pattern in microservice architectures**: fetching data from two or more independent downstream services **concurrently** (rather than sequentially, one after another) and combining their results once both complete — directly, precisely the scenario Module 15, Topic 6's `thenCombine` was introduced to solve, now shown solving an actual, real networking problem.

## Configuring the Client and Request

```java
HttpClient client = HttpClient.newBuilder()
    .connectTimeout(Duration.ofSeconds(10))
    .version(HttpClient.Version.HTTP_2)          // HTTP/2 support -- built in, unlike the old API
    .build();

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users"))
    .header("Content-Type", "application/json")
    .POST(HttpRequest.BodyPublishers.ofString("{\"name\":\"Aniket\"}"))   // Module 03, Topic 3's
    .timeout(Duration.ofSeconds(5))                                          // TEXT BLOCKS would make
    .build();                                                                  // this JSON body far
                                                                                   // more readable in
                                                                                   // real code!
```

**`BodyPublishers`/`BodyHandlers`** control how the request body is sent and the response body is received/converted — `ofString()` is the common case, with `ofByteArray()`, `ofFile(...)`, and others available for different data shapes.

## Real-World Analogy

Think of the synchronous `client.send(...)` like **placing a phone order and staying on the line until the food is fully described back to you** — simple, but you can't do anything else while waiting. Think of `client.sendAsync(...)` like **placing an order online and getting a confirmation with tracking, immediately free to do other things**, with the tracking page automatically updating (via your `thenApply`/`thenAccept` chain) the moment your order status actually changes — no need to keep refreshing or waiting on hold. `thenCombine` across two async requests is like **ordering from two different restaurants simultaneously for one meal**, and only sitting down to eat once **both** deliveries have actually arrived.

## Advantages

- Modern, builder-pattern-based API is significantly more ergonomic than the legacy `HttpURLConnection`.
- Built-in HTTP/2 support, without requiring a third-party library.
- Async requests integrate directly and seamlessly with `CompletableFuture` (Module 15, Topic 6) — no new concurrency vocabulary needed.

## Disadvantages / Trade-offs

- For very advanced HTTP needs (connection pooling tuning, retry policies, circuit breakers), production systems often still reach for a dedicated third-party HTTP client library or framework — `HttpClient` covers the core, standard cases very well, but isn't necessarily the final word for every sophisticated production need.
- Synchronous `send(...)` calls, if used carelessly at scale without Module 15's concurrency tools (thread pools or Virtual Threads), can reintroduce exactly the blocking-thread scalability concerns Modules 14–15 addressed in depth.

## Best Practices

- Prefer `sendAsync(...)` combined with `CompletableFuture` chaining for any scenario involving multiple concurrent requests or non-blocking needs.
- Reuse a single `HttpClient` instance across many requests rather than creating a new one per call — it's designed to be reused and manages connection pooling internally.
- Always set reasonable timeouts (`connectTimeout`, request-level `timeout`) — never allow an HTTP call to potentially hang indefinitely.

## Common Mistakes

- Creating a new `HttpClient` for every single request instead of reusing one instance.
- Using blocking `send(...)` calls in a context (like a Virtual-Thread-per-request server, Topic 2) where this actually works fine — but forgetting it would be a genuine scalability problem on a fixed platform-thread pool without Module 15, Topic 8's Virtual Threads.
- Omitting timeouts, risking indefinitely hanging requests.

## Interview Questions

1. **Q: Why did Java introduce a new `HttpClient` in Java 11, given `HttpURLConnection` already existed?**
   A: `HttpURLConnection` was a dated, awkward API with no built-in HTTP/2 support and no genuine asynchronous request model. `HttpClient` was a ground-up redesign providing a modern, builder-pattern-based API, built-in HTTP/2 support, and deliberate `CompletableFuture` integration for asynchronous requests.

2. **Q: What does `HttpClient.sendAsync(...)` return, and why is that significant?**
   A: A `CompletableFuture<HttpResponse<T>>` — this is a deliberate design choice letting all of Module 15, Topic 6's `CompletableFuture` chaining (`thenApply`, `thenCombine`, `exceptionally`) apply directly to handling asynchronous HTTP responses, with no additional concurrency vocabulary needed.

3. **Q: How would you fetch data from two independent services concurrently and combine their results?**
   A: Issue both requests via `sendAsync(...)`, obtaining two `CompletableFuture<HttpResponse<T>>`s, then combine them with `.thenCombine(...)` (Module 15, Topic 6) once both complete — running both requests concurrently rather than sequentially.

## Summary

- **`java.net.http.HttpClient`** (Java 11+) is Java's modern, builder-pattern-based HTTP client, with built-in HTTP/2 support, replacing the dated `HttpURLConnection`.
- **`client.send(...)`** is synchronous/blocking; **`client.sendAsync(...)`** returns a `CompletableFuture<HttpResponse<T>>`, integrating directly with Module 15, Topic 6's async chaining and combination tools.
- Reuse a single `HttpClient` instance, always set timeouts, and prefer async requests (with `CompletableFuture` chaining) when making multiple concurrent calls.

## Module-Wide Quick Revision

- The client-server model: IP address + port identify a specific service; `Socket`/`ServerSocket` provide raw TCP communication via the exact same I/O stream API from Module 13 (Topic 1).
- A single-threaded `accept()` loop can only serve one client at a time; `ExecutorService`-based thread-per-connection fixes moderate concurrency, and Virtual Threads (Module 15, Topic 8) fully resolve Module 14's C10K problem for I/O-bound servers with minimal code change (Topic 2).
- `HttpClient` provides modern, builder-pattern HTTP requests, synchronous (`send`) or asynchronous (`sendAsync`, returning `CompletableFuture`, directly reusing Module 15, Topic 6's chaining) (this topic).

## Common Pitfalls (Module-Wide)

- Forgetting sockets are `AutoCloseable` and need try-with-resources.
- Handling clients fully on the `accept()`-loop thread, serving only one at a time.
- Assuming a fixed platform-thread pool scales indefinitely.
- Creating a new `HttpClient` per request instead of reusing one.
- Omitting request/connection timeouts.