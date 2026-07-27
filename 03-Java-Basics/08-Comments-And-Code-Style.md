# Comments & Code Style

## Learning Objectives

- Use all three Java comment forms correctly, including Javadoc
- Generate documentation from Javadoc comments
- Have all of Java's naming conventions consolidated in one place, as a single reference

## Prerequisites

[01 — Variables & Identifiers](01-Variables-And-Identifiers.md)

## Motivation

This is a deliberately shorter, more reference-oriented topic — closing out Module 03's foundational vocabulary before Module 04 (Control Flow) starts building actual program logic. Comments and style don't affect what a program *does*, but they profoundly affect whether other humans (including future you) can understand it — and Javadoc specifically is how the entire Java ecosystem documents its own standard library, which you'll be reading constantly throughout this course.

## The Three Comment Forms

### 1. Single-Line Comment

```java
int retries = 3; // number of times to retry a failed request
```
Everything from `//` to the end of the line is ignored by the compiler.

### 2. Multi-Line (Block) Comment

```java
/*
 * This block handles retry logic for network requests.
 * It backs off exponentially between attempts.
 */
```
Everything between `/*` and `*/` is ignored, including across multiple lines. **Cannot be nested** — a `/*` inside an already-open block comment does not start a new nested comment, and will likely cause a confusing "unexpected `*/`" compile error at the first `*/` encountered, closing the *outer* comment prematurely.

### 3. Javadoc Comment

```java
/**
 * Calculates the total price including tax.
 *
 * @param subtotal the pre-tax amount, must be non-negative
 * @param taxRate the tax rate as a decimal (e.g., 0.08 for 8%)
 * @return the total price, including tax
 * @throws IllegalArgumentException if subtotal is negative
 */
public double calculateTotal(double subtotal, double taxRate) {
    // ...
}
```

A Javadoc comment starts with `/**` (two asterisks) specifically, placed directly above a class, method, or field declaration. It's structurally similar to a block comment, but the **`javadoc` tool** (bundled in the JDK — Module 01, Topic 4) specifically recognizes this `/** ... */` form and generates browsable HTML API documentation from it.

**Common Javadoc tags:**

| Tag | Meaning |
|---|---|
| `@param name description` | Documents a method parameter |
| `@return description` | Documents the return value |
| `@throws ExceptionType description` | Documents an exception the method may throw |
| `@author name` | Documents the author (class-level, less common in modern practice) |
| `@since version` | Documents which version introduced this |
| `@deprecated reason` | Marks something as discouraged, with an explanation |
| `{@link OtherClass#method}` | Cross-references another documented element |

**Why does Javadoc matter so much, specifically in Java?** The **entire standard library's official documentation** (`String`'s methods, `ArrayList`'s methods, everything) is generated *from* Javadoc comments written directly in the JDK's own source code. Every time you look up a standard library method's documentation, you're reading generated Javadoc output. Writing good Javadoc comments on your own public APIs (methods/classes other developers, or other teams, will use) is a genuine, expected professional practice, not an optional nicety, in most real Java codebases — full depth on writing effective API documentation for classes/interfaces: Module 06.

## Comment Best Practices — The "Why," Not the "What"

Recall from this course's own writing philosophy (stated in the introduction): a good comment explains **why**, not **what**. Well-named identifiers already communicate *what* code does — a comment repeating that is pure noise:

```java
// BAD -- repeats what the code already says
int retries = 3; // set retries to 3

// GOOD -- explains a non-obvious reason
int retries = 3; // 3 chosen empirically: fewer caused false failures on flaky test infra
```

## Naming Conventions — Full Consolidated Reference

You've seen pieces of this scattered across Topics 1 and 7 — here's the complete, single reference table:

| Element | Convention | Example |
|---|---|---|
| Class / Interface / Enum / Record | PascalCase (UpperCamelCase) | `BankAccount`, `Runnable`, `OrderStatus` |
| Method | camelCase, usually a verb/verb phrase | `calculateTotal`, `isValid`, `getName` |
| Variable (local, instance) | camelCase, usually a noun | `firstName`, `totalCount` |
| Parameter | camelCase, same as variables | `taxRate`, `accountId` |
| Constant (`static final`) | SCREAMING_SNAKE_CASE | `MAX_RETRIES`, `DEFAULT_TIMEOUT_MS` |
| Package | all lowercase, reverse-domain style, no underscores | `com.mycompany.billing.invoices` |
| Type parameter (generics — Module 11) | single uppercase letter, conventionally `T`, `E`, `K`, `V`, `R` | `List<T>`, `Map<K, V>` |

