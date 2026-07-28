# File & Path Handling

## Learning Objectives

- Use both `java.io.File` (legacy) and `java.nio.file.Path`/`Files` (modern) correctly
- Understand precisely why the modern API was introduced, and its concrete advantages
- Perform common file operations idiomatically in modern Java

## Prerequisites

[01 — IO Fundamentals, Streams & Encoding](01-io-fundamentals-streams-and-encoding.md), Module 12 Topic 4 (try-with-resources)

## Motivation

Before you can read or write a file's contents, you need to represent and manipulate the file's **location** and **metadata** (does it exist? is it a directory? what's its size?). Java has two generations of API for this — understanding why the second generation (NIO.2, Java 7+) replaced the first tells you which one to actually reach for in new code.

## `java.io.File` — The Original (Java 1.0) API

```java
File file = new File("data.txt");
file.exists();          // boolean
file.isDirectory();       // boolean
file.length();               // size in bytes
file.delete();                  // boolean -- returns false on failure, doesn't throw!
file.mkdir();                     // create a directory -- also returns boolean
```

**`File`'s design has several genuine, real limitations**, well-documented and widely acknowledged:

1. **Most methods return `boolean` for success/failure, rather than throwing an exception with a meaningful reason.** `file.delete()` returning `false` tells you *that* deletion failed, but not **why** (permission denied? file in use? path doesn't exist?) — directly contradicting Module 12's exception-based error-handling philosophy.
2. **No genuine symbolic link support.**
3. **Poor, inconsistent cross-platform behavior** for certain operations.
4. **No built-in support for efficient directory tree traversal or file-watching.**

## `java.nio.file.Path` and `Files` — The Modern (Java 7+) API

```java
import java.nio.file.*;

Path path = Path.of("data.txt");             // Java 11+ factory method (Paths.get(...) for 7-10)
Files.exists(path);                              // boolean
Files.isDirectory(path);
Files.size(path);                                    // size in bytes -- throws IOException on failure!
Files.delete(path);                                     // throws IOException with a MEANINGFUL reason on failure
Files.createDirectory(path);
```

**The single biggest, most consequential design improvement**: `Files`'s methods **throw checked `IOException`s with specific, meaningful messages** on failure, rather than silently returning `false` — directly applying Module 12's exception-based philosophy correctly, where the legacy `File` API notably didn't.

```java
try {
    Files.delete(path);
} catch (NoSuchFileException e) {
    System.out.println("File doesn't exist: " + e.getFile());
} catch (DirectoryNotEmptyException e) {
    System.out.println("Can't delete non-empty directory: " + e.getFile());
} catch (IOException e) {
    System.out.println("Deletion failed: " + e.getMessage());   // some OTHER, specific reason
}
```

**Compare this directly to `file.delete()` returning a bare `false`** — the modern API tells you **precisely** what went wrong, using Module 12's full exception-type-hierarchy expressiveness (specific exception subtypes for specific failure reasons), instead of forcing you to guess or write your own diagnostic logic from scratch.

## Reading and Writing Entire Files — The Convenient Modern Way

```java
// Reading an entire small/medium text file in one call:
String content = Files.readString(path);                    // Java 11+
List<String> lines = Files.readAllLines(path);                 // reads ALL lines into a List<String>

// Writing:
Files.writeString(path, "Hello, World!");                        // Java 11+
Files.write(path, List.of("line1", "line2"));                       // writes a List<String>, one per line
```

**These convenience methods are appropriate specifically for small-to-medium files** that comfortably fit in memory — recall Topic 1's motivation for streams in the first place: for genuinely large files, you'd still want incremental, stream-based processing (Topics 3–4) rather than loading everything into memory at once via these convenience methods.

## Directory Traversal — A Genuine Modern Improvement

```java
try (Stream<Path> paths = Files.walk(Path.of("src"))) {   // recursively walks an ENTIRE directory tree
    paths.filter(Files::isRegularFile)
         .filter(p -> p.toString().endsWith(".java"))
         .forEach(System.out::println);
}
```

**`Files.walk(...)` returns a `Stream<Path>`** (full Streams API depth: Module 18; usable now by pattern) — lazily traversing an entire directory tree, letting you filter/process it using the same functional-style operations you'll master soon. **The legacy `File` API had no equivalent** — recursive directory traversal required manually written recursive methods, a genuine, real inconvenience the modern API directly solves.

## `Path` Manipulation

```java
Path path = Path.of("/home/user/documents/report.txt");
path.getFileName();       // "report.txt"
path.getParent();           // "/home/user/documents"
path.resolve("data.csv");     // "/home/user/documents/data.csv" -- combines paths correctly,
                                 // handling platform-specific separators automatically
path.toAbsolutePath();          // resolves to a full, absolute path
```

**`resolve(...)` handling platform-specific path separators automatically** (`/` on Unix-like systems, `\` on Windows) is a genuinely important, real portability win — directly echoing Module 01, Topic 1's platform-independence theme, now applied specifically to file-path construction, historically a real, common source of cross-platform bugs when developers manually concatenated path strings with hardcoded separators.

## Full Comparison

| | `java.io.File` (legacy) | `java.nio.file.Path`/`Files` (modern, Java 7+) |
|---|---|---|
| Error reporting | `boolean` return values, no reason given | Specific, meaningful checked exceptions |
| Symbolic link support | Poor/absent | Full support |
| Directory tree traversal | Manual recursion required | `Files.walk(...)` — lazy `Stream<Path>` |
| Reading/writing whole files | Manual stream code required | `Files.readString`/`writeString` (11+) |
| Cross-platform path building | Manual, error-prone string concatenation | `resolve(...)`, handles separators automatically |
| Modern status | Legacy — avoid in new code | **Standard, recommended default** |

## Real-World Analogy

Think of `java.io.File` like an **old intercom system that only buzzes "yes" or "buzzes nothing" (silence)** when you ask to enter a building — you learn *whether* you got in, but never *why* you were denied (wrong code? door broken? building closed?). `java.nio.file.Path`/`Files` is like a **modern access system that speaks back with a specific, informative message** ("access denied: your badge has expired," "access denied: this door is under maintenance") — genuinely more useful for actually diagnosing and responding to the problem, exactly Module 12's exception-based philosophy correctly applied to file operations.

## Advantages

- `Files`'s exception-based error reporting provides genuinely actionable diagnostic information, unlike `File`'s bare booleans.
- `Files.walk`, `readString`/`writeString`, and `Path.resolve` provide substantial, real convenience improvements over manual, error-prone legacy code.
- Full symbolic link support and better cross-platform behavior overall.

## Disadvantages / Trade-offs

- The legacy `File` API remains present throughout older/legacy codebases and some older library APIs still expect it, requiring occasional interoperability (`Path.toFile()`/`File.toPath()` bridge methods exist for exactly this reason).
- The convenience whole-file-reading methods (`readString`, `readAllLines`) are inappropriate for very large files, where incremental stream processing (Topics 3–4) remains necessary.

## Best Practices

- Use `java.nio.file.Path`/`Files` for all new code — it's the modern, recommended standard, offering meaningfully better error reporting and functionality.
- Use `Files.readString`/`writeString`/`readAllLines` only for files you're confident are reasonably small; use streaming approaches (Topics 3–4) for large files.
- Use `Path.resolve(...)` instead of manual string concatenation when building file paths, for automatic cross-platform correctness.

## Common Mistakes

- Using the legacy `File` API in new code, missing out on meaningful error diagnostics and modern convenience methods.
- Manually concatenating path strings with hardcoded separators (`"/"` or `"\\"`), breaking cross-platform portability.
- Using `Files.readString`/`readAllLines` on genuinely large files, risking `OutOfMemoryError`.

## Interview Questions

1. **Q: What is the biggest design improvement `java.nio.file.Files` offers over `java.io.File`?**
   A: `Files`'s methods throw specific, meaningful checked exceptions (like `NoSuchFileException`, `DirectoryNotEmptyException`) on failure, providing genuinely actionable diagnostic information — `File`'s methods largely return bare `boolean` values with no explanation of *why* an operation failed.

2. **Q: How would you recursively list every `.java` file in a directory tree using the modern API?**
   A: `Files.walk(startPath)`, returning a lazily-traversed `Stream<Path>` (Module 18 preview), filtered with `.filter(p -> p.toString().endsWith(".java"))` — the legacy `File` API required manually written recursive traversal code to achieve the same result.

3. **Q: Why is `Path.resolve(...)` preferred over manual string concatenation for building file paths?**
   A: It automatically handles platform-specific path separators (`/` vs `\`) correctly, avoiding a historically common class of cross-platform path-construction bugs that manual string concatenation with hardcoded separators would introduce.

## Summary

- **`java.io.File`** (legacy, Java 1.0): `boolean`-based error reporting with no diagnostic detail, manual recursive traversal, poor symbolic link support.
- **`java.nio.file.Path`/`Files`** (modern, Java 7+): specific, meaningful checked exceptions, `Files.walk` for lazy directory traversal, `readString`/`writeString` convenience methods (Java 11+), and automatic cross-platform path handling via `resolve(...)`.
- Always prefer the modern API in new code; `Path.toFile()`/`File.toPath()` bridge the two when interoperating with legacy APIs.