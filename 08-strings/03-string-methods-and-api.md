# String Methods & API

## Learning Objectives

- Use the most important, most commonly needed `String` methods correctly and confidently
- Understand String indexing conventions precisely (0-based, end-exclusive ranges)
- Know the modern (Java 11+) whitespace/blank-checking methods and why they were added

## Prerequisites

[01 — String Immutability](01-string-immutability.md)

## Motivation

This topic is intentionally more reference-oriented than deeply conceptual — you'll use these methods constantly, in nearly every Java program you ever write, so fluency matters more than theory here. Every method shown returns a **new** `String` (Topic 1) — this is worth remembering once, rather than repeating for every single method below.

## Length and Character Access

```java
String s = "Hello";
s.length();          // 5
s.charAt(0);           // 'H'  -- 0-INDEXED, like arrays (Module 09)
s.charAt(4);            // 'o'
s.charAt(5);             // throws StringIndexOutOfBoundsException -- valid indices are 0 to length()-1
s.isEmpty();               // false -- true only for ""
s.isBlank();                 // false -- (Java 11+) true for "" OR strings containing ONLY whitespace
```

**Why does `isBlank()` exist alongside `isEmpty()` (Java 11+)?** `isEmpty()` only checks for **zero-length** strings — a string like `"   "` (all spaces) is **not** empty, but is very often *logically* meaningless the same way an empty string is, for real-world validation purposes (a form field containing only spaces should usually be treated the same as a blank one). `isBlank()` was added specifically to express this extremely common validation need directly, without manually calling `.trim().isEmpty()` every time.

## Substrings

```java
String s = "Hello, World!";
s.substring(7);         // "World!"        -- from index 7 to the END
s.substring(7, 12);       // "World"         -- from index 7 UP TO (NOT including) index 12
s.substring(0, 5);         // "Hello"
```

**Critical rule: `substring(begin, end)`'s `end` index is EXCLUSIVE** — the character *at* index `end` is **not** included. This "end-exclusive" convention appears constantly throughout Java's standard library (also in array-related methods, Module 09) — internalizing it now pays off repeatedly later. A quick way to verify: `end - begin` always equals the length of the resulting substring (`12 - 7 = 5`, matching `"World"`.length() == 5).

## Searching

```java
String s = "Hello, World!";
s.indexOf('o');            // 4    -- index of the FIRST occurrence
s.indexOf('o', 5);           // 8    -- first occurrence AT OR AFTER index 5
s.indexOf("World");            // 7
s.indexOf("xyz");                // -1   -- NOT FOUND, indicated by -1 (not an exception!)
s.lastIndexOf('o');                // 8    -- LAST occurrence
s.contains("World");                 // true
s.startsWith("Hello");                 // true
s.endsWith("!");                         // true
```

