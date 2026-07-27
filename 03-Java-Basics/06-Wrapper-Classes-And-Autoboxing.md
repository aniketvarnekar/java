# Wrapper Classes & Autoboxing

## Learning Objectives

- Explain why every primitive type has a corresponding wrapper class
- Understand autoboxing/unboxing and exactly when the compiler inserts them
- Explain the `Integer` caching behavior and the `==` vs `.equals()` trap it causes
- Know the `NullPointerException` risk unboxing introduces

## Prerequisites

[02 — Primitive Data Types](02-Primitive-Data-Types.md), Module 02 Topic 3 (Runtime Data Areas — Stack vs Heap)

## Motivation

This topic directly resolves a tension from Topic 2: Java has 8 non-object primitive types for performance, but Java is also supposed to be "everything is an object" (Module 01). Wrapper classes are the bridge — and the specific pitfall they introduce (`Integer` caching + `==`) is one of the single most common "gotcha" interview questions in all of Core Java.

## Problem Statement

Primitives can't be used everywhere objects are required:
- Generic collections (`ArrayList<T>`, `HashMap<K,V>` — Module 10/11) require **object** type parameters — `ArrayList<int>` is illegal; there's no way to write a generic class that works with raw, non-object primitive values directly.
- Primitives can't be `null` — sometimes you genuinely need to represent "no value present" for a numeric field (e.g., a database column that's nullable).
- Certain APIs (like reflection, or anything designed around `Object`) need a uniform, object-based way to represent *any* value, including what would otherwise be a primitive.

Java needs a way to "wrap" a primitive value inside a real object when one of these situations demands it.

## Concept: The 8 Wrapper Classes

Every primitive type has a corresponding wrapper class in `java.lang`:

| Primitive | Wrapper Class |
|---|---|
| `byte` | `Byte` |
| `short` | `Short` |
| `int` | `Integer` |
| `long` | `Long` |
| `float` | `Float` |
| `double` | `Double` |
| `char` | `Character` |
| `boolean` | `Boolean` |

A wrapper class is an ordinary object — it lives on the Heap, has methods, can be `null`, and can be used anywhere an `Object` is required (including generics):

```java
Integer boxedAge = Integer.valueOf(30);   // an actual object, on the Heap, wrapping the int value 30
int age = boxedAge.intValue();              // unwrapping it back to a primitive
```

## Autoboxing & Unboxing (Java 5+)

Manually calling `Integer.valueOf(...)` and `.intValue()` everywhere would be tedious, so since Java 5, the **compiler does this conversion for you automatically** wherever needed — this is called **autoboxing** (primitive → wrapper) and **unboxing** (wrapper → primitive):

```java
Integer boxedAge = 30;         // AUTOBOXING: compiler inserts Integer.valueOf(30) automatically
int age = boxedAge;              // UNBOXING: compiler inserts boxedAge.intValue() automatically

List<Integer> ages = new ArrayList<>();
ages.add(25);                     // autoboxing: int 25 -> Integer object, added to the list
int firstAge = ages.get(0);        // unboxing: Integer object -> int
```

**Critical thing to understand: this is purely a compile-time source-code convenience.** The compiler literally rewrites `Integer boxedAge = 30;` into `Integer boxedAge = Integer.valueOf(30);` behind the scenes, before generating bytecode — there's no runtime "magic," just automatic method-call insertion by `javac`.

## The `Integer` Cache — A Genuine, Famous Interview Trap

Here's where it gets interesting, and directly ties back to Module 02's Stack/Heap and reference model:

```java
Integer a = 100;
Integer b = 100;
System.out.println(a == b);       // prints: true  !!

Integer x = 200;
Integer y = 200;
System.out.println(x == y);        // prints: false  !!
```

**Why does the exact same-looking code behave differently just because the number changed from 100 to 200?**

`Integer.valueOf(int)` (which autoboxing calls behind the scenes) maintains an internal **cache** of `Integer` objects for the range **-128 to 127** — for any value in that range, `valueOf` returns a **shared, pre-existing** cached object instead of creating a new one. For values **outside** that range, it creates a genuinely **new** `Integer` object every time.

```
Integer a = 100;   -> Integer.valueOf(100) -> returns the SAME cached object every time (100 is in -128..127)
Integer b = 100;   -> Integer.valueOf(100) -> returns that SAME cached object again
a == b  ->  true (both variables reference the IDENTICAL object)

Integer x = 200;   -> Integer.valueOf(200) -> creates a NEW object (200 is outside -128..127)
Integer y = 200;   -> Integer.valueOf(200) -> creates ANOTHER NEW, different object
x == y  ->  false (two DIFFERENT objects, even though their wrapped values are equal)
```

