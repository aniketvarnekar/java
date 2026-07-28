# IO Fundamentals, Streams & Encoding

## Learning Objectives

- Understand the stream abstraction underlying all of `java.io`
- Understand precisely why Java maintains two parallel class hierarchies — byte streams and character streams
- Understand character encoding well enough to explain real, common "mojibake" (garbled text) bugs

## Prerequisites

Module 12 Topic 4 (try-with-resources), Module 08 Topic 1 (String's char-based, Unicode-aware design)

## Motivation

Nearly every I/O bug that isn't a simple "file not found" turns out to be an encoding problem — text that displays as garbled symbols, or an `OutOfMemoryError` from reading a huge file all at once. This topic builds the conceptual foundation that makes both of these fully explicable, rather than mysterious.

## The Stream Abstraction

> A **stream** represents a sequential flow of data, from a **source** (a file, network socket, memory buffer) to your program, or from your program to a **destination** — read or written **one chunk at a time**, rather than requiring the entire dataset to exist in memory at once.

**Why streams, rather than just "read the whole file into memory"?** A file could be gigabytes in size — loading it entirely into memory (Module 02, Topic 3's Heap) before processing a single byte would be wasteful at best, and could throw `OutOfMemoryError: Java heap space` (Module 02, Topic 3) for sufficiently large files. Streams let you process data **incrementally**, using a small, fixed amount of memory regardless of the total data size.

## Two Parallel Hierarchies — Byte Streams and Character Streams

```
                    BYTE STREAMS                      CHARACTER STREAMS
                (raw 8-bit data)                    (Unicode TEXT, char-based)

              InputStream (abstract)                  Reader (abstract)
                     │                                      │
        ┌────────────┼────────────┐            ┌────────────┼────────────┐
        ▼             ▼            ▼            ▼             ▼            ▼
 FileInputStream  BufferedInputStream  ...  FileReader  BufferedReader  InputStreamReader
                                                                        (the BRIDGE, Topic 4)

              OutputStream (abstract)                 Writer (abstract)
                     │                                      │
        ┌────────────┼────────────┐            ┌────────────┼────────────┐
        ▼             ▼            ▼            ▼             ▼            ▼
 FileOutputStream  BufferedOutputStream  ...  FileWriter  BufferedWriter  OutputStreamWriter
```

**Why does Java maintain two entirely separate class hierarchies, rather than one unified stream type?** This is the central question this topic answers precisely.

## The Core Problem: Bytes Are Not Characters

Recall Module 03, Topic 2: a `char` in Java is a 16-bit Unicode code unit. But **files and network connections fundamentally store and transmit raw bytes (8-bit values)** — there's no such thing as a "character" at the file-system or network level, only sequences of bytes. **Converting between "a sequence of bytes" and "a sequence of meaningful characters" requires an agreed-upon mapping — a character encoding** (UTF-8, UTF-16, ASCII, ISO-8859-1, and others).

```
 The letter "é" (e with an acute accent):

  In UTF-8 encoding:     2 bytes:  0xC3 0xA9
  In UTF-16 encoding:    2 bytes:  0x00 0xE9   (different bytes, SAME character!)
  In ISO-8859-1 encoding: 1 byte:   0xE9

  READING those bytes with the WRONG encoding produces GARBLED TEXT ("mojibake") --
  e.g., reading UTF-8 bytes 0xC3 0xA9 as if they were ISO-8859-1 produces "Ã©" instead of "é"
```

**This is precisely why "mojibake" (garbled text) bugs happen in the real world** — a file was **written** using one encoding, but **read** assuming a different encoding, and the byte-to-character mapping simply doesn't match, producing visually nonsensical output despite every individual byte being read correctly.

## Byte Streams — For Raw Binary Data (No Encoding Concept At All)

```java
try (InputStream in = new FileInputStream("image.png")) {
    int byteValue = in.read();   // returns ONE byte (0-255), or -1 at end of stream
    // ...
}
```

`InputStream`/`OutputStream` and their subclasses deal in **raw bytes**, with **zero concept of character encoding at all** — entirely appropriate for genuinely binary data (images, compiled `.class` files, audio, video) where there's no "text" interpretation to worry about in the first place.

## Character Streams — For Text, With Explicit Encoding Awareness

```java
try (Reader reader = new FileReader("notes.txt", StandardCharsets.UTF_8)) {   // ENCODING specified explicitly
    int charValue = reader.read();   // returns ONE character (properly decoded), or -1 at end of stream
}
```

`Reader`/`Writer` and their subclasses deal in **characters**, internally handling the byte-to-character encoding/decoding **for you** — this is precisely why a **separate** hierarchy exists: byte streams and character streams solve genuinely different problems (raw binary data vs. properly-encoded text), and conflating them into one API would either force unnecessary encoding overhead onto binary data, or force binary-data-style raw-byte handling onto text — both bad outcomes Java's designers deliberately avoided by keeping the two hierarchies cleanly separate.

## Why This Matters: Always Specify Your Encoding Explicitly

**A genuinely important, real-world best practice**: never rely on a platform's **default** character encoding (historically a common source of "works on my machine, breaks in production" bugs, since different operating systems/locales can have different defaults) — **always specify the encoding explicitly**:

```java
// RISKY -- uses the JVM's platform-default encoding, which VARIES across machines/environments:
Reader reader = new FileReader("notes.txt");

// CORRECT -- explicit, unambiguous, portable:
Reader reader2 = new InputStreamReader(new FileInputStream("notes.txt"), StandardCharsets.UTF_8);
```

**Since Java 11**, `FileReader`/`FileWriter` gained overloaded constructors accepting an explicit `Charset` directly (as shown in the character-stream example above) — before Java 11, achieving explicit encoding control required the `InputStreamReader`/`OutputStreamWriter` **bridge classes** (full depth: Topic 4), wrapping a byte stream and explicitly specifying the encoding to use for the byte-to-character conversion.

**Modern, universal guidance: default to UTF-8 everywhere**, explicitly, unless you have a specific, deliberate reason to use a different encoding — UTF-8 is the dominant, most broadly compatible modern standard, and Java 18+ actually changed the platform **default** encoding to UTF-8 specifically to reduce this entire class of cross-platform bugs (though explicit specification remains the best practice regardless, for clarity and to not depend on any default at all).

## Real-World Analogy

Think of byte streams like shipping a **sealed box of unspecified physical objects** — the shipping company (the `InputStream`/`OutputStream` API) transports it faithfully without ever needing to know or care what's inside. Think of character streams like shipping a **letter written in a specific language** — the postal service (the `Reader`/`Writer` API) doesn't just move paper around; it needs to know the shared **alphabet/encoding convention** both the sender and recipient are using, or the recipient will receive a technically-intact letter that reads as complete gibberish, despite every physical character on the page arriving perfectly preserved.

## Advantages

- Streams process data incrementally, using bounded memory regardless of total data size, directly avoiding `OutOfMemoryError` for large files.
- The byte/character split cleanly separates "raw binary data" concerns from "properly-encoded text" concerns, avoiding forcing inappropriate handling onto either kind of data.
- Explicit encoding specification (modern best practice) eliminates an entire, historically common class of cross-platform text-corruption bugs.

## Disadvantages / Trade-offs

- Two parallel class hierarchies genuinely add learning surface area compared to a single, unified stream API.
- Relying on platform-default encoding (still legal, if discouraged) remains a real, live footgun in code that doesn't follow the explicit-encoding best practice.

## Best Practices

- Always specify character encoding explicitly (`StandardCharsets.UTF_8`) when working with character streams — never rely on platform defaults.
- Use byte streams for genuinely binary data (images, compiled files); use character streams for text.
- Default to UTF-8 unless you have a specific, deliberate reason to use a different encoding.

## Common Mistakes

- Relying on the JVM's platform-default encoding, producing text-corruption bugs that only manifest on certain machines/environments.
- Using byte streams to read text data manually (byte by byte) instead of the properly encoding-aware character stream classes.
- Reading an entire large file into memory at once instead of processing it incrementally via streams.

## Interview Questions

1. **Q: Why does Java maintain two separate class hierarchies for I/O — byte streams and character streams?**
   A: Because files and network data are fundamentally sequences of raw bytes, while text is a sequence of meaningful characters, and converting between the two requires an explicit, agreed-upon character encoding (like UTF-8). Byte streams (`InputStream`/`OutputStream`) handle raw binary data with no encoding concept at all; character streams (`Reader`/`Writer`) handle text, performing byte-to-character encoding/decoding internally — conflating the two would force inappropriate handling onto one or the other.

2. **Q: What causes "mojibake" (garbled text) bugs, precisely?**
   A: Text data was written using one character encoding, but later read/interpreted assuming a different encoding — the same byte sequence maps to different characters under different encodings, so mismatched read/write encodings produce visually nonsensical output despite every byte being technically correct.

3. **Q: Why should you always specify character encoding explicitly rather than relying on defaults?**
   A: The JVM's platform-default encoding historically varied across operating systems/locales, producing "works on my machine, breaks elsewhere" bugs. Explicit encoding specification (typically UTF-8) makes behavior portable and predictable, independent of whatever machine the code happens to run on.

## Summary

- **Streams** process data incrementally, avoiding the need to hold entire datasets in memory at once.
- Java maintains two parallel hierarchies: **byte streams** (`InputStream`/`OutputStream`, raw bytes, no encoding concept) for binary data, and **character streams** (`Reader`/`Writer`, encoding-aware) for text.
- **Character encoding** (UTF-8, UTF-16, etc.) maps between bytes and characters — mismatched read/write encodings cause "mojibake" garbled text.
- Always specify encoding explicitly (prefer UTF-8) rather than relying on platform defaults, for portable, predictable behavior.

## Exercises

1. Explain, in your own words, why `InputStream` and `Reader` are separate class hierarchies rather than one unified stream type.
2. Explain precisely how the same underlying text data, written in UTF-8 but read assuming ISO-8859-1, could produce garbled output despite no bytes being lost or corrupted.
3. Rewrite `new FileReader("notes.txt")` to explicitly specify UTF-8 encoding, and explain why this is considered best practice over the implicit, platform-default version.

---

**Previous:** [00 — Module Overview](00-module-overview.md) · **Next:** [02 — File & Path Handling](02-file-and-path-handling.md)
