# `WatchService` & Modern NIO Recap

## Learning Objectives

- Use `WatchService` to react to filesystem changes
- Have a clear, practical decision framework for choosing classic IO vs. NIO vs. modern alternatives

## Prerequisites

All prior topics in this module, Module 13 (IO)

## Motivation

This closing topic covers one more practical NIO.2 tool (filesystem watching — genuinely useful, and much less conceptually demanding than Topic 3), then steps back to give you the decision-making framework this entire module has been building toward: **when do you actually reach for NIO, versus classic IO, versus something else entirely?**

## `WatchService` — Reacting to Filesystem Changes

```java
WatchService watchService = FileSystems.getDefault().newWatchService();
Path dir = Path.of("/path/to/watch");
dir.register(watchService, StandardWatchEventKinds.ENTRY_CREATE,
                             StandardWatchEventKinds.ENTRY_MODIFY,
                             StandardWatchEventKinds.ENTRY_DELETE);

while (true) {
    WatchKey key = watchService.take();   // BLOCKS until a filesystem event occurs
    for (WatchEvent<?> event : key.pollEvents()) {
        System.out.println(event.kind() + ": " + event.context());
    }
    key.reset();   // IMPORTANT -- re-arms the key to keep receiving future events
}
```

**`WatchService` lets your program be notified when files are created, modified, or deleted in a directory**, without needing to repeatedly poll the filesystem yourself (checking "has anything changed?" in a loop — wasteful, and introduces real latency between an actual change and your program noticing it). This is a genuinely common, practical need: development tools with "hot reload" (noticing a source file changed and automatically recompiling/restarting), log-file monitoring, configuration-file reload-on-change — all built on exactly this mechanism.

**`key.reset()` is essential and easy to forget** — without it, a `WatchKey` only reports events **once**; forgetting to reset it means your watcher silently stops receiving any further notifications for that directory after the first batch of events.

## The Complete Decision Framework — Classic IO vs. NIO vs. Modern Alternatives

This module, combined with Module 13, has given you two full I/O models. Here's the practical, honest guide for choosing between them (and their modern alternatives) in real code:

| Scenario | Recommended approach |
|---|---|
| Reading/writing a single file, small-to-medium size | `Files.readString`/`writeString` (Module 13, Topic 2) — simplest, most direct |
| Reading/writing a large text file, line by line | `BufferedReader`/`BufferedWriter` (Module 13, Topic 4) |
| Copying a file | `Files.copy(...)` or `FileChannel.transferTo` (Topic 2) — the latter for very large files/max performance |
| Random access into a very large file | Memory-mapped files (`FileChannel.map`, Topic 2) |
| Watching a directory for changes | `WatchService` (this topic) |
| Building a custom, high-performance network server from scratch | Raw `Selector`-based NIO (Topic 3) — but seriously consider a framework first |
| Building almost any real network server/application | A framework (Netty, or an HTTP framework built on it) or, in modern Java, **Virtual Threads** (Module 15) with simple blocking-style code |
| General application-level object serialization/data interchange | JSON (Module 13, Topic 5) — not raw byte-level NIO at all |

**The single most important, practical takeaway from this entire module**: **most application code should use the simplest tool that solves the actual problem** — `Files.readString` for a config file, `BufferedReader` for a log file, a well-established framework (or Virtual Threads) for a network server. **Raw NIO `Selector` code is a specialized, advanced tool** — genuinely important to understand conceptually (Topic 3), since it explains how the frameworks and modern language features you'll actually use are built, but rarely something you should reach for directly in typical application development.

## Real-World Analogy

Think of this module's full toolkit like a **professional workshop's complete tool wall** — you now know what a router, a lathe, and a precision milling machine each do, and *why* each exists (the specific problem each is uniquely good at solving). But for most everyday tasks — hanging a picture, assembling furniture — you reach for a simple screwdriver (`Files.readString`), not the precision milling machine (raw `Selector` code). Knowing the milling machine exists, and understanding its purpose, makes you a more capable craftsperson overall — even on days you never actually touch it.

## Advantages

- `WatchService` provides efficient, event-driven filesystem monitoring without wasteful manual polling.
- Having a complete mental map of the full I/O toolkit (Modules 13–14) lets you make genuinely informed tool choices, rather than defaulting to whatever's most familiar regardless of fit.

