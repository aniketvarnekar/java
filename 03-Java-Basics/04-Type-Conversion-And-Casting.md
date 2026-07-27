# Type Conversion & Casting

## Learning Objectives

- Distinguish widening (implicit) from narrowing (explicit) conversions
- Correctly use cast syntax, and understand exactly what happens to the bits/value during a narrowing cast
- Understand integer overflow behavior precisely
- Know the primitive conversion hierarchy from memory

## Prerequisites

[02 — Primitive Data Types](02-Primitive-Data-Types.md), [03 — Literals](03-Literals.md)

## Motivation

This is one of the most heavily interview-tested topics in Core Java, and one of the most common sources of subtle, silent production bugs (silent overflow, silent precision loss) — precisely *because* Java sometimes converts types automatically and silently, and sometimes refuses to, requiring an explicit cast. Knowing exactly which is which, and why, is essential.

## Problem Statement

Java is statically typed (Module 01) — every value has a fixed type. But real programs constantly need to combine values of *different* types: adding an `int` to a `double`, storing a `long` result into an `int` variable, etc. Java needs consistent, predictable rules for when this is allowed automatically, and when you must explicitly say "yes, I understand the risk, do it anyway."

## Concept: Two Kinds of Conversion

### Widening Conversion (Implicit, Automatic, Always Safe)

Converting a **smaller-range type to a larger-range type** — Java does this **automatically**, with no special syntax, because it can never lose information (every value the smaller type can hold, the larger type can also represent exactly, or with no meaningful/no loss for numeric purposes).

```
byte -> short -> int -> long -> float -> double
              ↖ char ↗
```

```java
int i = 100;
long l = i;        // automatic widening: int -> long, always safe
double d = l;       // automatic widening: long -> double
```

> **Subtle exception worth knowing:** `long` → `float` and `long` → `double` are technically widening conversions (allowed automatically), but **can** lose *precision* (not magnitude/range) — a very large `long` value may not be exactly representable in a `float`/`double`'s limited mantissa precision, silently rounding to the nearest representable floating-point value. This is a genuinely subtle, real gotcha: "widening" guarantees no loss of *range*, but does **not** always guarantee no loss of *precision*.

### Narrowing Conversion (Explicit, Requires a Cast, Potentially Unsafe)

Converting a **larger-range type to a smaller-range type** — Java **refuses to do this automatically**, because it genuinely might lose information, and forces you to write an explicit **cast** — `(targetType) value` — as a deliberate, visible acknowledgment that you understand the risk.

```java
double d = 3.99;
int i = (int) d;      // EXPLICIT cast required -- narrowing double -> int
                        // result: i = 3  (the fractional part is simply TRUNCATED, not rounded!)

long bigNumber = 130L;
byte b = (byte) bigNumber;  // EXPLICIT cast required -- narrowing long -> byte
                              // result: b = -126  (see "Overflow" below for why)
```

```java
double d = 3.99;
int i = d;    // COMPILE ERROR: incompatible types -- possible lossy conversion from double to int
              // (you MUST write "(int) d" explicitly)
```

## Why This Asymmetry Exists (The "Why")

This connects directly to Java's "Robust" feature from Module 01, Topic 3: **the compiler forces you to be explicit exactly at the point where data loss becomes possible.** Automatic widening is safe, so no ceremony is needed. Narrowing is a genuine risk (lost precision, lost magnitude, unexpected sign flips — see below), so Java requires you to write the cast as a deliberate, visible, "I know what I'm doing" signal — both for the compiler's benefit and, just as importantly, for the **next human** reading your code, who can now see exactly where a risky conversion is happening instead of it being invisible.

## Truncation, Not Rounding

A critical, commonly-misunderstood detail: casting a floating-point value to an integer type **truncates** (chops off the decimal part) — it does **not** round to the nearest whole number.

```java
int a = (int) 3.99;    // 3, NOT 4
int b = (int) -3.99;    // -3, NOT -4 (truncation rounds TOWARD ZERO, not "down")
```

**If you want proper rounding**, use `Math.round(...)` explicitly instead of relying on a cast.

