# Module 13 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-Module-Overview.md) has been covered:

- [x] **IO Fundamentals, Streams & Encoding** — the stream abstraction, the byte-stream/character-stream split, and precisely why character encoding makes that split necessary
- [x] **File & Path Handling** — legacy `java.io.File` vs. modern `java.nio.file.Path`/`Files`, and the concrete exception-based error-reporting improvement
- [x] **Byte Streams & Buffering** — `FileInputStream`/`FileOutputStream`, and the precise mechanism (system call reduction) behind buffering's real performance benefit
- [x] **Character Streams, Readers & Writers** — `FileReader`/`FileWriter`, `BufferedReader`/`BufferedWriter`, and the `InputStreamReader`/`OutputStreamWriter` bridge classes
- [x] **Object Serialization** — `Serializable`, `transient`, `serialVersionUID`, and an honest treatment of the well-documented security and fragility problems that have pushed modern systems toward JSON

## Practical Connections

- **Every Spring Boot application's `application.properties`/config file loading, log file writing, and file upload handling** uses exactly this module's `Path`/`Files`/buffered-stream patterns.
- **REST APIs universally use JSON** (Topic 5) rather than Java serialization for request/response bodies — you now understand the concrete, well-documented reasons why, not just "that's the convention."
- **Try-with-resources (Module 12, Topic 4) is used in essentially every code example in this module** — this is where that knowledge becomes genuine daily habit rather than a one-off lesson.
- **Reading configuration files, log files, and CSV/text data** in real backend code follows exactly Topic 4's `BufferedReader.readLine()` idiom.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| Byte streams vs character streams | `InputStream`/`OutputStream` handle raw bytes with no encoding concept; `Reader`/`Writer` handle encoding-aware text. |
| `java.io.File` vs `java.nio.file.Path`/`Files` | Legacy returns bare booleans on failure; modern throws specific, meaningful exceptions. |
| Unbuffered vs buffered streams | Unbuffered triggers a system call per operation; buffered batches many operations into far fewer, larger system calls. |
| `InputStreamReader` vs `BufferedReader` | The former bridges bytes to characters (encoding); the latter adds buffering performance plus `readLine()` convenience — often used together. |
| Java serialization vs JSON | Java serialization is Java-specific, tightly coupled to class structure, and carries a real deserialization security risk; JSON is human-readable, cross-language, and the modern default. |

## Consolidated Interview Questions (Module 13)

1. Why does Java maintain separate byte-stream and character-stream class hierarchies?
2. What causes "mojibake" (garbled text) bugs?
3. What is the biggest design improvement `java.nio.file.Files` offers over `java.io.File`?
4. Why is unbuffered, byte-at-a-time file I/O slow?
5. How does `BufferedInputStream` improve performance, mechanically?
6. What does `InputStreamReader` do, and why does it exist as a separate, explicit class?
7. What does `BufferedReader` add beyond plain buffering?
8. What is `transient`, and why would you use it?
9. Why should you always explicitly declare `serialVersionUID`?
10. What is the well-documented security risk with Java's built-in serialization?
11. Why have modern applications largely moved to JSON instead of Java serialization?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** From memory, draw the byte-stream and character-stream class hierarchies (Topic 1), and explain why they're kept separate.
2. **Hands-on:** Write code copying a text file using the full idiomatic modern pattern: `Files.exists` check, then `BufferedReader`/`BufferedWriter` with explicit UTF-8 encoding, reading and writing line by line.
3. **Hands-on:** Benchmark (using `System.nanoTime()`, a simple timing approach) copying a moderately large file with unbuffered vs. buffered streams, and observe the real performance difference firsthand.
4. **Hands-on:** Serialize and deserialize a small custom class implementing `Serializable`, with one `transient` field, and verify that field is `null`/default after deserialization.
5. **Conceptual:** Explain, referencing this module's security discussion, why a service accepting user-uploaded serialized Java objects for deserialization would be a serious, real security concern.
6. **Synthesis:** Write a method using `Files.walk(...)` (Topic 2) to find all `.log` files in a directory tree, then use buffered character streams (Topic 4) to read and print the first line of each one found.

## What's Next

Module 13 completed classic `java.io`-based file and stream handling. **Module 14 — NIO** goes further into Java's modern, non-blocking I/O model: channels, buffers, selectors, and the fundamentally different approach NIO takes to scalable I/O — essential foundation for understanding how high-performance network servers (and frameworks built on Netty, for instance) actually work under the hood.

---

**Previous:** [05 — Object Serialization](05-Object-Serialization.md) · **Module Overview:** [00 — Module Overview](00-Module-Overview.md)

**Type "Continue" to begin Module 14 — NIO.**
