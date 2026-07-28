# `Comparable` & `Comparator`

## Learning Objectives

- Implement `Comparable` correctly for natural ordering
- Write `Comparator`s for custom/alternative ordering, including modern lambda-friendly chaining
- Understand precisely when each is appropriate, and how they combine
- Understand the "consistent with equals" recommendation

## Prerequisites

Module 07 Topic 3 (`equals()`), [03 — `Set`](03-set-interface-and-implementations.md) and [04 — `Map`](04-map-interface-and-implementations.md) (`TreeSet`/`TreeMap` depend on this topic)

## Motivation

Topics 3–5 repeatedly referenced "requires `Comparable` or a `Comparator`" for `TreeSet`, `TreeMap`, and `PriorityQueue` — this topic delivers on that promise fully. Sorting is also one of the most common everyday operations on any collection, making this genuinely essential, practical knowledge.

## The `Comparable` Interface — An Object's "Natural" Ordering

```java
public class Employee implements Comparable<Employee> {
    String name;
    double salary;

    @Override
    public int compareTo(Employee other) {
        return Double.compare(this.salary, other.salary);   // natural order: BY SALARY, ascending
    }
}
```

`Comparable<T>` declares **one** method, `compareTo(T other)`, returning:
- **Negative** if `this` should sort **before** `other`
- **Zero** if they're considered **equal** for ordering purposes
- **Positive** if `this` should sort **after** `other`

**This is exactly Module 08, Topic 3's `String.compareTo` convention**, generalized to any type — `String` itself implements `Comparable<String>`, which is precisely why `TreeSet<String>` (Topic 3) worked without any extra setup.

```java
List<Employee> employees = new ArrayList<>(List.of(/* ... */));
Collections.sort(employees);       // uses Employee's compareTo() automatically -- "natural order"
// or, equivalently:
employees.sort(null);                // List.sort(null) also uses natural (Comparable) order
```

**Why "natural" ordering?** It represents the **single, canonical, most obviously default way** to order objects of this type — for `Employee`, perhaps by salary; for a `Product`, perhaps by price; for `String`, alphabetically. A class should implement `Comparable` only when there's a genuinely obvious, singular default ordering — not every class has one, and `Comparable` should never be implemented arbitrarily just to "enable sorting" if no truly natural order exists.

## The `Comparator` Interface — Custom, Alternative, or Multiple Orderings

**What if you need to sort `Employee`s by `name` sometimes, and by `salary` other times?** `Employee` can only have **one** `compareTo()` (its single natural order) — `Comparator<T>` provides an entirely separate, **external, pluggable** ordering strategy:

```java
Comparator<Employee> byName = new Comparator<Employee>() {
    @Override
    public int compare(Employee a, Employee b) {
        return a.name.compareTo(b.name);
    }
};

employees.sort(byName);   // sorts by NAME, regardless of Employee's own natural (salary-based) order
```

**This is precisely Module 06, Topic 6's anonymous class pattern** — and, being a **functional interface** (exactly one abstract method, `compare`), `Comparator` is one of the most common places you'll see **lambda expressions** used in modern Java (full depth Module 17; usable now by pattern recognition):

```java
Comparator<Employee> byName = (a, b) -> a.name.compareTo(b.name);   // lambda -- MUCH more concise
```

## Modern `Comparator` Construction — Static/Default Factory Methods (Java 8+)

```java
Comparator<Employee> byName = Comparator.comparing(e -> e.name);
Comparator<Employee> bySalaryDesc = Comparator.comparing((Employee e) -> e.salary).reversed();

// CHAINING -- sort by department first, THEN by salary (as a tiebreaker) -- extremely common, real need:
Comparator<Employee> combined = Comparator.comparing((Employee e) -> e.department)
                                            .thenComparing(e -> e.salary);

employees.sort(combined);
```

**`Comparator.comparing(...)`, `.reversed()`, and `.thenComparing(...)` are modern (Java 8+) additions specifically designed to make constructing rich, multi-level sort orders concise and highly readable** — replacing what used to require verbose, manually-written `compare()` method bodies with nested if/else tie-breaking logic. This is Module 05, Topic 6's `default`/`static` interface method story (the reason those were added to Java) directly delivering real, everyday ergonomic value here.

## `Comparable` vs. `Comparator` — The Full Comparison

| | `Comparable<T>` | `Comparator<T>` |
|---|---|---|
| Defined | **Inside** the class being compared (`class Employee implements Comparable<Employee>`) | **Externally**, separate from the class |
| Method | `compareTo(T other)` | `compare(T a, T b)` |
| How many orderings? | **Exactly one** — the class's single "natural" order | **Unlimited** — define as many different `Comparator`s as needed |
| Used automatically by | `Collections.sort(list)`, `TreeSet`, `TreeMap` (when no `Comparator` supplied) | Explicitly passed: `list.sort(comparator)`, `new TreeSet<>(comparator)` |
| Appropriate when | The class has one genuinely obvious default order | You need alternative, multiple, or externally-defined orderings — including for classes you don't own/can't modify |

