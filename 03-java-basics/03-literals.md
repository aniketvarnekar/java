# Literals

## Learning Objectives

- Write literal values correctly for every primitive type and String
- Use alternate integer literal bases (binary, octal, hex) and numeric underscores
- Use character escape sequences correctly
- Understand text blocks (Java 15+) as an evolution of String literals

## Prerequisites

[02 — Primitive Data Types](02-primitive-data-types.md)

## Motivation

A **literal** is a fixed value written directly into your source code — `42`, `3.14`, `'A'`, `"hello"`, `true`. You've been using them since `HelloWorld.java`. This topic makes the rules around them precise, because several of Java's literal rules (type suffixes, numeric underscores, escape sequences) are frequently tested and frequently used incorrectly by beginners.

## Concept: What Is a Literal?

> A **literal** is source code text that directly represents a fixed value of a specific type, as opposed to a variable (which represents a *named, changeable* storage location) or an expression (which represents a *computed* value).

## Integer Literals

By default, an integer literal is typed `int`. To specify `long`, append `L` (uppercase preferred — lowercase `l` is legal but easily confused with the digit `1`, a real, common style mistake to avoid):

```java
int a = 100;
long b = 100L;              // 'L' suffix required if outside int's range, or for clarity
long big = 3000000000L;      // REQUIRED here -- 3 billion exceeds int's ~2.1 billion max
```

```java
long broken = 3000000000;    // COMPILE ERROR: integer literal too large -- it's parsed as an int FIRST
```

### Alternate Number Bases

Java supports writing integer literals in four bases:

```java
int decimal    = 42;         // base 10 (default, no prefix)
int binary      = 0b101010;   // base 2  (0b or 0B prefix)   -- equals 42
int octal        = 052;        // base 8  (leading 0 prefix)   -- equals 42
int hexadecimal  = 0x2A;       // base 16 (0x or 0X prefix)    -- equals 42
```

**Why does this matter practically?** Hexadecimal is extremely common for representing bit patterns, color values (`0xFF0000` for red), and memory-related/bitwise work (Topic 5 — Operators). Binary literals are useful for making bitmask/flag logic self-documenting. Octal is rare in modern code but occasionally appears in legacy systems and Unix file permission values — and is a **notorious beginner trap**: `int x = 010;` does **not** equal 10 — it equals 8, because the leading `0` makes it an octal literal!

### Underscores in Numeric Literals (Java 7+)

For readability, you can insert underscores anywhere **between** digits (not at the start, end, or adjacent to a decimal point or type suffix):

```java
int million = 1_000_000;          // reads far more clearly than 1000000
long creditCardNumber = 1234_5678_9012_3456L;
double pi = 3.14_159;
```

```java
int bad1 = _1000;      // ILLEGAL -- can't start with underscore
int bad2 = 1000_;       // ILLEGAL -- can't end with underscore
double bad3 = 3._14;     // ILLEGAL -- can't be adjacent to the decimal point
```

## Floating-Point Literals

By default, a decimal literal is typed `double`. Append `f`/`F` for `float`, `d`/`D` for `double` (redundant but sometimes used for clarity):

```java
double d1 = 3.14;      // double by default
float f1 = 3.14f;       // 'f' suffix REQUIRED -- without it, this is a double, and assigning
                          // a double to a float without an explicit cast is a compile error
                          // (narrowing conversion -- full rules in Topic 4)
double d2 = 2.5e3;      // scientific notation: 2.5 x 10^3 = 2500.0
```

## Character Literals

A `char` literal is a **single** character enclosed in **single** quotes:

```java
char letter = 'A';
char digit = '7';        // this is the CHARACTER '7', not the number 7 (its numeric/Unicode value is 55)
char unicode = '\u0041';  // Unicode escape -- also represents 'A'
```

### Escape Sequences

Certain characters can't be typed directly into a literal (or need special representation) — these use a backslash `\` escape:

| Escape | Meaning |
|---|---|
| `\n` | Newline |
| `\t` | Tab |
| `\\` | A literal backslash |
| `\'` | A literal single quote |
| `\"` | A literal double quote |
| `\r` | Carriage return |
| `\b` | Backspace |
| `\0` | Null character |
| `\uXXXX` | A Unicode character by its 4-digit hex code point |

```java
char newline = '\n';
char quote = '\'';
String path = "C:\\Users\\Aniket";   // each \\ represents one literal backslash
```

**Why do escape sequences exist?** Certain characters are either impossible to represent visually in source code (newline, tab) or would be ambiguous with the syntax itself (a literal `"` inside a `String` literal would otherwise prematurely end the string) — the backslash lets you unambiguously say "treat the next character(s) specially, not literally."

## Boolean Literals

Exactly two, always lowercase, and — unlike C, where any nonzero integer is "truthy" — Java's `boolean` accepts **only** these two literal values, with **no implicit conversion** from `int` at all:

```java
boolean isActive = true;
boolean isDone = false;
```

```java
if (1) { ... }   // COMPILE ERROR in Java -- unlike C, an int is never usable as a boolean
```

**Why?** This is a deliberate robustness choice (Module 01, Topic 3) — a classic category of C bugs is accidentally writing `if (x = 1)` (assignment, always "truthy") instead of `if (x == 1)` (comparison). Since Java's `boolean` is a completely separate type from `int`, with zero implicit conversion either direction, this entire bug category is **impossible** in Java — it simply won't compile.

## String Literals

