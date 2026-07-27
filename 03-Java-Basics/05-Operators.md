# Operators

## Learning Objectives

- Use arithmetic, relational, logical, bitwise, and assignment operators correctly
- Understand short-circuit evaluation and why it exists
- Understand operator precedence and associativity well enough to predict evaluation order
- Understand exactly what bitwise operators do, bit by bit

## Prerequisites

[02 — Primitive Data Types](02-Primitive-Data-Types.md), [04 — Type Conversion & Casting](04-Type-Conversion-And-Casting.md)

## Motivation

Operators are how you actually *do* things with the values you've learned to declare and convert. Most are intuitive if you've programmed before — this topic focuses extra attention on the parts that are genuinely Java-specific or commonly misunderstood: short-circuit evaluation, integer division truncation, and bitwise operations.

## Categories of Operators

### 1. Arithmetic Operators

| Operator | Meaning | Example |
|---|---|---|
| `+` | Addition (also String concatenation — Module 08) | `5 + 3` → `8` |
| `-` | Subtraction | `5 - 3` → `2` |
| `*` | Multiplication | `5 * 3` → `15` |
| `/` | Division | `5 / 3` → `1` (integer division!) |
| `%` | Modulus (remainder) | `5 % 3` → `2` |
| `++` | Increment (pre/post) | `x++`, `++x` |
| `--` | Decrement (pre/post) | `x--`, `--x` |

**Integer division truncates, it does not round:**
```java
int result = 5 / 3;      // 1, NOT 1.67 or 2 -- the fractional part is simply discarded
double result2 = 5 / 3;    // STILL 1.0! Both operands are int, so int division happens FIRST,
                             // THEN the int result (1) is widened to double (1.0)
double result3 = 5.0 / 3;   // 1.6666... -- at least one operand must be a floating type
                             // BEFORE the division happens, not after
```
**This is a genuinely common bug source** — the type of a division depends entirely on the types of its **operands**, evaluated *before* any assignment or widening to the result variable happens.

**Division by zero:**
```java
int x = 5 / 0;       // throws ArithmeticException: / by zero
double y = 5.0 / 0;    // does NOT throw -- produces Infinity (a valid double value, per IEEE 754)
double z = 0.0 / 0;    // produces NaN ("Not a Number")
```
**Why the difference?** Integer division by zero is mathematically undefined with no sensible representable result, so Java throws an exception. Floating-point arithmetic (IEEE 754, the standard `float`/`double` follow) has well-defined special values — `Infinity`, `-Infinity`, `NaN` — specifically designed to let floating-point computation continue rather than crash, which is why it doesn't throw.

### Pre-Increment vs. Post-Increment — Precisely

```java
int a = 5;
int b = a++;   // POST-increment: b gets the value of 'a' BEFORE incrementing. b=5, a=6
int c = 5;
int d = ++c;    // PRE-increment: 'c' is incremented FIRST, then that new value is used. d=6, c=6
```

```
POST (a++):  1. read current value of a  2. use that value in the expression  3. THEN increment a
PRE  (++a):  1. increment a first        2. THEN use the new value in the expression
```

> **Best practice, stated up front:** avoid combining `++`/`--` with other operations on the same variable **within the same statement** (e.g., `int x = a++ + ++a;`) — this is legal but genuinely hard to read correctly, and different developers frequently misjudge the evaluation order. Keep increment/decrement as their own separate statement wherever reasonably possible.

### 2. Relational (Comparison) Operators

| Operator | Meaning |
|---|---|
| `==` | Equal to |
| `!=` | Not equal to |
| `>` , `<` | Greater than, less than |
| `>=` , `<=` | Greater/less than or equal to |

All return a `boolean`. **Critical warning, previewed here and covered fully in Modules 06–08:** `==` on **object** references (including `String`) compares whether two references point to the **same object in memory** — not whether the objects are "logically equal." For primitives, `==` correctly compares values directly. This distinction is one of the most common Java bugs for beginners, and gets its own full, dedicated treatment once you've learned about objects and references (Module 07) and Strings specifically (Module 08).

