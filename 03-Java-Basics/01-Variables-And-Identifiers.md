# Variables & Identifiers

## Learning Objectives

- Declare, initialize, and use variables correctly
- Know Java's identifier naming rules (enforced by the compiler) and naming conventions (enforced by convention/tooling only)
- Understand the three kinds of variables (local, instance, static) at a preview level
- Use `var` (Java 10+) correctly and know its real limitations

## Prerequisites

Module 01 (Introduction), Module 02 Topic 3 (Runtime Data Areas)

## Motivation

A variable is the most basic unit of state in any program — but Java attaches more rules to it than many languages you may have used before (explicit types, strict naming rules, distinct kinds of variables with different lifetimes). Understanding these rules precisely now prevents a long tail of confusing errors later.

## Problem Statement

A program needs to **name** and **store** data so it can be used and changed later. Different languages solve "naming data" differently — Python lets you name anything without declaring its type; C requires an explicit type; Java requires an explicit type **and** enforces a specific set of rules about what names are even legal, checked by the compiler before your program ever runs.

## Concept: What Is a Variable in Java?

> A **variable** is a named piece of storage, of a specific, fixed **type**, that holds a value which can change over the variable's lifetime.

Every variable declaration in Java has this shape:

```java
type name = value;   // declaration + initialization (combined)
type name;            // declaration only (no value yet)
name = value;          // separate assignment, later
```

```java
int age = 30;          // type = int, name = age, value = 30
double price;          // declared, not yet initialized
price = 19.99;          // assigned later
```

## Identifiers: The Rules (Compiler-Enforced)

An **identifier** is any name you choose — for a variable, a method, a class, etc. Java's rules for what makes a *legal* identifier are strict and checked by `javac`:

- Must start with a letter, `_` (underscore), or `$` (dollar sign) — never a digit.
- Subsequent characters can be letters, digits, `_`, or `$`.
- Cannot be a **reserved keyword** (`class`, `int`, `public`, `static`, `if`, `for`, etc. — words the language itself uses).
- Case-sensitive: `age`, `Age`, and `AGE` are three completely different identifiers.
- No length limit, and no restriction against starting with an underscore (though see the Java 21+ note on the single underscore `_` below).

```java
int age;        // legal
int _score;      // legal (unusual style, but legal)
int $value;       // legal (unusual style, but legal)
int 2ndPlace;     // ILLEGAL -- starts with a digit
int class;         // ILLEGAL -- 'class' is a reserved keyword
int my-name;       // ILLEGAL -- hyphens are not allowed in identifiers
```

> **Java 21+ note:** the single underscore `_` used *by itself* is reserved as an **"unnamed variable"** marker (finalized in newer Java versions) — used to explicitly signal "I don't care about this value" in contexts like pattern matching or lambda parameters (covered fully in Module 23 — Modern Java). You can still use identifiers that merely *contain* underscores (`my_variable`) freely; it's specifically the single, standalone `_` that has special modern meaning.

## Naming Conventions (Convention-Only — NOT Compiler-Enforced)

These are not language rules — `javac` will happily compile code that violates every one of them — but violating them will make you look inexperienced to any real Java team, and most tooling (IDEs, linters) will warn you.

| Kind | Convention | Example |
|---|---|---|
| Variables & method names | camelCase, starting lowercase | `firstName`, `calculateTotal` |
| Class & interface names | PascalCase, starting uppercase | `BankAccount`, `Runnable` |
| Constants (`static final`) | SCREAMING_SNAKE_CASE | `MAX_RETRY_COUNT` |
| Packages | all lowercase, reverse-domain style | `com.mycompany.billing` |

**Why does this convention exist, specifically?** A Java developer reading unfamiliar code can instantly tell — from casing alone, with zero other context — whether an identifier refers to a variable, a type, or a constant. This is a real, practical readability aid the whole Java ecosystem has agreed on for 25+ years; breaking it imposes a real cognitive tax on anyone reading your code.

## The Three Kinds of Variables (Preview)

You'll see all three in depth in Module 06 (Classes), but you need the vocabulary now:

```java
public class Account {
    static int totalAccounts = 0;   // STATIC variable — one copy shared by the whole class
    double balance;                  // INSTANCE variable — one copy per object

    void deposit(double amount) {    // 'amount' is a parameter (a kind of local variable)
        double newBalance = balance + amount;  // LOCAL variable — exists only during this call
        balance = newBalance;
    }
}
```

| Kind | Where it lives (Module 02 terms) | Lifetime | Default value if uninitialized? |
|---|---|---|---|
| **Local variable** | JVM Stack (as part of the method's frame) | From declaration until the enclosing block/method ends | **No** — must be explicitly initialized before use, or `javac` refuses to compile |
| **Instance variable** (field) | Heap (as part of the object) | From object creation until the object is garbage collected | **Yes** — automatically defaulted (`0`, `false`, `null`, etc.) |
| **Static variable** | Method Area/Metaspace | From class initialization until the class is unloaded (rare) | **Yes** — automatically defaulted |

> **This table directly connects to Module 02.** Local variables living on the Stack is exactly why they're thread-private and destroyed instantly on method return (Module 02, Topic 3). Instance variables living on the Heap as part of their object is exactly why they persist as long as the object is reachable. Static variables living in the Method Area is exactly why there's only ever *one* copy, shared by every instance of the class — full depth on all three arrives in Module 06.

## Why Local Variables Have No Default Value (And Fields Do)

This is a deliberate, surprising-at-first design decision, worth understanding rather than memorizing:

- A **local variable**'s value is meaningless until you explicitly set it — there is no sensible "default" for, say, a loop counter or an intermediate calculation the compiler could safely guess. So Java forces you to initialize it yourself, and `javac` performs **definite assignment analysis** — a compile-time check that a local variable is guaranteed to be assigned before any code path that reads it. This eliminates an entire class of "uninitialized variable" bugs common in languages like C, at zero runtime cost (it's a compile-time-only check).
- A **field** (instance or static), on the other hand, is initialized automatically to a safe default — because fields are commonly accessed from *many* different methods, in orders the compiler cannot fully predict ahead of time, so guaranteeing a safe default is the only way to make sure no field is ever read in a genuinely undefined state.

```java
void demo() {
    int x;
    System.out.println(x);  // COMPILE ERROR: "variable x might not have been initialized"
}
```

## `var` — Local Variable Type Inference (Java 10+)

```java
var age = 30;              // compiler infers: int
var name = "Aniket";        // compiler infers: String
var prices = new double[5]; // compiler infers: double[]
```

**What `var` actually is — and is not:** `var` is **not** a dynamic type, and Java has **not** become dynamically typed. `var` simply tells the *compiler* to infer the exact static type from the right-hand side, **once, at compile time** — the variable's type is then fixed forever, exactly as if you'd written it explicitly. This is purely a **source-code readability feature** — the compiled bytecode is completely identical to the explicit-type version.

```java
var age = 30;
age = "thirty";   // COMPILE ERROR: incompatible types: String cannot be converted to int
                    // -- proves 'age' really is a fixed 'int', not a dynamic type
```

**Restrictions on `var` (deliberate, not arbitrary):**
- Only for **local variables** (and a few other local contexts, like enhanced-for loop variables) — never for fields, method parameters, or return types. **Why this restriction?** Fields and public API signatures (parameters/return types) are part of a class's contract, read far more often than written, often by other developers or tools who benefit enormously from an explicit, visible type — `var` optimizes for the writer's convenience in a narrow, local scope, deliberately not at the cost of a public contract's clarity.
- Must be initialized on the same line as declaration (the compiler needs the right-hand side *right there* to infer from) — `var x;` alone is illegal.
- Cannot be initialized with `null` alone (there's nothing to infer a specific type from).

## Real-World Analogy

Think of a variable declaration like labeling a **storage box** with both a **name tag** and a **size/shape specification** (the type) before you're allowed to put anything in it — `int` is a small, fixed-size box that only holds whole numbers; `String` is a differently-shaped slot that holds a reference to text stored elsewhere (Module 02's Stack/Heap model, again). `var` doesn't remove the size/shape specification — it just lets you say "make the box exactly the right shape for whatever I'm about to put in it," decided once, at labeling time, and fixed forever after.

