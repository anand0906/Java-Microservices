# ☕ Java OOP Notes

> **Java Versions Referenced:** Java 5, Java 8, Java 11, Java 17 (LTS), Java 21 (LTS)

---

## 📑 Index

1. [Method Signature](#1-method-signature)
2. [Polymorphism](#2-polymorphism)
3. [Method Overloading](#3-method-overloading)
   - [Automatic Promotion in Overloading](#31-automatic-promotion-in-overloading)
   - [Overloading Resolution Priority](#32-overloading-resolution-priority)
4. [Method Overriding](#4-method-overriding)
   - [Rules for Overriding](#41-rules-for-overriding)
   - [Co-variant Return Types](#42-co-variant-return-types)
   - [Overriding with Static Methods](#43-overriding-with-respect-to-static-methods)
   - [Overriding with Var-arg Methods](#44-overriding-with-respect-to-var-arg-methods)
   - [Overriding with Variables](#45-overriding-with-respect-to-variables)
5. [Overloading vs Overriding — Comparison](#5-overloading-vs-overriding--comparison)
6. [Method Hiding](#6-method-hiding)
7. [Checked vs Unchecked Exceptions](#7-checked-vs-unchecked-exceptions)
8. [Ways to Create an Object](#8-ways-to-create-an-object)
9. [Constructors](#9-constructors)
   - [Rules for Constructors](#91-rules-for-constructors)
   - [Default Constructor](#92-default-constructor)
   - [super() vs this()](#93-super-vs-this)
   - [Overloaded Constructors](#94-overloaded-constructors)
   - [Constructor vs Instance Block](#95-constructor-vs-instance-block)

---

## 1. Method Signature

In Java, a **method signature = method name + argument types** (order matters).

```
public void methodOne(int i, float f)
              ↓
         methodOne(int, float)   ← this is the signature
```

- **Return type is NOT part of the signature.**
- The compiler uses the signature to resolve method calls.
- Two methods with the **same signature** in the same class → Compile Error.

```java
// ❌ Compile Error — same signature methodOne()
public void methodOne() { }
public int  methodOne() { return 10; }
// Error: methodOne() is already defined
```

> 💡 **Java 21 — Pattern Matching** uses method signatures heavily when dispatching across sealed class hierarchies.

---

## 2. Polymorphism

> **"Same name, different forms."**

The 3 pillars of OOP and what they offer:

```
              OOP
           /   |   \
Inheritance  Polymorphism  Encapsulation
    |              |              |
Reusability   Flexibility     Security
```

**Types of Polymorphism:**

```
            Polymorphism
           /             \
  Compile-time         Runtime
  (Static / Early)   (Dynamic / Late)
      /     \               \
Overloading  Method        Overriding
             Hiding
```

**Examples:**
- `abs(int)`, `abs(long)`, `abs(float)` — same name, different argument types → overloading
- `List l = new ArrayList()` — same reference type, different runtime objects → runtime polymorphism

---

## 3. Method Overloading

> Two methods are overloaded when they have the **same name but different argument types**.

- Java allows multiple methods with the same name — unlike C (which needed `abs`, `labs`, `fabs` etc.)
- Main advantage: **reduces programming complexity**
- Method resolution in overloading is done by the **compiler based on reference type** → compile-time polymorphism / static polymorphism / early binding.

```java
class Calculator {
    int add(int a, int b)       { return a + b; }
    double add(double a, double b) { return a + b; }   // overloaded
    int add(int a, int b, int c) { return a + b + c; } // overloaded
}
```

### 3.1 Automatic Promotion in Overloading

If the compiler **can't find an exact match**, it automatically promotes the argument to the next compatible type before giving up.

**Promotion chain:**

```
byte ──> short ─┐
                 ├──> int ──> long ──> float ──> double
char ───────────┘
```

```java
// Only methodOne(int) and methodOne(float) exist
t.methodOne('a');   // 'a' (char) promoted to int  → int version called
t.methodOne(101);   // int  → int version called
t.methodOne(10.5);  // 10.5 is double — no double method → Compile Error
```

### 3.2 Overloading Resolution Priority

When multiple matches are possible, the compiler follows this priority order:

| Priority | Rule |
|---|---|
| 1st | **Exact match** |
| 2nd | **Child class type** over parent class type |
| 3rd | **Automatic promotion** (widening) |
| 4th | **Var-arg method** (lowest — like `default` in switch) |

**Example — Child vs Parent priority:**

```java
class Animal {}
class Monkey extends Animal {}

void methodOne(Animal a)  { ... }
void methodOne(Monkey m)  { ... }

Animal a1 = new Monkey();
t.methodOne(a1);   // Animal version — based on REFERENCE type, not runtime object
```

> ⚠️ In overloading, the **runtime object doesn't matter** — only the reference type does.

**Example — Null ambiguity:**

```java
void methodOne(String s)       { ... }
void methodOne(StringBuffer sb) { ... }

t.methodOne(null);  // ❌ Compile Error: ambiguous
// String and StringBuffer are at same level (both extend Object directly)
```

**Example — Var-arg gets least priority:**

```java
void methodOne(int i)     { System.out.println("general"); }
void methodOne(int... i)  { System.out.println("var-arg"); }

t.methodOne();       // var-arg (no exact match for 0 args)
t.methodOne(10, 20); // var-arg (no exact match for 2 args)
t.methodOne(10);     // general (exact match wins)
```

> 💡 **Var-args** were introduced in **Java 5**.

---

## 4. Method Overriding

> If a child is not satisfied with the parent's implementation of a method, it can **redefine** it in its own way. This is overriding.

- The parent method being replaced → **overridden method**
- The child method doing the replacing → **overriding method**
- Method resolution is by **JVM at runtime based on the actual object** → runtime polymorphism / dynamic polymorphism / late binding / **dynamic method dispatch**

```java
class Parent {
    void greet() { System.out.println("Hello from Parent"); }
}

class Child extends Parent {
    @Override
    void greet() { System.out.println("Hello from Child"); }  // overriding
}

Parent p1 = new Child();
p1.greet();   // "Hello from Child" — runtime object decides, not reference type
```

> 💡 Always use `@Override` annotation (Java 5+). It makes the compiler verify you are actually overriding and not accidentally overloading.

### 4.1 Rules for Overriding

| Rule | Detail |
|---|---|
| Method name | Must be same |
| Arguments | Must be same (including order) |
| Return type | Same (or co-variant from Java 5+) |
| Access modifier | Cannot be **reduced** (can be widened) |
| `private` methods | Cannot be overridden (not visible to child) |
| `final` methods | Cannot be overridden |
| `static` methods | Cannot be overridden (method hiding instead) |
| `abstract` methods | **Must** be overridden in non-abstract child |
| `synchronized`, `strictfp` | No restrictions on overriding |

**Access modifier widening rules:**

```
private < default < protected < public

Parent: public  → Child must be: public            ✅
Parent: protected → Child can be: protected or public ✅
Parent: default → Child can be: default, protected, or public ✅
Parent: private → Overriding not applicable ❌
```

**Modifier override summary table:**

| Parent modifier | Can child change to | Allowed? |
|---|---|---|
| `final` | `nonfinal` | ❌ |
| `nonfinal` | `final` | ✅ |
| `native` | `nonnative` | ✅ |
| `abstract` | `nonabstract` | ✅ |
| `nonabstract` | `abstract` | ✅ |
| `synchronized` | `nonsynchronized` | ✅ |

### 4.2 Co-variant Return Types

> From **Java 5** onwards, the child's overriding method can return a **subtype** of the parent's return type.

```
Parent return type    Child return type     Valid?
─────────────────    ─────────────────     ──────
Object               String                ✅  (String is child of Object)
Number               Integer               ✅  (Integer is child of Number)
String               Object                ❌  (Object is parent of String)
double               int                   ❌  (only for object types, not primitives)
```

```java
class Parent {
    public Object getData() { return null; }
}
class Child extends Parent {
    @Override
    public String getData() { return "hello"; }  // ✅ String is child of Object
}
```

### 4.3 Overriding with respect to Static Methods

```
Parent static  →  Child non-static  → ❌ Compile Error
Parent non-static → Child static    → ❌ Compile Error
Parent static  →  Child static      → ✅ Valid, but it's METHOD HIDING, not overriding
```

```java
class Parent {
    static void show() { System.out.println("parent"); }
}
class Child extends Parent {
    static void show() { System.out.println("child"); }  // method hiding
}

Parent p = new Child();
p.show();   // "parent" — resolved at compile time by reference type (not runtime object)
```

### 4.4 Overriding with respect to Var-arg Methods

A **var-arg method can only be overridden by a var-arg method**. If you override with a normal method, it becomes **overloading**, not overriding.

```java
class Parent {
    void show(int... i) { System.out.println("parent"); }
}
class Child extends Parent {
    void show(int i) { System.out.println("child"); }  // overloading, NOT overriding
}

Parent p1 = new Child();
p1.show(10);  // "parent" — still calls parent because it's overloading, not overriding
```

### 4.5 Overriding with respect to Variables

> **Overriding is NOT applicable for variables.** Variable resolution is always done by the **compiler based on reference type**.

```java
class Parent { int x = 888; }
class Child extends Parent { int x = 999; }

Parent p = new Child();
System.out.println(p.x);  // 888 — reference type is Parent, so Parent's x

Child c = new Child();
System.out.println(c.x);  // 999 — reference type is Child
```

> This behavior is the same whether `x` is static or non-static.

---

## 5. Overloading vs Overriding — Comparison

| Property | Overloading | Overriding |
|---|---|---|
| Method names | Same | Same |
| Arguments | Must be **different** | Must be **same** (including order) |
| Method signature | Must be different | Must be same |
| Return types | No restriction | Same / co-variant (Java 5+) |
| `private`, `static`, `final` | Can be overloaded | Cannot be overridden |
| Access modifiers | No restriction | Cannot reduce scope |
| `throws` clause | No restriction | Child can't add new checked exceptions |
| Method resolution | Compiler (reference type) | JVM (runtime object) |
| Also known as | Compile-time / Static / Early binding | Runtime / Dynamic / Late binding |

---

## 6. Method Hiding

> When **both parent and child** methods are `static`, it looks like overriding but it is actually **method hiding**.

All rules of method hiding are the same as overriding **except**:

| | Overriding | Method Hiding |
|---|---|---|
| Methods must be | Non-static | Static |
| Resolution by | JVM (runtime object) | Compiler (reference type) |
| Also known as | Runtime polymorphism | Compile-time polymorphism |

```java
class Parent {
    static void show() { System.out.println("parent"); }
}
class Child extends Parent {
    static void show() { System.out.println("child"); }
}

Parent p = new Child();
p.show();   // "parent" — reference type decides (method hiding)

// If both were non-static → overriding → output would be "child"
```

---

## 7. Checked vs Unchecked Exceptions

**Checked exceptions** — checked by the compiler at compile time for smooth runtime.

**Unchecked exceptions** — not checked by the compiler.

```
Object
  └── Throwable
        ├── Exception
        │     ├── RuntimeException  ← Unchecked
        │     │     ├── ArithmeticException
        │     │     ├── NullPointerException
        │     │     ├── ArrayIndexOutOfBoundsException
        │     │     ├── ClassCastException
        │     │     └── IllegalArgumentException
        │     │           └── NumberFormatException
        │     ├── IOException       ← Checked
        │     │     ├── EOFException
        │     │     └── FileNotFoundException
        │     └── SQLException      ← Checked
        └── Error                   ← Unchecked
              ├── StackOverflowError
              ├── OutOfMemoryError
              └── VirtualMachineError
```

> **Rule:** `RuntimeException` + its children + `Error` + its children = **Unchecked**. Everything else = **Checked**.

### Exception rule in Overriding

> If the child class method throws a **checked exception**, the parent method **must** throw the same checked exception or its parent.

```
Parent: void m() throws IOException           → Child: void m()                           ✅ valid
Parent: void m()                              → Child: void m() throws IOException         ❌ invalid
Parent: void m() throws IOException           → Child: void m() throws IOException         ✅ valid
Parent: void m() throws IOException           → Child: void m() throws Exception           ❌ invalid (broader)
Parent: void m() throws IOException           → Child: void m() throws EOFException        ✅ valid (narrower)
Parent: void m() throws IOException           → Child: void m() throws EOFException, FileNotFoundException ✅ valid
Parent: void m() throws IOException           → Child: void m() throws EOFException, InterruptedException ❌ invalid
```

> **No restrictions for unchecked exceptions** in overriding.

---

## 8. Ways to Create an Object

There are **5 ways** to get/create an object in Java:

```java
// 1. new operator (most common)
Test t = new Test();

// 2. newInstance() — Reflection (deprecated in Java 9, removed in Java 17)
Test t = (Test) Class.forName("Test").newInstance();
// Modern way (Java 9+):
Test t = Test.class.getDeclaredConstructor().newInstance();

// 3. clone()
Test t1 = new Test();
Test t2 = (Test) t1.clone();

// 4. Factory methods
Runtime r = Runtime.getRuntime();
DateFormat df = DateFormat.getInstance();

// 5. Deserialization
FileInputStream fis = new FileInputStream("abc.ser");
ObjectInputStream ois = new ObjectInputStream(fis);
Test t = (Test) ois.readObject();
```

> 💡 **Java 14+** introduced `record` types — objects created with `new` but immutable by default.

---

## 9. Constructors

> **Purpose: Initialize an object.** When an object is created, a piece of code runs automatically to set it up — that's the constructor.

```java
class Student {
    String name;
    int rollno;

    Student(String name, int rollno) {   // constructor
        this.name = name;
        this.rollno = rollno;
    }
}

Student s1 = new Student("Ravi", 101);
Student s2 = new Student("Priya", 102);
```

```
s1 ──> [ name="Ravi",  rollno=101 ]
s2 ──> [ name="Priya", rollno=102 ]
```

### 9.1 Rules for Constructors

- Constructor name must **match the class name**.
- **No return type** — not even `void`. If you write a return type, it becomes a method, not a constructor.
- Valid modifiers: `public`, `protected`, `default`, `private` only.
- `static`, `final`, `abstract`, `synchronized` are **not** allowed.

```java
class Test {
    void Test() { }   // ⚠️ This is a METHOD named Test, NOT a constructor
    static Test() { } // ❌ Compile Error: modifier static not allowed here
}
```

### 9.2 Default Constructor

- If you write **no constructor**, the compiler generates a default one.
- If you write **even one constructor**, the compiler does **not** generate a default one.
- The default constructor is always a **no-argument constructor**.
- Its access modifier matches the class modifier (only `public` or `default`).
- It contains only one line: `super();`

```java
// What you write:         What compiler generates:
class Test { }            class Test {
                              Test() { super(); }
                          }

public class Test { }     public class Test {
                              public Test() { super(); }
                          }

class Test {              class Test {
    Test(int i) { }           Test(int i) { super(); }
}                         }
// No default constructor generated — you wrote one already!
// new Test(); would now FAIL
```

> ✅ **Best practice:** Whenever you write an argument constructor, also write a no-arg constructor.

### 9.3 super() vs this()

Both must appear as the **first line** of any constructor.

```
super() ──────> calls the superclass constructor
this()  ──────> calls another constructor in the same class

Rules:
  1. Only in the FIRST line of a constructor
  2. Only ONE of them — not both
  3. Only INSIDE a constructor (never inside a method)
```

```java
// ❌ super() not on first line
Test() {
    System.out.println("hello");
    super();   // Compile Error: call to super must be first statement
}

// ❌ both super() and this()
Test() {
    super();
    this();    // Compile Error
}

// ❌ super() inside a method
void methodOne() {
    super();   // Compile Error
}
```

**super() vs this keyword:**

| | `super()` / `this()` | `super` / `this` keyword |
|---|---|---|
| What it is | Constructor calls | Keywords |
| Used for | Invoke constructors | Access parent/current instance members |
| Where allowed | First line of constructor only | Anywhere in instance area (not static) |

### 9.4 Overloaded Constructors

A class can have **multiple constructors with different arguments** — these are overloaded constructors.

```java
class Box {
    double width, height, depth;

    Box()                          { width = height = depth = 0; }
    Box(double side)               { width = height = depth = side; }
    Box(double w, double h, double d) { width = w; height = h; depth = d; }
}

Box b1 = new Box();           // no-arg constructor
Box b2 = new Box(5);          // single-arg constructor
Box b3 = new Box(3, 4, 5);    // three-arg constructor
```

**Important notes:**
- Parent class constructors are **not inherited** by child classes.
- Constructors **cannot be overridden** — only overloaded.
- Constructors are allowed in abstract classes but **not in interfaces**.

```
class Test       → ✅ constructor allowed
abstract class   → ✅ constructor allowed (called during child object creation)
interface        → ❌ constructor NOT allowed
```

**Why does an abstract class need a constructor?**
When a child class object is created, the parent (abstract) class constructor is called to initialize the part of the child object inherited from the parent.

```java
abstract class Animal {
    String type;
    Animal(String type) { this.type = type; }
}
class Dog extends Animal {
    Dog() { super("mammal"); }
}
```

### 9.5 Constructor vs Instance Block

```java
class Test {
    static int count = 0;

    {
        count++;   // instance block — runs before constructor, for every object
    }

    Test() { }      // constructor — runs after instance block
    Test(int i) { }
}
```

| | Instance Block `{ }` | Constructor |
|---|---|---|
| Runs | Before constructor, for every object | After instance block |
| Takes arguments | No | Yes |
| Purpose | Activity common to all constructors | Object initialization |
| Can replace the other | No | No |

**Compiler's 3 responsibilities:**
1. If no constructor written → generate default constructor.
2. If constructor doesn't start with `super()` or `this()` → add `super()`.
3. If recursive constructor invocation detected → Compile Error.

```java
// Recursive constructor → Compile Error
class Test {
    Test(int i) { this(); }   // calls no-arg
    Test()      { this(10); } // calls int-arg → loop → CE
}

// Recursive method → Runtime Error (StackOverflowError)
void methodOne() { methodOne(); }  // RE at runtime
```

---

## 🔁 Quick Summary — Part 2

```
┌─────────────────────┬────────────────────────────────────────────────┐
│ Concept             │ Key Point                                       │
├─────────────────────┼────────────────────────────────────────────────┤
│ Method Signature    │ name + arg types (return type NOT included)    │
│ Overloading         │ Same name, different args — compile-time       │
│ Overriding          │ Same signature — runtime (JVM decides)         │
│ Method Hiding       │ Like overriding but for static methods         │
│ Co-variant return   │ Child can return subtype (Java 5+)             │
│ Constructors        │ Same name as class, no return type             │
│ Default constructor │ Generated only if no constructor written       │
│ super() / this()    │ Must be 1st line, only one, only in constructor│
└─────────────────────┴────────────────────────────────────────────────┘
```

### Java Version Quick Ref — Part 2

| Feature | Introduced In |
|---|---|
| Var-args | Java 5 |
| Co-variant return types | Java 5 |
| `@Override` annotation | Java 5 |
| `newInstance()` deprecated | Java 9 |
| `getDeclaredConstructor().newInstance()` | Java 9+ |
| Records (constructor auto-generated) | Java 16 |
| Sealed classes (overriding control) | Java 17 |
| Pattern matching in switch | Java 21 |
