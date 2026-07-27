# Module 03 — Java Basics

## Module Goal

You now know *what* Java is and *how the JVM* runs it. This module starts actually **writing Java logic**: declaring data, choosing the right type for it, converting between types safely, and manipulating it with operators. Everything here is small in isolation, but it's the vocabulary every later module (Control Flow, OOP, Collections, everything) is written in — get it precise now.

## Topics Covered in This Module

1. **[Variables & Identifiers](01-Variables-And-Identifiers.md)** — declaring variables, naming rules and conventions, `var` type inference, a first look at scope.
2. **[Primitive Data Types](02-Primitive-Data-Types.md)** — all 8 primitives, their sizes/ranges/defaults, and why Java keeps primitives separate from objects at all.
3. **[Literals](03-Literals.md)** — how to write literal values of every type, numeric literal formats (binary, hex, underscores), escape sequences.
4. **[Type Conversion & Casting](04-Type-Conversion-And-Casting.md)** — widening vs. narrowing conversions, implicit vs. explicit casting, overflow behavior.
5. **[Operators](05-Operators.md)** — arithmetic, relational, logical, bitwise, assignment, ternary operators, and precedence/associativity rules.
6. **[Wrapper Classes & Autoboxing](06-Wrapper-Classes-And-Autoboxing.md)** — why every primitive has an object wrapper, automatic boxing/unboxing, and the infamous `Integer` caching pitfall.
7. **[Constants & `final`](07-Constants-And-Final.md)** — declaring unchangeable data, compile-time constants, and naming conventions.
8. **[Comments & Code Style](08-Comments-And-Code-Style.md)** — comment types, Javadoc basics, and Java's naming conventions consolidated in one place.
9. **[Module Summary, Interview Questions & Exercises](09-Module-Summary-Exercises.md)** — consolidated recap, quiz, and practice problems.

## Prerequisites

- Module 01 (Introduction) — especially knowing how to compile/run a program.
- Module 02 (JVM) — especially the Stack vs. Heap model (Topic 3), which this module builds on directly when explaining where primitives and objects actually live.

## How to Study This Module

Topics 1–3 are foundational vocabulary — read them in order. Topic 4 (Type Conversion) is where most beginner bugs and a disproportionate share of interview questions come from — don't rush it. Topic 6 (Wrapper Classes) revisits the Stack/Heap model from Module 02 in a very concrete, commonly-tested way (`==` vs `.equals()` on boxed values) — this sets up Module 08 (Strings), which has the exact same pitfall in an even more common form.

---

**Previous module:** [02 — JVM](../02-JVM/00-Module-Overview.md) · **Next:** [01 — Variables & Identifiers](01-Variables-And-Identifiers.md)
