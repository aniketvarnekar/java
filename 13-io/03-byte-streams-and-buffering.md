# Byte Streams & Buffering

## Learning Objectives

- Use `FileInputStream`/`FileOutputStream` correctly
- Understand precisely why unbuffered I/O is slow, with a concrete mechanism
- Use `BufferedInputStream`/`BufferedOutputStream` (the Decorator pattern) correctly

## Prerequisites

[01 — IO Fundamentals, Streams & Encoding](01-io-fundamentals-streams-and-encoding.md), Module 12 Topic 4 (try-with-resources)

## Motivation

This topic explains one of the most impactful, easy-to-apply performance improvements in all of everyday Java programming — wrapping a stream in a buffer — and, more importantly, explains **why** it works, so the technique generalizes to any I/O-bound code you'll ever write, not just the specific classes shown here.

## `FileInputStream` and `FileOutputStream` — The Basics

```java
try (InputStream in = new FileInputStream("data.bin");
     OutputStream out = new FileOutputStream("copy.bin")) {

    int byteValue;
    while ((byteValue = in.read()) != -1) {   // read() returns -1 at end of stream
        out.write(byteValue);
    }
}
```

**This works correctly** — but is a genuine, real performance trap, explained precisely below.

## Why Unbuffered, One-Byte-at-a-Time I/O Is Slow

**Every single call to `in.read()` (reading just one byte) typically triggers a real, expensive system call** — a request from your Java program, through the JVM, to the underlying **operating system**, which handles actual disk (or network) access. System calls have genuine, measurable overhead — context-switching between your program and the OS kernel, potential disk-seek operations, and more — **regardless of how much data is actually being transferred in that one call**.

```
 UNBUFFERED: one system call PER BYTE

 read() -> SYSTEM CALL (expensive overhead) -> returns 1 byte
 read() -> SYSTEM CALL (expensive overhead) -> returns 1 byte
 read() -> SYSTEM CALL (expensive overhead) -> returns 1 byte
 ... (repeated for EVERY SINGLE byte in the file -- for a 1MB file, that's potentially
      OVER A MILLION separate, expensive system calls!)
```

**Reading a file one byte at a time means paying that expensive system-call overhead once per byte** — for a 1 MB file, that's potentially **over a million** separate system calls, each with its own real overhead, when the *actual* disk read could have been accomplished in a small handful of much larger, more efficient operations.

## `BufferedInputStream`/`BufferedOutputStream` — The Fix

```java
try (InputStream in = new BufferedInputStream(new FileInputStream("data.bin"));
     OutputStream out = new BufferedOutputStream(new FileOutputStream("copy.bin"))) {

    int byteValue;
    while ((byteValue = in.read()) != -1) {   // SAME code as before!
        out.write(byteValue);
    }
}
```

**Notice: the actual reading/writing loop code is completely unchanged** — only the stream **construction** changed, wrapping the original stream in a `BufferedInputStream`/`BufferedOutputStream`. This is the **Decorator design pattern** (a genuinely important, widely-used pattern beyond just I/O, worth recognizing by name): `BufferedInputStream` **wraps** an existing `InputStream`, adding buffering behavior **transparently**, without changing the interface your code interacts with at all.

```
 BUFFERED: one system call per BUFFER-FULL (typically several KB at a time)

 read() -> checks INTERNAL BUFFER first -- if it has data, return it INSTANTLY, no system call!
 read() -> ...
 (after the buffer is exhausted:)
 read() -> BUFFER EMPTY -- ONE system call fills the ENTIRE buffer (e.g., 8KB) at once
 read() -> checks buffer -- HAS data now, returns instantly, no system call
 read() -> checks buffer -- HAS data, returns instantly
 ... (thousands of fast, in-memory reads, for every ONE real system call)
```

**`BufferedInputStream` maintains an internal, in-memory buffer (typically several kilobytes).** When you call `read()`, it first checks whether the buffer already has data — if so, it returns a byte from that buffer **instantly, with no system call at all**. Only when the buffer is genuinely exhausted does it perform **one** real system call to refill the **entire** buffer at once — meaning the expensive system-call overhead is paid **once per buffer-full**, not once per byte, a **dramatic**, real, measurable performance improvement for any non-trivial amount of data.

## The General Principle — Applies Far Beyond Just These Two Classes

**This exact "wrap in a buffering decorator" pattern applies to every stream-based I/O class in Java** — `BufferedReader`/`BufferedWriter` (Topic 4) work identically, for exactly the same underlying reason. **The general, transferable lesson**: whenever you're doing small, repeated I/O operations (reading/writing a little bit at a time, in a loop), buffering the underlying stream is almost always a substantial, easy performance win, because it converts "many small, expensive operations" into "few large, efficient operations" — a genuinely universal systems-programming principle, not something specific to Java's particular class names.