## Integer Overflow: What Actually Happens to the Bits

This is the deepest, most interview-relevant part of this topic. When you narrow a larger integer type into a smaller one, Java doesn't "clamp" the value to the smaller type's max/min — it simply **discards the higher-order bits** and reinterprets whatever bits remain, using two's complement representation for negative numbers.

```java
long bigNumber = 130L;              // binary: ...10000010
byte b = (byte) bigNumber;           // byte keeps only the lowest 8 bits: 10000010
                                       // as a SIGNED 8-bit two's complement value, that's -126
System.out.println(b);               // prints: -126
```

```
long value 130 in binary (relevant low byte shown):   ... 0000 0000 1000 0010
                                                                     └────┬────┘
                                          byte keeps ONLY these 8 bits ───┘
byte's 8 bits: 1000 0010
  - the leftmost bit is 1 -> this is a NEGATIVE number in two's complement
  - value = -126
```

**Why does this matter practically?** This is exactly what causes **silent integer overflow bugs** — no exception is thrown, no warning is printed, the program simply produces a wrong, wrapped-around value and keeps running. This is a real, historically significant category of production bugs (and famously, security vulnerabilities) across many languages, Java included, for regular arithmetic overflow too, not just explicit casts:

```java
int max = Integer.MAX_VALUE;  // 2147483647
int overflowed = max + 1;      // silently wraps to -2147483648, NO exception thrown!
```

**Why doesn't Java throw an exception on overflow, given its "Robust" design philosophy?** Historically, this mirrors how CPU integer arithmetic actually works at the hardware level (fixed-width registers wrap around identically), and checking for overflow on *every single* arithmetic operation would impose a real, constant performance cost that Java's designers judged not worth paying by default. **If you need guaranteed overflow detection**, Java's standard library provides `Math.addExact()`, `Math.multiplyExact()`, etc., which explicitly **throw** `ArithmeticException` on overflow instead of silently wrapping — an opt-in, deliberate choice you make only where it matters.

```java
int safe = Math.addExact(Integer.MAX_VALUE, 1);  // throws ArithmeticException: integer overflow
```

## `char` Conversions — A Special Case

`char` is really an unsigned 16-bit integer under the hood (representing a UTF-16 code unit), so it participates in numeric conversions, with some special rules:

```java
char c = 'A';       // Unicode code point 65
int i = c;            // automatic WIDENING: char -> int is always safe (char is unsigned, fits in int)
                        // i = 65

int j = 66;
char c2 = (char) j;    // NARROWING required: int -> char, since char's range differs from int's
                          // c2 = 'B'

char c3 = 65;           // this specific case is legal WITHOUT a cast -- because 65 is a compile-time
                          // CONSTANT that the compiler can verify fits within char's range
```

## Widening Reference Types vs. Widening Primitive Types (Preview)

Everything above is about **primitive** conversions. Once you learn inheritance (Module 05), you'll see an analogous — but conceptually distinct — idea for **object references**: a subclass reference can be automatically "widened" (upcast) to a superclass reference type, while the reverse (downcasting) requires an explicit cast and can fail at runtime with a `ClassCastException`. The vocabulary ("widening"/"narrowing", "implicit"/"explicit cast") is intentionally parallel to what you just learned here — full treatment in Module 05.

## Real-World Analogy

Think of **widening like pouring a small cup of water into a bigger bucket** — always safe, nothing is lost, no special care needed. Think of **narrowing like pouring a large bucket of water into a small cup** — you *can* do it, but you're explicitly acknowledging that some water (information) is very likely to spill out (be lost), which is exactly why Java makes you explicitly say "yes, pour it, I accept the spillage" via the cast syntax, rather than doing it silently.

## Advantages

- Automatic widening removes unnecessary ceremony for genuinely safe conversions.
- Mandatory explicit casts for narrowing conversions make risky conversions visible in the source code, both to the compiler and to human reviewers.
- Opt-in overflow-checking methods (`Math.addExact`, etc.) let you choose safety where it specifically matters, without imposing that cost everywhere by default.

