# Your First Java Program

## Learning Objectives

- Write, compile, and run a real Java program
- Explain every single keyword in the classic `HelloWorld.java` — nothing left unexplained
- Recognize and fix the most common beginner compilation errors
- Know the modern (Java 21+) simplified alternative and how it differs

## Prerequisites

[04 — JDK vs JRE vs JVM](04-JDK-vs-JRE-vs-JVM.md), [05 — How Java Works Internally](05-How-Java-Works.md), [06 — Setting Up Java](06-Setting-Up-Java.md)

## Motivation

Every Java tutorial shows you `HelloWorld.java` in the first five minutes — and then rushes past it without explaining half the words in it. That's backwards for this course. We are going to slow down and explain **every single keyword and symbol**, because this one tiny program actually contains the seeds of five major topics you'll study in depth later: classes (Module 05/06), access modifiers (Module 05), the `static` keyword (Module 06), arrays (Module 09), and packages/imports (Module 06/21).

## Problem

Write a program that prints the text `Hello, World!` to the screen.

## The Code

**File name: `HelloWorld.java`** (this must exactly match the public class name — explained below)

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

## Line-by-Line Explanation

### Line 1: `public class HelloWorld {`

| Token | Meaning |
|---|---|
| `public` | An **access modifier**. It means this class is visible/usable from *any* other code, in any package. (Full depth on access modifiers: Module 05.) |
| `class` | A **keyword** declaring that we're defining a new class — a blueprint for objects, and in this case, also a self-contained unit that can hold our program's entry point. (Full depth: Module 05.) |
| `HelloWorld` | The **class name** — chosen by you. Java convention: class names use **PascalCase** (each word capitalized, no underscores/spaces) — e.g., `HelloWorld`, `BankAccount`, `OrderProcessor`. |
| `{` | Opens the **class body** — everything inside these braces belongs to the `HelloWorld` class. |

> **Critical rule:** if a class is declared `public`, the **file name must exactly match the class name**, including capitalization: `HelloWorld.java`, not `helloworld.java` or `Hello_World.java`. This is enforced by `javac` — get it wrong and compilation fails immediately. **Why does this rule exist?** So the compiler and class loader can locate a public class's source/bytecode file by a predictable naming convention, without having to scan every file's contents first.

### Line 2: `    public static void main(String[] args) {`

This is the **method signature** for `main` — the most important line in the file, because the JVM specifically looks for **exactly this signature** as the program's entry point.

| Token | Meaning |
|---|---|
| `public` | Same access modifier as before — `main` must be `public` so the JVM (which is, conceptually, "outside" your class) can call it. |
| `static` | Means this method belongs to the **class itself**, not to any particular object/instance of the class. **Why is this required for `main`?** Because when the JVM starts your program, **no objects exist yet** — there's nothing to call an instance method *on*. `static` lets the JVM invoke `main` directly via the class, without needing to construct a `HelloWorld` object first. (Full depth on `static`: Module 06.) |
| `void` | The **return type** — `void` means this method returns no value. `main` doesn't hand a result back to the JVM through a return value (a program's *exit code*, if needed, is set differently — via `System.exit(int)`, covered later). |
| `main` | The exact **method name** the JVM looks for. This is not arbitrary — it's a hardcoded convention the JVM launcher searches for. |
| `(String[] args)` | The **parameter list**: a single parameter named `args`, of type `String[]` — an **array** of `String` objects. This holds any command-line arguments passed when launching the program (e.g., `java HelloWorld foo bar` → `args = ["foo", "bar"]`). Arrays are covered fully in Module 09; for now, just know `String[]` means "a list of `String` values." |
| `{` | Opens the **method body**. |

> **Why must the signature match exactly?** The JVM's launcher performs a very literal lookup: "does this class have a method named `main`, that is `public`, `static`, returns `void`, and takes a single `String[]` parameter?" If any part differs — wrong name, missing `static`, wrong parameter type — the JVM reports `Error: Main method not found` and refuses to start, **even if the code otherwise compiles perfectly fine**. This is a runtime launch-time check, not a compile-time one.