## Reading/Writing Larger Chunks Directly — An Alternative to Buffering

```java
byte[] buffer = new byte[8192];    // an 8KB buffer, MANUALLY managed
int bytesRead;
while ((bytesRead = in.read(buffer)) != -1) {    // reads UP TO buffer.length bytes in ONE call
    out.write(buffer, 0, bytesRead);               // writes exactly the bytes actually read
}
```

**`read(byte[] buffer)` reads multiple bytes in a single call**, filling as much of the provided array as it can — this achieves a similar performance benefit to `BufferedInputStream` by manually batching reads into larger chunks, without needing the wrapper class at all. **In practice, using `BufferedInputStream` combined with simple, one-byte-at-a-time-style code is usually preferred** for its simplicity — you get the performance benefit of batching without needing to manage a buffer array and its exact fill-length bookkeeping yourself.

## Real-World Analogy

Think of unbuffered I/O like **walking to the mailbox and back to fetch a single letter, one trip at a time, for every single piece of mail you're expecting** — technically correct, but if you're expecting a thousand pieces of mail, that's a thousand separate, tiring round trips. `BufferedInputStream` is like **checking the mailbox once, grabbing the entire stack of accumulated mail in one trip**, and then sorting through that stack from the comfort of your own home — the "expensive trip to the mailbox" (the system call) happens far less often, even though you're ultimately processing the exact same total amount of mail.

## Advantages

- Buffering provides a dramatic, real, easily-applied performance improvement for any non-trivial I/O workload, with zero change to your actual reading/writing logic.
- The Decorator pattern (wrapping one stream in another) is transparent — code written against the base `InputStream`/`OutputStream` interface works identically whether or not it's actually buffered underneath.

## Disadvantages / Trade-offs

- Buffered streams consume a modest, fixed amount of extra memory for their internal buffer — negligible for virtually all practical purposes, but technically a real, non-zero cost.
- Forgetting to buffer (using raw `FileInputStream`/`FileOutputStream` directly, in a byte-at-a-time loop) is easy to do accidentally, and the resulting performance problem may not be obvious until tested against realistically-sized data.

## Best Practices

- Always wrap `FileInputStream`/`FileOutputStream` (and similar raw streams) in their `Buffered` counterparts, unless you have a specific reason not to.
- Recognize the Decorator pattern as a general, transferable technique — you'll encounter (and can apply) it well beyond just I/O.
- Prefer `BufferedInputStream` wrapping over manually managing your own buffer array, for simpler, equally performant code.

## Common Mistakes

- Reading/writing files byte-by-byte without buffering, and being surprised by poor performance on realistically large files.
- Assuming buffering changes the stream's interface or behavior in any observable way (beyond performance) — it doesn't; the Decorator pattern is specifically designed to be transparent.
- Forgetting to close (or use try-with-resources for) buffered streams — an unflushed buffer can mean data written to it never actually reaches the underlying file.

## Interview Questions

1. **Q: Why is reading a file one byte at a time, using an unbuffered `FileInputStream`, slow?**
   A: Each individual `read()` call typically triggers a real, expensive operating-system-level system call, regardless of how much data is actually transferred — for a large file, this means potentially millions of separate, expensive system calls instead of a much smaller number of larger, efficient ones.

2. **Q: How does `BufferedInputStream` improve performance, mechanically?**
   A: It maintains an internal, in-memory buffer; `read()` calls are served instantly from this buffer with no system call, until the buffer is exhausted, at which point one real system call refills the entire buffer at once — converting many small, expensive operations into few large, efficient ones.

3. **Q: What design pattern does wrapping `new BufferedInputStream(new FileInputStream(...))` demonstrate?**
   A: The Decorator pattern — `BufferedInputStream` wraps an existing stream, transparently adding buffering behavior without changing the interface the calling code interacts with at all.

## Summary

- `FileInputStream`/`FileOutputStream` provide basic, unbuffered byte-stream access to files.
- Unbuffered, byte-at-a-time I/O is slow because each `read()`/`write()` call typically triggers an expensive OS-level system call, regardless of data size.
- **`BufferedInputStream`/`BufferedOutputStream`** wrap an existing stream (the Decorator pattern), maintaining an internal buffer that batches many small operations into far fewer, larger, more efficient system calls — a dramatic, easily-applied performance improvement.
- This general "batch small operations into fewer large ones" principle applies broadly across I/O and beyond, not just to these specific classes.