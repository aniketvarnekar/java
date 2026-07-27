# Glossary

Every technical term used across this course, defined in plain language the moment it matters. Organized by the module that introduces it. Use Ctrl+F / Cmd+F to jump to a term.

> This file grows with every module. Terms are never removed, only added.

---

## Module 01 — Introduction

| Term | Definition |
|---|---|
| **Java** | A general-purpose, class-based, object-oriented programming language designed to have as few implementation dependencies as possible, so compiled code can run on any machine that has a Java Virtual Machine (JVM). |
| **JVM (Java Virtual Machine)** | A virtual (software-based) computer that executes Java bytecode. It is what makes Java "write once, run anywhere" — the JVM, not your code, is what changes per operating system. |
| **JRE (Java Runtime Environment)** | The JVM plus the standard Java class libraries needed to *run* Java programs. It does not include development tools like the compiler. |
| **JDK (Java Development Kit)** | The JRE plus development tools like the compiler (`javac`), debugger, and other utilities needed to *write and build* Java programs. |
| **Bytecode** | The intermediate, platform-independent instruction set that the Java compiler (`javac`) produces. It is not machine code (not directly understood by hardware) and not source code (not human-oriented) — it's a middle format the JVM interprets or compiles further. |
| **Compiler (`javac`)** | The tool that translates human-readable `.java` source files into `.class` files containing bytecode. |
| **Platform Independence** | The property that compiled Java code (bytecode) runs unmodified on any operating system that has a compatible JVM. The trade-off: you don't compile to native machine code directly — the JVM does that translation at run time. |
| **WORA (Write Once, Run Anywhere)** | Java's foundational marketing principle and design goal: a `.class` file compiled on one OS runs unmodified on any other OS with a JVM. |
| **Class Loader** | The JVM subsystem responsible for finding, loading, and initializing `.class` files into memory at runtime. |
| **`main` method** | The designated entry point of a standalone Java application; the JVM looks for `public static void main(String[] args)` (or the Java 21+ unnamed-class equivalent) to begin execution. |
| **Interpreter** | A JVM component that reads bytecode instructions one at a time and executes them directly, without producing native machine code first. |
| **JIT (Just-In-Time) Compiler** | A JVM component that compiles frequently executed bytecode into native machine code at run time, to make repeated execution faster than pure interpretation. |

---

## Module 02 — JVM

| Term | Definition |
|---|---|
| **Class Loader Subsystem** | The JVM component that finds, loads, links, and initializes `.class` files, turning them into in-memory `Class` objects the rest of the JVM can use. |
| **Loading** | The first class-loading phase: locating and reading a class's bytecode, and creating its in-memory `Class` object. |
| **Linking** | The second class-loading phase, itself split into Verification (safety checks), Preparation (default values for static fields), and Resolution (turning symbolic references into direct ones). |
| **Initialization** | The third class-loading phase: running static initializer blocks and assigning real values to static fields. Triggered lazily, by "active use," not by mere reference. |
| **Bootstrap ClassLoader** | The topmost class loader, written in native code, responsible for loading core JDK classes (`java.lang.*`, etc.). |
| **Platform ClassLoader** | Loads other JDK platform modules (e.g., `java.sql`); called the "Extension ClassLoader" before Java 9. |
| **Application (System) ClassLoader** | Loads your own application's classes from the classpath/modulepath. |
| **Parent Delegation Model** | The rule that a class loader always asks its parent to load a class first, only attempting to load it itself if every ancestor fails — primarily exists to prevent core classes like `java.lang.String` from being spoofed by application code. |
| **Method Area / Metaspace** | The shared JVM memory region holding class metadata, static fields, and the runtime constant pool. Since Java 8, implemented as native (off-heap) memory called Metaspace, replacing the older, leak-prone PermGen. |
| **PermGen (Permanent Generation)** | The pre-Java-8 implementation of the Method Area, carved out of the Heap with a fixed size; notorious for `OutOfMemoryError: PermGen space`. Replaced by Metaspace. |
| **Heap** | The shared JVM memory region where every object created with `new` (and its instance fields) actually lives; reclaimed by the Garbage Collector. |
| **JVM Stack** | A per-thread memory region holding a stack of frames, one per active (not-yet-returned) method call, storing local variables, parameters, and the operand stack. |
| **Stack Frame** | One entry in a JVM Stack, representing one active method call's local variables and intermediate calculation state. |
| **PC (Program Counter) Register** | A small, per-thread piece of data tracking which bytecode instruction the thread is currently executing. |
| **Native Method Stack** | A per-thread memory region analogous to the JVM Stack, but used for calls into native (non-Java) code via JNI. |
| **StackOverflowError** | Thrown when a thread's JVM Stack exceeds its configured size, almost always from runaway/excessively deep recursion. |
| **OutOfMemoryError** | Thrown when the JVM cannot allocate more memory in a given region (e.g., "Java heap space" for the Heap, "Metaspace" for the Method Area) because it's exhausted. |
| **Reference** | A value (conceptually a pointer/memory address) held by an object-typed variable, pointing to the actual object data on the Heap. Primitive variables hold their value directly, with no reference involved. |
| **Interpreter (JVM)** | The Execution Engine component that reads and executes bytecode instructions one at a time, while also profiling method/branch execution counts. |
| **JIT (Just-In-Time) Compiler** | Compiles frequently executed ("hot") bytecode into native machine code at runtime, based on live profiling data gathered by the interpreter. |
| **Tiered Compilation** | HotSpot's strategy of layering two JIT compilers — C1 for fast, light compilation of warm code, and C2 for slow, heavy optimization of the hottest code — to balance startup speed against peak performance. |
| **C1 (Client Compiler)** | The fast, lightly-optimizing JIT compiler tier, used for warm (moderately hot) methods. |
| **C2 (Server Compiler)** | The slower, heavily-optimizing JIT compiler tier, reserved for the JVM's hottest methods. |
| **De-optimization** | The JVM discarding a speculatively JIT-compiled method's native code and falling back to interpretation, when a runtime assumption the compilation relied on turns out to be false. |
| **Garbage Collector (GC)** | The Execution Engine component that continuously identifies and reclaims Heap objects no longer reachable from any active reference. |
| **JNI (Java Native Interface)** | The framework letting Java code call native (C/C++) code, and native code call back into Java, via methods declared `native`. |
| **native keyword** | Marks a Java method as having no bytecode body — its implementation is provided by a loaded native library instead. |
| **JVM Specification** | The document defining required JVM behavior (class file format, bytecode semantics, etc.) that any conforming implementation (HotSpot, OpenJ9, GraalVM) must satisfy. |
| **HotSpot** | The default, most widely used JVM implementation, underlying most JDK distributions (Oracle JDK, OpenJDK, Temurin, Corretto, Zulu); named for its hot-spot-driven JIT optimization strategy. |
| **Eclipse OpenJ9** | An alternative JVM implementation (rooted in IBM J9) engineered for fast startup and low memory footprint. |
| **GraalVM** | An Oracle JVM distribution offering an alternative high-performance JIT compiler and, notably, **Native Image**. |
| **GraalVM Native Image** | Ahead-of-time (AOT) compilation of a Java application into a standalone native executable, trading runtime adaptivity and cross-platform bytecode portability for near-instant startup and lower memory footprint. |