```java
String greeting = "Hello, World!";
String empty = "";
String withEscapes = "Line1\nLine2\tTabbed";
```

String literals get special, important treatment in the JVM — the **String Constant Pool** — covered in full depth in Module 08 (Strings), where you'll learn exactly why `"hello" == "hello"` behaves differently than `new String("hello") == new String("hello")`.

### Text Blocks (Java 15+)

For multi-line text, the classic approach requires awkward `\n` and `+` concatenation:

```java
// Old way (pre-Java 15):
String json = "{\n" +
              "  \"name\": \"Aniket\",\n" +
              "  \"age\": 30\n" +
              "}";
```

**Text blocks** solve this directly, using triple-quote delimiters (`"""`):

```java
// Modern way (Java 15+):
String json = """
              {
                "name": "Aniket",
                "age": 30
              }
              """;
```

**Why introduced?** Multi-line text (JSON payloads, SQL queries, HTML fragments embedded in Java code — extremely common in real backend development) was genuinely painful and error-prone to write with classic String literals, requiring constant manual escaping and concatenation. Text blocks eliminate that ceremony entirely, while intelligently handling indentation (the compiler strips a common leading-whitespace amount based on the closing `"""`'s position) so your source code can stay nicely indented without that indentation leaking into the actual string value. (Full syntax rules and edge cases: Module 23 — Modern Java.)

## Advantages of Java's Literal Rules

- Multiple integer bases and underscore separators make numeric code (especially bitwise/flag-heavy code) significantly more self-documenting.
- Strict `boolean` typing (no int-to-boolean implicit conversion) eliminates a real, historically common bug category from C-family languages.
- Text blocks (modern Java) dramatically improve readability for embedded multi-line text, a very common real-world need.

## Disadvantages / Trade-offs

- More literal syntax rules to memorize compared to more permissive languages.
- The octal literal's `0`-prefix trap (`010` meaning 8, not 10) is a genuine, sharp-edged legacy wart most developers only learn about after being bitten once.

## Best Practices

- Prefer underscores in long numeric literals for readability (`1_000_000` over `1000000`).
- Always double check leading zeros on integer literals — they mean octal, not "padding," a subtle and easy-to-miss mistake.
- Use text blocks (Java 15+) for any embedded multi-line text (SQL, JSON, HTML) instead of manual `\n` + concatenation.

## Common Mistakes

| Mistake | Correction |
|---|---|
| `long big = 3000000000;` (no `L` suffix) | The literal `3000000000` is parsed as `int` *first* (before any assignment happens), and overflows `int`'s range — you must write `3000000000L`. |
| `float f = 3.14;` (no `f` suffix) | `3.14` is a `double` literal by default; assigning it to a `float` variable without a cast or `f` suffix is a compile error (narrowing conversion — Topic 4). |
| `int x = 010;` assuming it equals 10 | A leading `0` makes it an **octal** literal — `010` actually equals 8. |
| Using `if (someInt)` expecting truthy/falsy behavior like C/Python/JS | Illegal in Java — `boolean` and `int` have zero implicit conversion between them. |

## Interview Questions

1. **Q: Why does `long big = 3000000000;` fail to compile without an `L` suffix, even though the target type is `long`?**
   A: Integer literals are parsed as `int` by default, regardless of the variable they're being assigned to. `3000000000` exceeds `int`'s maximum value (~2.1 billion), so the literal itself fails to compile before assignment is even considered — the `L` suffix is what tells the compiler to parse it directly as a `long` literal instead.

2. **Q: Why doesn't Java allow `if (1)` the way C does?**
   A: Java's `boolean` is a completely separate type from `int`, with no implicit conversion in either direction — a deliberate robustness decision that makes an entire historical category of C bugs (accidental assignment `=` instead of comparison `==` inside a condition) impossible to even compile in Java.

3. **Q: What's the difference between `010` and `10` as Java integer literals?**
   A: `10` is decimal ten. `010`, with its leading zero, is interpreted as an **octal** literal, equal to decimal 8 — a well-known, sharp-edged trap in Java (and C).

## Summary

- Literals are fixed values written directly in source code; each primitive type (and `String`) has its own literal syntax.
- Integer literals default to `int` (need `L` for `long`); decimal literals default to `double` (need `f`/`F` for `float`).
- Java supports binary (`0b`), octal (leading `0`), and hexadecimal (`0x`) integer literal bases, plus underscore separators for readability.
- `boolean` accepts only `true`/`false`, with zero implicit conversion from `int` — a deliberate robustness decision.
- Text blocks (Java 15+) modernize multi-line String literals, eliminating manual escaping/concatenation for embedded multi-line text.

## Exercises

1. What is the value (in decimal) of the literal `010`? Explain why, and rewrite it unambiguously as a decimal literal.
2. Why does `float f = 3.14;` fail to compile, and what are the two ways to fix it? (Hint: one involves a literal suffix, the other involves an explicit cast — Topic 4 covers the second option fully.)
3. Rewrite this legacy-style multi-line String concatenation as a Java 15+ text block:
   ```java
   String msg = "Dear " + name + ",\n" + "Thank you for your order.\n" + "Regards,\nThe Team";
   ```
4. Explain, precisely, why `if (x = 5)` (a common typo for `x == 5`) is a compile error in Java but a real, historically common runtime bug in C.

---

**Previous:** [02 — Primitive Data Types](02-primitive-data-types.md) · **Next:** [04 — Type Conversion & Casting](04-type-conversion-and-casting.md)