**Why these specific conventions, restated one final time for this module:** every convention here serves the same underlying goal — letting a reader instantly classify *what kind of thing* an identifier refers to from its spelling alone, with zero additional context. This is a genuinely load-bearing readability convention across the entire, enormous Java ecosystem (every library, every open-source project, every company codebase you'll ever encounter), not a stylistic preference specific to this course.

## Real-World Analogy

Think of naming conventions like **road sign color-coding** — in most countries, without reading a single word, you instinctively know a *red* sign means prohibition/danger, a *green* sign means direction/confirmation, a *yellow* sign means caution. You didn't memorize this per-sign; you learned the *color system* once, and now every new sign you've never seen before still communicates its category instantly. Java's PascalCase/camelCase/SCREAMING_SNAKE_CASE system works exactly the same way for code.

## Advantages

- Consistent naming conventions make any unfamiliar Java codebase substantially faster to navigate.
- Javadoc turns documentation into a first-class, tool-supported part of the language itself, rather than an external, easily-outdated wiki.

## Disadvantages / Trade-offs

- Writing high-quality Javadoc for every public API takes real, deliberate effort — teams under time pressure frequently let it decay, becoming stale relative to the actual code.
- Naming conventions are not compiler-enforced (Topic 1) — a codebase without discipline (or without automated tooling like Checkstyle) can still drift from them over time.

## Best Practices

- Write Javadoc for every `public` class and method that other developers/teams will actually call — it's part of the API's real contract, not decoration.
- Keep inline comments focused on **why**, never restating **what** the code already makes obvious through good naming.
- Use an IDE's or build tool's style-checking integration (not covered in depth in this Core Java course, but worth knowing exists — tools like Checkstyle) to enforce naming conventions automatically in real team settings.

## Common Mistakes

- Writing comments that restate the obvious instead of explaining non-obvious reasoning.
- Confusing `/** */` (Javadoc, tool-recognized) with `/* */` (plain block comment, not processed by the `javadoc` tool) — using the wrong one means your documentation simply won't be picked up by documentation generation.
- Letting comments go stale — a comment that no longer matches the code it describes is actively worse than no comment at all, since it actively misleads a reader.

## Interview Questions

1. **Q: What's the difference between a regular block comment (`/* */`) and a Javadoc comment (`/** */`)?**
   A: Syntactically, a Javadoc comment starts with an extra asterisk (`/**` instead of `/*`). Practically, only Javadoc-form comments, placed directly above a class/method/field declaration, are recognized and processed by the `javadoc` tool to generate browsable HTML API documentation — regular block comments are ignored by that tooling entirely.

2. **Q: Why does the entire Java standard library's documentation come from Javadoc comments in the JDK's own source?**
   A: Because Javadoc is a first-class, tool-supported documentation mechanism built directly into the language and JDK — writing documentation as structured comments directly alongside the code it describes (rather than in a separate, easily-diverging document) keeps documentation close to, and more likely to stay consistent with, the actual implementation.

## Summary

- Java has three comment forms: `//` (single-line), `/* */` (block), and `/** */` (Javadoc — tool-processed to generate API documentation).
- Good comments explain **why**, not **what** — well-named code already communicates what it does.
- Java's naming conventions (PascalCase for types, camelCase for variables/methods, SCREAMING_SNAKE_CASE for constants, lowercase reverse-domain for packages) are convention-only but universally expected across the entire Java ecosystem.

## Module-Wide Quick Revision

- Variables: typed, named storage; local (Stack, no default, must be definitely assigned), instance (Heap, defaulted), static (Method Area, defaulted); `var` is compile-time type inference, not dynamic typing (Topic 1).
- 8 primitives (`byte`/`short`/`int`/`long`/`float`/`double`/`char`/`boolean`), stored inline for performance, unlike objects (Topic 2).
- Literals: integer literals default `int` (need `L` for `long`), decimal literals default `double` (need `f` for `float`); watch for the leading-zero octal trap (Topic 3).
- Widening = automatic/safe; narrowing = explicit cast required, can silently lose data via truncation or bit-discarding overflow (Topic 4).
- Operators: integer division truncates; `&&`/`||` short-circuit (essential for null-guards) while `&`/`|` don't; `+=` implicitly narrows-casts, unlike plain `=` (Topic 5).
- Wrapper classes bridge primitives into Java's object world; autoboxing/unboxing is compiler-inserted; the `Integer` cache (-128 to 127) makes `==` on wrapper objects unreliable — always use `.equals()` (Topic 6).
- `final` locks a variable's *binding*, not the mutability of the object it may point to; compile-time constants get inlined directly into bytecode (Topic 7).
- Three comment forms, Javadoc generates real API docs, and Java's naming conventions let readers classify identifiers by spelling alone (this topic).

## Common Pitfalls (Module-Wide)

- Treating `var` as dynamic typing.
- Forgetting `char`'s default value is NUL, not `'0'`.
- Assuming `0.1 + 0.2 == 0.3` for `double`/`float`.
- Assuming `(int) someDouble` rounds instead of truncates.
- Assuming integer overflow throws an exception by default.
- Comparing wrapper objects (or, foreshadowing Module 08, Strings) with `==` instead of `.equals()`.
- Believing `final` makes an object immutable, not just its variable binding.

## Mini Quiz (Module-Wide)

1. What's the compiled type of a plain integer literal like `42`, and what suffix would make it a `long`?
2. Why does `byte b = 5; b += 10;` compile while `byte b = 5; b = b + 10;` does not?
3. What does `Integer a = 127; Integer b = 127; a == b` evaluate to, and why would `1000` instead of `127` change the answer?
4. Does `final Map<String, String> config = new HashMap<>();` prevent `config.put(...)` from working? Why or why not?
5. What's the practical difference between `/* */` and `/** */`?

*(Answers are derivable from Topics 3, 5, 6, 7, and this topic, respectively.)*

---

**Previous:** [07 — Constants & `final`](07-Constants-And-Final.md) · **Next:** [09 — Module Summary, Interview Questions & Exercises](09-Module-Summary-Exercises.md)