---

## Module 03 — Java Basics

| Term | Definition |
|---|---|
| **Identifier** | A name you choose for a variable, method, class, etc.; subject to compiler-enforced rules (can't start with a digit, can't be a reserved keyword). |
| **Reserved keyword** | A word the Java language itself uses (`class`, `int`, `if`, `static`, etc.) that cannot be used as an identifier. |
| **`var`** | Local variable type inference (Java 10+); the compiler infers and fixes the variable's exact static type from its initializer at compile time. Not dynamic typing. |
| **Local variable** | A variable declared inside a method/block; lives on the JVM Stack, has no default value, must be definitely assigned before use. |
| **Instance variable (field)** | A variable declared inside a class but outside any method, one copy per object; lives on the Heap as part of its object, automatically defaulted. |
| **Static variable** | A variable declared `static`; one copy shared by the whole class; lives in the Method Area/Metaspace, automatically defaulted. |
| **Definite assignment analysis** | The compile-time check ensuring a local variable is guaranteed assigned before any code path reads it. |
| **Primitive type** | One of Java's 8 non-object built-in types (`byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean`), stored directly/inline rather than as a Heap-allocated object. |
| **Literal** | Source code text directly representing a fixed value (e.g., `42`, `3.14`, `'A'`, `"hello"`, `true`). |
| **Widening conversion** | An automatic, always-safe conversion from a smaller-range type to a larger-range type. |
| **Narrowing conversion** | A conversion from a larger-range type to a smaller-range type, requiring an explicit cast because it can lose information. |
| **Cast** | Explicit syntax (`(type) value`) telling the compiler to perform a narrowing (or reference-type) conversion, acknowledging possible data loss. |
| **Truncation** | Discarding a value's fractional part when casting a floating-point value to an integer type (rounds toward zero, not to the nearest integer). |
| **Integer overflow** | When an arithmetic result exceeds its type's representable range; Java silently wraps around by default rather than throwing an exception. |
| **Short-circuit evaluation** | The behavior of `&&`/`\|\|` to skip evaluating their right-hand operand when the result is already determined by the left-hand operand. |
| **Wrapper class** | An object class (`Integer`, `Double`, `Boolean`, etc.) that wraps a primitive value, enabling it to be used with generics, `Object`-based APIs, or represent `null`. |
| **Autoboxing** | Compiler-inserted automatic conversion of a primitive value into its corresponding wrapper object. |
| **Unboxing** | Compiler-inserted automatic conversion of a wrapper object back into its primitive value; throws `NullPointerException` if the wrapper is `null`. |
| **Integer cache** | `Integer.valueOf(int)`'s internal cache of pre-created objects for values -128 to 127, reused rather than newly allocated — the root cause of `==` behaving inconsistently on boxed `Integer` values. |
| **`final` (variable)** | Prevents a variable from being reassigned after its initial assignment; does NOT make the referenced object's own internal state immutable. |
| **Compile-time constant** | A `final` primitive or `String` initialized with a compile-time-computable expression; the compiler inlines its literal value at every usage site in bytecode. |
| **Javadoc** | A `/** ... */` comment form, recognized by the `javadoc` tool, used to generate the JDK's (and any project's) browsable HTML API documentation. |

---

## Module 04 — Control Flow

