# String Formatting & Text Blocks

## Learning Objectives

- Use `String.format()` and `printf()` with common format specifiers
- Fully master text blocks (Java 15+), including indentation handling rules
- Know what's next in Java's string-templating evolution

## Prerequisites

[04 — StringBuilder & StringBuffer](04-StringBuilder-And-StringBuffer.md), Module 03 Topic 3 (Literals — text blocks preview)

## Motivation

Concatenating variables into readable, well-formatted output (currency, padded numbers, dates) via `+` quickly becomes unreadable. This topic covers Java's dedicated formatting tools, and gives text blocks (previewed briefly in Module 03) the full, complete treatment promised there.

## `String.format()` and `printf()`

```java
String s = String.format("Name: %s, Age: %d, Score: %.2f", "Aniket", 30, 95.5);
System.out.println(s);   // "Name: Aniket, Age: 30, Score: 95.50"

System.out.printf("Name: %s, Age: %d%n", "Priya", 28);   // printf() = format() + print directly
```

**`String.format(...)` returns a formatted `String`**; **`printf(...)`** (available on `System.out` and other `PrintStream`s) does the same formatting and **prints it immediately** — equivalent to `System.out.print(String.format(...))`. Both use the exact same **format specifier** syntax.

### Common Format Specifiers

| Specifier | Meaning | Example |
|---|---|---|
| `%s` | String (calls `toString()` on the argument — Module 07, Topic 2!) | `%s` → `"Aniket"` |
| `%d` | Integer (decimal) | `%d` → `42` |
| `%f` | Floating-point | `%f` → `3.140000` (default 6 decimal places) |
| `%.2f` | Floating-point, 2 decimal places | `%.2f` → `3.14` |
| `%n` | Platform-specific newline (prefer over `\n` for portability) | |
| `%%` | A literal `%` character | |
| `%5d` | Integer, right-padded to width 5 | `%5d` with `42` → `"   42"` |
| `%-5d` | Integer, left-aligned in width 5 | `%-5d` with `42` → `"42   "` |
| `%05d` | Integer, zero-padded to width 5 | `%05d` with `42` → `"00042"` |
| `%x` | Hexadecimal | `%x` with `255` → `"ff"` |

```java
System.out.printf("%-10s | %5d | %8.2f%n", "Widget", 42, 19.999);
// "Widget     |    42 |    20.00"
```

**Why `%n` instead of `\n`?** `\n` is specifically the Unix-style line feed character; `%n` inserts whatever the **current platform's** native line separator actually is (`\n` on Linux/macOS, `\r\n` on Windows) — a small but genuine portability detail, directly connected to Module 01's platform-independence discussion: your *text output formatting* can still have platform-specific concerns, even though your *bytecode* is fully portable.

**Why does `%s` call `toString()`?** This is precisely why Module 07, Topic 2's advice to override `toString()` pays off yet again here — `%s` works correctly and meaningfully for **any** object, not just Strings, by calling its `toString()` automatically.

## Text Blocks — Full Depth (Previewed in Module 03)

Recall Module 03, Topic 3's brief introduction. Here's the complete picture:

```java
String json = """
              {
                "name": "Aniket",
                "age": 30
              }
              """;
```

A text block starts with **three double-quotes followed immediately by a line break** (`"""` then newline — content cannot start on the same line as the opening `"""`), and ends with a closing `"""`.

### Indentation Handling — The Rule That Surprises Beginners

Text blocks perform **automatic, deliberate "incidental whitespace" stripping**, based on the **least-indented line** in the block (including the closing `"""`'s own line):

```java
void demo() {
    String html = """
                  <html>
                    <body>
                    </body>
                  </html>
                  """;
    System.out.println(html);
}
```

Even though this text block is indented to match the surrounding Java code (good practice for readability in your source file), the **printed output** has that common leading indentation **stripped away**, leaving only the *relative* indentation between lines intact:

```
<html>
  <body>
  </body>
</html>
```

**How is the "common leading whitespace" amount determined, precisely?** The compiler looks at **every line's** leading whitespace (including the closing `"""` delimiter's own line) and strips exactly the **minimum common amount** shared by all of them — this is what lets you indent the entire text block naturally to match your surrounding code's indentation level, without that indentation leaking into the actual string value. **Moving the closing `"""` further left than the content** effectively reduces how much gets stripped (since it becomes the new "least indented line"), giving you deliberate control over the final result's leading whitespace, if you genuinely need some.

### Other Text Block Rules

```java
String s1 = """
            Line one
            Line two""";     // NO trailing newline (closing """ immediately follows the last line)

String s2 = """
            Line one
            Line two
            """;               // trailing newline included (closing """ is on its OWN line)

String s3 = """
            Trailing spaces preserved with \s at end of a line \s
            """;                  // \s = an explicit space that WON'T be stripped as trailing whitespace
```

**Escape sequences (`\n`, `\"`, etc.) still work normally inside text blocks** — the main practical difference from regular String literals is you generally **don't need** to escape internal `"` characters (since the block is delimited by `"""`, not a single `"`), and you don't need `\n` for line breaks (an actual newline in the source does that for you).

### Why Text Blocks Were Added (Recap, With Full Context Now)

Recall Module 03, Topic 3: embedded multi-line text (JSON, SQL, HTML) is extremely common in real backend development, and the old approach (`"line1\n" + "line2\n" + ...`) was verbose and error-prone. Text blocks solve this while **intelligently preserving your source code's own readable indentation**, without that indentation corrupting the actual string content — a genuinely well-designed feature addressing a real, everyday pain point.

## A Brief Look Ahead: String Templates