### 3. Logical Operators

| Operator | Meaning |
|---|---|
| `&&` | Logical AND (short-circuit) |
| `\|\|` | Logical OR (short-circuit) |
| `!` | Logical NOT |
| `&` | Logical AND (non-short-circuit — also a bitwise operator, see below) |
| `\|` | Logical OR (non-short-circuit — also a bitwise operator, see below) |

### Short-Circuit Evaluation — Why It Exists

`&&` and `||` **skip evaluating their right-hand operand entirely** if the result is already determined by the left-hand operand alone:

```java
if (obj != null && obj.getValue() > 0) {
    // if obj IS null, "obj.getValue() > 0" is NEVER EVALUATED --
    // short-circuit && stops immediately since the whole expression
    // is already guaranteed false
}
```

If `&&` were **not** short-circuit, `obj.getValue()` would still be called even when `obj` is `null`, throwing a `NullPointerException` — the classic **null-check guard pattern** shown above only works correctly *because* of short-circuit evaluation. This is precisely why Java provides **both** a short-circuit (`&&`/`||`) and a non-short-circuit (`&`/`|`) version of logical AND/OR — the non-short-circuit versions exist for the rare cases where you deliberately *want* both sides evaluated regardless (e.g., both sides have necessary side effects you need to happen every time) — but `&&`/`||` are overwhelmingly the correct default choice.

```
&&  :  left is false  -> STOP, result is false, right side never runs
||  :  left is true   -> STOP, result is true,  right side never runs
&   :  ALWAYS evaluates both sides, regardless
|   :  ALWAYS evaluates both sides, regardless
```

### 4. Bitwise Operators

These operate on the **individual bits** of integer types:

| Operator | Meaning |
|---|---|
| `&` | Bitwise AND |
| `\|` | Bitwise OR |
| `^` | Bitwise XOR |
| `~` | Bitwise complement (NOT) |
| `<<` | Left shift |
| `>>` | Signed right shift (preserves sign) |
| `>>>` | Unsigned right shift (fills with zeros) |

```
  a = 0110  (6)
  b = 0011  (3)
  ─────────────
a & b = 0010 (2)   -- 1 only where BOTH bits are 1
a | b = 0111 (7)   -- 1 where EITHER bit is 1
a ^ b = 0101 (5)   -- 1 where bits DIFFER
  ~a  = 1001 (in a full 32-bit int, this becomes -7 due to two's complement)
```

**Shift operators:**
```java
int x = 1 << 3;    // 1 shifted left 3 places = 8   (each left shift doubles the value)
int y = 8 >> 2;      // 8 shifted right 2 places = 2  (each right shift halves it, rounding toward -infinity)
int z = -8 >>> 28;    // UNSIGNED right shift -- fills with 0s regardless of sign, ignoring the sign bit
```

**`>>` vs `>>>` — the critical difference:** `>>` (signed) fills the newly-vacated high-order bits with copies of the **original sign bit** (preserving whether the number was negative), while `>>>` (unsigned) **always** fills with `0`s, regardless of the original sign — meaning `>>>` on a negative number produces a large *positive* number, since the sign bit itself gets overwritten with `0`. `>>>` has no meaningful use on `long`/`int` values you're treating as ordinary signed numbers — it's specifically for when you're treating the value as a raw, unsigned bit pattern.

**Why do bitwise operators matter practically, given how rarely most application code touches raw bits?** They're heavily used for **flags/bitmasks** (packing many boolean options into a single compact integer, common in low-level APIs, file formats, and permission systems), hashing algorithms, and performance-critical numeric code (bit shifts are extremely fast, hardware-native operations — `x << 1` is a common, fast idiom for "multiply by 2"). You'll see `&`/`|` used for real flag-combination logic in some standard library APIs (e.g., some historical I/O and concurrency flag constants).

### 5. Assignment Operators