**Why does `==` behave this way at all?** Recall from Module 02, Topic 3: for **object** types, a variable holds a **reference** (a Heap address), and `==` on object references compares whether they point to the **exact same object**, not whether the wrapped values are logically equal. `100 == 100` (primitives) always compares values directly and is always `true` — but `Integer(100) == Integer(100)` (objects) depends entirely on whether both references happen to point at the *same* object, which depends on this caching implementation detail.

**Why does the cache exist, and why specifically -128 to 127?** Small integer values are used *extremely* frequently in typical programs (loop counters, small counts, array indices), so pre-creating and reusing a fixed pool of common small `Integer` objects avoids constant, wasteful re-allocation on the Heap for these extremely common values — a genuine, deliberate performance optimization. The range -128 to 127 was chosen as a reasonable, historically-established default; it's technically guaranteed by the JLS to include at least this range and is tunable via a JVM flag (`-XX:AutoBoxCacheMax`) in HotSpot, though relying on that is not standard practice.

### The Correct Fix

**Always use `.equals()`, never `==`, to compare wrapper objects for logical equality:**

```java
Integer x = 200;
Integer y = 200;
System.out.println(x.equals(y));   // true -- correctly compares the wrapped VALUES, not object identity
```

> This exact same "`==` compares identity, `.equals()` compares logical value" distinction reappears, even more commonly, with `String` in Module 08 — this topic is deliberately your first, foundational encounter with a pattern you'll see constantly for the rest of the course.

## Unboxing and `NullPointerException`

Since a wrapper object *can* be `null` (unlike a primitive, which never can be), **unboxing a `null` wrapper throws `NullPointerException`**:

```java
Integer count = null;
int total = count;    // throws NullPointerException! (compiler inserts count.intValue(),
                        // and calling a method on a null reference always throws NPE)
```

This is a genuinely common, real production bug — especially when a wrapper-typed field (e.g., from a database row, or an API response) is unexpectedly `null` and gets silently, automatically unboxed somewhere deep in ordinary-looking arithmetic or comparison code.

```java
Integer accountBalance = getBalanceFromDatabase();  // might return null if no record found
if (accountBalance > 0) {   // if accountBalance is null, THIS LINE throws NullPointerException
    // ...
}
```

## Performance Cost of Autoboxing in Loops

Because autoboxing/unboxing happens **silently**, it's easy to accidentally introduce significant, invisible overhead in performance-sensitive code:

```java
Long sum = 0L;                       // sum is a WRAPPER (Long), not a primitive (long)
for (long i = 0; i < 1_000_000; i++) {
    sum += i;    // EVERY iteration: unbox sum, add, autobox the NEW result back into a
                   // brand new Long object -- a million needless Heap allocations!
}
```
**Fix:** use the primitive type (`long sum = 0L;`) for accumulator variables in hot loops — this is a real, practical performance best practice directly explained by everything you've learned in this topic and Module 02.

## When Wrapper Classes Are Genuinely Necessary

- Generic collections: `List<Integer>`, `Map<String, Double>`, etc. (Modules 10–11) — generics fundamentally require object type parameters.
- Representing "no value" (`null`) for what's conceptually a numeric field — e.g., an optional configuration value, or a nullable database column mapped into Java.
- APIs designed around `Object` (older reflection-based or generic utility code).

## Real-World Analogy

Think of a primitive as **cash in your pocket** — direct, immediate, no container needed. Think of a wrapper object as **that same cash sealed inside a labeled envelope** — now it can be filed in a folder (a generic collection, which only accepts "envelopes," not raw cash), handed around as a single object, or the envelope itself can be empty (`null`) to represent "no cash here at all," which loose cash in your pocket can never represent. The `Integer` cache is like a **pre-printed, reusable set of envelopes for common small amounts** (-128 to 127) that the bank hands out instead of sealing a brand-new envelope every single time — two people asking for a "$50 envelope" might get handed the exact same physical envelope, while two people asking for a "$50,000 envelope" each get their own distinct one.

## Advantages

- Enables primitives to participate in generics and any `Object`-based API, closing the gap left by Topic 2's performance-driven primitive/object split.
- Autoboxing/unboxing removes tedious manual conversion boilerplate.
- The `Integer` cache is a genuine, effective performance optimization for extremely common small values.

## Disadvantages / Trade-offs

- The caching behavior creates a real, sharp-edged `==` vs `.equals()` trap that has caused countless real bugs and is a near-universal interview question.
- Silent unboxing of `null` risks `NullPointerException` in code that looks completely unremarkable.
- Silent, repeated autoboxing in loops/hot paths introduces invisible Heap allocation overhead.

