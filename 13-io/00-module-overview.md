# Module 13 — IO

## Module Goal

Every real program eventually needs to read from or write to something outside itself — a file, a network connection, standard input. This module covers Java's classic `java.io` I/O model: the byte-stream/character-stream split (and precisely why it exists), file and path handling, buffering (and why it matters enormously for performance), and object serialization. Try-with-resources (Module 12, Topic 4) gets used constantly throughout — this is the module where that knowledge becomes daily, practical habit.

## Topics Covered in This Module

1. **[IO Fundamentals, Streams & Encoding](01-io-fundamentals-streams-and-encoding.md)** — the byte-stream vs. character-stream split, and why character encoding makes this distinction unavoidable.
2. **[File & Path Handling](02-file-and-path-handling.md)** — the legacy `java.io.File` class vs. the modern `java.nio.file.Path`/`Files` API.
3. **[Byte Streams & Buffering](03-byte-streams-and-buffering.md)** — `FileInputStream`/`FileOutputStream`, and why unbuffered I/O is a real, measurable performance trap.
4. **[Character Streams, Readers & Writers](04-character-streams-readers-and-writers.md)** — `FileReader`/`FileWriter`, `BufferedReader`/`BufferedWriter`, and the bridge classes connecting bytes to characters.
5. **[Object Serialization](05-object-serialization.md)** — `Serializable`, `transient`, `serialVersionUID`, and the well-known, real problems with Java's built-in serialization.
6. **[Module Summary, Interview Questions & Exercises](06-module-summary-exercises.md)** — consolidated recap, quiz, and practice problems.

## Prerequisites

- Module 12 (Exceptions), especially Topic 4 (try-with-resources) — used constantly throughout this module.
- Module 08 (Strings), especially Topic 1 (encoding is directly relevant to character streams).
- Module 07 (Objects), especially Topic 4 (Object Cloning — serialization is a related, alternative way to "copy" object state).

## How to Study This Module

Topic 1 is the conceptual foundation the rest of the module builds on — understanding *why* bytes and characters need separate class hierarchies (not just "Java has two APIs, memorize both") makes Topics 3–4 feel like natural applications rather than parallel, redundant systems to learn independently.

---

**Previous module:** [12 — Exceptions](../12-exceptions/00-module-overview.md) · **Next:** [01 — IO Fundamentals, Streams & Encoding](01-io-fundamentals-streams-and-encoding.md)