```java
int x = 10;
x += 5;    // equivalent to: x = x + 5;   -> 15
x -= 3;     // x = x - 3;   -> 12
x *= 2;      // x = x * 2;   -> 24
x /= 4;       // x = x / 4;   -> 6
x %= 4;        // x = x % 4;   -> 2
```

**A subtle but real gotcha:** compound assignment operators (`+=`, etc.) perform an **implicit narrowing cast** back to the variable's original type — something a plain `=` would NOT let you do without an explicit cast:

```java
byte b = 10;
b = b + 5;     // COMPILE ERROR! "b + 5" promotes to int, and int cannot be assigned to byte without a cast
b += 5;         // LEGAL -- compiles fine, because += implicitly includes a narrowing cast back to byte
                 // this is EXACTLY equivalent to: b = (byte) (b + 5);
```
**This is a genuinely surprising, real interview trap** — many developers assume `x += y` and `x = x + y` are always perfectly interchangeable; for narrower-than-`int` types, they are not, because `+=` silently performs a cast that `=` alone would refuse to do implicitly.

### 6. Ternary (Conditional) Operator

```java
int max = (a > b) ? a : b;   // "if a > b, then a, else b" -- as a single expression, not a statement
```
`condition ? valueIfTrue : valueIfFalse` — the only operator in Java that takes **three** operands. It's an **expression** (it produces a value you can assign/use), not a control-flow **statement** like `if`/`else` (full control flow coverage: Module 04).

## Operator Precedence & Associativity

When an expression mixes multiple operators, Java evaluates them in a fixed precedence order (highest to lowest, abbreviated to the most practically relevant ones):

```
Java Operator Precedence (Highest → Lowest)

┌────┬─────────────────────┬──────────────────────────────────────────────┐
│ No │ Precedence          │ Operators                                    │
├────┼─────────────────────┼──────────────────────────────────────────────┤
│ 1  │ Postfix             │ expr++  expr--                               │
│ 2  │ Unary               │ ++expr  --expr  +expr  -expr  !  ~  (cast)   │
│ 3  │ Multiplicative      │ *  /  %                                      │
│ 4  │ Additive            │ +  -                                         │
│ 5  │ Shift               │ <<  >>  >>>                                  │
│ 6  │ Relational          │ <  >  <=  >=  instanceof                     │
│ 7  │ Equality            │ ==  !=                                       │
│ 8  │ Bitwise AND         │ &                                            │
│ 9  │ Bitwise XOR         │ ^                                            │
│ 10 │ Bitwise OR          │ |                                            │
│ 11 │ Logical AND         │ &&                                           │
│ 12 │ Logical OR          │ ||                                           │
│ 13 │ Ternary             │ ?:                                           │
│ 14 │ Assignment          │ =  +=  -=  *=  /=  %=                        │
│    │                     │ &=  ^=  |=  <<=  >>=  >>>=                   │
└────┴─────────────────────┴──────────────────────────────────────────────┘

Highest Precedence
        ▲
        │
        │
        ▼
Lowest Precedence
```

```java
int result = 2 + 3 * 4;    // 14, NOT 20 -- * has higher precedence than +, evaluated first
boolean b = 5 > 3 && 2 < 4;  // true -- relational (>, <) evaluated before logical (&&)
```

**Best practice, stated directly: use parentheses liberally**, even where technically unnecessary, whenever an expression mixes more than 2 distinct operator types. Precedence tables are a real thing to *understand*, not something a reader should be forced to *recall from memory* while reading your code.

## Advantages

- Short-circuit evaluation (`&&`/`||`) enables the extremely common and important null-check guard pattern, and avoids unnecessary computation.
- A rich, well-defined operator set (including dedicated bitwise operators) supports both high-level application logic and low-level, performance-critical code in the same language.

## Disadvantages / Trade-offs

- Integer division truncation and the `+=` implicit-narrowing-cast behavior are both genuine, sharp-edged surprises for anyone not specifically taught them.
- Deeply nested expressions relying purely on precedence rules (without parentheses) are a real, common source of subtle bugs and code review friction.