Java experimented with a **String Templates** preview feature (introduced as a preview in Java 21, intended to let you embed expressions directly inside string literals, e.g., something like `STR."Hello, \{name}!"` instead of `"Hello, " + name + "!"` or `String.format(...)`) as a further evolution beyond text blocks and `String.format`. **As of this writing, this feature's exact final form is still evolving** and was withdrawn as a preview in Java 23 for redesign — worth knowing the *direction* Java is exploring (more ergonomic, safer string interpolation), without over-committing to specifics that may still change. Module 23 (Modern Java) will cover whatever the finalized state is by the time you reach it, with full detail.

## Real-World Analogy

Think of `String.format`/`printf`'s specifiers like a **fill-in-the-blank form template** — `"Dear %s, your balance is %.2f"` is the blank form; the arguments are what you write into each blank, formatted according to that blank's specific rules (text vs. a 2-decimal number). Text blocks are like being handed a **pre-formatted document you can type directly into**, preserving its natural layout, rather than having to manually reconstruct that layout line by line with explicit line-break markers.

## Advantages

- `String.format`/`printf` provide precise, readable control over numeric formatting, padding, and alignment — far cleaner than manual concatenation for anything beyond trivial cases.
- Text blocks dramatically improve readability for embedded multi-line text, with genuinely thoughtful automatic indentation handling.

## Disadvantages / Trade-offs

- Format specifier syntax (`%-10s`, `%08.2f`, etc.) has a real learning curve and can be cryptic to read until memorized.
- Text blocks' automatic indentation-stripping rule, while sensible once understood, can produce initially surprising results if you don't understand exactly how the "minimum common indentation" is computed.

## Best Practices

- Use `String.format`/`printf` for any output involving numeric alignment, padding, or fixed decimal precision — far more reliable and readable than manual string-building for these cases.
- Use text blocks for any embedded multi-line text (SQL, JSON, HTML) instead of manual `+`/`\n` concatenation.
- Prefer `%n` over `\n` in format strings for platform-correct line separators.

## Common Mistakes

- Forgetting `%s` calls `toString()` and being surprised when a custom object without a good `toString()` (Module 07, Topic 2) prints unhelpful default output inside a formatted string.
- Misjudging text blocks' indentation-stripping rule, especially when the closing `"""` isn't aligned the way the content is.
- Forgetting `%d` is for integers and `%f` is for floating-point — using the wrong specifier throws `IllegalFormatConversionException` at runtime.

## Interview Questions

1. **Q: What's the difference between `String.format(...)` and `printf(...)`?**
   A: `String.format(...)` returns a formatted `String`; `printf(...)` performs the same formatting and prints the result immediately — functionally equivalent to `System.out.print(String.format(...))`.

2. **Q: How does a text block determine how much leading whitespace to strip from each line?**
   A: The compiler finds the minimum common leading whitespace shared across every line in the block (including the closing `"""` delimiter's own line) and strips exactly that amount from all lines, preserving each line's *relative* indentation while removing the whitespace incidental to matching your source code's own formatting.

3. **Q: Why does `%s` work correctly for any object, not just String arguments?**
   A: `%s` internally calls the argument's `toString()` method (Module 07, Topic 2) — which is why overriding `toString()` meaningfully on custom classes pays off in formatted output too, not just direct `println` calls.

## Summary

- `String.format`/`printf` use `%`-prefixed format specifiers (`%s`, `%d`, `%.2f`, `%n`, etc.) for precise, readable text formatting; `%s` calls `toString()` on any object.
- **Text blocks** (`"""`) support clean, multi-line embedded text with automatic, deliberate common-indentation stripping, letting source-code indentation stay natural without corrupting the actual string value.
- Java's string-formatting story continues evolving (String Templates), with the finalized modern state covered fully in Module 23.

## Module-Wide Quick Revision

- `String` is immutable by class design (not the same as `final`); every "modifying" method returns a new object (Topic 1).
- Immutability enables security, thread-safety, safe pooling, and cacheable hash codes (Topic 1).
- The String Constant Pool shares identical literals; `new String(...)` and runtime concatenation bypass it; `intern()` manually opts in; **always use `.equals()`, never `==`** (Topic 2).
- Core String API: 0-based indexing, `substring`'s exclusive end index, `indexOf` returns -1 for "not found," `strip()`/`isBlank()` (Java 11+) over `trim()`/`isEmpty()` for Unicode correctness (Topic 3).
- Repeated concatenation in loops is expensive due to immutability; `StringBuilder` mutates one buffer in place; `StringBuffer` adds unnecessary synchronization overhead for the common single-threaded case (Topic 4).
- `String.format`/`printf` for precise formatting; text blocks for clean multi-line text (this topic).

## Common Pitfalls (Module-Wide)

- Forgetting to capture a String method's return value, expecting in-place mutation.
- Comparing Strings with `==` instead of `.equals()`.
- Building large strings with repeated `+=` inside loops instead of `StringBuilder`.
- Off-by-one errors with `substring`'s exclusive end index.
- Misjudging text blocks' automatic indentation stripping.

## Mini Quiz (Module-Wide)

1. Does `s.trim()` modify `s` in place? Why or why not?
2. Why does `"hello" == "hello"` return `true` but `new String("hello") == "hello"` return `false`?
3. What does `"hello".substring(1, 3)` evaluate to?
4. Why is `StringBuilder` preferred over `+=` concatenation inside a loop?
5. How does a text block decide how much leading whitespace to strip?

*(Answers are derivable from Topics 1, 2, 3, 4, and this topic, respectively.)*

---

**Previous:** [04 — StringBuilder & StringBuffer](04-StringBuilder-And-StringBuffer.md) · **Next:** [06 — Module Summary, Interview Questions & Exercises](06-Module-Summary-Exercises.md)