## Best Practices

- **Always** use `.equals()` (never `==`) to compare wrapper objects for value equality.
- Use primitive types, not wrapper types, for local loop counters/accumulators wherever a `null` value is never genuinely needed.
- Be deliberately cautious around unboxing any wrapper-typed value that could plausibly be `null` (data from a database, an external API, or any "optional" field) — check for `null` explicitly before letting it participate in arithmetic or comparisons.

## Common Mistakes

| Mistake | Correction |
|---|---|
| `if (wrapperA == wrapperB)` to check value equality | Use `.equals()` — `==` on wrapper objects compares object identity, which is only reliably `true` for cached small values (-128 to 127). |
| Assuming `Integer x = 200; Integer y = 200; x == y` is always `true` because it "worked" for smaller numbers in testing | The `Integer` cache only covers -128 to 127 by default — behavior silently changes outside that range. |
| Using a wrapper type for a loop accumulator | Introduces unnecessary autoboxing/unboxing overhead on every iteration — use the primitive type instead. |
| Comparing a wrapper to a primitive with `==` (e.g., `Integer wrapped = 200; if (wrapped == 200)`) | This one is actually **safe** — comparing an object reference to a primitive with `==` forces **unboxing** of the wrapper first (since a reference and a primitive can't be compared directly), so this correctly compares values, not identity. The trap is specifically **wrapper-to-wrapper** `==` comparison. |

## Interview Questions

1. **Q: Why does `Integer a = 100; Integer b = 100; a == b` print `true`, but the same code with `200` instead of `100` prints `false`?**
   A: `Integer.valueOf(int)`, which autoboxing calls internally, caches and reuses `Integer` objects for values in the range -128 to 127. `100` falls in this range, so both variables reference the same cached object, making `==` (identity comparison) true. `200` falls outside the cache range, so two genuinely distinct `Integer` objects are created, making `==` false — even though their wrapped values are logically equal.

2. **Q: What is autoboxing, and is it a runtime or compile-time mechanism?**
   A: Autoboxing is the automatic conversion of a primitive value into its corresponding wrapper object (and unboxing, the reverse) — it's inserted entirely by the **compiler** at compile time (e.g., `Integer x = 5;` is rewritten to `Integer x = Integer.valueOf(5);`), not a runtime feature.

3. **Q: Why can unboxing a wrapper object throw `NullPointerException`?**
   A: Because wrapper objects, unlike primitives, can be `null`. Unboxing inserts an implicit method call (e.g., `.intValue()`) on the wrapper reference — calling any method on a `null` reference throws `NullPointerException`, and this can happen silently in code that looks like ordinary primitive arithmetic.

4. **Q: Why shouldn't you use a wrapper type (like `Long`) as an accumulator variable in a performance-sensitive loop?**
   A: Each update requires unboxing the current value, computing the new value, then autoboxing the result back into a brand-new wrapper object on the Heap — repeated, unnecessary Heap allocation on every iteration, compared to a primitive accumulator which updates in place with no allocation at all.

## Summary

- Every primitive has a corresponding wrapper class (`int`→`Integer`, etc.) — real Heap-allocated objects that bridge primitives into Java's object-oriented world (generics, `null`, `Object`-based APIs).
- **Autoboxing/unboxing** (Java 5+) is a compiler-inserted, compile-time convenience — not runtime magic.
- `Integer` (and other wrapper types) cache small values (-128 to 127 by default) for performance, which means **`==` on wrapper objects is unreliable** for anything outside that range — always use `.equals()` for value comparison.
- Unboxing a `null` wrapper throws `NullPointerException`, a real and common production bug source.
- Prefer primitives over wrappers for performance-sensitive local variables (loop counters/accumulators) to avoid invisible autoboxing overhead.

## Exercises

1. Without running code, predict the output of comparing `Integer a = -128; Integer b = -128; a == b` and `Integer a = -129; Integer b = -129; a == b`, and explain the boundary precisely.
2. Explain why `Integer count = null; if (count == 0) { ... }` throws an exception, and identify exactly which line and mechanism causes it.
3. Rewrite this loop to avoid unnecessary autoboxing overhead: `Integer total = 0; for (int i = 0; i < 100; i++) { total += i; }`
4. Explain, in your own words, why comparing a wrapper object to a primitive with `==` (e.g., `Integer x = 500; x == 500`) is actually safe, while comparing two wrapper objects with `==` is not.

---

**Previous:** [05 — Operators](05-Operators.md) · **Next:** [07 — Constants & `final`](07-Constants-And-Final.md)