## Disadvantages / Trade-offs

- Silent overflow (with no cast even needed — just from arithmetic exceeding a type's range) is a genuine, easy-to-miss bug source if you're not deliberately aware of a value's realistic range.
- Truncation-not-rounding on `double`→`int` casts is a common source of off-by-one-style logic errors for developers coming from languages with different (or configurable) rounding behavior.

## Best Practices

- Choose integer types (`int` vs `long`) based on the realistic maximum value the data could reach — not just "whatever compiles today."
- Use `Math.round()` explicitly whenever you actually want rounding behavior, never rely on a narrowing cast for that.
- Use `Math.addExact()`/`Math.subtractExact()`/`Math.multiplyExact()` in code where silent overflow would be a serious correctness or security concern (e.g., calculating array sizes, financial totals before moving to `BigDecimal`).

## Common Mistakes

| Mistake | Correction |
|---|---|
| Assuming `(int) 3.99` rounds to `4` | It truncates to `3` — casting a float/double to an integer type always truncates toward zero, never rounds. |
| Assuming overflow throws an exception | It silently wraps around by default; use `Math.addExact()` etc. if you need guaranteed detection. |
| Forgetting an explicit cast is required for any narrowing conversion, even when you're "sure" the value fits | The compiler doesn't know your runtime value is safe — it enforces the cast requirement based on the *types* involved, not the specific value. |
| Confusing widening's "no data loss" guarantee for precision, not just range (with `long` -> `float`/`double`) | Widening always preserves *magnitude/range* but can still lose *precision* in the `long`→`float`/`double` case specifically. |

## Interview Questions

1. **Q: What's the difference between widening and narrowing conversions in Java?**
   A: Widening converts a smaller-range type to a larger-range type and happens automatically, since it's always safe. Narrowing converts a larger-range type to a smaller one and requires an explicit cast, because it can lose information (magnitude, precision, or sign) — the cast is a deliberate, visible acknowledgment of that risk.

2. **Q: What does `(int) 9.99` evaluate to, and why?**
   A: `9` — casting a floating-point value to an integer type truncates the fractional part (rounds toward zero); it does not round to the nearest integer.

3. **Q: What happens when you cast a `long` value that doesn't fit into a `byte`?**
   A: Java discards the higher-order bits beyond the byte's 8 bits and reinterprets the remaining 8 bits as a signed two's-complement `byte` value — which can produce a surprising, seemingly unrelated (and possibly negative) result, not an exception or a clamped value.

4. **Q: Does Java throw an exception on integer overflow by default?**
   A: No — arithmetic overflow silently wraps around (e.g., `Integer.MAX_VALUE + 1` becomes `Integer.MIN_VALUE`) with no exception and no warning. `Math.addExact()` and similar methods are available as an explicit, opt-in way to get `ArithmeticException` on overflow instead.

## Summary

- **Widening** conversions (small type → large type) are automatic and always range-safe (though `long`→`float`/`double` can still lose precision).
- **Narrowing** conversions (large type → small type) require an explicit cast `(type) value`, because they can lose information.
- Casting a floating-point value to an integer type **truncates**, never rounds.
- Integer overflow **silently wraps around** by default (no exception); use `Math.addExact()` and friends for guaranteed detection.
- These same "widening/narrowing, implicit/explicit" concepts reappear for object references in Module 05 (upcasting/downcasting), with different underlying mechanics but parallel vocabulary.

## Exercises

1. Without running any code, predict the output of: `byte b = (byte) 200;` and explain the bit-level reasoning behind your answer.
2. Explain why `int total = 2_000_000_000 + 2_000_000_000;` produces a negative number, and rewrite it using a `Math` method that would instead throw an exception on this overflow.
3. Predict the value of `int x = (int) -7.8;` and explain why it isn't `-8`.
4. Explain, in your own words, why `long l = 100;` needs no cast, but `int i = 100L;` does — even though `100L` is a value that obviously fits within `int`'s range.

---

**Previous:** [03 — Literals](03-Literals.md) · **Next:** [05 — Operators](05-Operators.md)
