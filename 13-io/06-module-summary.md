# Module 13 Summary

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-module-overview.md) has been covered:

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

## What's Next

Module 13 completed classic `java.io`-based file and stream handling. **Module 14 — NIO** goes further into Java's modern, non-blocking I/O model: channels, buffers, selectors, and the fundamentally different approach NIO takes to scalable I/O — essential foundation for understanding how high-performance network servers (and frameworks built on Netty, for instance) actually work under the hood.