## Best Practices

- Use parentheses to make evaluation order explicit whenever mixing more than two operator categories in one expression.
- Prefer `&&`/`||` over `&`/`|` for boolean logic unless you specifically, deliberately need both sides always evaluated.
- Be deliberate about operand types before dividing — cast explicitly (`(double) a / b`) when you want floating-point division between integer variables.

## Common Mistakes

| Mistake | Correction |
|---|---|
| `double avg = sum / count;` where both are `int` | Integer division happens first, truncating, THEN widens to double — cast at least one operand: `(double) sum / count`. |
| Assuming `x += y` and `x = x + y` are always equivalent | For types narrower than `int` (`byte`, `short`, `char`), `+=` silently performs an implicit narrowing cast that plain `=` would refuse to compile without an explicit cast. |
| Using `&`/`|` for boolean conditions where `&&`/`||` was intended | Loses short-circuit evaluation, potentially causing unnecessary computation or even `NullPointerException`s in guard-pattern conditions. |
| Forgetting `>>` preserves sign while `>>>` does not | Using `>>` when unsigned shifting was actually intended produces sign-extended (often unexpectedly negative) results. |

## Interview Questions

1. **Q: Why does `5 / 2` evaluate to `2` instead of `2.5`?**
   A: When both operands are `int`, Java performs integer division, which truncates the result toward zero, discarding any fractional part — the truncated `int` result (2) only gets converted to a floating type afterward, if the *assignment target* requires it, which is too late to recover the lost fraction.

2. **Q: What is short-circuit evaluation, and why does it matter for null checks?**
   A: `&&` and `||` skip evaluating their right operand when the left operand alone already determines the result. This is essential for the common `if (obj != null && obj.getValue() > 0)` guard pattern — without short-circuiting, the right-hand method call would still execute even when `obj` is `null`, throwing a `NullPointerException`.

3. **Q: Why does `byte b = 10; b += 5;` compile, but `byte b = 10; b = b + 5;` does not?**
   A: `b + 5` promotes to `int` (per Java's arithmetic promotion rules), and assigning an `int` to a `byte` requires an explicit cast. `+=` is defined to implicitly include that narrowing cast back to the variable's original type, while plain `=` is not — making these two forms genuinely non-equivalent for narrower-than-`int` types.

## Summary

- Arithmetic operators follow standard rules, with two critical Java specifics: **integer division truncates**, and **integer division-by-zero throws `ArithmeticException`** while floating-point division-by-zero produces `Infinity`/`NaN` instead.
- `&&`/`||` are **short-circuit**; `&`/`|` (also usable as bitwise operators) always evaluate both sides — short-circuiting is essential for safe null-check guard patterns.
- Bitwise operators (`&`, `|`, `^`, `~`, `<<`, `>>`, `>>>`) manipulate raw bits; `>>` preserves sign, `>>>` does not.
- Compound assignment operators (`+=`, etc.) include an **implicit narrowing cast**, unlike plain `=` — a genuinely surprising, real interview trap.
- Operator precedence follows a fixed hierarchy; use parentheses liberally to make evaluation order explicit rather than relying on a reader's memorized precedence table.

## Exercises

1. Predict the output of `System.out.println(7 / 2 + 0.5);` and explain the exact order of operations that produces it.
2. Explain, using the null-check guard pattern, why `&&` must be short-circuit for `if (obj != null && obj.isValid())` to be safe.
3. Explain precisely why `byte b = 5; b += 10;` compiles successfully while `byte b = 5; b = b + 10;` does not.
4. Without running code, predict the value of `-8 >> 1` and `-8 >>> 1` and explain why they differ.

---

**Previous:** [04 — Type Conversion & Casting](04-Type-Conversion-And-Casting.md) · **Next:** [06 — Wrapper Classes & Autoboxing](06-Wrapper-Classes-And-Autoboxing.md)
