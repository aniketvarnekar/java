# Channels & Memory-Mapped Files

## Learning Objectives

- Use `FileChannel` for buffer-based file I/O
- Understand precisely how a `Channel` differs from a `java.io` `Stream`
- Understand memory-mapped files and why they can be dramatically faster for certain workloads

## Prerequisites

[01 — Buffers](01-buffers.md), Module 02 Topic 3 (Heap, memory model)

## Motivation

Channels are NIO's replacement for streams — bidirectional, buffer-oriented, and (crucially, Topic 3) capable of non-blocking operation in a way streams fundamentally are not. Memory-mapped files, this topic's second half, represent one of the most powerful, genuinely different techniques NIO enables — worth understanding even if you never write raw channel code yourself.

## `Channel` vs. `Stream` — The Fundamental Difference

| | `java.io` Stream | `java.nio` Channel |
|---|---|---|
| Direction | Unidirectional (`InputStream` OR `OutputStream`) | **Bidirectional** — a single `FileChannel` can both read and write |
| Data model | Reads/writes bytes (or chars) directly | Reads/writes into/from a **`Buffer`** (Topic 1) |
| Blocking behavior | Always blocking | Can be **non-blocking** (Topic 3) |
| Bulk transfer | Byte-by-byte or manual buffering (Module 13, Topic 3) | Built-in, highly efficient bulk transfer operations |

## `FileChannel` — Basic Usage

```java
try (FileChannel channel = FileChannel.open(Path.of("data.bin"), StandardOpenOption.READ)) {
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    int bytesRead = channel.read(buffer);   // reads INTO the buffer
    buffer.flip();                             // Topic 1's essential step!
    while (buffer.hasRemaining()) {
        System.out.print((char) buffer.get());
    }
}
```

**Notice the constant interplay with `Buffer` (Topic 1)** — a `Channel` never reads/writes raw bytes directly the way a `java.io` stream does; it always reads **into** or writes **from** a `Buffer` object. This is precisely why Topic 1 had to come first — nothing in this module makes sense without it.

## Efficient File Copying — `transferTo`/`transferFrom`

```java
try (FileChannel source = FileChannel.open(Path.of("input.bin"), StandardOpenOption.READ);
     FileChannel dest = FileChannel.open(Path.of("output.bin"),
             StandardOpenOption.CREATE, StandardOpenOption.WRITE)) {

    source.transferTo(0, source.size(), dest);   // copies the ENTIRE file, often using OS-level
}                                                    // "zero-copy" optimizations under the hood
```

**`transferTo`/`transferFrom` can leverage OS-level "zero-copy" transfer** — on supporting platforms, the operating system can copy data directly from the source file to the destination **without ever routing the bytes through your Java program's own memory space at all**, a genuinely significant performance advantage for large file copies compared to the read-into-buffer-then-write-from-buffer pattern shown above (or Module 13's classic stream-based copying).

## Memory-Mapped Files — A Fundamentally Different Access Model

```java
try (FileChannel channel = FileChannel.open(Path.of("huge-file.bin"), StandardOpenOption.READ)) {
    MappedByteBuffer mapped = channel.map(FileChannel.MapMode.READ_ONLY, 0, channel.size());

    byte firstByte = mapped.get(0);          // reads DIRECTLY from the mapped memory region --
    byte lastByte = mapped.get((int) channel.size() - 1);   // NO explicit read() call needed at all!
}
```

**This is genuinely different from every other I/O technique in this course.** `map(...)` asks the operating system to **map the file directly into your process's virtual memory address space** — instead of explicitly calling `read()` to copy bytes from the file into a buffer, the file's contents become **directly, transparently accessible as if it were an in-memory array**, and the OS handles loading the actual disk pages **on demand**, behind the scenes, exactly like Module 02's virtual memory concepts operate for ordinary program memory.

```
 NORMAL file reading:                     MEMORY-MAPPED file access:

 Disk ──▶ OS kernel buffer ──▶ Java Heap    Disk ──▶ mapped directly into your
          (explicit read() call,                      process's virtual address space --
           copies data)                                 accessed like a plain array, OS
                                                          loads pages from disk LAZILY,
                                                          on-demand, as you actually touch them
```

