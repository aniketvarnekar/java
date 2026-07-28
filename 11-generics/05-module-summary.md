# Module 11 Summary

## Topic Coverage Verification

Every topic promised in the [Module Overview](00-module-overview.md) has been covered:

- [x] **Why Generics — Introduction** — the concrete pre-generics problem (raw types, unsafe casting, runtime `ClassCastException`), writing generic classes/interfaces, naming conventions, raw types' continued legality for backward compatibility
- [x] **Generic Methods** — declaring a method's own independent type parameter, type inference, recognizing generic methods throughout the standard library, bounded type parameter preview
- [x] **Bounded Types & Wildcards** — full bounded type parameter syntax, why generics are deliberately invariant (contrasted with array covariance's real design wart), `? extends`/`? super`, and the PECS principle
- [x] **Type Erasure** — the complete erasure mechanism, why it exists (backward compatibility), and how it directly explains `new T[]`'s illegality, `instanceof List<String>`'s illegality, static-context restrictions, and bridge methods

## Practical Connections

- **Every Spring/Hibernate repository interface** (`JpaRepository<Entity, ID>`) is a generic interface, using exactly Topic 1's mechanics — you can now read and understand these declarations precisely rather than by pattern-matching alone.
- **Stream API method signatures** (Module 18, coming soon) make extremely heavy use of bounded types and wildcards — this module is essential preparation for genuinely understanding `Stream<T>`'s and `Collector`'s type signatures rather than just using them by imitation.
- **`Collections.sort`, `Collections.max/min`, and countless standard library utility methods** use exactly the PECS-following wildcard patterns from Topic 3 — you can now read their real signatures in the JDK source and understand precisely why they're written that way.
- **API design decisions** at real companies (should this method accept `List<T>` or `List<? extends T>`?) are informed directly by PECS — this is genuinely everyday, practical knowledge for designing clean, flexible Java APIs.

## Frequently Confused Concepts (Recap)

| Confused pair | The distinction |
|---|---|
| Generic class vs. generic method | A class's type parameter is fixed once, at construction (`new Box<String>()`); a method's own type parameter is inferred fresh at each individual call. |
| Raw type vs. generic type | Raw types (`List`) offer zero compile-time type safety, reverting to pre-2004 behavior; generic types (`List<String>`) are fully checked. |
| Array covariance vs. generics invariance | Arrays allow `Object[] o = new Integer[3];` (unsafe, checked only at runtime via `ArrayStoreException`); generics deliberately disallow the equivalent (`List<Object> l = new ArrayList<Integer>();`) at compile time. |
| `? extends T` vs `? super T` | Producer/read-only vs. Consumer/write-only — remembered via PECS. |
| Compile-time generics vs. runtime erasure | Generics provide full compile-time type safety, but that type information is completely erased from the actual runtime bytecode. |

## What's Next

Module 11 completed the "modern Java fundamentals" arc that began with Collections in Module 10 — you now fully understand the type-safety machinery underlying everything you'll build going forward. **Module 12 — Exceptions** shifts focus to Java's structured error-handling model: checked vs. unchecked exceptions, `try`/`catch`/`finally`, try-with-resources (finally delivering on Module 07's `AutoCloseable` preview in full), custom exceptions, and the exception hierarchy.