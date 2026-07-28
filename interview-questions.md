# Interview Questions — Consolidated

All interview questions from every module, collected here for quick revision before an interview. Each entry links back to the module section that explains the full reasoning — don't just memorize the short answer, understand the "why" behind it.

---

## Module 01 — Introduction

**Q1. Why is Java called "platform independent"? Doesn't the OS matter at all?**
> Java source code compiles to bytecode, not native machine code. Bytecode is not tied to any OS or CPU architecture — it's tied to the *JVM specification*. Each OS has its own JVM implementation that knows how to translate that same bytecode into instructions its own hardware understands. So the OS absolutely matters — but it's the JVM's problem, not your `.class` file's problem. Your compiled code never changes; only the JVM underneath it changes.

**Q2. What is the difference between JDK, JRE, and JVM?**
> JVM executes bytecode. JRE = JVM + standard libraries (enough to *run* Java programs). JDK = JRE + development tools like `javac`, `javadoc`, debuggers (enough to *build* Java programs). A production server only needs a JRE (or increasingly, just a JDK since JRE-only distributions are rare post Java 11); a developer's machine needs a JDK.

**Q3. Is Java 100% "write once, run anywhere" in practice?**
> Mostly, but not perfectly. Nuances: OS-specific file path separators, native library dependencies (JNI), GUI look-and-feel differences, timezone/locale data updates, and OS-level resource differences can leak platform-specific behavior. The bytecode itself is portable — but a "Java program" can still call into non-portable OS features.

**Q4. Why does Java compile to an intermediate bytecode instead of directly to native machine code like C/C++?**
> Two reasons: (1) Portability — one compilation output works everywhere a JVM exists, instead of needing to recompile per OS/CPU. (2) It enables the JVM to apply runtime optimizations (JIT compilation) informed by *actual* execution behavior (which methods are hot, which branches are taken), something a purely static, ahead-of-time compiler cannot do as effectively. The cost is a startup/warm-up overhead compared to native-compiled languages.

**Q5. What is the entry point of a Java application and why must it be `static`?**
> `public static void main(String[] args)`. It must be `static` because the JVM calls it *before any object of your class exists* — there is nothing to invoke an instance method on yet. `static` means "belongs to the class itself, not to an instance," so the JVM can call it directly via the class, with zero objects created.

---

## Module 02 — JVM

**Q1. What are the major subsystems of the JVM?**
> The Class Loader Subsystem (finds/loads/links/initializes classes), the Runtime Data Areas (Method Area/Metaspace, Heap, JVM Stacks, PC Registers, Native Method Stacks — where everything lives in memory), the Execution Engine (Interpreter, JIT compiler(s), Garbage Collector), and the Native Method Interface (bridges to native code).

**Q2. What are the three phases of class loading?**
> Loading (locate and read bytecode, create the in-memory `Class` object), Linking (Verification, Preparation, Resolution), and Initialization (run static initializers, assign real static field values).

**Q3. When exactly is a Java class initialized?**
> Lazily — only upon "active use": instantiation, a static method/non-constant static field access, or initialization of a subclass. Merely declaring a variable of that type does not trigger it.

**Q4. What is the parent delegation model and why does it exist?**
> Each class loader asks its parent to load a requested class first, all the way up to the Bootstrap loader, before trying itself. This prevents application code from spoofing core classes like `java.lang.String`, since any request for a core class always resolves to the trusted Bootstrap-loaded version first.

**Q5. What's the difference between Stack and Heap?**
> The Stack (per-thread) holds local variables, parameters, and call frames — fast, private, destroyed when a method returns. The Heap (shared) holds all actual object data created with `new` — reclaimed only by the GC. Object-typed variables hold a *reference* to Heap data on the Stack; the object itself always lives on the Heap.

**Q6. What replaced PermGen, and why?**
> Metaspace, since Java 8. PermGen was a fixed-size region carved from the Heap, notorious for `OutOfMemoryError: PermGen space` under repeated class loading. Metaspace lives in native (off-heap) memory and grows dynamically by default.

**Q7. What causes a `StackOverflowError` vs an `OutOfMemoryError: Java heap space`?**
> `StackOverflowError`: a thread's per-thread JVM Stack exceeds its size, almost always from runaway recursion. `OutOfMemoryError: Java heap space`: the shared Heap can't allocate more memory because too many objects are still reachable (a memory leak in the "unintentionally retained reference" sense, since Java has no manual `free()`).

**Q8. What is tiered JIT compilation, and why does HotSpot use two compilers?**
> C1 (fast, light optimization) compiles warm methods quickly; C2 (slow, heavy optimization) is reserved for the genuinely hottest methods. This balances fast startup against excellent peak throughput, rather than sacrificing one for the other.

**Q9. What is JIT de-optimization?**
> The JVM discarding a speculatively compiled native method and reverting to interpretation when a runtime assumption it optimized for (e.g., "this branch is always taken") turns out to be false — a capability only possible because Java compiles based on live runtime behavior, not purely ahead of time.

**Q10. What is JNI, and what does it cost?**
> The Java Native Interface, letting Java call native C/C++ code via `native` methods. It costs platform independence (native libraries must be built per OS/architecture) and the JVM's usual memory safety at that boundary (native code can crash the whole process).

**Q11. Is "the JVM" a single piece of software?**
> No — it's a specification. HotSpot (the default for most JDK distributions), Eclipse OpenJ9 (optimized for fast startup/low memory), and GraalVM (alternative JIT + Native Image) are independent implementations with different internal engineering trade-offs.

