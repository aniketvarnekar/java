# Setting Up Java

## Learning Objectives

- Install a JDK on your machine
- Understand `PATH` and `JAVA_HOME`, and why both exist
- Know how to verify your installation
- Understand the JDK distribution landscape (Oracle vs OpenJDK vs Temurin, etc.) and how to choose one

## Prerequisites

[04 — JDK vs JRE vs JVM](04-JDK-vs-JRE-vs-JVM.md)

## Motivation

You can't run any of the code examples in the rest of this course without a working JDK installation. This section is deliberately practical — get through it once, verify it works, and you won't think about it again until Module 21 (Modules) or Module 22 (Performance), where installation choices become relevant again.

## Problem Statement

"Install Java" is deceptively ambiguous:
- *Which* Java version? (17? 21? 25? — Topic 8 explains LTS choice)
- *Which* distribution/vendor? (There isn't one single "Java" download the way there's one Python.org installer — more below.)
- How does your terminal *find* the `java`/`javac` commands after installing?

## Concept: The JDK Distribution Landscape

Here's something that surprises beginners: **Java itself (the language spec and the JVM spec) is open, and multiple vendors ship their own compliant JDK builds.** This is different from, say, Python, where there's essentially one canonical CPython download.

| Distribution | Vendor | Notes |
|---|---|---|
| **OpenJDK** | The open-source reference implementation | The upstream project nearly everything else is built from |
| **Eclipse Temurin** (formerly AdoptOpenJDK) | Eclipse Foundation | Extremely popular free, community-supported OpenJDK build — a very common default choice today |
| **Oracle JDK** | Oracle | Oracle's own OpenJDK-based build; free for development/most uses, with specific licensing terms for certain production scenarios — always check current terms before enterprise production use |
| **Amazon Corretto** | AWS | Free, production-ready OpenJDK build, tuned for/by AWS, popular for teams already on AWS |
| **Azul Zulu** | Azul Systems | Free and commercial OpenJDK builds, wide platform support |
| **GraalVM** | Oracle | Adds a high-performance polyglot runtime and **Native Image** (ahead-of-time compilation to a native executable — referenced in Topic 5, deep dive in Module 22) |

**Why does this matter to a beginner?** Because they're all (Oracle JDK included) built from the same open **OpenJDK** source and implement the same **JVM specification** — so for learning purposes, *any* of them work identically for the code in this course. What differs between them is licensing terms, support timelines, and specialized performance tuning — decisions that matter for production deployments (Module 22), not for learning.

> **Recommendation for this course:** install **Eclipse Temurin** or **OpenJDK** directly — both are free, unambiguous, and widely used in the industry. Pick the latest LTS version (Java 21 or Java 25 at the time of writing — see Topic 8 for what LTS means).

## Installation Steps

### macOS