**`indexOf` returning `-1` for "not found" (rather than throwing an exception) is a deliberate design choice** — searching for something that might not exist is a perfectly normal, expected outcome, not an exceptional/error condition (a distinction that becomes fully precise once you reach Module 12's checked-vs-unchecked exception philosophy).

## Comparison

```java
String s1 = "Hello";
s1.equals("Hello");             // true  -- content equality (ALWAYS prefer this over ==, Topic 2)
s1.equalsIgnoreCase("HELLO");    // true
s1.compareTo("World");            // negative -- 'H' comes before 'W' alphabetically (lexicographic order)
s1.compareTo("Hello");              // 0 -- equal
s1.compareTo("Apple");                // positive -- 'H' comes after 'A'
```

`compareTo` returns a **negative**, **zero**, or **positive** `int` — not specifically `-1`/`0`/`1` (though often implemented that way) — following the same "negative/zero/positive" convention used throughout Java's `Comparable` interface (full depth: Module 10/11). It compares strings **lexicographically** — character by character, using each character's underlying numeric/Unicode value (Module 03, Topic 3's `char` discussion).

## Modification (Returns a New String — Recall Topic 1)

```java
String s = "  Hello World  ";
s.trim();                  // "Hello World"  -- removes LEADING/TRAILING whitespace (ASCII-only rules)
s.strip();                   // "Hello World"  -- (Java 11+) Unicode-aware version of trim()
s.toUpperCase();                // "  HELLO WORLD  "
s.toLowerCase();                  // "  hello world  "
s.replace('o', '0');                // "  Hell0 W0rld  "  -- replaces ALL occurrences of a CHARACTER
s.replace("World", "Java");           // "  Hello Java  "    -- replaces ALL occurrences of a SUBSTRING
```

**Why `strip()` alongside `trim()` (Java 11+)?** `trim()` predates Java's full Unicode-awareness maturity and only recognizes a narrow, ASCII-defined notion of "whitespace." `strip()` correctly handles the **full Unicode** definition of whitespace, which matters for genuinely international text processing. **Best practice in modern Java: prefer `strip()` over `trim()`** unless you have a specific reason to rely on `trim()`'s narrower, legacy ASCII-only behavior.

## Splitting and Joining

```java
String csv = "apple,banana,cherry";
String[] fruits = csv.split(",");     // ["apple", "banana", "cherry"]  -- an ARRAY (Module 09!)

String joined = String.join(", ", "apple", "banana", "cherry");  // "apple, banana, cherry"
String joined2 = String.join("-", fruits);                          // "apple-banana-cherry"
```

`split(...)` takes a **regular expression** (regex — a pattern-matching mini-language, worth knowing exists even before dedicated regex coverage) — for simple delimiters like `","`, this works exactly as expected, but be aware that characters with special regex meaning (like `.` or `|`) need escaping if used as a literal delimiter (`split("\\.")` to split on a literal period, for instance).

## Conversion To/From Other Types

```java
String s = String.valueOf(42);        // "42"  -- converts virtually ANY type to a String
String s2 = String.valueOf(3.14);        // "3.14"
String s3 = String.valueOf(true);          // "true"
String s4 = Integer.toString(42);            // "42" -- an alternative, type-specific approach

int n = Integer.parseInt("42");                 // 42  -- String -> int
double d = Double.parseDouble("3.14");             // 3.14
```

**`Integer.parseInt(...)` throws `NumberFormatException`** (a real, common exception — full depth Module 12) if the String isn't a validly formatted number — always be prepared to handle this when parsing untrusted input (user input, file/network data).

```java
int n = Integer.parseInt("abc");   // throws NumberFormatException: For input string: "abc"
```

## Immutable Doesn't Mean "No Useful Char Array Access"

```java
char[] chars = "Hello".toCharArray();   // converts to a char[] (Module 09) -- a genuine, independent COPY,
                                           // NOT a live view into the String's internal data (which,
                                           // recall Topic 1, is never exposed for mutation anyway)
```

## Quick Reference Table

| Category | Methods |
|---|---|
| Info | `length()`, `isEmpty()`, `isBlank()` (11+), `charAt(int)` |
| Search | `indexOf`, `lastIndexOf`, `contains`, `startsWith`, `endsWith` |
| Compare | `equals`, `equalsIgnoreCase`, `compareTo` |
| Extract | `substring(begin)`, `substring(begin, end)` |
| Transform | `trim`, `strip` (11+), `toUpperCase`, `toLowerCase`, `replace` |
| Split/Join | `split(regex)`, `String.join(delim, ...)` |
| Convert | `String.valueOf(...)`, `Integer.parseInt(...)`, `toCharArray()` |

## Real-World Analogy

Think of these methods like a **fully-stocked kitchen full of pre-made tools** — a peeler, a slicer, a measuring cup. None of them modify the original ingredient bag itself; each one takes what you give it and hands you back a **new, prepared result** (a peeled potato, a measured cup of flour), leaving the original bag of ingredients completely untouched — exactly mirroring `String`'s "every method returns something new" immutability from Topic 1.

## Advantages

- A rich, comprehensive, well-designed API covers the overwhelming majority of everyday text-processing needs without external libraries.
- Consistent conventions (0-based indexing, end-exclusive ranges, `-1` for "not found") reduce the cognitive load of learning new methods, once the conventions themselves are internalized.

## Disadvantages / Trade-offs

- The sheer number of methods can feel overwhelming to a beginner — fluency comes from repeated, practical use, not memorization in one sitting.
- Regex-based methods (`split`, and others not covered here like `matches`/`replaceAll`) carry real regex-specific gotchas (special characters needing escaping) — a full regex treatment is beyond this module's scope but worth knowing exists.

## Best Practices

- Prefer `strip()` over `trim()` in modern Java for correct Unicode handling.
- Always handle `NumberFormatException` when parsing untrusted numeric input.
- Remember `substring`'s end-exclusive convention — verify with `end - begin == expected length` when unsure.

## Common Mistakes

- Off-by-one errors with `substring(begin, end)`, forgetting `end` is exclusive.
- Forgetting `indexOf` returns `-1` (not an exception) for "not found," and failing to check for it before using the result as an index.
- Assuming `split(",")` and similar simple-looking delimiters never need special handling — forgetting that `split` takes a regex, which matters once the delimiter itself is a regex special character (like `.` or `|`).

## Interview Questions

1. **Q: Is `substring`'s ending index inclusive or exclusive?**
   A: Exclusive — `s.substring(begin, end)` includes characters from index `begin` up to, but not including, index `end`. The resulting substring's length always equals `end - begin`.

2. **Q: What does `indexOf` return when the target isn't found, and why isn't an exception thrown instead?**
   A: `-1`. Not finding a substring is a normal, expected outcome of a search, not an exceptional/error condition — throwing an exception for a routine "not found" result would be inappropriate and inefficient for such a common case.

3. **Q: Why was `strip()` added in Java 11 when `trim()` already existed?**
   A: `trim()` only recognizes a narrow, ASCII-defined notion of whitespace, predating Java's full Unicode maturity. `strip()` correctly handles the full Unicode definition of whitespace, making it the more correct choice for genuinely international text — modern Java code should generally prefer it.

## Summary

- Every `String` method that appears to modify content actually returns a **new** `String` (Topic 1's immutability, applied practically).
- Indexing is 0-based; `substring`'s end index is exclusive; `indexOf` returns `-1` for "not found," never throwing.
- Java 11 added Unicode-aware, validation-friendly methods (`strip()`, `isBlank()`) alongside their older, narrower counterparts (`trim()`, `isEmpty()`).
- `split()` takes a regex; parsing methods (`Integer.parseInt`, etc.) throw `NumberFormatException` on invalid input.

## Exercises

1. Given `String s = "  The Quick Brown Fox  ";`, write the exact method chain to produce `"the_quick_brown_fox"` (trimmed, lowercase, spaces replaced with underscores).
2. Explain why `"hello".substring(2, 5)` produces `"llo"` and not `"llo "` or `"ll"`, referencing the exclusive-end-index convention precisely.
3. Write a small method that safely parses a `String` to an `int`, returning `-1` if the input isn't a valid number, using a `try`/`catch` around `Integer.parseInt` (based on what you've seen of exceptions so far — full depth comes in Module 12).

---

**Previous:** [02 — The String Constant Pool](02-string-constant-pool.md) · **Next:** [04 — StringBuilder & StringBuffer](04-stringbuilder-and-stringbuffer.md)