**Q12. What is GraalVM Native Image, and what does it trade away?**
> Ahead-of-time compilation of a Java app into a standalone native executable, eliminating interpreter/JIT warm-up for near-instant startup. In exchange, it loses runtime-adaptive JIT optimization (no live profiling data informs its static compilation) and loses bytecode portability (must build per target platform, like C).

---

## Module 03 — Java Basics

**Q1. What does `var` actually do, and is Java still statically typed with it?**
> `var` (Java 10+) is local variable type inference — the compiler infers and permanently fixes the exact static type from the initializer, at compile time. Java remains fully statically typed; `var` is purely a source-readability feature with zero effect on the compiled bytecode.

**Q2. Why does Java have primitive types alongside its object-oriented design?**
> Performance and memory efficiency — primitives are stored directly (inline on the Stack or inline within an object's fields) with no Heap allocation, object header overhead, or reference indirection, unlike objects. A deliberate, pragmatic exception to full OO purity.

**Q3. Why does `long big = 3000000000;` fail without an `L` suffix?**
> Integer literals are parsed as `int` by default regardless of the assignment target; `3000000000` exceeds `int`'s range, so the literal itself fails to compile before assignment is even considered.

**Q4. What's the difference between widening and narrowing conversions?**
> Widening (small type → large type) is automatic and always safe. Narrowing (large type → small type) requires an explicit cast because it can lose information — magnitude, precision, or sign.

**Q5. Does Java throw on integer overflow?**
> No — it silently wraps around by default (e.g., `Integer.MAX_VALUE + 1` becomes `Integer.MIN_VALUE`). `Math.addExact()` and similar methods opt into `ArithmeticException` on overflow.

**Q6. What is short-circuit evaluation and why does it matter for null checks?**
> `&&`/`||` skip evaluating the right operand once the left alone determines the result — essential for the `obj != null && obj.getValue() > 0` guard pattern, which would otherwise throw `NullPointerException` when `obj` is `null`.

**Q7. Why does `byte b = 5; b += 10;` compile but `b = b + 10;` doesn't?**
> `b + 10` promotes to `int`, which requires an explicit cast to assign back to `byte`. `+=` implicitly includes that narrowing cast; plain `=` does not.

**Q8. Why does `Integer a = 100; Integer b = 100; a == b` print `true`, but `200` instead of `100` prints `false`?**
> `Integer.valueOf(int)` caches and reuses objects for -128 to 127; `100` hits the cache (same object, `==` true), `200` doesn't (two distinct objects, `==` false) — always use `.equals()` for wrapper value comparison.

**Q9. Why can unboxing throw `NullPointerException`?**
> Unboxing inserts an implicit method call (e.g., `.intValue()`) on the wrapper reference; if that reference is `null`, calling a method on it throws NPE — a real, silent risk in code that looks like ordinary primitive arithmetic.

**Q10. Does `final` make an object immutable?**
> No — `final` only prevents reassigning the variable's binding. It says nothing about whether the referenced object's own state can still be mutated via its methods; that depends entirely on the object's own class design.

---

## Module 04 — Control Flow

**Q1. Why does `if (someInt)` fail to compile in Java?**
> Java's `if` condition must be a genuine `boolean` expression — there's no implicit int-to-boolean ("truthy") conversion, a deliberate robustness decision eliminating a historical C-family bug class (accidental `=` instead of `==`).

**Q2. In a brace-less nested `if`/`else`, which `if` does the `else` bind to?**
> The nearest, most recently unmatched `if`, regardless of source indentation — this is why braces should always be used, even for single-statement bodies.

**Q3. What is switch fall-through, and is it ever intentional?**
> Without a `break`, execution continues into the next `case` block regardless of whether it matches. It's dangerous when accidental (a forgotten `break`), but legitimate when deliberately grouping multiple case values to share behavior (stacked empty cases).

**Q4. What does the modern `switch` expression (Java 14+) improve over the classic statement?**
> It directly produces a value, uses arrow syntax with no fall-through ever, supports comma-separated multi-value case labels, and provides compiler-enforced exhaustiveness checking for `enum`/`sealed` subjects.

**Q5. What's the fundamental difference between `while` and `do-while`?**
> `while` checks its condition before running the body (may run zero times); `do-while` checks after (always runs at least once).

**Q6. Is a `for` loop functionally different from a `while` loop?**
> No — any `for` loop can be mechanically rewritten as an equivalent `while` loop. The one genuine practical difference: a `for` loop's counter is scoped only to the loop itself, while an equivalent `while` loop's counter remains in scope afterward.

**Q7. When would you prefer an indexed `for` loop over an enhanced for-each loop?**
> When you need the index itself — to write back to a specific position, iterate backward, or process multiple collections in lockstep — for-each deliberately provides no index at all.

**Q8. By default, does `break` inside a nested loop affect the outer loop too?**
> No — `break` and `continue` affect only the innermost enclosing loop by default. Reaching an outer loop from within a nested inner one requires a labeled `break`/`continue`.

---

## Module 05 — OOP

**Q1. What is the difference between a class and an object?**
> A class is a blueprint defining structure/behavior, holding no actual data itself. An object is a specific instance created from that blueprint, with its own independent field values on the Heap.

**Q2. What is encapsulation, and why isn't a private field with a trivial getter/setter "real" encapsulation?**
> Encapsulation bundles data with the behavior operating on it and restricts direct access so the object can enforce its own invariants. A trivial pass-through getter/setter with no validation provides the mechanism (private + accessors) without the purpose (protecting invariants) — "fake encapsulation."

**Q3. What's the difference between abstraction and encapsulation?**
> Encapsulation controls *who* can access data (protection, via access modifiers). Abstraction controls *how much* a user needs to understand to use something (simplification, via a clean interface hiding complexity). Complementary, not synonyms.

**Q4. Why does Java restrict classes to single inheritance?**
> To avoid the Diamond Problem — genuine ambiguity about which superclass's implementation/state should apply when two parents provide conflicting definitions. A deliberate simplicity/safety trade-off over C++-style multiple inheritance's complex disambiguation rules.

**Q5. What's the difference between overloading and overriding?**
> Overloading: same class, different parameter lists, resolved at compile time (static binding). Overriding: subclass redefines an inherited method with an identical signature, resolved at runtime based on the object's actual type (dynamic binding).

**Q6. How does the JVM decide which overridden method to invoke at runtime?**
> It consults the object's actual runtime type's method table (not the variable's declared type) — dynamic method dispatch. The JIT compiler optimizes this via inline caching when a call site consistently resolves to the same implementation, de-optimizing if that assumption is later violated.

**Q7. What's the fundamental difference between an abstract class and an interface?**
> Abstract classes can hold real state/constructors and support only single inheritance. Interfaces are traditionally stateless (only constants) and support multiple implementation; Java 8+ `default` methods let interfaces carry limited implementation specifically for safe evolution.

**Q8. Why were default methods added to interfaces in Java 8?**
> To let existing interfaces (like `List`) gain new methods without breaking every class that already implemented them across the ecosystem — essential infrastructure for cleanly introducing Streams and Lambdas.

**Q9. What's the difference between aggregation and composition?**
> Aggregation: the "part" can exist independently of the "whole" and outlive it. Composition: the "part"'s lifecycle is entirely bound to the "whole," typically created/destroyed by it.

**Q10. What does "favor composition over inheritance" mean?**
> Default to composition for code reuse; reserve inheritance for genuine IS-A relationships where polymorphism is actually needed — not "never use inheritance."

---

## Module 06 — Classes

**Q1. Is Java pass-by-value or pass-by-reference?**
> Always pass-by-value. For objects, the copied value is the reference itself — enabling visible mutation of shared object state through that reference, but never letting a method reassign the caller's own variable.

**Q2. When does the compiler generate a default constructor?**
> Only when a class defines zero constructors of its own. Writing any constructor yourself stops the compiler from generating the default one.

**Q3. What is `this`, mechanically?**
> An implicit reference to the current object instance, passed as an invisible first parameter to every instance method — which is exactly why `static` methods (not invoked on any instance) have no `this` at all.

**Q4. Can static methods be overridden?**
> No — only hidden. A static method call is resolved at compile time by the reference's declared type, unlike instance methods, which use runtime dynamic dispatch based on the object's actual type.

**Q5. How does the Singleton pattern guarantee only one instance exists?**
> A `private static` field holds the single instance; a `private` constructor prevents outside code from calling `new` directly; a `public static getInstance()` method is the only path to a reference.

**Q6. What is the complete order of execution when a subclass object is created for the first time?**
> Class-level static initialization (superclass before subclass, once ever) → per `new`: superclass's entire object-level initialization (fields defaulted, `super()`, instance blocks/initializers, constructor body) fully completes → then the subclass's own object-level initialization runs.

**Q7. What's the difference between a static nested class and a non-static inner class?**
> A static nested class has no connection to any outer instance and can be created independently. A non-static inner class holds an implicit reference to one specific outer instance and requires one to exist (`outer.new Inner()`).

---

## Module 07 — Objects

**Q1. What does `Object`'s default `toString()` produce?**
> `fully.qualified.ClassName@hexadecimalHashCode` — identifies the type and a hash-derived identifier, but reveals nothing about the object's actual state.

**Q2. What is the formal `equals()` contract?**
> Reflexive (`x.equals(x)` always true), symmetric (`x.equals(y)` == `y.equals(x)`), transitive, consistent across repeated calls, and never true for `null`.

**Q3. Why is `getClass()` generally safer than `instanceof` inside `equals()`?**
> `instanceof` can silently break symmetry across inheritance hierarchies (a superclass might consider a subclass instance equal while the subclass's own `equals()` rejects the superclass instance). `getClass()` requires an exact type match in both directions, guaranteeing symmetry.

**Q4. What is the equals/hashCode contract, and what breaks if you violate it?**
> Objects equal per `equals()` must produce equal `hashCode()`s. Violating it (e.g., overriding `equals()` without `hashCode()`) sends logically-equal objects to different `HashMap`/`HashSet` buckets, causing lookups to silently fail even when a matching object is present.

**Q5. Why is Java's `Cloneable`/`clone()` design widely considered broken?**
> `Cloneable` is a methodless marker interface checked only at runtime (throwing a checked exception), `Object.clone()` is `protected` requiring awkward widening, the default copy is shallow (often unexpectedly), and it bypasses normal constructor-based object creation entirely.

**Q6. What's the difference between a shallow copy and a deep copy?**
> Shallow: reference fields copied by address, sharing the same underlying mutable object. Deep: referenced objects recursively copied too, giving full independence.

**Q7. What is the modern, recommended alternative to `clone()`?**
> A copy constructor (or static copy factory method) — explicit, uses normal constructor-based initialization, and makes shallow-vs-deep decisions visible per field.

**Q8. What does it mean for an object to be "reachable"?**
> There's a chain of references leading to it starting from a GC Root (active Stack locals/parameters, active static fields, or JNI references). Unreachable objects become eligible for garbage collection, at an unpredictable later time.

**Q9. Why is `finalize()` deprecated?**
> No guaranteed timing (might run late or never), real GC performance overhead, can "resurrect" objects, and silently swallows exceptions. `AutoCloseable`/try-with-resources replaced it for deterministic cleanup.

---

## Module 08 — Strings

**Q1. Is `String` immutable, and what benefits does that enable?**
> Yes — no method mutates it in place; every "modifying" operation returns a new object. This enables security (no post-validation mutation attacks), thread-safety (safe sharing with zero synchronization), safe pooling (the String Constant Pool), and a cacheable, precomputed `hashCode()`.

**Q2. Does `final String s = "Hello";` make the content immutable?**
> No — `final` only prevents reassigning the variable; content-level immutability is a separate, class-design-level guarantee present for every `String` regardless of `final`.

**Q3. Why does `"hello" == "hello"` evaluate to `true` but `new String("hello") == "hello"` evaluate to `false`?**
> Identical literals are automatically shared via the String Constant Pool, so both reference the same object. `new String(...)` explicitly creates a separate object outside the pool, regardless of an identical existing literal.

**Q4. Should you ever use `==` to compare String content?**
> No — always use `.equals()`. Even "safe-seeming" cases are fragile under refactoring or when a String originates from a non-literal source (user input, file/network data), which is never automatically pooled.

**Q5. Is `substring`'s ending index inclusive or exclusive?**
> Exclusive — `s.substring(begin, end)` includes indices `begin` through `end-1`; the result's length equals `end - begin`.

**Q6. Why is repeated String concatenation inside a loop inefficient?**
> Since `String` is immutable, each `+`/`+=` creates a brand-new object, discarding the previous one — many iterations create significant GC pressure from short-lived discarded objects. `StringBuilder`, used once across the loop, mutates one buffer in place instead.

**Q7. What's the difference between `StringBuilder` and `StringBuffer`?**
> Identical API; `StringBuffer`'s methods are `synchronized` (thread-safe, slower), `StringBuilder`'s are not (faster, the correct default for single-threaded use, which is the common case).

**Q8. How does a text block decide how much leading whitespace to strip?**
> The compiler finds the minimum common leading whitespace across every line (including the closing `"""`'s own line) and strips exactly that amount, preserving relative indentation while removing whitespace incidental to source formatting.

---

## Module 09 — Arrays

**Q1. Is an array a primitive or an object?**
> An object — even an array of primitives lives on the Heap and is accessed via a reference, exactly like any other object.

**Q2. Can an array's size change after creation?**
> No — permanently fixed at creation. "Resizing" always means creating a new array (e.g., via `Arrays.copyOf`).

**Q3. Does Java have a true, native 2D array type?**
> No — a "2D array" is an array of references to other 1D arrays, which is exactly what enables jagged (non-rectangular) arrays with zero special syntax.

**Q4. Why doesn't `println(array)` print its contents?**
> Arrays don't override `Object`'s default `toString()`, so `println` falls back to the unhelpful `ClassName@hashCode` format. Use `Arrays.toString`/`deepToString` instead.

**Q5. Why shouldn't you use `.equals()` to compare array contents?**
> Arrays don't override `equals()` either, so it falls back to identity comparison. Use `Arrays.equals()`/`deepEquals()` instead.

**Q6. Why does `Arrays.binarySearch` require pre-sorted input?**
> It works by repeatedly halving the search range assuming ordered elements — violating that assumption produces silently undefined, incorrect results with no warning.

**Q7. How does `ArrayList` achieve dynamic resizing if arrays can't resize?**
> Internally it maintains an ordinary backing array; when capacity is exceeded, it allocates a new, larger array and copies existing elements into it (conceptually identical to `Arrays.copyOf`), transparently to the caller.

**Q8. What does `Arrays.asList(arr).add(x)` do?**
> Throws `UnsupportedOperationException` — `Arrays.asList()` returns a fixed-size view directly backed by the original array, not a genuinely resizable list.

---

## Module 10 — Collections

**Q1. Why doesn't `Map` extend `Collection`?**
> `Collection` represents individual elements (`add(E)`); `Map` represents key-value pairs, whose natural `put(K,V)` operation doesn't fit that single-element contract. Kept separate, bridged via `keySet()`/`values()`/`entrySet()`.

**Q2. Why does `ArrayList` usually beat `LinkedList` in practice despite worse Big-O for some operations?**
> Locating an insertion point is O(n) for both in typical usage; `ArrayList`'s contiguous memory gives dramatically better real-world CPU cache locality than `LinkedList`'s scattered nodes.

**Q3. Why does `HashSet` require correct `equals()`/`hashCode()`?**
> `HashSet` is internally a `HashMap` wrapper — it depends entirely on the equals/hashCode contract (Module 07) for correct duplicate detection.

**Q4. Walk through `HashMap.put(key, value)` step by step.**
> Compute `hashCode()`, apply hash-spreading, compute bucket index via bitwise AND, then place directly (empty bucket) or walk the chain calling `equals()` to find/overwrite a match or append a new entry.

**Q5. What is HashMap's Java 8 treeification optimization?**
> When a bucket's collision chain exceeds 8 entries in a large table, it's converted from a linked list to a balanced Red-Black Tree, improving worst-case lookup from O(n) to O(log n).

**Q6. Does `TreeSet`/`TreeMap` use `equals()` for duplicate detection?**
> No — they use `compareTo`/`Comparator` (returning 0), which can disagree with `equals()` if the two aren't kept consistent.

**Q7. Why does modifying a list directly during a for-each loop throw `ConcurrentModificationException`?**
> The active iterator captured the collection's modification count at creation and checks it on every `next()`; a direct structural change outside the iterator's own `remove()` causes a detected mismatch, triggering an immediate, deliberate "fail-fast" exception.

**Q8. What's the difference between `Comparable` and `Comparator`?**
> `Comparable` is one built-in natural order defined on the class itself; `Comparator` is an external, pluggable ordering strategy supporting unlimited alternatives, including for types you don't own.

**Q9. Does `Collections.unmodifiableList` create an independent copy?**
> No — it's a restricted view; changes to the original collection remain visible through it. Use `List.of(...)`/`List.copyOf(...)` (Java 9+) for a genuinely independent immutable collection.

---

## Module 11 — Generics

**Q1. What problem did generics solve when introduced in Java 5?**
> Before generics, collections held plain `Object` references with no compile-time type safety — mismatches were only caught as `ClassCastException` at runtime, and every retrieval required a manual, unsafe cast. Generics let the compiler enforce type consistency and eliminate manual casts.

**Q2. Can a method be generic even if its enclosing class is not?**
> Yes — a method declares its own type parameter with `<T>` before its return type, entirely independent of the enclosing class's own generics.
**Q3. Why is `List<Integer>` not a subtype of `List<Object>`?**
> Generics are deliberately invariant, closing the unsafe-assignment loophole array covariance historically leaves open (which only fails at runtime via `ArrayStoreException`).

**Q4. What does PECS mean?**
> "Producer Extends, Consumer Super" — use `? extends T` for read-only parameters, `? super T` for write-only parameters.

**Q5. Why can't you call `.add(...)` on a `List<? extends Number>`?**
> The compiler doesn't know the list's exact underlying type parameter, so it conservatively forbids insertion entirely — only reading (guaranteed to be at least a `Number`) is safe.

**Q6. What is type erasure, and why did Java implement generics this way?**
> The compiler fully type-checks generic code, then removes all generic type information from the generated bytecode (unbounded `T` → `Object`, bounded `T` → its bound). It exists for backward compatibility with pre-2004 Java code and bytecode.

**Q7. Why can't you write `new T[10]` inside a generic class?**
> Arrays retain their element type at runtime, but after erasure there's no real type information left for `T` to give the array.

**Q8. What is a bridge method?**
> A compiler-generated synthetic method that repairs a signature mismatch erasure would otherwise create when a generic method is overridden with a more specific type, preserving correct polymorphic dispatch.

---

## Module 12 — Exceptions

**Q1. What's the difference between `Error` and `Exception`?**
> Both extend `Throwable`. `Error` represents serious, typically unrecoverable JVM/system-level problems that shouldn't be caught. `Exception` represents application-handleable conditions, split into checked and unchecked.

**Q2. What's the mechanical difference between checked and unchecked exceptions?**
> Checked extends `Exception` directly and requires compiler-enforced handling (`catch` or `throws`); unchecked extends `RuntimeException` and requires none.

**Q3. What are the main real-world criticisms of checked exceptions?**
> They encourage "swallow and ignore" anti-patterns, compose poorly with functional interfaces/lambdas, make API evolution a breaking change, and most other mainstream languages deliberately chose not to adopt the feature.

**Q4. Does `finally` run if `try` contains a `return`? What if `finally` also has a `return`?**
> `finally` always runs, even after a `try`-block `return`. But if `finally` itself contains a `return` (or `throw`), it silently overrides/discards whatever the `try`/`catch` was doing — including a propagating exception. Never put `return`/`throw` in `finally`.

**Q5. What is stack unwinding?**
> The JVM walking back up the call stack after a `throw`, popping frames with no matching `catch` until one is found or the stack is exhausted.

**Q6. In what order do multiple try-with-resources resources close?**
> Reverse of declaration order.

**Q7. What are suppressed exceptions?**
> When a resource's `close()` throws while a primary exception is already propagating in try-with-resources, the close-triggered exception is attached to the primary as "suppressed" (via `getSuppressed()`) instead of being discarded.

**Q8. What is exception chaining, and why does it matter?**
> Passing an original exception as the `cause` of a new one when translating across abstraction layers, preserving full diagnostic information rather than silently discarding it.

**Q9. Why is an empty `catch` block the most damaging exception anti-pattern?**
> It silently discards all information that a failure occurred, directly violating Java's fail-loudly robustness philosophy, often leaving the program in a subtly broken state discovered only much later.

---

## Module 13 — IO

**Q1. Why does Java maintain separate byte-stream and character-stream hierarchies?**
> Files/networks store raw bytes with no inherent "character" concept; converting to meaningful text requires an explicit encoding. Byte streams handle raw data with no encoding concept; character streams handle encoding-aware text.

**Q2. What's the biggest design improvement of `java.nio.file.Files` over `java.io.File`?**
> `Files` throws specific, meaningful checked exceptions on failure; `File` largely returns bare booleans with no explanation.

**Q3. Why is unbuffered, byte-at-a-time file I/O slow?**
> Each `read()`/`write()` call typically triggers an expensive OS-level system call regardless of data size — buffering batches many small operations into far fewer, larger ones.

**Q4. What does `InputStreamReader` do?**
> Bridges a byte stream into a character stream, decoding raw bytes into characters using an explicit `Charset` — usable for any byte source (files, sockets, console input).

**Q5. What is `transient`, and why use it?**
> A field modifier excluding a field from serialization (left at its default value on deserialization) — used for security-sensitive, non-meaningful, or non-serializable fields.

**Q6. Why should you always explicitly declare `serialVersionUID`?**
> An auto-computed one changes whenever the class structure changes even slightly, causing `InvalidClassException` when deserializing older data — explicit declaration gives deliberate control over compatibility.

**Q7. What is the security risk with Java's built-in serialization?**
> Deserializing untrusted data can be exploited to construct arbitrary objects and execute code during deserialization — a real, historically significant vulnerability class. Never deserialize untrusted data with it.

---

## Module 14 — NIO

**Q1. What does `Buffer.flip()` do, and why is it necessary?**
> Sets `limit = position` and resets `position = 0`, converting write-mode bookkeeping into read-mode bookkeeping — necessary because the buffer has no inherent way to know "writing is done, now read."

**Q2. What's the fundamental difference between a `Channel` and a `Stream`?**
> Channels are bidirectional and always operate through `Buffer` objects; streams are unidirectional and read/write bytes directly. Channels can also operate non-blocking.

**Q3. What is a memory-mapped file, and when does it help?**
> `FileChannel.map(...)` maps a file directly into the process's virtual memory, letting the OS lazily load pages on demand — best for large files with random/repeated access; overhead isn't worth it for small or purely sequential reads.

**Q4. What is the C10K problem?**
> The challenge of handling 10,000+ concurrent connections on one server — the thread-per-connection model requires one OS thread (with ~1MB stack) per connection, most idle most of the time, consuming enormous memory/scheduling overhead at scale.

**Q5. What does `Selector` do, and why is it needed even with non-blocking channels?**
> Lets one thread efficiently monitor many channels at once, blocking only until at least one is ready — without it, non-blocking channels alone would require wasteful busy-waiting.

**Q6. How do Virtual Threads change the practical need for hand-written NIO Selector code?**
> They let developers write simple, blocking-style code while the JVM handles scaling efficiently internally — reducing (but not eliminating the conceptual importance of) the need for manual event-loop code.

---

## Module 15 — Concurrency

**Q1. Why is `count++` not thread-safe?**
> It compiles to multiple bytecode instructions (read, add, write) — interleaving across threads can silently lose an update.

**Q2. Does `volatile` make compound operations thread-safe?**
> No — it guarantees visibility only, not atomicity; `count++` on a volatile field is still a race condition.

**Q3. What does `synchronized` do, and why must `wait()` be in a `while` loop?**
> It acquires an object's intrinsic lock, ensuring mutual exclusion and establishing happens-before on unlock/lock. `wait()` needs a `while` loop because by the time a woken thread re-acquires the lock, another thread may have already changed the condition again.

**Q4. How does `AtomicInteger` achieve thread safety without locking?**
> Via Compare-And-Swap (CAS), a hardware instruction that atomically checks-and-updates a value, retrying in a loop if another thread changed it first — no blocking involved.

**Q5. Why is creating a new Thread per task poor practice at scale?**
> Real per-thread memory/OS-scheduling overhead repeats Module 14's C10K problem; `ExecutorService` reuses a pool of threads across many tasks instead. 

**Q6. What limitation of `Future` does `CompletableFuture` solve?**
> `Future` can only be blocked on via `.get()`; `CompletableFuture` allows chaining dependent, non-blocking callbacks (`thenApply`/`thenCompose`/`thenCombine`).

**Q7. How does `ConcurrentHashMap` outperform `synchronizedMap`?**
> Fine-grained locking lets unrelated operations proceed concurrently instead of serializing behind one map-wide lock, plus it provides atomic compound operations (`putIfAbsent`, `computeIfAbsent`).

**Q8. What is a Virtual Thread, and how does it resolve the C10K problem?**
> A lightweight, JVM-managed thread not mapped one-to-one to an OS thread; many share a small pool of carrier threads, with the JVM transparently unmounting a blocked Virtual Thread to free its carrier. This lets simple blocking-style code scale to hundreds of thousands of concurrent tasks.

---

## Module 16 — JVM Internals

**Q1. What does `invokevirtual` do, and why is it significant?**
> Invokes an instance method using dynamic dispatch — the JVM consults the object's actual runtime class's method table, the concrete bytecode-level mechanism behind polymorphism.

**Q2. What is the generational hypothesis?**
> The empirical observation that most objects die young, motivating the split into a frequently-collected Young Generation and a less-frequently-collected Old Generation.

**Q3. Why can a reader thread see a stale value even after observing a "ready" flag, without synchronization?**
> The compiler/CPU can reorder independent operations absent an established happens-before relationship — a write to data and a write to a ready flag can become visible to another thread out of source-code order.

**Q4. What special guarantee do properly-initialized `final` fields receive?**
> Safe publication — any thread later obtaining the object's reference is guaranteed to see the final field's correctly-initialized value, with zero explicit synchronization needed.

**Q5. How does a framework like Jackson or Spring actually work internally?**
> Via Reflection — inspecting a class's fields/methods at runtime (looked up by name) and setting/invoking them dynamically, without compile-time knowledge of the specific class.

**Q6. Why do framework annotations need `RUNTIME` retention?**
> The framework must discover them via Reflection while the program is running; `SOURCE`/`CLASS` retention wouldn't be available at that point.

**Q7. How does Spring's `@Transactional` actually work?**
> Spring wraps the bean in a Dynamic Proxy; calls are intercepted, a transaction starts, the real method runs via Reflection, then the transaction commits or rolls back based on the outcome.

**Q8. What does `invokedynamic` enable, and how does it relate to lambdas?**
> It lets a call site's implementation be linked dynamically at runtime via a bootstrap method — Java 8 uses this to implement lambda expressions efficiently, without generating a separate compiled class per lambda.

---

## Module 17 — Functional Programming

**Q1. What is a functional interface, precisely?**
> An interface with exactly one abstract method — default/static/Object-inherited methods don't count.

**Q2. Why must local variables captured by a lambda be effectively final?**
> A lambda might outlive the method call that created it; it captures a value snapshot rather than a live reference, since the original Stack frame may already be destroyed by the time the lambda runs (Module 02). Allowing reassignment would create a confusing divergence between the snapshot and the "real" variable.

**Q3. What's the structural difference between a lambda and an equivalent anonymous class?**
> An anonymous class compiles to a separate `.class` file at compile time; a lambda uses `invokedynamic` (Module 16) to synthesize its implementation dynamically at runtime, making it generally lighter-weight.

**Q4. What are the four kinds of method references?**
> Static (`Class::staticMethod`), bound instance (`object::instanceMethod`), unbound instance (`Class::instanceMethod`), and constructor (`Class::new`).

**Q5. Why do primitive-specialized functional interfaces like `IntPredicate` exist?**
> Generic interfaces require object type parameters, so `Predicate<Integer>` autoboxes every `int` on every call — a real overhead in high-iteration code; primitive variants avoid this entirely.

**Q6. What's the difference between `andThen` and `compose`?**
> `f.andThen(g)` runs f first, then g. `f.compose(g)` runs g first, then f — reversed order.

**Q7. What problem does `Optional` solve?**
> It makes "this value might be absent" explicit in a method's return type, unlike a plain possibly-null `T` return type, which gives callers no signature-level warning.

**Q8. Why is calling `Optional.get()` without checking presence an anti-pattern?**
> It reproduces the exact "forgot to handle absence" bug Optional was designed to prevent, just as `NoSuchElementException` instead of `NullPointerException`.

---

## Module 18 — Streams

**Q1. Is a `Stream` a data structure?**
> No — it's a one-time-use pipeline describing operations over a data source; it stores no elements itself.

**Q2. Why don't intermediate operations execute immediately when called?**
> Streams are lazy — intermediate operations only build up a pipeline description; the whole pipeline executes only when a terminal operation is called, enabling optimizations like short-circuiting.

**Q3. What's the difference between `map` and `flatMap`?**
> `map` transforms each element 1-to-1. `flatMap` is used when the mapping function itself produces a stream per element, flattening those into one unified stream instead of a nested `Stream<Stream<T>>`.

**Q4. What are `reduce`'s two arguments?**
> An identity value (starting point, and result for an empty stream) and an accumulator function combining the running total with each next element. 

**Q5. Why does `Collectors.toMap` throw on duplicate keys by default?**
> A `Map` can't have duplicate keys, and `toMap` refuses to silently pick a winner — supply a merge function (third argument) to resolve conflicts explicitly.

**Q6. What's the difference between `groupingBy` and `partitioningBy`?**
> `groupingBy` partitions into any number of groups by a classifier function; `partitioningBy` always produces exactly two groups (`true`/`false`) based on a `Predicate`.

**Q7. How do parallel streams actually execute work internally?**
> Via the fork/join framework over the JVM's shared common `ForkJoinPool` — data is recursively split, processed independently, and recombined.

**Q8. Why might a parallel stream be slower than a sequential one?**
> Fork/join coordination overhead can exceed the parallelism benefit for small datasets/cheap work, and contention on the shared common pool (also used by `CompletableFuture`) can cause unexpected slowdowns elsewhere.

---

## Module 19 — Networking

**Q1. How do you read and write data over a `Socket`?**
> Via ordinary `InputStream`/`OutputStream`, typically bridged to character streams via `InputStreamReader`/`PrintWriter` — exactly Module 13's standard I/O API, applied to a network connection.

**Q2. Why can't a single-threaded server serve multiple clients concurrently?**
> The main thread is fully occupied handling one client (including blocking reads) before it can `accept()` again; any other client must wait.

**Q3. How does swapping to `Executors.newVirtualThreadPerTaskExecutor()` change a server's scalability?**
> It resolves the C10K problem for that server — Virtual Threads transparently unmount from their carrier thread whenever blocking I/O is waiting, letting a small number of carriers serve huge numbers of concurrent connections, with the handler code itself unchanged.

**Q4. What does `HttpClient.sendAsync(...)` return?**
> A `CompletableFuture<HttpResponse<T>>` — a deliberate design choice letting Module 15's `CompletableFuture` chaining apply directly to async HTTP responses.

---

## Module 20 — JDBC

**Q1. How does JDBC achieve database-vendor neutrality?**
> Application code is written against standard JDBC interfaces; vendor-specific drivers implement them, translating standard calls into each database's actual protocol.

**Q2. What is SQL injection, and how does it work?**
> An attack where user input concatenated into a SQL query is interpreted as additional SQL syntax rather than pure data, letting an attacker manipulate query logic or execute destructive statements.

**Q3. Why does `PreparedStatement` prevent SQL injection?**
> It sends query structure to the database separately from, and before, parameter values — the database fixes the structure first, so values are always treated as pure data, never as SQL syntax, regardless of their content.

**Q4. What do `commit()` and `rollback()` do?**
> `commit()` makes every statement since the last commit permanent together, atomically. `rollback()` undoes them all, as if none had executed — typically called on exception, mirroring Module 12's exception-handling discipline.

**Q5. Why does connection pooling exist?**
> Establishing a `Connection` is genuinely expensive (network/authentication overhead) — a pool establishes connections once and reuses them, directly paralleling Module 15's thread-pool reasoning.

---

## Module 21 — Modules (JPMS)

**Q1. What problem does "JAR hell" refer to?**
> Different libraries depending on conflicting versions of the same shared dependency, with the flat, unordered classpath offering no built-in conflict detection — whichever version is found first (unpredictably) is used for everyone.

**Q2. What does `exports` control, and how does it differ from `public`?**
> It controls whether a package is visible to code in *other modules* at all — a class can be `public` yet completely inaccessible externally if its package isn't exported, a stronger, module-level encapsulation layer beyond `public`/`private`/`protected`.

**Q3. Why does `opens` exist separately from `exports`?**
> Ordinary compile-time access and deep Reflective access (`setAccessible(true)`-style) are different capabilities with different risk profiles; `opens` grants the latter deliberately and independently.

**Q4. What is an automatic module?**
> A non-modularized JAR placed on the module path, automatically treated as a module with an inferred name and all packages implicitly exported — a bridge enabling gradual JPMS migration.

**Q5. What does `jlink` produce, and why does JPMS make it possible?**
> A minimal, custom Java runtime image containing only the modules an application actually needs, based on its declared `requires` dependencies — reliable only because JPMS makes true dependencies explicit and verifiable.

---

## Module 22 — Performance

**Q1. Why might you set `-Xms` equal to `-Xmx`?**
> To pre-allocate the full heap upfront, avoiding dynamic resizing overhead and improving consistency — common for latency-sensitive production services.

**Q2. Why is a naive `System.nanoTime()` loop benchmark usually misleading?**
> It blends slow, interpreted "warm-up" execution with fast, JIT-compiled steady-state execution into one uninterpretable number, and risks dead-code elimination silently invalidating the measurement.

**Q3. What does JMH do differently from a naive benchmark?**
> Runs explicit warm-up iterations before measuring, measures only after warm-up, prevents dead-code elimination, and applies real statistical rigor across multiple runs.

**Q4. What is escape analysis?**
> A JIT optimization that, when it can prove an object never "escapes" the method that creates it, may allocate it on the Stack or eliminate the allocation entirely, transparently — refining, not contradicting, the Heap-allocation model.

**Q5. When would you choose GraalVM Native Image over a standard JVM deployment?**
> When startup latency is the dominant, measured concern (serverless functions, short-lived CLI tools) — deliberately trading away runtime-adaptive JIT optimization and bytecode portability for near-instant startup.

---

## Module 23 — Modern Java

**Q1. What specific problem do records solve, and which Java version finalized them?**
> They eliminate the boilerplate of hand-writing a constructor, accessors, `equals()`, `hashCode()`, and `toString()` for simple immutable data-holder classes — finalized in Java 16.

**Q2. Why can't a record extend another class?**
> Every record implicitly extends `java.lang.Record`, and Java has no multiple inheritance of state — a record can still implement any number of interfaces.

**Q3. What must every direct permitted subtype of a sealed type declare, and why?**
> Exactly one of `final`, `sealed`, or `non-sealed`, so the entire hierarchy's openness/closedness is fully explicit from its declarations alone.

**Q4. How do sealed classes and `switch` pattern matching combine to provide a guarantee classic Java never had?**
> Because a sealed type's complete set of permitted subtypes is known to the compiler, a `switch` over it can be checked for exhaustiveness at compile time — a missing case is a compile error, not a silent runtime gap.

**Q5. What is a record pattern?**
> A pattern like `case Point(int x, int y) ->` that simultaneously checks the type AND destructures its components in one step.

**Q6. Why is `ScopedValue` considered a better fit than `ThreadLocal` for Virtual-Thread-heavy code?**
> It is immutable for the duration of a single binding scope and automatically, deterministically unbound when that scope ends, eliminating both the mutation risk and the memory-leak risk `ThreadLocal`'s manual lifecycle carries.

**Q7. What happened to Java's String Templates feature?**
> Previewed in Java 21-22, then withdrawn from preview in Java 23 for redesign — not part of Java 25.

---

## Module 24 — Interview Preparation

**Q1. Why is `volatile` required in double-checked-locking singleton implementations?**
> Object construction isn't atomic; without `volatile`'s JMM ordering guarantee, another thread could observe a non-null but only partially-constructed instance.

**Q2. Why must `wait()` be called inside a `while` loop, not an `if`?**
> `wait()` can return due to a spurious wakeup without an actual `notify()`, so the condition must be re-checked after waking, not assumed true.

**Q3. Why doesn't marking every field `final` guarantee genuine immutability?**
> `final` only prevents reassigning a reference; if a field's type is itself mutable, defensive copies are still required on both construction and access.

**Q4. Why should you state assumptions before answering an open-ended system-design-adjacent question?**
> These questions are evaluated on structured reasoning under ambiguity, not one correct answer — skipping assumptions signals jumping to a solution without understanding the actual problem.

**Q5. What's the correct first step when diagnosing high GC pause times in production?**
> Measure first (JFR/GC logs) — never tune speculatively.

**Q6. Why should a technical answer lead with a direct answer before explaining the mechanism?**
> An interviewer's first impression often forms from the opening sentence; hedging before answering reads as uncertainty even when a correct answer follows.
