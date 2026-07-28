# Module 04 — Control Flow

## Module Goal

Every program you've written so far in this course has executed **top to bottom, one statement after another, with no decisions and no repetition**. That changes here. This module gives you the constructs that let a program actually *branch* (do different things depending on data) and *repeat* (do the same thing many times) — the two capabilities that turn a fixed script into a genuinely dynamic program.

## Topics Covered in This Module

1. **[If-Else & Conditional Logic](01-if-else-and-conditional-logic.md)** — `if`, `else`, `else if` chains, nested conditionals, and the classic "dangling else" ambiguity.
2. **[Switch Statement & Expression](02-switch-statement-and-expression.md)** — the classic `switch` statement (and its fall-through behavior), and the modern `switch` **expression** (Java 14+) that fixes fall-through's biggest footgun.
3. **[While & Do-While Loops](03-while-and-do-while-loops.md)** — condition-first vs. condition-last repetition, and when each is the right tool.
4. **[For Loops](04-for-loops.md)** — the classic three-part `for` loop mechanics, plus a first look at the enhanced for-each loop (full depth once you reach Arrays/Collections in Modules 09–10).
5. **[Break, Continue & Labeled Statements](05-break-continue-and-labeled-statements.md)** — early loop exit, skipping an iteration, and controlling nested loops precisely with labels.
6. **[Module Summary, Interview Questions & Exercises](06-module-summary-exercises.md)** — consolidated recap, quiz, and practice problems.

## Prerequisites

- Module 03 (Java Basics), especially Topic 5 (Operators) — every condition in this module is a `boolean` expression built from what you already learned there.

## How to Study This Module

This module is shorter and more mechanical than Modules 01–03 — the concepts themselves are intuitive if you've programmed before in any language. The value here is in Java's *specific* syntax and *specific* gotchas (fall-through in classic `switch`, the dangling-else trap, labeled break/continue) — read for precision, not just familiarity. Everything here is used constantly, starting immediately in Module 05 (OOP).

---

**Previous module:** [03 — Java Basics](../03-java-basics/00-module-overview.md) · **Next:** [01 — If-Else & Conditional Logic](01-if-else-and-conditional-logic.md)
