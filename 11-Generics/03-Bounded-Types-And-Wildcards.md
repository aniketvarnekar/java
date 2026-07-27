# Bounded Types & Wildcards

## Learning Objectives

- Use bounded type parameters (`<T extends X>`) correctly, including multiple bounds
- Understand precisely why `List<Integer>` is NOT a `List<Object>` — the core problem wildcards solve
- Use `? extends` and `? super` wildcards correctly
- Apply the PECS principle confidently

## Prerequisites

[02 — Generic Methods](02-Generic-Methods.md), Module 05 Topic 5 (Polymorphism — upcasting)

## Motivation

This is the topic most learners find genuinely difficult on first exposure — not because any single piece is complicated, but because it requires holding several ideas in your head simultaneously. Take it slowly; the PECS mnemonic at the end will give you a fast, reliable way to apply everything correctly without re-deriving it from scratch every time.

## Bounded Type Parameters — Restricting What `T` Can Be

Topic 2 previewed this with `findMax`. The full picture:

```java
public class NumberBox<T extends Number> {   // T must be Number OR A SUBCLASS of Number
    private T value;
    public NumberBox(T value) { this.value = value; }
    public double asDouble() {
        return value.doubleValue();   // LEGAL -- the bound GUARANTEES value has doubleValue(),
    }                                    // since Number declares it (Module 03's wrapper classes
}                                         // ALL extend Number!)

NumberBox<Integer> b1 = new NumberBox<>(5);      // OK -- Integer extends Number
NumberBox<Double> b2 = new NumberBox<>(3.14);      // OK -- Double extends Number
NumberBox<String> b3 = new NumberBox<>("hi");        // COMPILE ERROR -- String does NOT extend Number
```

**Why does bounding exist at all?** Without a bound, `T` could be *any* type — meaning the compiler can only assume it has the methods **every** type has (from `Object`, Module 07, Topic 1). Bounding `T extends Number` tells the compiler "whatever `T` ends up being, I guarantee it's at least a `Number`" — unlocking the ability to call `Number`'s own methods (`doubleValue()`, `intValue()`, etc.) directly inside the generic class, something an unbounded `T` could never safely permit.

**Multiple bounds** are also legal, using `&` (at most one **class** bound, which must come first, plus any number of **interface** bounds):

```java
public static <T extends Number & Comparable<T>> T max(T a, T b) {
    return a.compareTo(b) > 0 ? a : b;
}
```

## The Core Problem: Generics Are NOT Covariant (Unlike Arrays!)

This is the single most important, most surprising fact in this entire topic:

```java
List<Integer> integers = new ArrayList<>();
List<Object> objects = integers;   // COMPILE ERROR !! -- even though Integer IS-A Object (Module 05)!
```

**Why does this fail, when `Integer extends Object` (Module 05, Topic 4) is obviously true?** This is a **deliberate, safety-preserving restriction**, not an oversight. If it were allowed:

```java
List<Integer> integers = new ArrayList<>();
List<Object> objects = integers;    // hypothetically ALLOWED
objects.add("a string!");             // objects thinks it's a List<Object> -- adding a String seems fine...
Integer first = integers.get(0);        // ...but 'integers' is ACTUALLY the same list!
                                           // THIS would throw ClassCastException at runtime --
                                           // exactly the bug generics were introduced to PREVENT (Topic 1)!
```

**This is genuinely important to contrast with arrays**, which — for historical reasons predating generics — **are** covariant, and this is now considered a real design wart:

```java
Integer[] intArray = new Integer[3];
Object[] objArray = intArray;         // LEGAL for arrays! (arrays are covariant, historically)
objArray[0] = "a string!";              // COMPILES... but throws ArrayStoreException AT RUNTIME!
                                           // (a runtime safety net arrays have specifically BECAUSE
                                           //  they allow this unsafe-looking assignment in the first place)
```

**Generics deliberately do NOT repeat this mistake** — `List<Integer>` and `List<Object>` are treated as **completely unrelated types**, full stop, **despite** `Integer` and `Object` themselves having a genuine IS-A relationship. This is exactly what "generics are invariant" means, and it's a **deliberate, compile-time-safety-preserving design choice**, closing off the exact loophole arrays' covariance leaves open.

## Wildcards — The Controlled, Safe Escape Hatch

Sometimes you genuinely **do** want some flexibility — a method that can accept a `List<Integer>`, a `List<Double>`, or any other `List` of some `Number` subtype, without caring about the *exact* specific type. **Wildcards** (`?`) provide this, safely:

### `? extends T` — "Producer" — Read-Only Access

