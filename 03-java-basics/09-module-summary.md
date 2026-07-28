# Module 03 Summary

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

## What's Next

Module 03 gave you the full vocabulary of values, types, and operators. **Module 04 — Control Flow** puts that vocabulary into motion: `if`/`else`, the classic `switch` statement *and* the modern `switch` expression (Java 14+), `for`/`while`/`do-while` loops, and branching statements (`break`/`continue`/labeled loops) — the constructs that let a program actually make decisions and repeat work.