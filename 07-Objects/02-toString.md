# `toString()`

## Learning Objectives

- Understand exactly what `Object`'s default `toString()` produces, and why
- Override `toString()` correctly and idiomatically
- Know precisely when `toString()` is called implicitly by the language/standard library

## Prerequisites

[01 — The `Object` Class](01-The-Object-Class.md)

## Motivation

`toString()` is likely the single most commonly overridden method in all of Java — nearly every class you write in real projects benefits from a custom one. Understanding exactly *why* the default is unhelpful, and *when* the method gets called implicitly (often surprising beginners), makes this small feature much more useful.

## The Default `toString()` — What It Actually Produces

```java
class Point {
    int x, y;
}

Point p = new Point();
System.out.println(p);          // Point@1b6d3586   (or similar)
```

`Object`'s default `toString()` implementation produces a string in the format:

```
fully.qualified.ClassName@hexadecimalHashCode
```

Specifically, it's `getClass().getName() + "@" + Integer.toHexString(hashCode())` — the class name (via `getClass()`, Topic 1), an `@` symbol, and the object's default hash code (Topic 3) in hexadecimal. **This is almost never useful information for debugging or logging** — it tells you the type and a memory-adjacent identifier, but nothing about the object's actual *state* (its field values), which is virtually always what you actually want to see.

## Overriding `toString()` — The Idiomatic Way

```java
class Point {
    int x, y;

    Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public String toString() {
        return "Point(x=" + x + ", y=" + y + ")";
    }
}

Point p = new Point(3, 4);
System.out.println(p);   // Point(x=3, y=4)
```

**Always use `@Override`** (Module 05, Topic 5) — since `toString()` is inherited from `Object`, this protects against signature typos (e.g., accidentally writing `tostring()`, which would silently create an unrelated new method instead of overriding anything).

## When Is `toString()` Called Implicitly?

This is the part beginners are frequently surprised by — `toString()` is called **automatically**, without you ever writing `.toString()` explicitly, in several extremely common situations:

```java
System.out.println(p);              // println calls p.toString() internally
String message = "Point is: " + p;    // STRING CONCATENATION calls p.toString() internally!
System.out.println("Point: " + p);     // same as above
```

**Why does `"text" + p` call `toString()`?** Recall Module 03's operators: `+` performs **String concatenation** when either operand is a `String`. When the *other* operand is an object (not already a `String`), Java automatically calls that object's `toString()` to obtain text to concatenate with — this is a compiler-inserted, implicit call, exactly analogous to autoboxing (Module 03, Topic 6) being a compiler-inserted, implicit conversion. **This is precisely why overriding `toString()` has such broad, automatic payoff** — you write it once, and it improves *every* place your object ever gets printed or concatenated into a string, throughout your entire codebase, without needing to remember to call it explicitly anywhere.

## Real-World Analogy

Think of `toString()` like a **name tag at a conference**. `Object`'s default is like a name tag that only shows a randomly assigned attendee ID number (`Point@1b6d3586`) — technically unique and "identifying," but useless for actually recognizing who someone is. An overridden `toString()` is like writing your **actual name and role** on the tag (`Point(x=3, y=4)`) — genuinely useful information, visible automatically to anyone who glances at it (every `println`, every string concatenation), without them needing to specifically ask "what's your ID number, and can you look it up in a database for me."

## Advantages

- Dramatically improves debugging and logging output for essentially zero ongoing cost — write it once, benefit everywhere the object is ever printed/concatenated.
- Implicit invocation (via `println`, string concatenation) means the improvement is automatic and pervasive, not something callers need to remember to invoke.

## Disadvantages / Trade-offs

- A `toString()` that includes sensitive data (passwords, tokens, PII) can inadvertently leak that information into logs — a real, practical security/privacy concern worth being deliberate about when designing what a `toString()` actually includes.
- For classes with many fields, a naive `toString()` can become long/unwieldy — worth being deliberate about which fields are actually useful to include, rather than mechanically dumping everything.

## Best Practices

- Override `toString()` for essentially every class you write that isn't purely a stateless utility/behavior class — the payoff for debugging is consistently high relative to the (small) effort.
- Include the class name and the most identifying/useful fields; omit sensitive data.
- Always use `@Override` to guard against signature typos.

## Common Mistakes

- Assuming an object with no explicit `toString()` will fail to print or throw an error — it always has `Object`'s default, it just produces unhelpful output.
- Not realizing `+` concatenation implicitly calls `toString()` — leading to confusion about why a custom `toString()` "magically" improves output in places it was never explicitly called.
- Accidentally including sensitive fields in a `toString()` output that later ends up in application logs.

## Interview Questions

1. **Q: What does `Object`'s default `toString()` produce, and why is it generally unhelpful?**
   A: `fully.qualified.ClassName@hexadecimalHashCode` — it identifies the type and a hash-derived identifier, but reveals nothing about the object's actual field values/state, which is almost always what's actually useful for debugging.

2. **Q: When is `toString()` called implicitly, without an explicit `.toString()` call in the source?**
   A: Whenever an object is passed to `System.out.println()` (and similar print methods), and whenever an object is used as an operand in String concatenation (`"text" + object`) — the compiler inserts an implicit `toString()` call in both cases.

## Summary

- `Object`'s default `toString()` produces `ClassName@hexHashCode` — rarely useful for debugging.
- Overriding `toString()` to show meaningful field values pays off automatically, everywhere the object is printed or concatenated into a String, since `println` and `+` concatenation both call it implicitly.
- Always use `@Override`, and be deliberate about excluding sensitive data.

## Exercises

1. Write a `Book` class with `title` and `author` fields, override `toString()` appropriately, and demonstrate it being called implicitly via both `System.out.println(book)` and `"Book: " + book`.
2. Explain precisely why `"Total: " + 5 + 3` produces `"Total: 53"` while `"Total: " + (5 + 3)` produces `"Total: 8"`, connecting your answer to how `+` concatenation and `toString()`/numeric addition interact (revisit Module 03's operator precedence if needed).
3. Explain one real risk of including every field in a class's `toString()` without consideration, and how you'd address it.

---

**Previous:** [01 — The `Object` Class](01-The-Object-Class.md) · **Next:** [03 — `equals()` and `hashCode()`](03-equals-And-hashCode.md)
