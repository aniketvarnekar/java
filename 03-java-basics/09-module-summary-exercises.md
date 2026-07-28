# Module 03 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-module-overview.md) has been covered:

- [x] **Variables & Identifiers** — declaration, compiler-enforced naming rules vs. convention-only style, local/instance/static variable kinds tied to Module 02's memory areas, `var` type inference
- [x] **Primitive Data Types** — all 8 primitives with sizes/ranges/defaults, why primitives exist alongside objects, `float`/`double` precision limits
- [x] **Literals** — integer/floating/char/boolean/String literal forms, alternate number bases, underscores, escape sequences, text blocks
- [x] **Type Conversion & Casting** — widening vs. narrowing, truncation vs. rounding, integer overflow bit-level mechanics, `Math.*Exact()` methods
- [x] **Operators** — arithmetic (including division/modulus edge cases), relational, logical (short-circuit vs. not), bitwise, assignment (including the `+=` implicit-cast trap), ternary, precedence
- [x] **Wrapper Classes & Autoboxing** — why wrappers exist, autoboxing/unboxing mechanics, the `Integer` cache and `==` trap, NPE-on-unboxing risk
- [x] **Constants & `final`** — `final` on variables/fields, the "final ≠ immutable object" distinction, compile-time constants and their bytecode-inlining behavior
- [x] **Comments & Code Style** — all three comment forms, Javadoc, consolidated naming conventions

## Practical Connections

- **Spring Boot configuration properties** are frequently bound to wrapper types (`Integer`, `Boolean`) rather than primitives specifically so that "not configured" can be represented as `null` — directly using the wrapper-vs-primitive distinction from Topic 6.
- **Database ORM frameworks** (like Hibernate/JPA, built on core Java) map nullable database columns to wrapper types for the exact same reason — a primitive `int` field literally cannot represent SQL `NULL`.
- **Financial/e-commerce backend code** universally uses `BigDecimal`, never `double`/`float`, for money — directly motivated by Topic 2's explanation of binary floating-point imprecision.
- **Code review culture** at any real Java company will flag `==` used to compare `Integer`/`String`/any object type — this module (Topic 6) is your first, foundational encounter with a pattern that recurs constantly (Strings in Module 08, custom `equals()` in Module 07).
- **Microservices/REST APIs**: request/response DTOs (data classes) overwhelmingly use wrapper types for optional fields and primitives for guaranteed-present fields — a direct, everyday application of Topic 6's core lesson.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| `int` vs `Integer` | `int` is a primitive, stored inline, never `null`, no methods. `Integer` is a Heap-allocated object wrapping an `int`, can be `null`, usable in generics. |
| Widening vs narrowing conversion | Widening (small→large type) is automatic and safe. Narrowing (large→small type) requires an explicit cast and can lose data. |
| `==` vs `.equals()` on wrapper objects | `==` compares object identity (same Heap object); `.equals()` compares logical value equality — use `.equals()` for wrapper/object value comparisons. |
| `&&`/`\|\|` vs `&`/`\|` | The double-symbol forms short-circuit (skip evaluating the right side when already determined); the single-symbol forms always evaluate both sides. |
| `final` variable vs immutable object | `final` locks the variable's binding (can't reassign); it says nothing about whether the referenced object's own state can still be mutated via its methods. |
| Truncation vs rounding | Casting a float/double to an integer type **truncates** (discards the fraction, toward zero) — it never rounds to the nearest value. |

## Consolidated Interview Questions (Module 03)

1. What does `var` actually do at compile time, and why is Java still statically typed despite it?
2. Name Java's 8 primitive types with their sizes. Why does Java have primitives at all, given it's object-oriented?
3. Why does `long big = 3000000000;` fail to compile without an `L` suffix?
4. What's the difference between widening and narrowing conversions? Why does one require an explicit cast and the other doesn't?
5. What actually happens, bit by bit, when you narrow a `long` value that doesn't fit into a `byte`?
6. Does Java throw an exception on integer overflow by default? How can you opt into guaranteed overflow detection?
7. Why does `5 / 2` evaluate to `2`? What about `5.0 / 2`?
8. What is short-circuit evaluation, and why is it essential for the common null-check guard pattern (`obj != null && obj.getValue() > 0`)?
9. Why does `byte b = 5; b += 10;` compile but `byte b = 5; b = b + 10;` does not?
10. What is autoboxing? Is it a compile-time or runtime mechanism?
11. Why does `Integer a = 100; Integer b = 100; a == b` print `true`, while the same code with `200` prints `false`?
12. Why can unboxing a wrapper object throw `NullPointerException`?
13. Does `final` make an object immutable? What does it actually guarantee?
14. What is a compile-time constant, and why does referencing one not trigger the defining class's initialization?
15. What's the difference between a block comment and a Javadoc comment, practically?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** From memory, list all 8 primitive types with sizes, then all 8 corresponding wrapper class names.
2. **Hands-on:** Write and run a program demonstrating the `Integer` cache boundary — print the result of `==` comparison for wrapper `Integer` values at `127`, `128`, `-128`, and `-129`, and confirm your predictions match the actual output.
3. **Hands-on:** Write a program that deliberately triggers `StackOverflowError`-style... no — instead, deliberately trigger silent `int` overflow (`Integer.MAX_VALUE + 1`), print the surprising result, then rewrite the same calculation using `Math.addExact()` and observe the thrown exception.
4. **Hands-on:** Write a method with a `final` parameter that receives a mutable `List`, and inside the method, both mutate the list's contents (legal) and attempt to reassign the parameter (illegal) — confirm which one the compiler rejects and why.
5. **Conceptual:** Explain, tying together Topics 2, 4, and 6, why `ArrayList<int>` is illegal but `ArrayList<Integer>` works — reference primitives' inline storage and generics' object requirement specifically.
6. **Conceptual:** A teammate writes `if (userInputFlag == 1)` expecting it to work like a boolean check. Explain precisely why this fails to compile in Java, referencing the specific design decision behind it.
7. **Synthesis:** Write a short method that takes two `double` parameters representing money, adds them, and explain — referencing Topic 2 specifically — why this is inappropriate for a real financial application, and what type should be used instead.

## What's Next

Module 03 gave you the full vocabulary of values, types, and operators. **Module 04 — Control Flow** puts that vocabulary into motion: `if`/`else`, the classic `switch` statement *and* the modern `switch` expression (Java 14+), `for`/`while`/`do-while` loops, and branching statements (`break`/`continue`/labeled loops) — the constructs that let a program actually make decisions and repeat work.

---

**Previous:** [08 — Comments & Code Style](08-comments-and-code-style.md) · **Module Overview:** [00 — Module Overview](00-module-overview.md)

**Type "Continue" to begin Module 04 — Control Flow.**
