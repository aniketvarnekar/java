# Module 09 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-Module-Overview.md) has been covered:

- [x] **Array Fundamentals** — arrays as Heap-allocated objects, the reference model applied concretely, fixed size, `.length` field, default values, runtime bounds checking (`ArrayIndexOutOfBoundsException`)
- [x] **Multidimensional Arrays** — the "array of arrays" model, rectangular vs. jagged arrays, correct traversal bounds
- [x] **The `Arrays` Utility Class** — why `println`/`equals` don't work as expected on arrays, `sort`, `binarySearch` (and its precondition), `fill`, `equals`/`deepEquals`, `copyOf`/`copyOfRange`
- [x] **Array vs. `ArrayList`** — the complete comparison, exactly how `ArrayList` implements dynamic resizing internally (using the exact mechanism from Topic 3), `Arrays.asList()`'s fixed-size-view quirk, and genuine guidance on when each is appropriate

## Practical Connections

- **Every `ArrayList`/`HashMap`/collection you'll use starting in Module 10** is built, at some level, on the array fundamentals from this module — understanding raw arrays first is precisely what makes Collections Framework internals transparent rather than magical.
- **Image processing, scientific computing, and game development** rely heavily on raw primitive arrays (and multidimensional arrays specifically) for performance — the `int[]`-vs-`ArrayList<Integer>` memory/performance distinction from Topic 4 is a real, everyday consideration in these domains.
- **`main(String[] args)`** — the very first Java syntax you learned in Module 01 — is itself an array, and everything in this module (bounds checking, default behavior, `.length`) applies directly to it.
- **JSON/CSV parsing** frequently produces jagged, dynamically-sized data — Topic 2's jagged array understanding, combined with Topic 4's array-vs-List guidance, directly informs how you'd model such data in real parsing code.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| `array.length` vs `list.size()` vs `string.length()` | Field, method, and method respectively — a small but consistently confusing set of API inconsistencies worth memorizing deliberately. |
| `new Person[5]` vs. 5 actual `Person` objects | Creates 5 `null` slots; each must be individually assigned an actual object before use. |
| Rectangular vs. jagged arrays | Rectangular: all rows same length. Jagged: rows independently sized, since each is its own array object. |
| `array.equals()` vs `Arrays.equals()` | The former is unhelpful identity comparison (arrays don't override `equals()`); the latter correctly compares content. |
| `Arrays.asList()` vs `new ArrayList<>(...)` | The former is a fixed-size view backed by the original array (no add/remove); the latter is a genuinely independent, fully resizable list. |

## Consolidated Interview Questions (Module 09)

1. Is an array a primitive or an object in Java?
2. Can an array's size change after creation? What must you do instead if you need "more room"?
3. What happens when you create `new Person[5]` — does it create 5 usable objects?
4. What exception is thrown for an out-of-bounds array access, and why does Java guarantee this check happens?
5. Does Java have a true, native 2D array type?
6. What is a jagged array, and why is it possible in Java without special syntax?
7. Why doesn't `System.out.println(someArray)` print the array's contents?
8. Why shouldn't you use `.equals()` to compare two arrays' contents?
9. Why does `Arrays.binarySearch` require its input to be pre-sorted?
10. Why can't arrays grow or shrink, while `ArrayList` can — and how does `ArrayList` actually achieve this internally?
11. What does `Arrays.asList(someArray).add(x)` do, and why?
12. When would you genuinely still prefer a raw array over `ArrayList` in modern application code?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** From memory, draw the Stack/Heap memory diagram for an `int[]` array with a few assigned values, labeling the reference and the Heap-allocated object precisely.
2. **Hands-on:** Write a method that takes a jagged `int[][]` and returns the sum of all elements, using correctly-bounded nested loops (`array[row].length`, not a fixed value).
3. **Hands-on:** Demonstrate `Arrays.asList()`'s fixed-size quirk experimentally — create a list from an array, attempt `.add()` (catching/observing the exception), then successfully use `.set()` and confirm the original array was mutated too.
4. **Conceptual:** Explain, step by step, exactly what happens internally when an `ArrayList` with backing-array capacity 4 receives a 5th `add()` call — referencing `Arrays.copyOf`'s mechanism directly.
5. **Conceptual:** A colleague is processing a 10-million-element dataset of raw `int` values and considers using `ArrayList<Integer>`. Explain, referencing autoboxing (Module 03, Topic 6), why a plain `int[]` would likely be the better choice here.
6. **Synthesis:** Write a method that takes two `int[]` arrays, correctly checks whether they contain the same elements (using the appropriate `Arrays` method, not `==`/`.equals()`), and if so, returns a new, independently-copied array combining both (hint: `Arrays.copyOf` combined with manual index management, or research `System.arraycopy`).

## What's Next

Module 09 gave you complete mastery of Java's foundational, fixed-size data structure — and, critically, explained exactly why `ArrayList` exists and how it works internally. **Module 10 — Collections** now takes this foundation and builds the full Java Collections Framework on top of it: `List`, `Set`, `Map`, and `Queue`, their major implementations (`ArrayList`, `LinkedList`, `HashSet`, `TreeSet`, `HashMap`, `TreeMap`, and more), and the algorithmic trade-offs between them — directly extending everything you now understand about arrays, `equals()`/`hashCode()` (Module 07), and generics (previewed in Module 05, full depth Module 11 shortly after).

---

**Previous:** [04 — Array vs. `ArrayList`](04-Array-vs-ArrayList.md) · **Module Overview:** [00 — Module Overview](00-Module-Overview.md)

**Type "Continue" to begin Module 10 — Collections.**