```java
public static double sumAll(List<? extends Number> list) {   // accepts List<Integer>, List<Double>,
    double total = 0;                                           // List<Number> itself, ANY Number subtype
    for (Number n : list) {
        total += n.doubleValue();     // READING is safe -- whatever it actually is, it's AT LEAST a Number
    }
    return total;
}

sumAll(List.of(1, 2, 3));        // works -- List<Integer>
sumAll(List.of(1.5, 2.5));         // works -- List<Double>
```

```java
List<? extends Number> list = new ArrayList<Integer>();
list.add(5);   // COMPILE ERROR !! -- even though 5 is an int/Integer...
```

**Why can't you `add` to a `List<? extends Number>`, even an `Integer`?** The compiler genuinely doesn't know the list's **exact** underlying type — it could be `List<Integer>`, `List<Double>`, or any other `Number` subtype's list, and it deliberately refuses to guess. Adding an `Integer` might be perfectly safe (if it's really a `List<Integer>`) or genuinely unsafe (if it's actually a `List<Double>`) — since the compiler can't tell which, it conservatively **forbids all insertion**, permitting only **reading** (since reading an element and treating it as *at least* a `Number` is always safe, regardless of the exact underlying type).

### `? super T` — "Consumer" — Write-Access, Restricted Reading

```java
public static void addNumbers(List<? super Integer> list) {   // accepts List<Integer>, List<Number>,
    list.add(1);                                                 // List<Object> -- ANY Integer SUPERtype
    list.add(2);                                                   // ADDING is safe -- an Integer is always
}                                                                    // legal to add to any of these

List<Number> numbers = new ArrayList<>();
addNumbers(numbers);   // works -- List<Number> IS a valid "? super Integer"

List<Object> objects = new ArrayList<>();
addNumbers(objects);     // ALSO works -- List<Object> is ALSO a valid "? super Integer"
```

```java
List<? super Integer> list = new ArrayList<Number>();
Integer x = list.get(0);   // COMPILE ERROR !! -- get() only guarantees an Object, not specifically an Integer
Object o = list.get(0);       // this works -- Object is always a safe, guaranteed catch-all type here
```

**Why can you `add` but not safely `get` a specific type from `? super Integer`?** The compiler knows the list holds **Integer, or some supertype of Integer** — so adding an `Integer` is always safe (an `Integer` fits into any of those possibilities). But **reading**, the compiler only knows the result is *at least* an `Object` (Module 07, Topic 1's universal root) — it can't guarantee anything more specific, since the list might actually be a `List<Object>` holding all sorts of unrelated things beyond just Integers.

## PECS — "Producer Extends, Consumer Super"

This is the industry-standard mnemonic (coined by Joshua Bloch, a genuinely famous, foundational figure in Java API design) for remembering which wildcard to use, without re-deriving the reasoning from scratch every time:

> **PECS: if a parameterized type is a "Producer" (you only ever READ FROM it), use `? extends T`. If it's a "Consumer" (you only ever WRITE TO it), use `? super T`.**

```
 PRODUCER  (you READ values OUT)      ──▶  use  ?  extends  T
 CONSUMER  (you WRITE values IN)      ──▶  use  ?  super     T
 BOTH (read AND write)                  ──▶  use  T  directly (no wildcard) -- accept only the EXACT type
```

```java
// Collections.copy's actual real signature is a perfect, canonical PECS example:
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    //                              ↑ CONSUMER: we WRITE into dest      ↑ PRODUCER: we READ from src
    for (T item : src) {
        dest.add(item);
    }
}
```

**This single real, standard-library method signature demonstrates both wildcard forms simultaneously, each used for precisely the reason PECS predicts** — `src` is only ever read from (a Producer, `? extends T`), `dest` is only ever written to (a Consumer, `? super T`).

## Unbounded Wildcard — `?` Alone

```java
public static void printSize(List<?> list) {   // accepts a List of ANY type whatsoever
    System.out.println("Size: " + list.size());   // size() doesn't depend on the element type at all
}
```

Used when a method genuinely doesn't care about the element type at all — only about `Collection`-level operations that don't depend on the specific type (`size()`, `isEmpty()`, iterating and treating everything as `Object`).

## Real-World Analogy

Think of `? extends Number` like a **"deliveries only" dock** — trucks can drop off any kind of Number-family cargo (Integers, Doubles), and you can safely unload and inspect whatever arrives (treating it generically as "at least a Number"), but you're not allowed to load anything **onto** the trucks yourself, since you genuinely don't know which specific Number sub-type each truck is actually configured to carry. Think of `? super Integer` like a **"pickups only" dock accepting Integer-compatible cargo** — you can confidently load Integers onto any truck here (since every truck is guaranteed to accept at least Integers, being an Integer or some Integer supertype), but you can't reliably inspect what's already on a given truck beyond "it's cargo of some kind" (`Object`), since the truck might be configured for a broader category than just Integers.

## Advantages

- Generics' deliberate invariance closes the exact unsafe-assignment loophole array covariance leaves open (Module 09's arrays now shown to have a real, historical design wart generics deliberately avoided repeating).
- Wildcards provide precisely the safe flexibility needed for read-only/write-only scenarios, without reopening that unsafe loophole.
- PECS gives a fast, reliable, memorable rule for choosing the correct wildcard, without needing to re-derive the underlying reasoning every time.