Using [Homebrew](https://brew.sh) (recommended):

```bash
brew install openjdk@21
```

Or via [SDKMAN!](https://sdkman.io) (recommended if you'll ever need to switch between multiple Java versions — very common in real development):

```bash
curl -s "https://get.sdkman.io" | bash
sdk install java 21.0.5-tem   # installs Eclipse Temurin build of Java 21
```

### Windows

- Download an installer from Eclipse Temurin (adoptium.net) or use `winget`:
```powershell
winget install EclipseAdoptium.Temurin.21.JDK
```
- Or use [SDKMAN! for Windows/WSL](https://sdkman.io) inside WSL, following the macOS/Linux instructions above.

### Linux (Debian/Ubuntu-based)

```bash
sudo apt update
sudo apt install openjdk-21-jdk
```

Or, again, SDKMAN! works identically across macOS/Linux.

> **Tip: Why SDKMAN! is worth learning early.** Real Java projects frequently pin specific Java versions per-project. SDKMAN! (and similar tools like `jenv`) let you install multiple JDK versions side-by-side and switch between them per-project, instead of fighting with a single system-wide Java install. This becomes especially relevant once you're juggling an older LTS project alongside a newer one.

## `PATH` and `JAVA_HOME` — What They Are and Why Both Exist

These are two *different* environment variables that solve two *different* problems, and beginners frequently confuse them.

### `PATH`

**Problem it solves:** When you type `java` or `javac` in a terminal, how does your shell know *where on disk* those programs live?

**Mechanism:** `PATH` is an environment variable containing a list of directories. When you run a command, your shell searches each directory in `PATH`, in order, until it finds an executable with that name. Installing a JDK typically adds its `bin/` directory (containing `java`, `javac`, etc.) to your `PATH` automatically — but sometimes you must add it manually.

```
PATH = /usr/local/bin : /usr/bin : /opt/homebrew/opt/openjdk@21/bin : ...
                                    └──────────────┬──────────────┘
                                    this is where `java` and `javac` actually live
```

### `JAVA_HOME`

**Problem it solves:** Many Java-based build tools (Maven, Gradle, IDEs) don't just need to *find and run* `java` — they need to know the **root installation directory** of a specific JDK, so they can locate its libraries, headers, and other bundled resources beyond just the `bin/` executables.

**Mechanism:** `JAVA_HOME` is an environment variable pointing directly at a JDK's installation root (e.g., `/opt/homebrew/opt/openjdk@21`, or `C:\Program Files\Eclipse Adoptium\jdk-21`). Tools read this variable directly, rather than trying to reverse-engineer it from `PATH`.

**Why both are needed, not just one:** `PATH` answers "how do I *run* a command by name," a shell-level concern. `JAVA_HOME` answers "where is the *whole JDK installation* located," a tool-configuration-level concern. A build tool could technically search `PATH` and infer the JDK root, but that's fragile (what if `java` on `PATH` is a symlink, or multiple JDKs are installed?) — so the convention became: set `JAVA_HOME` explicitly, and typically also add `$JAVA_HOME/bin` to `PATH`, so both needs are satisfied from one source of truth.

```bash
# Typical shell profile (~/.zshrc or ~/.bashrc) setup:
export JAVA_HOME="/opt/homebrew/opt/openjdk@21"
export PATH="$JAVA_HOME/bin:$PATH"
```

## Verifying Your Installation

```bash
java -version
```

Expected output (version number will vary):
```
openjdk version "21.0.5" 2024-10-15
OpenJDK Runtime Environment Temurin-21.0.5+11 (build 21.0.5+11)
OpenJDK 64-Bit Server VM Temurin-21.0.5+11 (build 21.0.5+11, mixed mode, sharing)
```

```bash
javac -version
```

Expected output:
```
javac 21.0.5
```

> **If `javac -version` fails but `java -version` works:** you likely have only a JRE-style installation, or your `PATH` points at a JRE-only `bin` directory rather than a full JDK's. Revisit Topic 4 — remember, only the JDK includes `javac`.

## Common Mistakes

- Installing a JDK but never updating `PATH`/`JAVA_HOME`, then wondering why the terminal says `command not found: java`.
- Having **multiple** JDKs installed with a stale `PATH` pointing at an old one — always double check `java -version` matches what you intended to install, especially after installing a new version.
- Confusing `JAVA_HOME` pointing at the JDK's **install root** vs. its `bin/` subfolder — `JAVA_HOME` should point at the root (the folder that *contains* `bin/`, `lib/`, etc.), not at `bin/` itself.

## Best Practices

- Use a version manager (SDKMAN!, jenv, or your IDE's built-in JDK management) from day one rather than a single system-wide install — you will need multiple Java versions eventually, almost certainly.
- Always verify with both `java -version` and `javac -version` after installing — confirming only one doesn't confirm you have a full JDK.
- Match your installed JDK version to the LTS version your course/company/project targets (Topic 8 explains LTS).

## Interview Questions

1. **Q: What's the difference between `PATH` and `JAVA_HOME`, and why do build tools like Maven need `JAVA_HOME` specifically?**
   A: `PATH` lets the shell locate and run an executable by name (like `java`). `JAVA_HOME` points at the JDK's full installation root, which tools like Maven/Gradle need to locate the entire JDK's resources (libraries, headers, etc.), not just the `java` executable itself — a distinction that matters because a tool can't safely infer the whole install root just from where one executable happens to be on `PATH`.

2. **Q: Are Oracle JDK and OpenJDK functionally different?**
   A: Both implement the same OpenJDK-derived codebase and the same JVM/language specification, so functionally they behave the same for almost all practical/learning purposes. They primarily differ in licensing terms and specific vendor support arrangements, which matters for production/enterprise decisions but not for learning the language.

## Summary

- Multiple vendors ship JDK builds (OpenJDK, Temurin, Oracle JDK, Corretto, Zulu, GraalVM) from the same underlying open-source project — any is fine for learning; pick Temurin or OpenJDK if unsure.
- `PATH` lets your shell find `java`/`javac` by name; `JAVA_HOME` tells build tools where the full JDK lives — they solve related but distinct problems.
- Always verify with `java -version` **and** `javac -version`.
- Consider a version manager (SDKMAN!) early — you'll likely need multiple JDK versions across different projects.

## Exercises

1. Install a JDK on your machine (any LTS version, any distribution mentioned above) and confirm both `java -version` and `javac -version` work.
2. Print your current `PATH` and `JAVA_HOME` (`echo $PATH` / `echo $JAVA_HOME` on macOS/Linux, `echo %PATH%` / `echo %JAVA_HOME%` on Windows) and identify which directory entry corresponds to your JDK installation.
3. Explain in your own words why a build tool would fail without `JAVA_HOME` set, even if `java -version` works fine from the same terminal.

---

**Previous:** [05 — How Java Works Internally](05-How-Java-Works.md) · **Next:** [07 — Your First Java Program](07-First-Java-Program.md)
