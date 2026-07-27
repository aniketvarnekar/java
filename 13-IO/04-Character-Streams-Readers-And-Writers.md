# Character Streams, Readers & Writers

## Learning Objectives

- Use `FileReader`/`FileWriter` and `BufferedReader`/`BufferedWriter` correctly and idiomatically
- Understand `InputStreamReader`/`OutputStreamWriter` as the explicit "bridge" between byte streams and character streams
- Read/write text files line-by-line, the standard, idiomatic Java pattern

## Prerequisites

[01 — IO Fundamentals, Streams & Encoding](01-IO-Fundamentals-Streams-And-Encoding.md), [03 — Byte Streams & Buffering](03-Byte-Streams-And-Buffering.md)

## Motivation

This topic completes the character-stream half of Topic 1's dual hierarchy, applying Topic 3's buffering lesson identically (the same Decorator pattern, same underlying performance reasoning) — plus revealing the explicit "bridge" classes that connect the byte and character worlds when you need fine-grained encoding control.

## `FileReader`/`FileWriter` — Basic Character Stream Access

```java
try (Reader reader = new FileReader("notes.txt", StandardCharsets.UTF_8)) {   // Java 11+ overload
    int c;
    while ((c = reader.read()) != -1) {
        System.out.print((char) c);
    }
}
```

This mirrors Topic 3's `FileInputStream` pattern exactly, but operating in terms of **characters** rather than raw bytes — `FileReader` internally handles the byte-to-character decoding for you (Topic 1), using the specified encoding.

## `BufferedReader`/`BufferedWriter` — The Same Decorator Pattern, Applied Here

```java
try (BufferedReader reader = new BufferedReader(new FileReader("notes.txt", StandardCharsets.UTF_8))) {
    String line;
    while ((line = reader.readLine()) != null) {   // readLine() -- reads an ENTIRE line at once!
        System.out.println(line);
    }
}
```

**This is exactly Topic 3's Decorator pattern and buffering rationale, applied identically to character streams** — `BufferedReader` wraps a `Reader`, adding an internal buffer for the same system-call-reduction performance benefit. **`BufferedReader` also adds a genuinely valuable new capability beyond plain buffering: `readLine()`** — reading an entire line of text at once (returning `null` at end-of-stream, not `-1`, since it returns a `String`, not a `char`/`int`), which plain `Reader` doesn't provide at all. **This `readLine()`-based pattern is the standard, idiomatic way to read text files line-by-line in Java** — you'll see and write this pattern constantly in real code.

```java
try (BufferedWriter writer = new BufferedWriter(new FileWriter("output.txt", StandardCharsets.UTF_8))) {
    writer.write("Line 1");
    writer.newLine();                     // platform-appropriate line separator (recall Module 08,
    writer.write("Line 2");                  // Topic 5's %n discussion -- same portability principle)
}
```

## `InputStreamReader`/`OutputStreamWriter` — The Explicit Bridge Classes

Recall Topic 1: before Java 11's `FileReader(String, Charset)` constructor overload existed, achieving **explicit** encoding control required these bridge classes directly:

```java
try (Reader reader = new InputStreamReader(new FileInputStream("notes.txt"), StandardCharsets.UTF_8)) {
    // 'reader' is a genuine character stream, built by explicitly bridging
    // a BYTE stream (FileInputStream) with a specified ENCODING
}
```

**`InputStreamReader` takes any `InputStream` (byte stream) and an explicit `Charset`, and produces a `Reader` (character stream)** — this is the literal, concrete mechanism that makes Topic 1's byte-to-character conversion happen, made fully visible and explicit rather than hidden inside `FileReader`'s convenience constructor. **`OutputStreamWriter` does the reverse** — taking a `Writer`'s character-based output and encoding it into bytes for an underlying `OutputStream`.

**Why does this bridge matter beyond just files?** It's the standard way to apply character-stream, encoding-aware handling to **any** byte-based source — most notably, **network sockets** (Module 19) and `System.in`:

```java
BufferedReader consoleReader = new BufferedReader(
    new InputStreamReader(System.in, StandardCharsets.UTF_8));   // reading CONSOLE input as encoded text

String userInput = consoleReader.readLine();
```