## Disadvantages / Trade-offs

- Wildcard syntax (`? extends`, `? super`) and their asymmetric read/write restrictions are genuinely one of the harder concepts in the entire language to internalize on first exposure.
- Bounded type parameters and wildcards together create a real learning curve before advanced generic API design (like writing your own PECS-following utility methods) feels natural.

## Best Practices

- Apply PECS directly: read-only parameter → `? extends T`; write-only parameter → `? super T`; both read and write → no wildcard, use the exact type `T`.
- Remember generics are invariant by design — `List<Integer>` is never a `List<Object>`, even though `Integer` IS-A `Object`.
- Use bounded type parameters (`<T extends X>`) whenever a generic method/class needs to call `X`'s specific methods on values of type `T`.

## Common Mistakes

- Assuming `List<Integer>` can be assigned to a `List<Object>` variable, based on `Integer`'s IS-A relationship with `Object` — generics don't work this way, deliberately.
- Attempting to `add()` to a `List<? extends T>` — always a compile error, by design.
- Attempting to retrieve anything more specific than `Object` from a `List<? super T>` — the compiler correctly refuses to guess a more specific type.
- Misapplying PECS backward (using `? super` for read-only, or `? extends` for write-only parameters).

## Interview Questions

1. **Q: Why is `List<Integer>` not considered a subtype of `List<Object>`, even though `Integer` IS-A `Object`?**
   A: Generics are deliberately invariant, closing a real unsafe-assignment loophole that arrays' covariance (a historical, pre-generics design choice) leaves open — allowing it would let code add, say, a `String` to what's actually a `List<Integer>` through a mistakenly-typed `List<Object>` reference, producing a runtime `ClassCastException` on later retrieval, exactly the class of bug generics were introduced to eliminate.

2. **Q: What does the PECS mnemonic mean, and how do you apply it?**
   A: "Producer Extends, Consumer Super" — use `? extends T` for a parameter you only read values from (a "producer"), and `? super T` for a parameter you only write values into (a "consumer"). If both reading and writing are needed, use the exact type `T` with no wildcard.

3. **Q: Why can't you call `list.add(...)` on a `List<? extends Number>`?**
   A: The compiler doesn't know the list's exact underlying type parameter — it could be `List<Integer>`, `List<Double>`, or any other `Number` subtype — so it conservatively forbids insertion entirely, since any specific addition might not actually match the list's real, unknown-to-the-compiler element type. Reading is safe, since any element is guaranteed to be at least a `Number`, regardless of the exact subtype.

## Summary

- Bounded type parameters (`<T extends X>`) let a generic class/method rely on `X`'s specific methods being available on `T`.
- Generics are **invariant** by design: `List<Integer>` and `List<Object>` are unrelated types, deliberately closing the unsafe loophole array covariance historically leaves open.
- **`? extends T`** ("producer," read-only) and **`? super T`** ("consumer," write-only) are the controlled, safe escape hatches for needed flexibility.
- **PECS** ("Producer Extends, Consumer Super") is the reliable mnemonic for choosing correctly between them.

## Exercises

1. Explain, using the hypothetical unsafe-assignment example from this topic, precisely why `List<Integer> objects = someListOfObject;`-style covariant assignment would be dangerous if generics allowed it.
2. Write a method `static double sumAll(List<? extends Number> list)` and explain, referencing PECS, why `? extends Number` (not `? super Number` or a plain `List<Number>`) is the correct choice here.
3. Given `Collections.copy(List<? super T> dest, List<? extends T> src)`, explain in your own words why `dest` uses `? super T` and `src` uses `? extends T`, applying PECS directly.
4. Explain why `List<?> list` (unbounded wildcard) is an appropriate parameter type for a method that only calls `list.size()`, and why `List<Object>` would be a worse choice for the same method.

---

**Previous:** [02 — Generic Methods](02-Generic-Methods.md) · **Next:** [04 — Type Erasure](04-Type-Erasure.md)