**A genuinely important, related use case**: `Comparator` lets you sort/order objects from a class **you don't control** (e.g., a standard library class, or a class from a third-party dependency) — since you can't add `implements Comparable` to code you don't own, `Comparator` is the **only** option for imposing an ordering on such types.

## "Consistent with `equals`" — A Recommendation, Not a Hard Requirement

Recall Topic 3's warning: `TreeSet`/`TreeMap` use `compareTo`/`Comparator` (returning `0`) for duplicate detection, **not** `equals()`/`hashCode()`. The `Comparable` documentation **strongly recommends** — though doesn't strictly, compiler-enforce — that `compareTo` be **"consistent with equals"**: `x.compareTo(y) == 0` should ideally hold **if and only if** `x.equals(y)` is `true`.

**Why does this matter practically?** If they're inconsistent, a `TreeSet` and a `HashSet` containing the exact same objects can report **different** sizes/contents for what a developer would reasonably expect to be "the same set of unique items" — a genuinely confusing, real inconsistency if not deliberately intended. **Sometimes inconsistency is deliberate and fine** (e.g., a `Comparator` sorting `Employee`s by `salary` alone, where two different employees might coincidentally share a salary — `compareTo` returns `0`, but `equals()` correctly says they're different people) — the key is being **aware** of the distinction and its consequences, not necessarily avoiding it entirely.

## Real-World Analogy

Think of `Comparable` like a **person's own, single, self-declared "default sort key"** printed directly on their employee badge (say, employee ID number) — built into the badge itself, one canonical way to order badges. Think of `Comparator` like **a separate sorting instruction sheet you bring with you** — "sort these badges by last name," "sort these badges by hire date" — completely independent of whatever's printed on the badges themselves, and you can bring as many different instruction sheets as you have different sorting needs, even for badges you didn't design or print yourself.

## Advantages

- `Comparable` provides a convenient, built-in default order directly on the type itself, requiring zero extra objects to use `Collections.sort`/`TreeSet`/`TreeMap` out of the box.
- `Comparator` supports unlimited alternative orderings, external definition (for types you don't own), and modern, highly composable chaining (`comparing`, `reversed`, `thenComparing`).

## Disadvantages / Trade-offs

- A class can have only **one** `compareTo()`-defined natural order — genuinely limiting when multiple, equally valid orderings are needed (directly motivating `Comparator`'s existence).
- Inconsistency between `compareTo`/`Comparator` and `equals()` can produce genuinely confusing behavior across different collection types if not deliberately understood.

## Best Practices

- Implement `Comparable` only when a class has one truly obvious, canonical natural order; use `Comparator` for everything else, including alternative/multiple orderings.
- Prefer modern `Comparator.comparing(...)`/`thenComparing(...)`/`reversed()` chaining over manually-written `compare()` method bodies.
- Be deliberate and aware when `compareTo`/`Comparator` logic isn't fully consistent with `equals()` — understand the specific consequence for whichever collection types you're using.

## Common Mistakes

- Implementing `Comparable` for a class with no genuinely obvious natural order, forcing an arbitrary, potentially confusing choice.
- Forgetting `Comparator` is required (not `Comparable`) for sorting/ordering a class you don't own or can't modify.
- Assuming `TreeSet`/`TreeMap` use `equals()` for duplicate detection when a custom `Comparator` is supplied — they always use the comparison logic (natural or supplied), never `equals()` directly.

## Interview Questions

1. **Q: What's the difference between `Comparable` and `Comparator`?**
   A: `Comparable` is implemented by the class itself, defining exactly one "natural" ordering via `compareTo(T other)`. `Comparator` is a separate, external object defining a `compare(T a, T b)` ordering strategy — you can define any number of different `Comparator`s, including for classes you don't own, unlike `Comparable`'s single, built-in order.

2. **Q: When would you use a `Comparator` instead of implementing `Comparable`?**
   A: When you need multiple or alternative orderings for the same type (e.g., sort by name sometimes, by salary other times), or when you need to order a type you don't control and can't add `implements Comparable` to.

3. **Q: What does "consistent with equals" mean for `compareTo`, and why does it matter?**
   A: It means `x.compareTo(y) == 0` should ideally hold if and only if `x.equals(y)` is `true`. It matters because `TreeSet`/`TreeMap` use `compareTo` (not `equals()`) for duplicate detection — inconsistency can cause a `TreeSet` and a `HashSet` of the "same" objects to disagree on size/contents, which should be a deliberate, understood choice, not an accident.

## Summary

- **`Comparable<T>`**: implemented by the class itself, exactly one `compareTo()`-defined "natural" order, used automatically by `Collections.sort`, `TreeSet`, `TreeMap` when no `Comparator` is supplied.
- **`Comparator<T>`**: an external, pluggable ordering strategy, supporting unlimited alternative orderings and usable on types you don't own; modern `Comparator.comparing`/`.reversed`/`.thenComparing` chaining makes rich, multi-level sorts concise.
- `TreeSet`/`TreeMap` use comparison logic (not `equals()`/`hashCode()`) for duplicate detection — "consistent with equals" is a strong recommendation, not a compiler-enforced rule.