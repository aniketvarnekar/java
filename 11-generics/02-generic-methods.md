# Generic Methods

## Learning Objectives

- Write generic methods, independent of whether their enclosing class is generic
- Understand type inference and when explicit type witnesses are needed
- Recognize generic methods throughout the standard library you've already used

## Prerequisites

[01 — Why Generics — Introduction](01-why-generics-introduction.md)

## Motivation

Topic 1 showed generic **classes** (`Box<T>`). A method can **independently** be generic too — even inside an entirely non-generic class — which is precisely how many standard library utility methods you've already used (`Collections.sort`, `Arrays.asList`, `List.of`) actually work.

## Problem Statement

Sometimes you need a method whose behavior is identical regardless of the specific type it operates on — "swap two elements in an array," "find the first matching element," "print every item" — but that still needs full type safety without resorting to `Object` and casting (Topic 1's pre-generics problem, at the method level instead of the class level).

## Writing a Generic Method

```java
public class ArrayUtils {                  // NOTE: this class itself is NOT generic!
    public static <T> void swap(T[] array, int i, int j) {   // <T> HERE declares the method's OWN
        T temp = array[i];                                     // type parameter, independent of the
        array[i] = array[j];                                    // enclosing class
        array[j] = temp;
    }
}
```

```java
Integer[] nums = {1, 2, 3};
ArrayUtils.swap(nums, 0, 2);      // T is inferred as Integer

String[] words = {"a", "b", "c"};
ArrayUtils.swap(words, 0, 1);       // T is inferred as String -- the SAME method, different type
```

**The `<T>` immediately before the return type** is what declares this as a generic method, introducing `T` as a fresh type parameter scoped to **this method only** — genuinely independent of any type parameter the enclosing class might (or might not) have. `ArrayUtils` itself has zero type parameters; `swap` has its own, entirely self-contained.

## Type Inference

In the calls above, you never wrote `ArrayUtils.<Integer>swap(nums, 0, 2)` — the compiler **inferred** `T = Integer` automatically, from the argument types you actually passed. This is called **type inference**, and it works correctly the overwhelming majority of the time — you rarely need to specify a generic method's type argument explicitly.

**When explicit type witnesses ARE occasionally needed** — when the compiler genuinely cannot infer the type from context alone (e.g., an empty collection literal with no other type-carrying arguments to infer from):

```java
List<String> empty = Collections.<String>emptyList();    // explicit type witness -- rarely needed in
                                                              // modern code, since target-typing (the
                                                              // compiler using the ASSIGNMENT's declared
                                                              // type, List<String>, to infer) usually
                                                              // handles this automatically since Java 8
List<String> empty2 = Collections.emptyList();               // usually just works, via target-typing
```

## Generic Methods You've Already Used

Recognizing generics you've used throughout this course, now with full understanding of their declaration:

```java
public static <T> void sort(List<T> list, Comparator<? super T> c) { ... }   // Collections.sort (Module 10)
public static <T> List<T> asList(T... a) { ... }                                // Arrays.asList (Module 09)
public static <T> List<T> of(T... elements) { ... }                                // List.of (Module 10)
```

`Collections.sort`'s exact signature (shown simplified above) references **wildcards** (`? super T`) — the subject of Topic 3, coming next. For now, simply recognize that these familiar, already-used standard library methods are genuine generic methods, using exactly the `<T>` declaration syntax you just learned.

## A Generic Method Finding the Maximum Element

```java
public static <T extends Comparable<T>> T findMax(List<T> list) {   // BOUNDED type parameter --
    T max = list.get(0);                                                // preview of Topic 3
    for (T item : list) {
        if (item.compareTo(max) > 0) {
            max = item;
        }
    }
    return max;
}
```

```java
List<Integer> nums = List.of(5, 2, 8, 1);
Integer max = findMax(nums);   // 8

List<String> words = List.of("banana", "apple", "cherry");
String maxWord = findMax(words);   // "cherry" (alphabetically last)
```

**`<T extends Comparable<T>>`** is a **bounded** type parameter — it restricts `T` to only types that implement `Comparable<T>` (Module 10, Topic 7), which is precisely what's needed here, since `findMax` calls `.compareTo()` internally. Without this bound, the compiler couldn't verify `item.compareTo(max)` is even a legal call, since a completely unconstrained `T` might be any type at all, with no guarantee it has a `compareTo` method. **Full depth on bounded type parameters is Topic 3** — this is a deliberate, motivating preview.

## Real-World Analogy

Think of a generic method like a **universal power tool attachment** that works with any material you feed it — the attachment (the method) itself doesn't care whether you're cutting wood, plastic, or metal (the type argument); it applies the exact same mechanical process (the method's logic) regardless, and the tool automatically recognizes what material you've loaded (type inference) without you needing to manually configure it every time.

## Advantages

- Enables writing genuinely type-safe, reusable utility logic once, working correctly across any type, without duplication or unsafe casting.
- Type inference means callers almost never need to write out explicit type arguments — the syntax stays clean and natural in practice.
- A method can be generic independently of its enclosing class, offering maximum flexibility in API design.

## Disadvantages / Trade-offs

- The declaration syntax (`<T>` before the return type) is genuinely unfamiliar and slightly awkward on first encounter, compared to ordinary method declarations.
- Bounded type parameters (Topic 3) add real additional syntax complexity when a method needs to call specific methods on its type parameter (like `compareTo` above).

## Best Practices

- Prefer letting the compiler infer type arguments; only supply an explicit type witness when inference genuinely fails.
- Use bounded type parameters (`<T extends SomeInterface>`) whenever your generic method needs to call specific methods on values of type `T`.
- Recognize and read standard library generic method signatures confidently — they follow exactly the patterns taught in this topic.

## Common Mistakes

- Confusing a generic method's own `<T>` with a type parameter belonging to its enclosing generic class — they're independent, even when both exist and happen to share a name.
- Forgetting a type parameter needs a bound (`extends`) before calling type-specific methods (like `compareTo`) on it inside the method body.

## Interview Questions

1. **Q: Can a method be generic even if its enclosing class is not?**
   A: Yes — a method declares its own type parameter(s) with `<T>` immediately before its return type, entirely independent of whether the enclosing class itself has any type parameters.

2. **Q: What is type inference in the context of generic methods?**
   A: The compiler automatically determining a generic method's type argument(s) from the arguments actually passed at the call site (or, since Java 8, from the target/assignment context) — meaning callers rarely need to explicitly specify type arguments themselves.

3. **Q: Why does `findMax` need `<T extends Comparable<T>>` instead of just `<T>`?**
   A: Because its implementation calls `.compareTo()` on values of type `T` — an unconstrained `T` offers no guarantee such a method exists at all; bounding `T` to `Comparable<T>` gives the compiler the guarantee needed to verify that call is legal.

## Summary

- A method declares its own type parameter with `<T>` immediately before its return type, independent of its enclosing class's own generics (if any).
- Type inference lets the compiler determine type arguments automatically from context in the overwhelming majority of cases; explicit type witnesses are a rare fallback.
- Many familiar standard library methods (`Collections.sort`, `Arrays.asList`, `List.of`) are themselves generic methods, using exactly this declaration syntax.
- Bounded type parameters (previewed here, full depth Topic 3) are needed whenever a generic method calls type-specific methods on its type parameter.