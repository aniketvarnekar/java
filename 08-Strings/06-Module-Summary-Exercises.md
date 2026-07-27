# Module 08 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-Module-Overview.md) has been covered:

- [x] **String Immutability** — precise definition, four concrete benefits (security, thread-safety, safe pooling, cacheable hashCode), `final` vs. immutability distinguished
- [x] **The String Constant Pool** — pool mechanics as a direct extension of Module 03's `Integer` cache, the complete `==` decision table across every common creation pattern, `intern()`, the absolute "always use `.equals()`" rule
- [x] **String Methods & API** — comprehensive practical coverage: length/access, substring (exclusive-end convention), searching, comparison, modification, split/join, conversion, Java 11's `strip()`/`isBlank()`
- [x] **StringBuilder & StringBuffer** — the concrete performance cost of loop-based concatenation, `StringBuilder`'s mutable buffer mechanism, the compiler's automatic single-expression optimization, `StringBuilder` vs `StringBuffer`'s synchronization trade-off
- [x] **String Formatting & Text Blocks** — `String.format`/`printf` specifiers, full text block depth (indentation-stripping rules), a preview of String Templates' ongoing evolution

## Practical Connections

- **REST API request/response handling** relies constantly on `String.format`/text blocks (Topic 5) for constructing JSON/error messages, and on the String Constant Pool understanding (Topic 2) to correctly reason about comparing header values, status strings, etc.
- **SQL query building** in JDBC code (Module 20) is one of the most common real-world uses of text blocks (Topic 5) — multi-line queries are dramatically more readable than the old concatenation approach.
- **Log message construction** in high-throughput services is a textbook real-world case for `StringBuilder` (Topic 4) over naive concatenation, given how frequently logging code runs in production.
- **Every `HashMap<String, ...>` in existence** depends on `String`'s cached, immutability-enabled `hashCode()` (Topic 1) for its performance characteristics — this is not an abstract concern; it's foundational to Module 10's entire Collections performance story.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| `final String` vs. String immutability | `final` prevents variable reassignment; immutability is a separate, stronger, class-design-level guarantee that no `String` object's content can ever be mutated, `final` or not. |
| `==` vs. `.equals()` for Strings | `==` compares identity (same pooled or non-pooled object); `.equals()` compares content — always use `.equals()`, no exceptions. |
| `trim()` vs. `strip()` | `trim()` uses narrow, legacy ASCII whitespace rules; `strip()` (Java 11+) is Unicode-aware and the modern preferred choice. |
| `String` concatenation vs. `StringBuilder` | A single-expression `+` is compiler-optimized automatically; loop-based repeated concatenation is not, and should use a manually-managed `StringBuilder` instead. |
| `StringBuilder` vs. `StringBuffer` | Identical API; `StringBuffer` is `synchronized` (thread-safe, slower); `StringBuilder` is not (faster, the correct default for single-threaded use). |

## Consolidated Interview Questions (Module 08)

1. Is `String` immutable? How do you know, and what are the concrete benefits this enables?
2. Does `final String s = "Hello";` make the content immutable? Why or why not?
3. Why does `"hello" == "hello"` evaluate to `true` but `new String("hello") == "hello"` evaluate to `false`?
4. What does `String.intern()` do?
5. Should you ever use `==` to compare String content in real code?
6. Is `substring`'s ending index inclusive or exclusive?
7. What does `indexOf` return for "not found," and why not an exception?
8. Why is repeated String concatenation inside a loop inefficient?
9. What's the difference between `StringBuilder` and `StringBuffer`?
10. Does a single-expression concatenation (`"a" + b + "c"`) suffer the same performance problem as loop-based concatenation?
11. How does a text block determine how much leading whitespace to strip?
12. Why does `%s` in `String.format` work correctly for any object?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** From memory, list all four concrete benefits `String`'s immutability enables, with a one-sentence mechanism for each.
2. **Hands-on:** Write a program demonstrating all five rows of Topic 2's `==` decision table, printing the actual result of each comparison to confirm your predictions.
3. **Hands-on:** Write a method that takes a `List<String>` and joins them into a single comma-separated `String` using `StringBuilder`, then rewrite it using `String.join(...)` (Topic 3) and compare the two approaches.
4. **Hands-on:** Write a text block representing a multi-line SQL query with intentional source-code indentation, print it, and verify the output has the expected (stripped) indentation.
5. **Conceptual:** Explain, referencing Module 07's `equals()`/`hashCode()` contract, why `String`'s immutability makes it an especially safe and efficient choice as a `HashMap` key type.
6. **Synthesis:** Write a method that builds a large formatted report (multiple lines, using `%-10s`/`%8.2f`-style specifiers) using `StringBuilder` and `String.format` together, demonstrating both this module's formatting tools and its performance guidance in one piece of code.

## What's Next

Module 08 completed your mastery of Java's most-used class. **Module 09 — Arrays** covers Java's most fundamental data structure: how arrays are actually stored in memory (tying directly back to Module 02's Heap model), multidimensional arrays, the `java.util.Arrays` utility class, and array-vs-`ArrayList` — setting up the entire Collections Framework in Module 10.

---

**Previous:** [05 — String Formatting & Text Blocks](05-String-Formatting-And-Text-Blocks.md) · **Module Overview:** [00 — Module Overview](00-Module-Overview.md)

**Type "Continue" to begin Module 09 — Arrays.**