## Advantages

- Static typing (even with `var`'s inference) catches type errors before the program runs, and enables full IDE tooling support (autocomplete, refactoring, "find usages").
- Strict naming conventions, though not compiler-enforced, create genuinely consistent, readable code across the entire Java ecosystem.
- Definite assignment analysis for local variables eliminates an entire bug category at zero runtime cost.

## Disadvantages / Trade-offs

- More verbose than dynamically-typed languages for simple scripts (mitigated significantly by `var` since Java 10, but never fully eliminated — Java remains statically typed).
- Naming conventions being convention-only (not enforced) means a poorly-disciplined codebase can still violate them freely — tooling (checkstyle, IDE inspections) is typically used in real projects to enforce them automatically.

## Best Practices

- Use `var` when the right-hand side already makes the type obvious (`var list = new ArrayList<String>();`) — but prefer an explicit type when inference would make the type genuinely unclear to a reader (`var result = process();` — what does `process()` return? Not obvious from this line alone).
- Follow naming conventions strictly — camelCase for variables/methods, PascalCase for types, SCREAMING_SNAKE_CASE for constants — every professional Java codebase expects this.
- Always initialize local variables as close to their first use as possible, rather than declaring a batch of variables upfront (a legacy C-style habit that reduces readability in Java).

## Common Mistakes

- Assuming `var` makes Java dynamically typed — it doesn't; the type is fixed at compile time and cannot change.
- Forgetting local variables have no default value, and being confused by "variable might not have been initialized" compiler errors — this is `javac`'s definite assignment analysis working as intended, not a bug.
- Using non-conventional casing (e.g., naming a class `bankAccount` instead of `BankAccount`) — legal, but immediately flags unfamiliarity with Java conventions to any experienced reader.

## Interview Questions

1. **Q: What is `var`, and does it make Java dynamically typed?**
   A: `var` is local variable type inference (Java 10+) — the compiler infers the variable's exact static type from its initializer, once, at compile time. The type is then completely fixed, exactly as if written explicitly; Java remains statically typed. It's purely a source-readability feature with zero effect on the compiled bytecode.

2. **Q: Why do local variables require explicit initialization before use, while instance/static fields don't?**
   A: The compiler performs definite assignment analysis on local variables specifically because there's no universally sensible default for a local computation — forcing explicit initialization eliminates uninitialized-variable bugs at compile time. Fields are automatically defaulted because they're accessed from many methods in orders the compiler can't fully predict, so a safe default guarantees no field is ever read in an undefined state.

3. **Q: What are Java's naming rules for identifiers, and are they enforced by the compiler or just convention?**
   A: Rules (compiler-enforced): must start with a letter/`_`/`$`, contain only letters/digits/`_`/`$` after that, and cannot be a reserved keyword. Conventions (not enforced, but expected): camelCase for variables/methods, PascalCase for types, SCREAMING_SNAKE_CASE for constants.

## Summary

- A variable is a named, typed storage location; Java enforces strict, compiler-checked identifier rules, plus separate, convention-only naming style rules.
- There are three kinds of variables — local (Stack), instance (Heap, part of an object), static (Method Area) — each with different lifetimes, directly tied to Module 02's Runtime Data Areas.
- Local variables have no default value and must be definitely assigned before use; fields are automatically defaulted.
- `var` (Java 10+) is compile-time type inference for local variables only — not dynamic typing.

## Exercises

1. Which of these are legal Java identifiers, and why/why not: `2total`, `_score`, `my-name`, `class`, `$balance`, `firstName`?
2. Write a small class with one static variable, one instance variable, and one local variable inside a method — label which Runtime Data Area (from Module 02) each one lives in.
3. Explain why `var x;` (with no initializer) is illegal, tying your answer to how `var` actually works.
4. Predict what error (if any) this produces, and explain why: `void demo() { int total; total = total + 1; }`

---

**Previous:** [00 — Module Overview](00-Module-Overview.md) · **Next:** [02 — Primitive Data Types](02-Primitive-Data-Types.md)
