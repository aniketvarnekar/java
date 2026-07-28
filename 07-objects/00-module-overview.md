# Module 07 — Objects

## Module Goal

Module 05 (Topic 4) told you every Java class ultimately inherits from `java.lang.Object`. This module makes that fact fully concrete: what `Object` actually provides, why `equals()` and `hashCode()` have a strict contract binding them together (and what breaks if you violate it), why `clone()` is widely considered a design mistake, and — closing the loop that started all the way back in Module 02 — precisely how an object's life ends via garbage collection.

## Topics Covered in This Module

1. **[The `Object` Class](01-the-object-class.md)** — every method `Object` provides, and what each is for.
2. **[`toString()`](02-tostring.md)** — the default, unhelpful format, and why overriding it is close to mandatory in real code.
3. **[`equals()` and `hashCode()`](03-equals-and-hashcode.md)** — the precise contract between them, why breaking it silently corrupts hash-based collections, and how to implement both correctly.
4. **[Object Cloning](04-object-cloning.md)** — `Cloneable`, shallow vs. deep copying, and why most modern Java code avoids `clone()` entirely in favor of copy constructors.
5. **[Object Lifecycle & Garbage Collection](05-object-lifecycle-and-garbage-collection.md)** — from `new` to unreachability, why `finalize()` is deprecated, and a preview of `AutoCloseable`/try-with-resources as the modern alternative.
6. **[Module Summary](06-module-summary.md)** — consolidated recap.

## Prerequisites

- Module 05 (OOP), especially Topic 4 (Inheritance — `Object` as the root of every hierarchy).
- Module 06 (Classes), especially Topic 5 (Object Creation Order).
- Module 02 (JVM), especially Topic 3 (Heap) and Topic 4 (Execution Engine — Garbage Collector).

## How to Study This Module

Topic 3 (`equals`/`hashCode`) is the most important, most heavily interview-tested topic in this module — it directly sets up Module 10 (Collections), where `HashMap`/`HashSet` correctness depends entirely on getting this contract right. Topic 5 closes the object lifecycle loop opened in Module 02 and previews concepts (try-with-resources, `AutoCloseable`) that get full treatment in Module 12 (Exceptions) and Module 13 (IO).