**Why can this be dramatically faster, for the right workload?** It eliminates an entire layer of explicit copying (from OS buffer into your Java buffer), and lets the OS's own, highly-optimized virtual memory/page-caching machinery handle data movement — particularly powerful for **randomly accessing** specific portions of a **very large** file (like a large database index or data file), since you only ever pay the cost of loading the *specific pages* you actually touch, rather than needing to read through the file sequentially.

**When it's NOT the right tool**: for small files, or purely sequential read-through-once access patterns, the setup overhead of memory-mapping isn't worth it over ordinary buffered stream reading (Module 13, Topic 3) — memory-mapped files shine specifically for **large files with random or repeated access patterns**.

## Real-World Analogy

Think of ordinary channel-based reading like **photocopying specific pages of a book you request, one request at a time** — you get exactly the pages you asked for, delivered to your desk. Think of a memory-mapped file like being **handed direct, permanent access to the book itself, sitting on a shelf right next to your desk** — you can flip to any page instantly whenever you want, and a diligent librarian (the OS) quietly ensures the specific pages you actually touch are always physically present and readable, without you ever needing to explicitly "request a copy" of anything.

## Advantages

- Channels support efficient bulk transfers (`transferTo`/`transferFrom`), sometimes using OS-level zero-copy optimizations.
- Memory-mapped files eliminate explicit copying and leverage the OS's highly optimized virtual memory machinery, particularly powerful for large files with random access patterns.

## Disadvantages / Trade-offs

- Memory-mapped files add real complexity (address space considerations, platform-specific behavior nuances) compared to straightforward stream-based reading.
- Not a universal win — small files or purely sequential access gain little or nothing from memory-mapping, and may even be slower due to setup overhead.

## Best Practices

- Use `transferTo`/`transferFrom` for efficient whole-file copying instead of manual buffer-based read/write loops.
- Reserve memory-mapped files for genuinely large files with random or repeated access patterns — not as a default choice for all file I/O.
- Always pair channel operations with correct buffer management (Topic 1's `flip()`, etc.).

## Common Mistakes

- Using memory-mapped files for small files or simple sequential reads, adding unnecessary complexity for no real performance benefit.
- Forgetting channels always operate through `Buffer` objects, unlike direct stream reads/writes.
- Not understanding that memory-mapped access loads pages lazily, on-demand — assuming (incorrectly) that mapping a huge file immediately loads its entire contents into memory.

## Interview Questions

1. **Q: What's the fundamental difference between a `java.nio` `Channel` and a `java.io` `Stream`?**
   A: A `Channel` is bidirectional (a single `FileChannel` can read and write) and always operates through `Buffer` objects, while a `Stream` is unidirectional and reads/writes bytes directly. Channels can also operate non-blocking (Topic 3), which streams fundamentally cannot.

2. **Q: What is a memory-mapped file, and why can it be significantly faster for certain workloads?**
   A: `FileChannel.map(...)` maps a file's contents directly into the process's virtual memory address space, letting the OS load disk pages lazily/on-demand as they're actually accessed, eliminating an explicit copy step. It's particularly beneficial for large files with random or repeated access patterns, since only the specific touched pages incur load cost.

3. **Q: When is memory-mapping NOT the right choice?**
   A: For small files or purely sequential read-through-once access, where memory-mapping's setup overhead outweighs any benefit compared to simple, ordinary buffered stream reading.

## Summary

- **Channels** are bidirectional, buffer-oriented, and can operate non-blocking — a fundamentally different model from `java.io` streams.
- `transferTo`/`transferFrom` provide efficient bulk file copying, sometimes using OS-level zero-copy optimizations.
- **Memory-mapped files** (`FileChannel.map(...)`) let a file's contents be accessed directly as if they were an in-memory array, with the OS lazily loading pages on demand — powerful for large files with random access, unnecessary overhead for small or purely sequential use.

## Exercises

1. Write code using `FileChannel` and a `ByteBuffer` to read the first 100 bytes of a file, correctly using `flip()` before processing.
2. Rewrite a file-copy operation using `transferTo` instead of a manual buffer loop, and explain the performance advantage this can offer.
3. Explain, in your own words, why memory-mapped files are well-suited for a large database index file accessed randomly, but poorly suited for a small configuration file read once at startup.

---

**Previous:** [01 — Buffers](01-buffers.md) · **Next:** [03 — Selectors, Non-Blocking I/O & the C10K Problem](03-selectors-non-blocking-io-and-the-c10k-problem.md)
