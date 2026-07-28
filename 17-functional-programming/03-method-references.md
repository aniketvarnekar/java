# Method References

## Learning Objectives

- Use all four kinds of method references correctly
- Recognize when a method reference is clearer than the equivalent lambda, and when it isn't

## Prerequisites

[02 — Lambda Expressions](02-lambda-expressions.md)

## Motivation

Sometimes a lambda's entire body is just "call this one existing method." Method references are Java's syntax for expressing exactly that redundancy-free — genuinely just a more concise notation for a specific, common lambda shape, not a new concept.

## The Core Idea

```java
// A lambda that does nothing but call an existing method:
Function<String, Integer> parser = s -> Integer.parseInt(s);

// The SAME thing, as a method reference:
Function<String, Integer> parser2 = Integer::parseInt;
```

**`Integer::parseInt` is pure syntactic sugar for `s -> Integer.parseInt(s)`** — when a lambda's entire job is "pass my parameter(s) directly through to an existing method," the method reference form (`ClassOrObject::methodName`) says exactly that, without the redundant "wrapper" ceremony of restating the parameter and re-invoking the method explicitly.

## The Four Kinds of Method References

### 1. Static Method Reference

```java
Function<String, Integer> parser = Integer::parseInt;   // ClassName::staticMethod
```

### 2. Instance Method Reference on a Particular, Already-Existing Object

```java
String greeting = "Hello";
Supplier<Integer> lengthGetter = greeting::length;   // particularObject::instanceMethod
```

### 3. Instance Method Reference on an Arbitrary Object of a Particular Type (a genuinely common, slightly less intuitive form)

```java
Function<String, Integer> lengthGetter = String::length;
// equivalent to: s -> s.length()
// the METHOD REFERENCE'S "parameter" becomes the OBJECT the instance method is called ON
```

**This third form is worth pausing on, since it's the one most learners initially find confusing**: `String::length` doesn't mean "call `length()` on some specific, already-known `String`" — it means **"take whatever `String` argument is passed in, and call `.length()` on THAT argument."** The functional interface's parameter **becomes** the receiver of the instance method call.

### 4. Constructor Reference

```java
Supplier<ArrayList<String>> listFactory = ArrayList::new;   // ClassName::new
ArrayList<String> newList = listFactory.get();                // calls new ArrayList<String>()
```

## The Quick Decision Rule

```
 Does your lambda's body do NOTHING except pass its parameter(s) straight through
 to one single existing method (or constructor), with no other logic at all?

    YES  ->  a method reference is available and generally preferred (more concise, clearer intent)
    NO   ->  write a regular lambda -- method references can't express any additional logic
```

```java
// Method reference works -- purely a direct pass-through:
names.forEach(System.out::println);

// Method reference does NOT work -- there's ADDITIONAL logic beyond a pure pass-through:
names.forEach(name -> System.out.println("Name: " + name));   // must stay a lambda
```

## Why Prefer Method References When Applicable

**Beyond pure brevity, a method reference states intent more directly**: `Integer::parseInt` immediately, visually communicates "this is literally just `Integer.parseInt`, nothing more, nothing hidden" — a reader doesn't need to parse through lambda parameter/body syntax to confirm there's no additional logic tucked inside. This is a genuinely real, if modest, readability win, exactly analogous to Module 06, Topic 3's "return `this`" fluent-chaining pattern being immediately recognizable once you know the idiom.

## Real-World Analogy

Think of a lambda like **writing out full, explicit instructions**: "take this ingredient, and put it directly into the mixer." A method reference is like a **shorthand notation everyone in the kitchen already understands**: "ingredient → mixer" — conveying the exact same instruction, with zero loss of meaning, but without restating the obvious mechanical steps explicitly every single time. If the actual instruction were more complex ("take this ingredient, weigh it first, then put only half into the mixer"), the shorthand notation genuinely couldn't capture that — you'd need the full, explicit instructions (a lambda) instead.

## Advantages

- More concise than the equivalent pure-pass-through lambda, with clearer, more immediately recognizable intent.
- Directly reuses existing, already-tested methods rather than re-wrapping them in a new lambda body.

## Disadvantages / Trade-offs

- Only applicable for the specific "pure pass-through, no additional logic" shape — most lambdas with any real logic cannot be expressed as a method reference at all.
- The third form (`String::length`-style, arbitrary-object-of-a-type) can be genuinely confusing on first encounter, since the "receiver" isn't a specific, already-known object.

## Best Practices

- Use a method reference whenever a lambda's body is a pure pass-through to an existing method or constructor — it's the more idiomatic, immediately recognizable modern Java style.
- Don't force a method reference where it doesn't naturally fit — a lambda with genuine logic should simply remain a lambda.

## Common Mistakes

- Attempting to use a method reference for a lambda that has any logic beyond a pure pass-through — this simply isn't expressible as a method reference at all.
- Misunderstanding the third form (`Type::instanceMethod`) as referring to some specific existing object, rather than "whatever argument gets passed in becomes the receiver."

## Interview Questions

1. **Q: What are the four kinds of method references in Java?**
   A: Static method reference (`ClassName::staticMethod`), instance method reference on a particular object (`object::instanceMethod`), instance method reference on an arbitrary object of a type (`ClassName::instanceMethod`), and constructor reference (`ClassName::new`).

2. **Q: What does `String::length` actually mean, given there's no specific String object mentioned?**
   A: It means "take whatever String argument is passed to this functional interface, and call `.length()` on that argument" — the functional interface's parameter becomes the receiver of the instance method call, rather than referring to some already-known, specific object.

3. **Q: When can a lambda NOT be rewritten as a method reference?**
   A: Whenever its body contains any logic beyond a pure, direct pass-through to a single existing method or constructor — method references can only express that one specific, redundancy-free shape.

## Summary

- A **method reference** (`ClassOrObject::methodName`) is concise syntax for a lambda whose entire body is a pure pass-through to an existing method or constructor — four forms: static, bound instance, unbound instance (arbitrary receiver), and constructor.
- Prefer method references over an equivalent pure-pass-through lambda for their conciseness and immediately recognizable intent; lambdas remain necessary for any logic beyond a direct pass-through.