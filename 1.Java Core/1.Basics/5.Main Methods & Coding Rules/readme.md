# ☕ Java — Main Method, Coding Standards & JVM Memory

> **Java Version Coverage:** Java 1.0 → Java 1.7+  
> Covers: main() Prototype · Overloading · Inheritance · Command Line Args · Coding Standards · JVM Memory

---

## 📑 Index

1. [main() Method](#1-main-method)
2. [Valid Variations of main()](#2-valid-variations-of-main)
3. [Overloading main()](#3-overloading-main)
4. [Inheritance & main()](#4-inheritance--main)
5. [Java 1.7 Enhancements](#5-java-17-enhancements)
6. [Command Line Arguments](#6-command-line-arguments)
7. [Java Coding Standards](#7-java-coding-standards)
8. [JVM Memory Areas](#8-jvm-memory-areas)

---

## 1. main() Method

> Compiler does **not** check for `main()` — that's JVM's job at **runtime**.  
> If JVM can't find a valid `main()` → `NoSuchMethodError: main`

```java
class Test {}
// javac Test.java  → ✅ compiles fine
// java Test        → ❌ RE: NoSuchMethodError: main
```

### Required Prototype

```java
public static void main(String[] args)
```

| Part | Reason |
|------|--------|
| `public` | JVM must be able to call it from outside |
| `static` | JVM calls it without creating an object |
| `void` | JVM doesn't expect a return value |
| `String[] args` | To receive command line arguments |

---

## 2. Valid Variations of main()

> The prototype is strict, but the following changes are **acceptable**:

```java
// ✅ Order of modifiers doesn't matter
static public void main(String[] args) {}

// ✅ String[] can be written in any form
public static void main(String[] args) {}
public static void main(String []args) {}
public static void main(String args[]) {}

// ✅ 'args' can be any valid identifier
public static void main(String[] x) {}
public static void main(String[] blah) {}

// ✅ var-arg is allowed
public static void main(String... args) {}

// ✅ Extra modifiers: final, synchronized, strictfp
static final synchronized strictfp public void main(String... ask) {
    System.out.println("valid main method");
}
```

### Valid / Invalid Declarations

```java
public static void main(String args){}                          // ❌ not an array/var-arg
public synchronized final strictfp void main(String[] args){}  // ❌ missing static
public static void Main(String... args){}                       // ❌ 'M' uppercase — wrong name
public static int main(String[] args){}                         // ❌ return type must be void
public void main(String[] args){}                               // ❌ missing static

public static void main(String... args){}                       // ✅
public static synchronized final strictfp void main(String... args){} // ✅
```

> ⚠️ **None of the above invalid cases cause a Compile Error.**  
> All invalid declarations compile fine but cause `NoSuchMethodError` at runtime.

---

## 3. Overloading main()

> Overloading is **possible**, but JVM always calls the `String[]` version.  
> Other overloaded versions must be called **explicitly**.

```java
class Test {
    public static void main(String[] args) {
        System.out.println("String[] main");   // JVM calls this ✅
    }

    public static void main(int[] args) {
        System.out.println("int[] main");      // must call explicitly
    }
}
// Output: String[] main
```

---

## 4. Inheritance & main()

> `main()` is a static method → inheritance applies.  
> If child class has no `main()`, parent's `main()` is executed.

### Child has no main()

```java
class Parent {
    public static void main(String[] args) {
        System.out.println("parent main");
    }
}

class Child extends Parent {}

// java Child  → Output: parent main  (inherits from Parent)
```

### Child has its own main() — Method Hiding (not Overriding)

```java
class Parent {
    public static void main(String[] args) {
        System.out.println("parent main");
    }
}

class Child extends Parent {
    public static void main(String[] args) {
        System.out.println("child main");
    }
}

// java Parent  → parent main
// java Child   → child main
```

> 💡 This looks like overriding but it's actually **method hiding** — because `main()` is static.

---

## 5. Java 1.7 Enhancements

### Better Error Message (Java 1.7+)

```java
class Test {}
```

| Version | Output |
|---------|--------|
| Java ≤ 1.6 | `RE: NoSuchMethodError: main` |
| Java 1.7+ | `Error: main method not found in class Test, please define the main method as public static void main(String[] args)` |

---

### Static Block Behavior

```java
class Test {
    static {
        System.out.println("static block");
    }
}
```

| Version | Output |
|---------|--------|
| Java ≤ 1.6 | `static block` → then `RE: NoSuchMethodError: main` |
| Java 1.7+ | Directly → `Error: main method not found...` (static block NOT executed) |

> 💡 From Java 1.7+, `main()` is **mandatory** to start execution — even static blocks won't run without it.

---

### Static Block + main() — Same in Both Versions

```java
class Test {
    static {
        System.out.println("static block");
    }
    public static void main(String[] args) {
        System.out.println("main method");
    }
}
// Both 1.6 and 1.7:
// static block
// main method
```

---

## 6. Command Line Arguments

> Arguments passed from the command prompt → available as `String[]` inside `main()`.  
> **All arguments are Strings** — `+` acts as **concatenation**, not addition.

### Basic Example

```java
class Test {
    public static void main(String[] args) {
        for (int i = 0; i < args.length; i++) {   // ✅ use < not <=
            System.out.println(args[i]);
        }
    }
}
// java Test x y z  → x, y, z printed
// java Test        → nothing printed (args.length = 0)
```

```java
// ❌ Common mistake — i <= args.length
for (int i = 0; i <= args.length; i++) {
    System.out.println(args[i]);   // RE: ArrayIndexOutOfBoundsException: 3
}
```

---

### String concatenation (not addition)

```java
class Test {
    public static void main(String[] args) {
        System.out.println(args[0] + args[1]);
    }
}
// java Test 10 20  → 1020  (String concat, not 30!)
```

---

### Reassigning args

```java
class Test {
    public static void main(String[] args) {
        String[] argh = {"X", "Y", "Z"};
        args = argh;                        // args now points to argh
        for (String s : args) {
            System.out.println(s);
        }
    }
}
// java Test A B C  → X, Y, Z  (argh overrides whatever was passed)
// java Test        → X, Y, Z
```

---

### Argument with spaces — use double quotes

```java
// java Test "Sai Charan"
System.out.println(args[0]);   // Sai Charan  (treated as one argument)
```

---

## 7. Java Coding Standards

> Names should **reflect purpose**. Follow these conventions for clean, readable code.

### Classes

```
- Noun
- Starts with Uppercase
- Multiple words → each word Uppercase (PascalCase)
```
```java
class Student {}
class CustomerAccount {}
class LinkedList {}
```

---

### Interfaces

```
- Adjective
- Starts with Uppercase
- PascalCase for multiple words
```
```java
interface Runnable {}
interface Serializable {}
interface Cloneable {}
```

---

### Methods

```
- Verb or Verb-Noun combination
- Starts with lowercase
- Multiple words → camelCase
```
```java
void print() {}
void getName() {}
void calculateSalary() {}
```

---

### Variables

```
- Noun
- Starts with lowercase
- Multiple words → camelCase
```
```java
int age;
String name;
double mobileNumber;
```

---

### Constants

```
- Noun
- ALL_UPPERCASE
- Multiple words → separated by underscore _
- Declare with: public static final
```
```java
public static final int MAX_VALUE = 100;
public static final int MIN_VALUE = 0;
public static final int NORM_PRIORITY = 5;
```

---

### Java Bean Standards

> A Java Bean = simple class with **private fields** + **public getters/setters**.

**Setter rules:**
- Prefix: `set`
- `public`, `void`, must have a parameter

**Getter rules:**
- Prefix: `get` (or `is` for boolean — `is` preferred)
- `public`, non-void return, no parameters

```java
class Student {
    private String name;
    private boolean active;

    public void setName(String name) { this.name = name; }   // ✅ setter
    public String getName() { return name; }                  // ✅ getter

    public boolean isActive() { return active; }              // ✅ boolean getter (is preferred)
    public boolean getActive() { return active; }             // ✅ also valid but not recommended
}
```

---

### Listener Standards

**Register a listener → prefix: `add`**

```java
public void addMyActionListener(MyActionListener l) {}    // ✅
public void registerMyActionListener(MyActionListener l) {} // ❌ wrong prefix
public void addMyActionListener(ActionListener l) {}      // ❌ wrong type
```

**Unregister a listener → prefix: `remove`**

```java
public void removeMyActionListener(MyActionListener l) {}   // ✅
public void unregisterMyActionListener(MyActionListener l) {} // ❌
public void deleteMyActionListener(MyActionListener l) {}   // ❌
```

---

## 8. JVM Memory Areas

```
JVM Memory
├── Method Area       → class-level binary data + static variables
├── Heap Area         → objects + instance variables
├── Stack (per Thread)→ method calls + local variables
│                       (each entry = Stack Frame / Action Record)
├── PC Registers      → address of next instruction to execute
└── Native Method Stack → native (non-Java) method calls
```

| Memory Area | Stores |
|-------------|--------|
| **Method Area** | Class bytecode, static variables |
| **Heap** | Objects, instance variables |
| **Stack** | Method calls, local variables (one stack per thread) |
| **PC Register** | Next instruction address (one per thread) |
| **Native Method Stack** | Native method invocations |

> 💡 **Quick recall:**
> - `static` → Method Area
> - `instance` → Heap
> - `local` → Stack