### Line 3: `        System.out.println("Hello, World!");`

| Token | Meaning |
|---|---|
| `System` | A built-in class from the standard library (package `java.lang`, covered in Module 06), representing system-level facilities. |
| `.out` | A **static field** on the `System` class, of type `PrintStream`, representing the "standard output" stream — conventionally, your terminal/console. |
| `.println(...)` | A **method** on that `PrintStream` object that prints its argument, followed by a newline character. |
| `"Hello, World!"` | A **String literal** — text data, enclosed in double quotes. (Full depth: Module 08.) |
| `;` | Every Java **statement** must end with a semicolon — it's how the compiler knows one instruction ends and the next begins. |

**Why the chained dots (`System.out.println`)?** This is simply accessing a static field (`out`) on a class (`System`), then calling a method (`println`) on the object that field holds. It reads left to right: "Go to the `System` class → get its `out` field → call `println` on it."

### Lines 4–5: `    }` and `}`

The first closing brace ends the `main` method body. The second ends the `HelloWorld` class body. Braces in Java always come in matched pairs, and indentation (while not syntactically required, unlike Python) is a strong convention to visually track which brace closes which block.

## Compiling and Running

```bash
javac HelloWorld.java     # produces HelloWorld.class (bytecode)
java HelloWorld           # JVM loads and runs it -- note: NO ".class" extension here
```

**Output:**
```
Hello, World!
```

> **Common typo:** running `java HelloWorld.class` (with the extension) — this is wrong. `javac` takes a filename with `.java`; `java` takes a **class name**, not a filename, and must be given *without* any extension.

## Internal Execution (Tying Back to Topic 5)

1. `javac HelloWorld.java` → produces `HelloWorld.class` containing bytecode.
2. `java HelloWorld` → JVM starts, Class Loader loads `HelloWorld.class`, Bytecode Verifier checks it.
3. The JVM's launcher looks specifically for `public static void main(String[] args)` inside `HelloWorld` and invokes it — **directly on the class, with zero `HelloWorld` objects ever created**, precisely because `main` is `static`.
4. Inside `main`, the interpreter executes the `System.out.println(...)` bytecode instructions, which ultimately call into native OS functionality to write text to your terminal.
5. `main` finishes (falls off the end of the method body) → the JVM has no more work to do → the JVM process exits.

## Common Mistakes (Beginner Compilation & Runtime Errors)

| Mistake | Symptom | Fix |
|---|---|---|
| File name doesn't match public class name (`Hello.java` containing `public class HelloWorld`) | `javac` error: *"class HelloWorld is public, should be declared in a file named HelloWorld.java"* | Rename the file to match the public class name exactly. |
| Forgetting a semicolon | `javac` error: *";" expected* | Add the missing `;` at the end of the statement. |
| Misspelling `main`, or wrong signature (e.g., `public void main(...)` — missing `static`) | Compiles fine, but running gives: *"Error: Main method not found in class HelloWorld"* | Match the exact required signature: `public static void main(String[] args)`. |
| Running with the `.class` extension (`java HelloWorld.class`) | `Error: Could not find or load main class HelloWorld.class` | Run `java HelloWorld` — no extension. |
| Mismatched braces | Confusing cascading compiler errors, sometimes far from the real mistake | Use an editor with bracket-matching/auto-indentation; count braces carefully. |
| Using `println` without `System.out.` (assuming it's a free-standing function like Python's `print`) | `javac` error: *cannot find symbol* | Java has no free-standing functions outside classes — you must call `println` *on* something, here `System.out`. |

## Variations

**Multiple statements:**
```java
public class Greeting {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
        System.out.println("Welcome to Java.");
    }
}
```