**This exact pattern — bridging `System.in` (a raw byte stream) through `InputStreamReader` into a `BufferedReader` — is the standard, idiomatic way to read line-based console input in classic Java** (modern alternatives like `Scanner` exist too, but this pattern reveals precisely what's happening underneath, which `Scanner` otherwise hides).

## The Complete Picture — Byte Stream to Buffered Character Stream

```
 FileInputStream            InputStreamReader              BufferedReader
 (raw BYTES from disk)  ──▶  (decodes bytes -> chars,  ──▶  (adds buffering +
                              using an explicit Charset)      readLine() capability)

      InputStream    ──bridge──▶     Reader          ──decorator──▶    Reader (buffered)
```

**This diagram is the complete, concrete resolution of Topic 1's "why two hierarchies" question** — `InputStreamReader` is the literal bridge connecting them, and `BufferedReader` then decorates the result with the same performance/convenience benefits Topic 3 established for byte streams.

## Real-World Analogy

Think of `InputStreamReader` like a **professional translator standing between a foreign-language radio broadcast (raw bytes) and you (expecting meaningful sentences)** — the translator needs to know **which specific language** the broadcast is actually in (the `Charset`) to translate correctly; get the language wrong, and you get Topic 1's "mojibake" nonsense instead of meaningful text. `BufferedReader` is like a **stenographer standing next to the translator, taking down entire sentences (`readLine()`) at once and efficiently batching their note-taking** rather than transcribing one word at a time.

## Advantages

- `BufferedReader`'s `readLine()` provides a genuinely convenient, idiomatic way to process text files line-by-line — far more ergonomic than manual character-by-character reading.
- `InputStreamReader`/`OutputStreamWriter` make the byte-to-character bridging fully explicit and controllable, applicable to any byte source (files, sockets, console input), not just files specifically.
- The same buffering performance benefits from Topic 3 apply identically here.

## Disadvantages / Trade-offs

- The full bridging chain (`FileInputStream` → `InputStreamReader` → `BufferedReader`) is genuinely more verbose than modern convenience methods (`Files.readString`/`readAllLines`, Topic 2) for simple, small-file use cases.
- Forgetting to specify an explicit `Charset` (Topic 1's warning) remains a real risk when using these classes' encoding-less constructor overloads.

## Best Practices

- Use `BufferedReader`/`BufferedWriter` for any non-trivial text file reading/writing — the performance and `readLine()`-convenience benefits are substantial.
- Always specify an explicit `Charset` (prefer `StandardCharsets.UTF_8`) when constructing `FileReader`/`FileWriter`/`InputStreamReader`/`OutputStreamWriter`.
- Use the `InputStreamReader`-bridging pattern specifically when you need character-stream handling over a non-file byte source (sockets, `System.in`).

## Common Mistakes

- Reading text files character-by-character instead of using `BufferedReader.readLine()`, producing more verbose, less idiomatic code.
- Confusing `read()`'s `-1` end-of-stream signal (for raw character reads) with `readLine()`'s `null` end-of-stream signal (since it returns a `String`).
- Constructing `FileReader`/`InputStreamReader` without an explicit `Charset`, reintroducing Topic 1's platform-default-encoding risk.

## Interview Questions

1. **Q: What does `InputStreamReader` do, precisely?**
   A: It bridges a byte stream (`InputStream`) into a character stream (`Reader`), decoding raw bytes into characters using an explicitly specified `Charset` — the literal, concrete mechanism underlying Topic 1's byte-to-character conversion, applicable to any byte source (files, sockets, console input).

2. **Q: What does `BufferedReader` add beyond plain buffering performance?**
   A: The `readLine()` method — reading an entire line of text in one call, returning `null` at end-of-stream — a genuinely convenient capability plain `Reader` doesn't provide, and the standard, idiomatic way to process text files line-by-line in Java.

3. **Q: How would you read console input as UTF-8-encoded text, line by line, using classic Java I/O classes?**
   A: `new BufferedReader(new InputStreamReader(System.in, StandardCharsets.UTF_8))`, then calling `.readLine()` — bridging `System.in` (a raw byte stream) through `InputStreamReader` (explicit encoding) into a `BufferedReader` (buffering + line-based reading).

## Summary

- `FileReader`/`FileWriter` provide basic character-stream file access; `BufferedReader`/`BufferedWriter` add Topic 3's buffering performance benefit plus `readLine()`/`newLine()` convenience.
- **`InputStreamReader`/`OutputStreamWriter`** are the explicit bridge classes connecting byte streams to character streams, with an explicit `Charset` — the literal mechanism behind Topic 1's byte-to-character conversion, usable for any byte source, not just files.
- The standard, idiomatic pattern for reading text (files, console input, sockets) is: byte stream → `InputStreamReader` (explicit encoding) → `BufferedReader` (buffering + `readLine()`).

## Exercises

1. Write code reading a text file line-by-line using `BufferedReader`, printing each line with its line number.
2. Rewrite `System.in` console-input reading using the full `InputStreamReader`-bridging pattern from this topic, and explain what each of the three wrapped layers contributes.
3. Explain precisely why `readLine()` returns `null` at end-of-stream while `read()` returns `-1`, connecting your answer to their different return types.

---

**Previous:** [03 — Byte Streams & Buffering](03-Byte-Streams-And-Buffering.md) · **Next:** [05 — Object Serialization](05-Object-Serialization.md)