| Term | Definition |
|---|---|
| **Dangling else** | The ambiguity in a brace-less nested `if`/`else` about which `if` an `else` belongs to; Java always resolves it by binding to the nearest unmatched `if`, regardless of indentation. |
| **Fall-through** | The classic `switch` statement's default behavior of continuing execution into the next `case` block when a `break` is missing. |
| **`switch` expression** | The modern (Java 14+) form of `switch` that directly produces a value, uses arrow (`->`) syntax, and never falls through. |
| **`yield`** | The keyword used inside a `switch` expression's multi-statement block branch to specify the value that branch produces, analogous to `return` for methods. |
| **Exhaustiveness checking** | Compiler verification that a `switch` (typically over an `enum` or `sealed` type) handles every possible case, refusing to compile otherwise without a `default`. |
| **Loop invariant** | A condition that remains true across every iteration of a loop; used to reason about a loop's correctness. |
| **Infinite loop** | A loop whose condition never becomes false; can be an accidental bug (missing update logic) or an intentional design (e.g., a server's request loop with an explicit `break`/`return` exit). |
| **Enhanced for-each loop** | The `for (Type var : collection)` syntax (Java 5+) for iterating every element of an array/collection without an explicit index. |
| **Labeled statement** | An identifier followed by a colon (`label:`) placed before a loop, allowing `break label;`/`continue label;` inside a nested loop to target that specific outer loop directly. |
| **`break`** | Immediately terminates the entire enclosing loop (or `switch` case); by default affects only the innermost enclosing loop unless labeled. |
| **`continue`** | Skips the remainder of the current loop iteration's body and proceeds to the next iteration; by default affects only the innermost enclosing loop unless labeled. |

---

## Module 05 — OOP

| Term | Definition |
|---|---|
| **Class** | A blueprint/template defining the fields and methods every object of that type will have; holds no actual data itself, lives in the Method Area/Metaspace. |
| **Object (instance)** | A specific, concrete entity created from a class, with its own independent field values, allocated on the Heap. |
| **Encapsulation** | Bundling data with the behavior that operates on it, and restricting direct external access, so an object can enforce its own invariants. |
| **Invariant** | A condition that must always hold true for an object to be in a valid state (e.g., "balance is never negative"). |
| **Access modifier** | `private`, package-private (default), `protected`, or `public` — controls which code can access a field, method, constructor, or class. |
| **JavaBeans convention** | The `getXxx()`/`setXxx()`/`isXxx()` naming standard for property accessors, relied upon by reflection-based tooling (Spring, Hibernate, Jackson). |
| **Abstraction** | Exposing only essential details while hiding implementation complexity behind a simple interface. |
| **Inheritance** | A subclass acquiring the fields/methods of a superclass via `extends`, modeling an IS-A relationship. |
| **Superclass / subclass** | The class being extended (superclass/parent) and the class doing the extending (subclass/child). |
| **`super`** | Used to call a superclass's constructor (`super(...)`, first line of a subclass constructor) or a superclass's overridden method (`super.method()`). |
| **IS-A relationship** | The conceptual test for whether inheritance is appropriate — "is this subclass genuinely a specialized kind of its superclass?" |
| **Diamond Problem** | The ambiguity that arises when a class could inherit conflicting implementations/state from two unrelated parents; the reason Java disallows multiple class inheritance. |
| **`java.lang.Object`** | The root of every Java class hierarchy; every class implicitly (or explicitly) extends it. |
| **Polymorphism** | The ability for code written against a general type to work correctly with any of its specific subtypes, each behaving per its own implementation. |
| **Overloading** | Multiple methods in the same class sharing a name but differing in parameter list; resolved at compile time (static/early binding). |
| **Overriding** | A subclass providing its own implementation of an inherited method with an identical signature; resolved at runtime based on the object's actual type (dynamic/late binding). |
| **Dynamic method dispatch** | The JVM mechanism of consulting an object's actual runtime type's method table to determine which overridden implementation to invoke. |
| **Inline caching** | A JIT optimization that compiles a polymorphic call site as a direct call when profiling shows it consistently resolves to the same implementation, with de-optimization as a fallback. |
| **Upcasting** | Treating a subtype reference as its supertype; implicit and always safe. |
| **Downcasting** | Treating a supertype reference as a specific subtype; requires an explicit cast and risks `ClassCastException`. |
| **`instanceof`** | An operator that checks an object's actual runtime type; modern Java (16+) supports pattern matching to combine the check and cast in one expression. |
| **Abstract class** | A non-instantiable class that can mix abstract (unimplemented) and concrete methods, hold state/constructors, and be extended by only one subclass at a time. |
| **Interface** | A contract type, traditionally stateless (only `public static final` constants), implementable by any number of classes simultaneously. |
| **`default` method** | An interface method with a body (Java 8+), automatically inherited unless overridden, enabling interfaces to evolve without breaking existing implementers. |
| **Association** | A relationship where two independent objects interact, with neither owning the other's lifecycle. |
| **Aggregation** | A HAS-A relationship where the "part" can exist independently of the "whole" and outlive it. |
| **Composition** | A HAS-A relationship where the "part"'s lifecycle is entirely bound to the "whole." |
| **Favor composition over inheritance** | The design principle of defaulting to composition for code reuse, reserving inheritance for genuine IS-A relationships where polymorphism is actually needed. |

---

## Module 06 — Classes

| Term | Definition |
|---|---|
| **Method signature** | A method's name plus its parameter types (in order); return type is not part of the signature. |
| **Varargs** | A `Type... name` parameter letting callers pass a comma-separated argument list; compiles to an ordinary array parameter, must be the last parameter, at most one per method. |
| **Pass-by-value** | Java's single, universal parameter-passing mechanism — a copy of the argument's value is passed; for objects, the copied value is the reference itself, not the object. |
| **Constructor** | A same-named-as-the-class, no-return-type block that runs automatically, exactly once, when an object is created via `new`. |
| **Default constructor** | The compiler-generated no-argument constructor, produced only when a class defines no constructors of its own. |
| **Constructor overloading** | Multiple constructors in the same class with different parameter lists. |
| **Constructor chaining (`this(...)`)** | Calling another constructor in the same class as a constructor's first statement, to avoid duplicating initialization logic. |
| **`this`** | An implicit reference to the object instance a method/constructor was invoked on; mechanically an invisible first parameter passed to every instance method. |
| **Static member** | A field, method, or block belonging to the class itself (one shared copy in the Method Area), not to any individual object instance. |
| **Method hiding** | What happens when a subclass defines a static method with the same signature as a superclass's static method; resolved at compile time by declared type, unlike instance method overriding. |
| **Static initialization block** | A `static { ... }` block running once, in textual order among other static initializers, during class initialization. |
| **Singleton pattern** | A design pattern guaranteeing exactly one instance of a class exists, via a `private` constructor and a `static` accessor method. |
| **Instance initializer block** | A `{ ... }` block (no `static`) running for every object, interleaved in textual order with field initializers, before the constructor body. |
| **Object creation order** | The precise sequence: class-level static initialization (superclass before subclass, once) → per-`new`: superclass's full object-level initialization completes → then the subclass's own. |
| **Static nested class** | A `static` class defined inside another class, with no implicit connection to any outer instance. |
| **Inner class** | A (non-static) class defined inside another class, holding an implicit reference to one specific enclosing outer instance. |
| **Local class** | A class defined inside a method body, visible only within that method. |
| **Anonymous class** | A nameless class declared and instantiated in a single expression, historically used for one-off interface implementations (largely superseded by lambdas for functional interfaces). |

---

## Module 07 — Objects

| Term | Definition |
|---|---|
| **`getClass()`** | Returns the `Class` object representing an object's exact runtime type — the same metadata object created during class loading (Module 02). |
| **`toString()`** | Returns a human-readable text representation of an object; `Object`'s default produces `ClassName@hexHashCode`; called implicitly by `println` and String concatenation. |
| **Equals contract** | The formal requirements any `equals()` override must satisfy: reflexive, symmetric, transitive, consistent, and never true for `null`. |
| **Equals/hashCode contract** | The rule that objects equal according to `equals()` must produce the same `hashCode()`; violating it silently breaks hash-based collection lookups. |
| **Hash collision** | When two unequal objects share the same `hashCode()` — permitted by the contract and handled correctly by hash-based collections. |
| **`Cloneable`** | A marker interface (no methods) that `Object.clone()` checks at runtime; without it, `clone()` throws `CloneNotSupportedException`. |
| **Shallow copy** | A copy where reference-typed fields are copied by address, so the original and copy share the same underlying mutable object. |
| **Deep copy** | A copy where referenced objects are recursively copied too, giving the original and copy no shared mutable state. |
| **Copy constructor** | A constructor taking an instance of the same class and explicitly copying its state — the modern, recommended alternative to `Cloneable`/`clone()`. |
| **Reachability** | Whether an object can be reached via a chain of references starting from a GC Root; the basis of Java's automatic memory management. |
| **GC Root** | A starting point the JVM always treats as alive: active Stack locals/parameters, active static fields, or JNI references. |
| **`finalize()`** | A deprecated `Object` method historically intended to run before GC reclaimed an object; unreliable timing and other flaws led to its deprecation. |
| **`AutoCloseable`** | An interface (`close()` method) enabling try-with-resources, guaranteeing deterministic cleanup at a precise point in the code — the modern replacement for `finalize()`-based cleanup. |
| **`WeakReference`** | A reference that doesn't by itself keep an object reachable/alive; used for advanced caching scenarios. |

---

## Module 08 — Strings

| Term | Definition |
|---|---|
| **Immutability (String)** | The guarantee that a `String` object's content can never change after construction; every apparent modification returns a new `String` object. |
| **String Constant Pool** | A dedicated area (part of the Heap since Java 7) where the JVM stores one shared instance per distinct String literal value, reusing it automatically. |
| **`intern()`** | A method that returns the pooled instance matching a String's content, adding it to the pool first if not already present. |
| **`isBlank()`** | (Java 11+) Returns true for empty strings or strings containing only whitespace, unlike `isEmpty()` which only checks for zero length. |
| **`strip()`** | (Java 11+) The Unicode-aware replacement for `trim()`, correctly handling the full Unicode definition of whitespace. |
| **`StringBuilder`** | A mutable, resizable character buffer for efficiently building/modifying text, avoiding the repeated object creation of String concatenation in loops. |
| **`StringBuffer`** | Functionally identical to `StringBuilder` but with `synchronized` methods, making it thread-safe at the cost of performance overhead. |
| **Text block** | A `"""`-delimited multi-line String literal (Java 15+) with automatic common-indentation stripping. |
| **Format specifier** | A `%`-prefixed placeholder (`%s`, `%d`, `%.2f`, `%n`) used by `String.format`/`printf` to control how an argument is rendered as text. |

---

## Module 09 — Arrays

| Term | Definition |
|---|---|
| **Array** | An object holding a fixed number of same-typed values in contiguous, zero-based indexed slots, living on the Heap. |
| **`.length`** | The field (not a method) exposing an array's fixed size. |
| **`ArrayIndexOutOfBoundsException`** | Thrown at runtime when accessing an array index outside its valid `0` to `length-1` range. |
| **Jagged array** | A multidimensional array whose rows are independently-sized arrays, so different rows can have different lengths. |
| **`Arrays.toString()` / `deepToString()`** | Utility methods producing readable string output for 1D and multidimensional arrays respectively, since arrays don't override `toString()`. |
| **`Arrays.equals()` / `deepEquals()`** | Utility methods for correct content-based array equality, since arrays don't override `equals()`. |
| **`Arrays.copyOf()` / `copyOfRange()`** | Utility methods that create a new array (optionally a subrange) — the only way to "resize" an array, and the conceptual mechanism `ArrayList` uses internally. |
| **`Arrays.binarySearch()`** | A fast search method requiring its input array to already be sorted; produces undefined results otherwise. |
| **`ArrayList`** | A resizable, object-based list implementation, internally backed by an ordinary array that's reallocated (via `Arrays.copyOf`-style copying) as needed to grow. |
| **`Arrays.asList()`** | Returns a fixed-size `List` view directly backed by the original array — supports `set()` but not `add()`/`remove()`. |

---

## Module 10 — Collections

| Term | Definition |
|---|---|
| **`Iterable`** | The root interface requiring an `iterator()` method; anything implementing it can be used in a for-each loop. |
| **`Collection`** | Extends `Iterable`; the shared interface for `List`, `Set`, and `Queue`, providing `add`, `remove`, `contains`, `size`, etc. |
| **`List`** | An ordered collection allowing duplicates, with index-based access (`get`, `set`, `add(index, ...)`). |
| **`Set`** | A collection guaranteeing no duplicate elements. |
| **`Map`** | A key-to-value association interface, deliberately separate from `Collection`. |
| **Amortized O(1)** | An operation that's usually constant time but occasionally costs more (e.g., `ArrayList.add` triggering a resize), with the expensive cost spread across many calls. |
| **Cache locality** | How efficiently a data structure's memory layout works with CPU caches; contiguous structures (arrays) have much better locality than scattered ones (linked lists). |
| **Bucket (HashMap)** | One slot in a `HashMap`'s internal array, potentially holding multiple colliding entries in a chain (or tree, since Java 8). |
| **Hash collision** | When two different keys map to the same bucket; resolved via a chain (or tree) within that bucket, disambiguated by `equals()`. |
| **Load factor** | The threshold (default 0.75) of `HashMap` fullness that triggers a resize/rehash. |
| **Treeification** | Java 8's optimization converting a `HashMap` bucket's long collision chain (8+ entries) into a balanced Red-Black Tree for O(log n) worst-case lookup. |
| **`PriorityQueue`** | A queue that retrieves elements by priority (via `Comparable`/`Comparator`), not insertion order; backed by a binary heap. |
| **`Deque`** | A double-ended queue supporting efficient insertion/removal at both ends; usable as both a queue and a stack. |
| **Fail-fast** | The behavior of most collections' iterators to throw `ConcurrentModificationException` immediately upon detecting an unexpected structural modification during iteration. |
| **`ConcurrentModificationException`** | Thrown when a collection is structurally modified outside of its active iterator's own `remove()` method during iteration. |
| **`Comparable`** | An interface a class implements to define its own single "natural" ordering via `compareTo()`. |
| **`Comparator`** | An external, pluggable object defining a `compare()`-based ordering strategy, supporting unlimited alternative orderings. |
| **Consistent with equals** | The recommendation that `compareTo`/`Comparator` equality (`0`) align with `equals()`, since `TreeSet`/`TreeMap` use comparison logic, not `equals()`/`hashCode()`, for duplicate detection. |

---

## Module 11 — Generics

| Term | Definition |
|---|---|
| **Type parameter** | A placeholder (conventionally `T`, `E`, `K`, `V`, `R`) for a type specified at the point a generic class/method is used. |
| **Raw type** | A generic class/interface used without a type argument (e.g., `List` instead of `List<String>`), reverting to unsafe pre-generics behavior; legal only for backward compatibility. |
| **Generic method** | A method declaring its own type parameter (`<T>` before the return type), independent of whether its enclosing class is generic. |
| **Type inference** | The compiler automatically determining a generic method's type argument(s) from context, without an explicit type witness. |
| **Bounded type parameter** | A type parameter restricted with `extends` (e.g., `<T extends Number>`), guaranteeing the bound's methods are callable on it. |
| **Invariance (generics)** | The rule that `List<Integer>` is not a subtype of `List<Object>`, even though `Integer` IS-A `Object` — a deliberate safety measure, unlike array covariance. |
| **Wildcard** | `?`, `? extends T`, or `? super T` — flexible generic type arguments used at a usage site (not a declaration), enabling safe, controlled variance. |
| **PECS** | "Producer Extends, Consumer Super" — the mnemonic for choosing `? extends T` (read-only) vs. `? super T` (write-only). |
| **Type erasure** | The compiler mechanism that fully type-checks generic code at compile time, then removes all generic type information from the generated bytecode. |
| **Bridge method** | A compiler-generated synthetic method that repairs a signature mismatch erasure would otherwise create when overriding a generic method. |

---

## Module 12 — Exceptions

| Term | Definition |
|---|---|
| **`Throwable`** | The root class of everything catchable/throwable in Java, splitting into `Error` and `Exception`. |
| **`Error`** | Serious, typically unrecoverable JVM/system-level problems (e.g., `OutOfMemoryError`) that application code generally shouldn't catch. |
| **Checked exception** | An `Exception` subclass not extending `RuntimeException`; must be caught or declared with `throws`, enforced at compile time. |
| **Unchecked exception** | A `RuntimeException` (or `Error`) subclass; requires no compile-time acknowledgment. |
| **Multi-catch** | A single `catch` block handling several unrelated exception types (`catch (A \| B e)`), Java 7+. |
| **Stack unwinding** | The process of the JVM walking back up the call stack after a `throw`, popping unmatched frames until a matching `catch` is found. |
| **Try-with-resources** | A `try (Resource r = ...)` statement guaranteeing deterministic `close()` invocation on any `AutoCloseable` resource. |
| **`AutoCloseable`** | An interface (`close()` method) enabling try-with-resources; resources close in reverse declaration order. |
| **Suppressed exception** | An exception thrown during resource `close()` while a primary exception is already propagating; attached via `addSuppressed()`/retrievable via `getSuppressed()` rather than discarded. |
| **Exception chaining** | Passing an original exception as the `cause` of a new one (`super(message, cause)`) when translating across abstraction layers, preserving full diagnostic context. |
| **Exception swallowing** | The anti-pattern of catching an exception with an empty (or purely cosmetic) `catch` block, silently discarding failure information. |

---

## Module 13 — IO

| Term | Definition |
|---|---|
| **Stream (I/O)** | A sequential flow of data between a source/destination and a program, processed incrementally rather than all at once. |
| **Byte stream** | `InputStream`/`OutputStream` and subclasses; handle raw bytes with no character-encoding concept. |
| **Character stream** | `Reader`/`Writer` and subclasses; handle encoding-aware text, converting between bytes and characters. |
| **Character encoding** | An agreed-upon mapping between bytes and characters (e.g., UTF-8); mismatched encodings between writer and reader cause "mojibake" (garbled text). |
| **`java.nio.file.Path`/`Files`** | The modern (Java 7+) file-handling API, offering specific, meaningful checked exceptions instead of legacy `File`'s bare booleans. |
| **Buffering (I/O)** | Wrapping a stream (e.g., `BufferedInputStream`) to batch many small read/write operations into far fewer, larger, more efficient system calls. |
| **Decorator pattern** | The design pattern where one class wraps another, transparently adding behavior (like buffering) without changing the wrapped object's interface. |
| **`InputStreamReader`/`OutputStreamWriter`** | The explicit "bridge" classes connecting byte streams to character streams via a specified `Charset`. |
| **`Serializable`** | A marker interface enabling an object's state to be converted to/from a byte stream via `ObjectOutputStream`/`ObjectInputStream`. |
| **`transient`** | A field modifier excluding that field from serialization; left at its default value upon deserialization. |
| **`serialVersionUID`** | An explicit version identifier for a serializable class, controlling compatibility across class changes; should always be declared explicitly. |

---

## Module 14 — NIO

| Term | Definition |
|---|---|
| **`Buffer`** | A fixed-capacity container (e.g., `ByteBuffer`) tracking capacity/position/limit, used for all NIO reads and writes. |
| **`flip()`** | Switches a buffer from write mode to read mode by setting `limit = position` and resetting `position = 0`. |
| **`Channel`** | NIO's bidirectional, buffer-oriented replacement for `java.io` streams; can operate in blocking or non-blocking mode. |
| **`FileChannel`** | A channel for file I/O, supporting buffer-based reads/writes, bulk transfer (`transferTo`/`transferFrom`), and memory-mapping. |
| **Memory-mapped file** | A file mapped directly into a process's virtual memory address space (`FileChannel.map`), with the OS lazily loading pages on demand. |
| **C10K problem** | The historical challenge of handling 10,000+ concurrent connections on one server; the thread-per-connection model fails to scale due to per-thread memory/OS-scheduling overhead. |
| **Non-blocking I/O** | An I/O mode where `read()`/`write()` return immediately, reporting what's currently available, rather than waiting. |
| **`Selector`** | Lets one thread efficiently monitor many channels at once, blocking only until at least one has something ready — solves C10K without needing one thread per connection. |
| **`WatchService`** | Provides efficient, event-driven notification of filesystem changes (create/modify/delete) in a registered directory. |

---

## Module 15 — Concurrency

| Term | Definition |
|---|---|
| **Process** | An independent running program instance with its own private memory space. |
| **Thread** | A unit of execution within a process, sharing the process's Heap while maintaining its own private Stack/PC Register. |
| **Race condition** | A bug where a multi-step operation's steps interleave across threads, causing lost or inconsistent updates, dependent on unpredictable timing. |
| **Visibility problem** | A hazard where one thread's write to shared memory is not promptly (or ever) observed by another thread, due to caching/reordering. |
| **Java Memory Model (JMM)** | The JVM specification's formal definition of when cross-thread memory visibility is guaranteed, centered on the `happens-before` relationship. |
| **`happens-before`** | A formal JMM guarantee that if action A happens-before action B, A's effects are guaranteed visible to B. |
| **`volatile`** | A field modifier guaranteeing visibility (not atomicity) — establishes happens-before between a write and subsequent reads by other threads. |
| **Intrinsic lock / monitor** | The built-in lock every Java object has, acquired via `synchronized`, and operated on by `wait()`/`notify()`/`notifyAll()`. |
| **`ReentrantLock`** | An explicit lock (`java.util.concurrent.locks`) offering `tryLock`, fairness, and multiple conditions, at the cost of manual unlocking. |
| **Deadlock** | Two or more threads each waiting on a lock the other holds, indefinitely; prevented by consistent lock-acquisition ordering. |
| **Compare-And-Swap (CAS)** | A hardware-level atomic instruction that checks-and-updates a value in one indivisible step, retrying on failure — the mechanism behind atomic classes. |
| **`ExecutorService`** | A managed pool of reusable worker threads plus a task queue, avoiding the overhead of creating a new thread per task. |
| **`Future`** | A handle to a possibly-not-yet-complete asynchronous computation's result, retrieved via blocking `.get()`. |
| **`CompletableFuture`** | A composable, chainable asynchronous computation supporting non-blocking callbacks (`thenApply`, `thenCompose`, `thenCombine`) and exception handling. |
| **`ConcurrentHashMap`** | A thread-safe map using fine-grained locking (not one map-wide lock) and atomic compound operations (`putIfAbsent`, `computeIfAbsent`, `merge`). |
| **`CopyOnWriteArrayList`** | A thread-safe list that copies its entire backing array on every write, enabling completely lock-free reads — ideal for read-heavy, rarely-modified data. |
| **`BlockingQueue`** | A queue whose `put`/`take` automatically block when full/empty, implementing producer-consumer coordination without manual `wait`/`notify`. |
| **Virtual Thread** | A lightweight, JVM-managed thread (Java 21) not mapped one-to-one to an OS thread; many share a small pool of "carrier" threads, transparently unmounted while blocked. |
| **Carrier thread** | The real OS (platform) thread a Virtual Thread is temporarily mounted on while actively running. |
| **Structured Concurrency** | A Java 21+ model enforcing that a group of related concurrent subtasks are treated as a single unit, unable to outlive their logical parent scope. |

---

## Module 16 — JVM Internals

| Term | Definition |
|---|---|
| **`javap`** | The JDK disassembler tool; `javap -c` prints a compiled class's actual bytecode instructions. |
| **`invokevirtual`/`invokeinterface`** | Bytecode instructions implementing runtime dynamic dispatch (polymorphism) for instance method calls. |
| **`invokestatic`/`invokespecial`** | Bytecode instructions for calls resolved entirely at compile time (static methods, constructors, `private`/`super` calls). |
| **Generational hypothesis** | The empirical observation that most objects die young, motivating the Young/Old generation GC split. |
| **Stop-the-world (STW) pause** | A pause where application threads halt so the GC can safely reclaim/move objects. |
| **G1 (Garbage-First) GC** | The modern default collector; divides the Heap into many small regions, prioritizing collection of the most garbage-dense ones within a target pause-time budget. |
| **ZGC / Shenandoah** | Mostly-concurrent, ultra-low-latency collectors achieving sub-millisecond pauses even on very large heaps. |
| **Reordering** | The compiler/CPU's permitted freedom to execute operations in a different order than source code, absent an established happens-before relationship. |
| **Memory barrier** | A low-level CPU/compiler instruction preventing certain reorderings and/or synchronizing a core's cache with main memory — the mechanism behind `synchronized`/`volatile`. |
| **`final` field safe publication** | The JMM guarantee that a properly-initialized `final` field is visible to any thread later obtaining the object's reference, with zero explicit synchronization needed. |
| **Reflection** | The ability of a running program to inspect and invoke a class's structure at runtime, looked up by name rather than compile-time reference. |
| **`setAccessible(true)`** | A Reflection call that bypasses `private`/`protected` access control — legitimate for framework code, an anti-pattern in ordinary application logic. |
| **`@Retention`** | An annotation meta-annotation controlling whether an annotation is discarded at compile time (`SOURCE`), retained in bytecode only (`CLASS`), or available via Reflection at runtime (`RUNTIME`). |
| **Dynamic Proxy** | A runtime-generated object implementing a given interface, routing every method call through a custom `InvocationHandler` — the mechanism behind Spring's AOP features like `@Transactional`. |
| **`MethodHandle`** | A lower-level, more JIT-friendly alternative to classic Reflection for dynamic method invocation. |
| **`invokedynamic`** | A bytecode instruction allowing a call site's implementation to be linked dynamically at runtime via a bootstrap method; the mechanism enabling efficient lambda expressions. |

---

## Module 17 — Functional Programming

| Term | Definition |
|---|---|
| **Functional interface** | An interface with exactly one abstract method (default/static/Object-inherited methods don't count); the prerequisite that makes lambdas type-safe. |
| **`@FunctionalInterface`** | A compiler-enforced annotation verifying and protecting an interface's "exactly one abstract method" property. |
| **Lambda expression** | Concise syntax (`(params) -> body`) for implementing a functional interface inline, generally lighter-weight than an equivalent anonymous class (via `invokedynamic`). |
| **Closure** | A lambda that captures and retains access to variables from its enclosing scope. |
| **Effectively final** | A local variable never reassigned after its initial assignment; lambdas may only capture final or effectively-final local variables (captured as a value snapshot). |
| **Method reference** | Concise syntax (`Class::method`, `object::method`, `Class::instanceMethod`, `Class::new`) for a lambda whose body is a pure pass-through to an existing method or constructor. |
| **`Function<T,R>`** | A functional interface transforming a value of type T into a value of type R. |
| **`Supplier<T>`** | A functional interface producing a value with no input. |
| **`Consumer<T>`** | A functional interface consuming a value with no output (a side effect). |
| **`Predicate<T>`** | A functional interface testing a condition, returning boolean. |
| **Primitive-specialized functional interface** | Variants like `IntPredicate`/`IntUnaryOperator` that avoid autoboxing overhead compared to their generic (`Predicate<Integer>`) equivalents. |
| **`Optional<T>`** | A container making "this value might be absent" explicit in the type system, intended as a return type, used idiomatically via functional chaining (`map`/`filter`/`orElse`). |

---

## Module 18 — Streams

| Term | Definition |
|---|---|
| **`Stream<T>`** | A one-time-use pipeline describing a sequence of operations over a data source; not itself a data structure. |
| **Intermediate operation** | A lazy stream operation (`map`, `filter`, etc.) returning a new stream, not executed until a terminal operation runs. |
| **Terminal operation** | A stream operation (`collect`, `forEach`, `reduce`, etc.) that triggers execution of the entire pipeline and produces a result or side effect. |
| **Laziness (streams)** | The property that intermediate operations build a pipeline description without executing until a terminal operation triggers it. |
| **Short-circuiting** | A terminal operation (`anyMatch`, `findFirst`, etc.) that can stop processing the stream early once its result is already determined. |
| **`flatMap`** | An intermediate operation that flattens a per-element stream-producing mapping into one unified stream, avoiding nested `Stream<Stream<T>>`. |
| **`reduce`** | A terminal operation combining every stream element into a single result via an identity value and an accumulator function. |
| **`Collectors.groupingBy`** | Partitions a stream into a `Map` keyed by a classifier function, optionally composed with a downstream collector. |
| **`Collectors.partitioningBy`** | A specialized binary `groupingBy` based on a `Predicate`, always producing exactly two keys (`true`/`false`). |
| **Fork/join framework** | The divide-and-conquer mechanism (recursive split, process, recombine) underlying parallel stream execution. |
| **Common `ForkJoinPool`** | The JVM-wide shared thread pool parallel streams (and `CompletableFuture`'s default async methods) use by default. |

---

## Module 19 — Networking

| Term | Definition |
|---|---|
| **IP address** | Identifies a specific machine on a network. |
| **Port** | A number (0–65535) identifying a specific application/service on a machine, allowing multiple services to run on one machine simultaneously. |
| **`ServerSocket`** | Listens on a port for incoming TCP connections; `accept()` blocks until a client connects. |
| **`Socket`** | Represents one TCP connection, read/written via ordinary `InputStream`/`OutputStream` (the same API as file I/O). |
| **Thread-per-connection** | A server design handing each accepted connection off to its own thread (platform or virtual) so the accept loop can immediately serve the next client. |
| **`java.net.http.HttpClient`** | Java's modern (11+), builder-pattern-based HTTP client with built-in HTTP/2 support, replacing the legacy `HttpURLConnection`. |
| **`sendAsync`** | `HttpClient`'s asynchronous request method, returning a `CompletableFuture<HttpResponse<T>>` for direct chaining/combination. |

---

## Module 20 — JDBC

| Term | Definition |
|---|---|
| **JDBC** | Java Database Connectivity — a standard, vendor-neutral API for database access, implemented per-database by vendor-specific drivers. |
| **Driver** | A vendor-specific library implementing the JDBC interfaces, translating standard calls into a specific database's wire protocol. |
| **`Connection`** | An active, stateful session with a database; genuinely expensive to establish, motivating connection pooling. |
| **`Statement`** | Executes SQL built via string concatenation; vulnerable to SQL injection when incorporating untrusted input. |
| **SQL injection** | An attack where user input, concatenated into a SQL query, is interpreted as additional SQL syntax rather than pure data. |
| **`PreparedStatement`** | Executes parameterized SQL, sending query structure and parameter values separately, structurally preventing SQL injection. |
| **`ResultSet`** | A cursor over query results, iterated via `next()`, with values retrieved by column name or index. |
| **Transaction** | A group of statements made permanent together (`commit()`) or undone together (`rollback()`), providing atomicity. |
| **ACID** | Atomicity, Consistency, Isolation, Durability — the reliability guarantees a properly-used transaction provides. |
| **Connection pooling** | Reusing a fixed set of pre-established database connections across many requests, avoiding the cost of creating one per request — directly paralleling thread pooling (Module 15). |

---

## Module 21 — Modules (JPMS)

| Term | Definition |
|---|---|
| **JPMS** | The Java Platform Module System (Java 9+), restructuring the JDK (and optionally applications) into modules with explicit dependencies and genuine, structural encapsulation. |
| **`module-info.java`** | The module descriptor file declaring a module's name and its relationships (`requires`, `exports`, etc.) to other modules. |
| **`requires`** | Declares an explicit, verified dependency on another module; missing dependencies are caught at compile/launch time. |
| **`exports`** | Declares which packages are genuinely, structurally visible to other modules — a non-exported package is inaccessible externally regardless of `public`. |
| **`opens`** | Grants deep Reflective access (e.g., `setAccessible(true)`) to a package from outside the module, separate from and in addition to `exports`'s normal access grant. |
| **`provides`/`uses`** | Formalize the Service Provider Interface pattern at the module-descriptor level — a module `uses` a service interface without knowing the implementation, and other modules `provide` implementations. |
| **Unnamed module** | The catch-all module traditional, non-`module-info.java` classpath code is automatically placed into, preserving full backward compatibility. |
| **Automatic module** | A non-modularized JAR placed on the module path, automatically treated as a module with an inferred name and all packages implicitly exported — a migration bridge. |
| **`jlink`** | A JDK tool building a minimal, custom Java runtime image containing only the specific modules an application actually needs. |

---

## Module 22 — Performance

| Term | Definition |
|---|---|
| **`-Xms`/`-Xmx`** | JVM flags controlling initial and maximum heap size; setting them equal avoids dynamic resizing overhead. |
| **JIT warm-up problem (benchmarking)** | The reason naive timing loops are misleading: they blend slow, interpreted execution with fast, JIT-compiled execution into one uninterpretable number. |
| **JMH (Java Microbenchmark Harness)** | The standard tool for correct microbenchmarking, explicitly separating warm-up from measurement and preventing dead-code elimination. |
| **Java Flight Recorder (JFR)** | A low-overhead, production-safe JDK profiler recording real JVM events (GC, lock contention, CPU sampling, allocation) from a running application. |
| **Escape analysis** | A JIT optimization that allocates a provably-non-escaping object's fields on the Stack (or eliminates the allocation) instead of the Heap, transparently. |

---

## Module 23 — Modern Java

| Term | Definition |
|---|---|
| **Record** | A compiler-generated, always-immutable data-carrier class declared as `record Name(fields...) { }`, auto-generating the constructor, accessors, `equals()`, `hashCode()`, and `toString()` (Java 16). |
| **Compact constructor** | A record constructor form omitting the parameter list and explicit field assignments, used purely to add validation/normalization logic. |
| **Sealed class/interface** | A type declaring an explicit, closed, compiler-enforced list of permitted direct subtypes via `sealed`...`permits` (Java 17). |
| **`non-sealed`** | A modifier a permitted subtype of a sealed type uses to deliberately reopen itself to unrestricted inheritance. |
| **Pattern matching (`instanceof`/`switch`)** | Language feature binding an already-cast, flow-scoped variable directly from a type check, eliminating redundant casts (Java 16 for `instanceof`, Java 21 for `switch`). |
| **Record pattern** | A pattern that simultaneously matches a record's type and destructures its components, e.g. `case Point(int x, int y) ->` (Java 21). |
| **Exhaustive `switch`** | A `switch` over a sealed type that the compiler proves covers every permitted subtype, with no `default` needed or allowed to silently hide a missing case. |
| **`SequencedCollection`/`SequencedSet`/`SequencedMap`** | Interfaces (Java 21) providing uniform `getFirst()`/`getLast()`/`reversed()` access across collections with a genuine, defined encounter order. |
| **Stream Gatherer** | A custom, composable intermediate Stream operation (`Stream.gather()`, finalized Java 25) for transformations `collect()` can't express as an intermediate step. |
| **Unnamed variable (`_`)** | Explicit syntax marking a variable (e.g., a `catch` parameter or pattern component) as intentionally unused (Java 21-22). |
| **`ScopedValue`** | An immutable-per-binding, automatically-unbound replacement for `ThreadLocal`, designed to pair safely with Virtual Threads at scale. |
| **Foreign Function & Memory API** | A pure-Java (`java.lang.foreign`) replacement for JNI, enabling native library calls and off-heap memory access without native glue code (Java 22). |

---

## Module 24 — Interview Preparation

| Term | Definition |
|---|---|
| **Double-checked locking** | A lazy-singleton pattern checking a `null` condition both outside and inside a `synchronized` block to avoid locking overhead once initialized; requires `volatile` to be correct under the JMM. |
| **Initialization-on-demand holder idiom** | A lazy-singleton pattern relying on the JVM's guarantee that a class is initialized lazily and thread-safely exactly once, needing no explicit locking or `volatile`. |
| **Token bucket algorithm** | A rate-limiting algorithm allowing controlled bursts up to a bucket's capacity while enforcing a steady long-term refill rate. |
| **Defensive copy** | Copying a mutable object on the way into (constructor) and/or out of (accessor) a class, required for genuine immutability when a field's type is itself mutable. |

---

*(This is the final module. The course is complete.)*
