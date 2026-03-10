# ☕ Java — Source File Structure, Import, Package

> **Java Version Coverage:** Java 1.0+ · Static Import (1.5v)  
> **Topics:** Source File Structure · Import Statements · Static Import · Package Statement 

---

## 📑 Index

| # | Topic |
|---|-------|
| 1 | [Java Source File Structure](#1-java-source-file-structure) |
| 2 | [Import Statement](#2-import-statement) |
| 3 | [Static Import (1.5v)](#3-static-import-15v) |
| 4 | [Package Statement](#4-package-statement) |
| 5 | [Java Source File — Complete Structure](#5-java-source-file--complete-structure) |

---

## 1. Java Source File Structure

### 📌 Key Rules

**Rule 1 — A Java program can contain any number of classes, but at most ONE class can be declared `public`.**

```java
class A { }         // ✅  non-public
public class B { }  // ✅  one public class
public class C { }  // ❌ CE: class C is public, should be declared in a file named C.java
```

**Rule 2 — If a `public` class exists, the filename MUST match the public class name.**

```
public class B  →  file must be named  B.java
```

**Rule 3 — If there is NO public class, any filename is allowed.**

---

### 📌 Cases

**Case 1 — No public class → any filename is valid**

```java
class A { }
class B { }
class C { }
```

Valid filenames: `A.java`, `B.java`, `C.java`, `Ashok.java` — all compile fine.

**Case 2 — One public class → filename must match**

```java
public class B { }  // File must be B.java
```

If file is named `Test.java` → ❌ CE: `class B is public, should be declared in a file named B.java`

**Case 3 — Two public classes in same file → CE**

```java
public class B { }
public class C { }  // ❌ CE: class C is public, should be declared in a file named C.java
```

> 💡 **Best Practice:** Keep only **one class per source file** and name the file same as the class. This improves readability and understandability.

---

### 📌 Compile vs Run

```
javac Bhaskar.java
    ↓         ↓         ↓         ↓
 A.class   B.class   C.class   D.class
```

```java
class A { public static void main(String args[]){ System.out.println("A class main method is executed"); } }
class B { public static void main(String args[]){ System.out.println("B class main method is executed"); } }
class C { public static void main(String args[]){ System.out.println("C class main method is executed"); } }
class D { }   // no main method
```

| Command | Output |
|---------|--------|
| `java A` | A class main method is executed |
| `java B` | B class main method is executed |
| `java C` | C class main method is executed |
| `java D` | ❌ RE: `NoSuchMethodError: main` |
| `java Ashok` | ❌ RE: `NoClassDefFoundError: Ashok` |

**Key Points:**
- We compile a **java source file** (`.java`) — NOT a class.
- We run a **java class** (`.class`) — NOT a source file.
- Compiling one `.java` file creates **one `.class` per class** inside it.
- If class has no `main()` → RE: `NoSuchMethodError: main`
- If `.class` file doesn't exist → RE: `NoClassDefFoundError`

---

## 2. Import Statement

### 📌 Why Import?

Without import, we must use the **fully qualified name** every time:

```java
// Without import — verbose:
java.util.ArrayList l = new java.util.ArrayList();

// With import — clean:
import java.util.ArrayList;
ArrayList l = new ArrayList();   // ✅
```

> 💡 Import allows using **short names** directly. It **decreases code length** and **improves readability**.

---

### 📌 Types of Import

**1. Explicit Class Import**

```java
import java.util.ArrayList;   // imports only ArrayList
```

- **Highly recommended** — improves readability.
- Best for production / Hi-Tech City environments where code clarity matters.

**2. Implicit Class Import (Wildcard)**

```java
import java.util.*;   // imports all classes in java.util package
```

- **Not recommended** — reduces readability (reader can't tell which classes are actually used).
- Best for fast typing / prototyping.

---

### 📌 Import Cases

**Case 1 — Which imports are meaningful?**

| Import Statement | Valid? | Reason |
|-----------------|:------:|--------|
| `import java.util;` | ❌ | Cannot import a package — must specify class or `.*` |
| `import java.util.ArrayList.*;` | ❌ | `ArrayList` is a class, not a package — cannot use `.*` after a class |
| `import java.util.*;` | ✅ | Wildcard import of all classes in `java.util` |
| `import java.util.ArrayList;` | ✅ | Explicit import of `ArrayList` class |

**Case 2 — Fully qualified name makes import optional**

```java
class MyArrayList extends java.util.ArrayList {
}  // ✅ compiles fine — no import needed when using fully qualified name
```

> 💡 Fully qualified name ↔ import statement are **alternatives** — you need only one.

**Case 3 — Ambiguity with wildcard imports**

```java
import java.util.*;
import java.sql.*;
class Test {
    Date d = new Date();  // ❌ CE: reference to Date is ambiguous
                          // both java.sql.Date and java.util.Date match
}
```

> ⚠️ `Date` exists in both `java.util` and `java.sql`. `List` exists in both `java.util` and `java.awt`.

**Case 4 — Compiler resolves ambiguity via priority order**

When the same class name exists in multiple imported packages, compiler uses this priority:

```
1. Explicit class import        (highest priority)
2. Classes in current directory (current working directory)
3. Implicit (wildcard) import   (lowest priority)
```

```java
import java.util.Date;   // explicit — takes priority
import java.sql.*;       // wildcard
class Test {
    Date d = new Date(); // ✅ uses java.util.Date (explicit wins)
}
```

**Case 5 — Sub-packages are NOT included in wildcard import**

```
java
└── util
    └── regex
        └── Pattern
```

To use `Pattern` class:

| Import | Works? |
|--------|:------:|
| `import java.*;` | ❌ sub-packages not included |
| `import java.util.*;` | ❌ `regex` is a sub-package, not a class in `util` |
| `import java.util.regex.*;` | ✅ |
| `import java.util.regex.Pattern;` | ✅ |

**Case 6 — Packages available by default (no import needed)**

Two packages are **automatically available** to every Java program:
1. `java.lang` — `String`, `System`, `Math`, `Integer`, etc.
2. **Default package** (current working directory)

**Case 7 — Import is a compile-time concept**

> `import` statements are **totally a compile-time concept**.  
> More imports → more compile time.  
> **No change in execution/runtime**.

---

### 📌 `#include` (C) vs `import` (Java)

| Feature | `#include` (C/C++) | `import` (Java) |
|---------|-------------------|-----------------|
| Language | C and C++ | Java |
| When executed | At compile time — compiler copies code from library into program | At runtime — JVM executes the library code when needed |
| Loading type | **Static loading** (all header files loaded at include time) | **Dynamic loading / load-on-demand / load-on-fly** (`.class` loaded only when actually used) |
| Memory | Wastage of memory | No wastage of memory |
| JSP equivalent | `<jsp:@ file="">` | `<jsp:include>` |

---

## 3. Static Import (1.5v)

### 📌 Introduction

Introduced in **Java 1.5**. Allows accessing **static members** (fields and methods) of a class **directly without the class name**.

> ⚠️ According to Sun: static import **improves readability**.  
> According to industry experts: static import **creates confusion and reduces readability**.  
> **Recommendation: Avoid static import unless there is a specific requirement.**

---

### 📌 Without vs With Static Import

```java
// Without static import:
class Test {
    public static void main(String args[]) {
        System.out.println(Math.sqrt(4));    // 2.0
        System.out.println(Math.max(10,20)); // 20
        System.out.println(Math.random());   // random value
    }
}

// With static import:
import static java.lang.Math.sqrt;
import static java.lang.Math.*;

class Test {
    public static void main(String args[]) {
        System.out.println(sqrt(4));     // 2.0  — no Math. needed
        System.out.println(max(10,20));  // 20   — no Math. needed
        System.out.println(random());    // random value
    }
}
```

---

### 📌 System.out.println() — What it means

```
System          →  class present in java.lang package
   .out         →  static variable of type PrintStream present in System class
      .println  →  method present in PrintStream class
```

To use `out.println()` without `System`:
```java
import static java.lang.System.out;
class Test {
    public static void main(String args[]) {
        out.println("hello");  // ✅
        out.println("hi");
    }
}
```

---

### 📌 Static Import Ambiguity

```java
import static java.lang.Integer.*;
import static java.lang.Byte.*;
class Test {
    public static void main(String args[]) {
        System.out.println(MAX_VALUE);  // ❌ CE: reference to MAX_VALUE is ambiguous
                                        // both Integer.MAX_VALUE and Byte.MAX_VALUE match
    }
}
```

> 💡 Two packages having the same class is **rare** → normal import ambiguity is rare.  
> But two classes having the same method/variable name is **very common** → static import ambiguity is very common.

**Compiler resolution priority for static members:**

```
1. Current class static members        (highest)
2. Explicit static import
3. Implicit (wildcard) static import   (lowest)
```

**Example:**

```java
// import static java.lang.Integer.MAX_VALUE;  ← line2 (commented)
import static java.lang.Byte.*;

class Test {
    // static int MAX_VALUE = 999;  ← line1 (commented)
    public static void main(String args[]) throws Exception {
        System.out.println(MAX_VALUE);
    }
}
```

| line1 active | line2 active | Output |
|:---:|:---:|--------|
| ✅ | any | `999` — current class member wins |
| ❌ | ✅ | `2147483647` — explicit import (Integer.MAX_VALUE) wins |
| ❌ | ❌ | `127` — implicit import (Byte.MAX_VALUE) used |

---

### 📌 Valid vs Invalid Static Import Syntax

```
normal import starts with → class name (Math) or wildcard (util.*)
static import starts with → member name (Math.sqrt) or class wildcard (Math.*)
```

| Statement | Valid? | Reason |
|-----------|:------:|--------|
| `import java.lang.Math.*;` | ❌ | Normal import — `Math` is a class, cannot use `.*` on class |
| `import static java.lang.Math.*;` | ✅ | Static import all members of Math |
| `import java.lang.Math;` | ✅ | Normal import of Math class |
| `import static java.lang.Math;` | ❌ | Cannot statically import a class |
| `import static java.lang.Math.sqrt.*;` | ❌ | `sqrt` is a method, not a package/class |
| `import java.lang.Math.sqrt;` | ❌ | Normal import cannot target a method |
| `import static java.lang.Math.sqrt();` | ❌ | Parentheses not allowed in import |
| `import static java.lang.Math.sqrt;` | ✅ | Static import of specific method |

---

### 📌 General Import vs Static Import

| Feature | General Import | Static Import |
|---------|---------------|---------------|
| Purpose | Import classes/interfaces from a package | Import static members of a class |
| Benefit | Use class/interface by short name — no fully qualified name needed | Use static members directly — no class name needed |
| Example | `import java.util.ArrayList;` → use `ArrayList` | `import static java.lang.Math.*;` → use `sqrt()` |

---

## 4. Package Statement

### 📌 What is a Package?

A package is an **encapsulation mechanism** to group related classes and interfaces into a single module.

**Objectives of packages:**
- Resolve name conflicts between classes
- Improve modularity of the application
- Provide security

**Naming Convention:** Use internet domain name in reverse.

```
com.icicibank.loan.housingloan.Account
 │              │      │          │
client domain  module submodule  class
(reversed)
```

---

### 📌 How to Compile a Package Program

```java
package com.durgajobs.itjobs;
class HydJobs {
    public static void main(String args[]) {
        System.out.println("package demo");
    }
}
```

**Method 1 — Simple compile (class placed in current working directory):**

```
javac HydJobs.java

cwd/
└── HydJobs.class   ← class in cwd, NOT inside package structure
```

**Method 2 — Compile with `-d` flag (creates proper package structure):**

```
javac -d . HydJobs.java
```

`-d` = destination for generated `.class` files  
`.`  = current working directory

```
cwd/
└── com/
    └── durgajobs/
        └── itjobs/
            └── HydJobs.class
```

**Compile to a specific drive:**

```
javac -d c: HydJobs.java

c:/
└── com/
    └── durgajobs/
        └── itjobs/
            └── HydJobs.class
```

> ❌ If destination does not exist → CE: destination directory does not exist  
> Example: `javac -d z: HydJobs.java` → CE if Z: drive doesn't exist

**Key Points:**
- If specified package structure doesn't exist, `-d` **creates it automatically**.
- As destination, any valid directory can be used.
- If destination is not available → CE.

---

### 📌 How to Execute a Package Program

```
java com.durgajobs.itjobs.HydJobs
```

> ⚠️ At the time of execution, the **fully qualified class name** is compulsory.

---

### 📌 Package Statement Rules

**Conclusion 1 — At most ONE package statement per source file.**

```java
package pack1;
package pack2;   // ❌ CE: class, interface, or enum expected
class A { }
```

**Conclusion 2 — Package statement must be the FIRST non-comment statement.**

```java
import java.util.*;
package pack1;   // ❌ CE: class, interface, or enum expected
class A { }
```

Correct order:
```java
package pack1;       // ✅ first
import java.util.*;  // then imports
class A { }          // then class declarations
```

---

## 5. Java Source File — Complete Structure

```
┌─────────────────────────────────┐  ←  order is IMPORTANT
│  package statement (at most 1)  │
│  import statements (any number) │
│  class/interface/enum (any num) │
└─────────────────────────────────┘
```

**All of the following are valid Java programs:**

| File Contents | Valid? |
|---------------|:------:|
| Empty file | ✅ An empty source file is a valid Java program |
| `package pack1;` only | ✅ |
| `import java.util.*;` only | ✅ |
| `package pack1; import java.util.*;` | ✅ |
| `class Test { }` only | ✅ |

> 💡 Note: An **empty source file** is a valid Java program.

---


## 📝 Quick Recap

| Topic | Key Rule |
|-------|----------|
| Public classes per file | At most **1** |
| Filename rule | If public class present → filename must match public class name |
| No public class | Any filename allowed |
| `java` vs `javac` | `javac` compiles `.java` file; `java` runs `.class` file |
| NoSuchMethodError | Class exists but has no `main()` method |
| NoClassDefFoundError | `.class` file not found |
| Explicit import | `import java.util.ArrayList;` — recommended for readability |
| Implicit (wildcard) import | `import java.util.*;` — not recommended |
| `import java.util;` | ❌ Invalid — cannot import package itself |
| `import java.util.ArrayList.*;` | ❌ Invalid — `ArrayList` is a class, not a package |
| Ambiguity in wildcard | `Date` in both `java.util` and `java.sql` → CE when both `.*` imported |
| Compiler priority (import) | Explicit → current dir → wildcard |
| Sub-packages | NOT included in wildcard import — must import separately |
| Auto-available packages | `java.lang` and default (current) package |
| Import is | Compile-time concept only — no effect on execution time |
| `#include` vs `import` | Static loading vs dynamic loading (load-on-fly) |
| Static import | `import static java.lang.Math.*;` — access static members without class name |
| Static import — NOT recommended | Creates confusion and reduces readability |
| Static member priority | Current class → explicit static import → wildcard static import |
| Package naming | Use internet domain in reverse: `com.company.module.Class` |
| `-d` flag | `javac -d . File.java` — creates package directory structure |
| Execution | Must use fully qualified name: `java com.pack.ClassName` |
| Package statement | At most ONE, must be FIRST non-comment statement |
| Source file order | package → import → class/interface/enum (order is mandatory) |
| Top-level class modifiers | Only: `public`, `default`, `final`, `abstract`, `strictfp` |
| Invalid top-level modifiers | `private`, `protected`, `static` → CE |
