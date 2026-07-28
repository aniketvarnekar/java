# Module 09 Summary

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-module-overview.md) has been covered:

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

## What's Next

Module 09 gave you complete mastery of Java's foundational, fixed-size data structure — and, critically, explained exactly why `ArrayList` exists and how it works internally. **Module 10 — Collections** now takes this foundation and builds the full Java Collections Framework on top of it: `List`, `Set`, `Map`, and `Queue`, their major implementations (`ArrayList`, `LinkedList`, `HashSet`, `TreeSet`, `HashMap`, `TreeMap`, and more), and the algorithmic trade-offs between them — directly extending everything you now understand about arrays, `equals()`/`hashCode()` (Module 07), and generics (previewed in Module 05, full depth Module 11 shortly after).