**Using command-line arguments:**
```java
public class Greeting {
    public static void main(String[] args) {
        if (args.length > 0) {
            System.out.println("Hello, " + args[0] + "!");
        } else {
            System.out.println("Hello, World!");
        }
    }
}
```
Run with: `java Greeting Aniket` → prints `Hello, Aniket!` (array indexing and the `+` string concatenation operator are covered fully in Modules 08–09 — don't worry about full understanding yet, just notice `args` in action).

**Single-file source-code launch (Java 11+):** you can skip the separate `javac` step entirely for quick scripts:
```bash
java HelloWorld.java
```
This compiles **in-memory** and runs immediately, without producing a `HelloWorld.class` file on disk — convenient for quick experiments and scripting, not typically used for real multi-file projects (which use build tools like Maven/Gradle instead — outside this course's Core Java scope, but worth knowing exists).

**Modern compact source files (Java 21+ preview, refined through Java 25):** Java has been actively simplifying the beginner on-ramp. A preview feature lets you skip the class/`public static`/`String[] args` ceremony entirely for simple programs:
```java
void main() {
    System.out.println("Hello, World!");
}
```
This is **not** a different language feature bolted on — under the hood, the compiler still generates an actual class with a proper `main` method; it just infers and hides the boilerplate for you. **Why mention this if it's not the "default" way?** Because Java's design has explicitly moved to reduce the intimidating ceremony beginners face on line one, while keeping full backward compatibility with the classic form — a good example of the backward-compatibility philosophy from Topic 2 in action: the old form still works forever, the new form is additive. We'll use the full classic form throughout most of this course since it's what you'll encounter in the overwhelming majority of real, existing Java code, but you should recognize the compact form when you see it.

## Best Practices

- Match file name to public class name — always, no exceptions.
- One public class per file (Java enforces this; you *can* have multiple non-public classes in one file, covered in Module 06).
- Use PascalCase for class names, and always start with a capital letter.
- Indent consistently (most teams use 4 spaces) — not required by the compiler, but essential for human readability.

## Interview Questions

1. **Q: Why must the `main` method be `static`?**
   A: Because the JVM invokes `main` before any instance of your class exists — `static` means the method belongs to the class itself, callable without an object, which is exactly what the JVM needs at program startup.

2. **Q: What happens if you name your `main` method correctly but forget the `public` modifier?**
   A: It will compile successfully (mostly) but the JVM's launcher requires `public` visibility to call `main` from outside the class, so you'll get a runtime "Main method not found" error when trying to run it, even though the code compiled.

3. **Q: What is `args` in `public static void main(String[] args)`, and where does its data come from?**
   A: It's an array of `String` values representing command-line arguments passed after the class name when launching the program (e.g., `java MyProgram arg1 arg2` gives `args = ["arg1", "arg2"]`); if none are passed, `args` is a valid, empty array (`args.length == 0`), never `null`.

## Summary

- `HelloWorld.java` looks simple but encodes five real Java concepts: classes, access modifiers, `static`, arrays, and the standard library.
- The file name must match the public class name exactly.
- `main`'s signature (`public static void main(String[] args)`) is a strict, JVM-enforced contract, not a stylistic convention.
- `javac` compiles; `java <ClassName>` (no extension) runs.
- Modern Java (11+ single-file launch, 21+ compact source) has progressively reduced beginner ceremony, while the classic form remains the standard you'll see in nearly all real-world code.

## Exercises

1. Type out `HelloWorld.java` yourself (don't copy-paste), compile it, and run it successfully.
2. Deliberately introduce each mistake from the "Common Mistakes" table, one at a time, and observe the exact error message `javac`/`java` produces. This builds pattern-recognition for real debugging later.
3. Modify the program to print your name using a command-line argument, as shown in the Variations section.
4. Without looking back at this page, write out the full required signature of `main` from memory.

---

**Previous:** [06 — Setting Up Java](06-Setting-Up-Java.md) · **Next:** [08 — Java Editions & Version Timeline](08-Java-Editions-And-Versions.md)