## Disadvantages / Trade-offs

- `WatchService`'s exact behavior/reliability can vary somewhat across operating systems — worth testing on your actual target platform for anything genuinely critical.
- The sheer breadth of Java's I/O toolkit (classic streams, NIO buffers/channels, NIO.2 files, WatchService, and more) means real judgment is needed to pick the right tool — this module's decision framework is meant to make that judgment easier.

## Best Practices

- Always call `key.reset()` after processing a `WatchKey`'s events, or you'll silently stop receiving further notifications.
- Default to the simplest applicable tool (Module 13's convenience methods) for typical application I/O; reserve raw NIO for genuinely specialized, performance-critical, or large-scale-concurrency scenarios.
- For building network servers specifically, strongly prefer an established framework or Virtual Threads (Module 15) over hand-rolled `Selector` loops, unless you have a specific, deep reason not to.

## Common Mistakes

- Forgetting `key.reset()`, causing filesystem watching to silently stop after the first event batch.
- Reaching for raw NIO `Selector` code for typical application development, when a much simpler tool (or an established framework) would serve the actual need better.
- Manually polling a directory for changes in a loop instead of using `WatchService`, introducing both wasted CPU and unnecessary notification latency.

## Interview Questions

1. **Q: What does `WatchService` provide, and why is `key.reset()` important?**
   A: Efficient, event-driven notification of filesystem changes (file creation/modification/deletion) in a registered directory, without manual polling. `key.reset()` re-arms the key to continue receiving future events — without it, a `WatchKey` only reports its first batch of events and then silently stops.

2. **Q: For a typical application reading a configuration file at startup, should you use raw NIO channels/buffers, or something simpler?**
   A: Something simpler — `Files.readString(path)` (Module 13, Topic 2) is the appropriate, idiomatic choice for a small, one-time file read; raw NIO channels/buffers are a specialized tool for scenarios genuinely requiring their specific capabilities (bulk transfer performance, memory-mapping, non-blocking scalability).

## Summary

- **`WatchService`** provides efficient, event-driven filesystem change notifications — remember to call `key.reset()` after processing events.
- The complete I/O decision framework: use the **simplest tool that solves the actual problem** — `Files` convenience methods for typical file access, buffered streams for line-based text processing, `FileChannel`/memory-mapping for large-file performance needs, and frameworks/Virtual Threads (rather than hand-written `Selector` loops) for scalable network servers.

## Module-Wide Quick Revision

- NIO is built around `Buffer` (capacity/position/limit, `flip()` before reading) rather than direct stream reads (Topic 1).
- `Channel`s are bidirectional and buffer-oriented; `transferTo`/`transferFrom` enable efficient bulk copying; memory-mapped files provide direct, OS-managed access to large files (Topic 2).
- Blocking I/O's thread-per-connection model doesn't scale past thousands of connections (the C10K problem); non-blocking I/O + `Selector` lets one thread efficiently monitor many channels; Virtual Threads (Module 15) offer a modern alternative to hand-written event loops (Topic 3).
- `WatchService` provides efficient filesystem change notifications; the practical guidance throughout is to default to the simplest applicable tool (this topic).

## Common Pitfalls (Module-Wide)

- Forgetting `flip()` before reading a buffer.
- Using memory-mapped files for small or purely sequential access.
- Assuming more threads always scale better, ignoring per-thread memory/scheduling overhead.
- Forgetting `key.reset()` in `WatchService` usage.
- Reaching for raw NIO when a simpler Module 13 tool would suffice.

## Mini Quiz (Module-Wide)

1. What does `flip()` do to a buffer's position and limit?
2. What's the fundamental difference between a `Channel` and a `Stream`?
3. Why does memory-mapping benefit large files with random access specifically?
4. What is the C10K problem?
5. What does `Selector` let one thread do?

*(Answers are derivable from Topics 1, 2, 2, 3, and 3 respectively.)*

---

**Previous:** [03 — Selectors, Non-Blocking I/O & the C10K Problem](03-Selectors-Non-Blocking-IO-And-The-C10K-Problem.md) · **Next:** [05 — Module Summary, Interview Questions & Exercises](05-Module-Summary-Exercises.md)
