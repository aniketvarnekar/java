# Module 07 Summary, Interview Questions & Exercises

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-Module-Overview.md) has been covered:

- [x] **The `Object` Class** — the complete method list every Java object inherits, `getClass()` tied to Module 02's class loading, a preview of `wait`/`notify`/`notifyAll`
- [x] **`toString()`** — the default's unhelpful format, correct overriding, and precisely when it's called implicitly (println, String concatenation)
- [x] **`equals()` and `hashCode()`** — the full formal contract (reflexive/symmetric/transitive/consistent/non-null), the `instanceof`-vs-`getClass()` symmetry pitfall, the equals/hashCode contract and exactly how violating it breaks `HashMap`/`HashSet`, the mutable-fields warning
- [x] **Object Cloning** — `Cloneable`/`clone()` mechanics and why the design is broken, shallow vs. deep copy with a full worked trace, copy constructors as the modern alternative
- [x] **Object Lifecycle & Garbage Collection** — reachability and GC Roots, unpredictable GC timing, why `finalize()` is deprecated, `AutoCloseable`/try-with-resources preview, weak references preview

## Practical Connections

- **Every entity class in a Spring/Hibernate application** needs correct `equals()`/`hashCode()` (Topic 3) — getting this wrong causes real, production-grade bugs when entities are stored in `Set`s or used as `Map` keys, a genuinely common real-world mistake this module specifically prepares you to avoid.
- **Logging frameworks** (Log4j, SLF4J) rely entirely on `toString()` (Topic 2) for meaningful object output in log statements — a well-written `toString()` is a direct, everyday productivity and debuggability win in real projects.
- **Try-with-resources** (Topic 5's preview) is the standard, idiomatic way every real Java codebase manages `Connection` (JDBC, Module 20), `InputStream`/`OutputStream` (Module 13), and countless other closeable resources — you'll use this constantly starting in Module 12.
- **Records** (Module 23) exist specifically to eliminate the equals/hashCode/toString boilerplate and correctness risk this module explored in such depth — understanding *why* it's hard to get right by hand is exactly what makes Records' automatic generation genuinely valuable, not just "less typing."

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| `Object`'s default `equals()` vs. an overridden one | Default is pure identity (`==`); an override typically compares logical field-based equality — but must satisfy the full formal contract. |
| Overriding `equals()` alone vs. overriding both `equals()`/`hashCode()` | Overriding only `equals()` leaves `Object`'s identity-based `hashCode()` in place, silently breaking `HashMap`/`HashSet` lookups for logically-equal objects. |
| `instanceof` vs. `getClass()` in `equals()` | `instanceof` can break symmetry across inheritance hierarchies; `getClass()` requires an exact type match, guaranteeing symmetry. |
| Shallow copy vs. deep copy | Shallow copies reference fields by address (shared mutable state); deep copies recursively copy referenced objects (full independence). |
| Unreachable vs. reclaimed | Becoming unreachable makes an object *eligible* for GC; actual reclamation happens later, at an unpredictable time. |
| `finalize()` vs. `AutoCloseable`/try-with-resources | `finalize()` has no timing guarantee and is deprecated; try-with-resources guarantees deterministic cleanup at a precise, predictable point in the code. |

## Consolidated Interview Questions (Module 07)

1. What methods does every Java object inherit from `Object`?
2. What does `getClass()` return, and how does it relate to Module 02?
3. What does `Object`'s default `toString()` produce, and when is `toString()` called implicitly?
4. What is the formal `equals()` contract (the five properties)?
5. Why is using `getClass()` generally safer than `instanceof` inside `equals()`?
6. What is the equals/hashCode contract, and what specifically breaks if you violate it?
7. Why should fields used in `equals()`/`hashCode()` ideally be `final`?
8. Why is Java's `Cloneable`/`clone()` design widely considered a mistake?
9. What's the difference between a shallow copy and a deep copy?
10. What is the modern, recommended alternative to `clone()`?
11. What does it mean for an object to be "reachable"? What are GC Roots?
12. Why is `finalize()` deprecated, and what replaced it?
13. Does `System.gc()` guarantee immediate garbage collection?

*(Full reasoning for every answer is in the respective topic file.)*

## Module Exercises

1. **Recall test:** From memory, list all five properties of the `equals()` contract, and explain in one sentence what "the equals/hashCode contract" (a distinct, sixth rule) requires.
2. **Hands-on:** Write a `Money` class (`long cents`, `String currency`) with correct `equals()`, `hashCode()`, and `toString()` implementations, then demonstrate it working correctly inside a `HashSet` (no duplicates for logically-equal `Money` instances).
3. **Hands-on:** Deliberately break the equals/hashCode contract (override `equals()` only), demonstrate the resulting `HashSet.contains()` failure experimentally, then fix it.
4. **Hands-on:** Implement a class with a mutable `List` field using a copy constructor (not `clone()`), demonstrating that mutating the copy's list doesn't affect the original's.
5. **Conceptual:** Explain, referencing GC Roots, why a local variable going out of scope at the end of a method can make a Heap object eligible for garbage collection.
6. **Synthesis:** Explain, end to end, why relying on `finalize()` to close a database connection promptly is unsafe, and write the correct modern replacement using try-with-resources syntax (based on this module's preview, even before Module 12's full depth).

## What's Next

Module 07 completed your understanding of the universal `Object` foundation every Java type builds on. **Module 08 — Strings** applies nearly everything from this module and Module 02 to Java's single most-used class: why `String` is immutable, the String Constant Pool (a direct extension of Topic 3's reference/equality discussion), `StringBuilder`/`StringBuffer` for efficient mutation, and the full String API.

---

**Previous:** [05 — Object Lifecycle & Garbage Collection](05-Object-Lifecycle-And-Garbage-Collection.md) · **Module Overview:** [00 — Module Overview](00-Module-Overview.md)

**Type "Continue" to begin Module 08 — Strings.**
