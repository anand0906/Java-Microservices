# ☕ Java Modifiers — Complete Notes

> **Java Versions covered:** Java 8, 11, 17, 21+  
> Based on Core Java (OCJP/SCJP concepts)

---

## 📑 Index
1. [What are Modifiers?](#2-what-are-modifiers)
3. [Class Modifiers](#3-class-modifiers)
   - [public](#31-public-class)
   - [default](#32-default-class)
   - [final](#33-final-class)
   - [abstract](#34-abstract-class)
   - [strictfp](#35-strictfp-class)
4. [Member Modifiers](#4-member-modifiers)
   - [public](#41-public-member)
   - [default](#42-default-member)
   - [private](#43-private-member)
   - [protected](#44-protected-member)
5. [Access Modifier Comparison Table](#5-access-modifier-comparison-table)
6. [Variable Modifiers](#6-variable-modifiers)
   - [final instance variables](#61-final-instance-variables)
   - [final static variables](#62-final-static-variables)
   - [final local variables](#63-final-local-variables)
7. [Method Modifiers](#7-method-modifiers)
   - [static](#71-static-modifier)
   - [abstract](#72-abstract-methods)
   - [final](#73-final-methods)
   - [native](#74-native-modifier)
   - [synchronized](#75-synchronized-modifier)
   - [strictfp](#76-strictfp-method)
8. [Other Modifiers](#8-other-modifiers)
   - [transient](#81-transient-modifier)
   - [volatile](#82-volatile-modifier)
9. [Illegal Combinations](#9-illegal-combinations)
10. [Modifier Summary Table](#10-modifier-summary-table)

---

## 1. What are Modifiers?

Modifiers are **keywords** that tell the JVM about the properties of a class, method, or variable — such as:
- Who can access it?
- Can it be extended/overridden?
- Is it shared or per-object?

In Java, **all** keywords (public, private, protected, final, abstract, etc.) are called **access modifiers**. Unlike C/C++, Java does not distinguish between "access specifiers" and "access modifiers."

---

## 3. Class Modifiers

### 3.1 `public` Class

A `public` class is accessible from **anywhere** — same package or different package.

```java
// File: pack1/Animal.java
package pack1;

public class Animal {
    public void speak() {
        System.out.println("Animal speaks");
    }
}

// File: pack2/Main.java
package pack2;
import pack1.Animal;

class Main {
    public static void main(String[] args) {
        Animal a = new Animal();  // ✅ Works — Animal is public
        a.speak();
    }
}
// Output: Animal speaks
```

> ⚠️ If `Animal` were not `public`, the import in `pack2` would cause a compile-time error.

---

### 3.2 `default` Class

No keyword = default access. Accessible **only within the same package**.  
Also called **"package-level access"**.

```java
// File: pack1/Dog.java
package pack1;

class Dog {                       // default — no modifier written
    void bark() {
        System.out.println("Woof!");
    }
}

// File: pack2/Test.java
package pack2;
import pack1.Dog;                 // ❌ Compile error!

class Test {
    public static void main(String[] args) {
        Dog d = new Dog();        // ❌ Dog is not visible outside pack1
    }
}
```

---

### 3.3 `final` Class

A `final` class **cannot be extended** (no child class allowed).

```java
final class Vehicle { }

class Car extends Vehicle { }   // ❌ Compile error: cannot inherit from final Vehicle
```

**Key rules:**
- Every method inside a `final` class is **implicitly final**.
- Variables inside a `final` class are **NOT** automatically final.
- ✅ Advantage: Security
- ❌ Disadvantage: Breaks inheritance (polymorphism lost)

> 📌 **Real-world example:** `java.lang.String` is a `final` class in Java.

---

### 3.4 `abstract` Class

An `abstract` class **cannot be instantiated** (no objects allowed).

```java
abstract class Shape {
    abstract double area();      // child must implement this
}

class Circle extends Shape {
    double radius;
    Circle(double r) { this.radius = r; }

    @Override
    double area() {
        return Math.PI * radius * radius;  // ✅ providing implementation
    }
}

// Shape s = new Shape();        // ❌ Compile error: Shape is abstract
Shape s = new Circle(5);         // ✅ Reference of parent, object of child
System.out.println(s.area());    // Output: 78.53...
```

**Key rules:**
- Abstract class can have **zero abstract methods** (still valid).
- If a class has even **one abstract method**, the class **must** be abstract.
- If a child class doesn't implement all abstract methods, it must also be declared `abstract`.

---

### 3.5 `strictfp` Class

Forces all floating-point calculations inside the class to follow **IEEE 754** standard — ensures consistent results across all platforms.

```java
strictfp class Calculator {
    double divide(double a, double b) {
        return a / b;   // follows IEEE 754 on all platforms
    }
}
```

> 📌 Introduced in **Java 1.2**. Made redundant in **Java 17** (JEP 306) — IEEE 754 is now always used by default. `strictfp` is still valid but has no effect in Java 17+.

---

## 4. Member Modifiers

### 4.1 `public` Member

Accessible from **anywhere**, but **the class must also be visible** first.

```java
// pack1/Box.java
package pack1;

class Box {                        // ← class is default (not public)
    public void open() {
        System.out.println("Box opened");
    }
}

// pack2/Test.java
package pack2;
import pack1.Box;                  // ❌ Error: Box is not public
```

> 💡 Both the **class** and the **method** must be public to access from outside the package.

---

### 4.2 `default` Member

Accessible **only within the same package**.

```java
// pack1/Cat.java
package pack1;

class Cat {
    void meow() {                  // default access
        System.out.println("Meow!");
    }
}

// pack1/Test.java (same package ✅)
package pack1;
class Test {
    public static void main(String[] args) {
        Cat c = new Cat();
        c.meow();                  // ✅ Works — same package
    }
}
```

---

### 4.3 `private` Member

Accessible **only within the same class**.

```java
class BankAccount {
    private double balance = 1000;   // hidden from outside

    public double getBalance() {     // public getter — controlled access
        return balance;
    }
}

class Test {
    public static void main(String[] args) {
        BankAccount acc = new BankAccount();
        // System.out.println(acc.balance);   // ❌ Compile error
        System.out.println(acc.getBalance()); // ✅ Output: 1000.0
    }
}
```

> 💡 `private` + `abstract` is **illegal** — abstract methods must be visible to child classes.

---

### 4.4 `protected` Member

Accessible:
- ✅ Anywhere within the **same package**
- ✅ In **child classes** of outside packages (but only via child reference)

```
protected  =  default  +  child classes (outside package)
```

```java
// pack1/Animal.java
package pack1;
public class Animal {
    protected void eat() {
        System.out.println("Eating...");
    }
}

// pack2/Dog.java
package pack2;
import pack1.Animal;

class Dog extends Animal {
    public static void main(String[] args) {
        Dog d = new Dog();
        d.eat();               // ✅ child reference — works

        Animal a = new Animal();
        a.eat();               // ❌ parent reference from outside package — error!
    }
}
```

---

## 5. Access Modifier Comparison Table

| Visibility | `private` | `default` | `protected` | `public` |
|---|:---:|:---:|:---:|:---:|
| Same class | ✅ | ✅ | ✅ | ✅ |
| Child class (same package) | ❌ | ✅ | ✅ | ✅ |
| Non-child class (same package) | ❌ | ✅ | ✅ | ✅ |
| Child class (outside package) | ❌ | ❌ | ✅ (child ref only) | ✅ |
| Non-child class (outside package) | ❌ | ❌ | ❌ | ✅ |

```
Least accessible ◄─────────────────────────► Most accessible
    private    <    default    <    protected    <    public
```

> 💡 Recommended: use `private` for variables, `public` for methods.

---

## 6. Variable Modifiers

### 6.1 Final Instance Variables

Instance variables = different value per object.  
If declared `final`, JVM **won't provide default values** — you must initialize manually.

**Valid initialization points (must be done before constructor completes):**

```java
// ✅ Option 1: At declaration
class Student {
    final int id = 101;
}

// ✅ Option 2: Inside instance initializer block
class Student {
    final int id;
    { id = 101; }              // instance block runs before constructor
}

// ✅ Option 3: Inside constructor
class Student {
    final int id;
    Student() { id = 101; }
}

// ❌ Invalid: Inside a method
class Student {
    final int id;
    void setId() { id = 101; } // Compile error: cannot assign to final variable
}
```

---

### 6.2 Final Static Variables

Static variables = one copy shared by all objects.  
If declared `final`, must be initialized before **class loading completes**.

```java
// ✅ Option 1: At declaration
class Config {
    final static int MAX = 100;
}

// ✅ Option 2: Inside static block
class Config {
    final static int MAX;
    static { MAX = 100; }

// ❌ Inside constructor or methods — compile error
```

---

### 6.3 Final Local Variables

Local variables are declared inside methods/blocks.  
JVM **never** provides default values for local variables (final or not).

```java
class Demo {
    public static void main(String[] args) {
        final int x;           // declared but not yet initialized — OK
        x = 50;                // ✅ initialized before use
        System.out.println(x); // Output: 50

        // x = 60;             // ❌ Cannot re-assign a final variable
    }
}
```

> 💡 `final` is the **only** modifier allowed for local variables. Using `private`, `public`, `static`, etc. on local variables causes a compile error.

---

## 7. Method Modifiers

### 7.1 `static` Modifier

- Belongs to the **class**, not to any object.
- Can be called without creating an object.
- Static methods **cannot access** instance (non-static) variables directly.

```java
class MathUtil {
    static int square(int n) {     // static method
        return n * n;
    }
}

// No object needed:
System.out.println(MathUtil.square(5)); // Output: 25
```

**Method Hiding (not overriding):**
```java
class Parent {
    static void show() { System.out.println("Parent"); }
}
class Child extends Parent {
    static void show() { System.out.println("Child"); }
    // This is METHOD HIDING, not overriding
}
```

---

### 7.2 `abstract` Methods

- Has **only declaration, no body**.
- Must end with semicolon.
- Child class **must** implement it.

```java
abstract class Animal {
    public abstract void sound();  // ✅ valid — no body, ends with ;
    // public abstract void sound(){} // ❌ invalid — has empty body
}

class Dog extends Animal {
    public void sound() {
        System.out.println("Woof!");   // ✅ providing implementation
    }
}
```

**Illegal combinations with `abstract`:**

```
abstract + final       ❌  (final can't be overridden, abstract must be)
abstract + static      ❌  (static = implementation exists, abstract = no body)
abstract + private     ❌  (private = not visible to child, abstract = must be visible)
abstract + native      ❌
abstract + synchronized ❌
abstract + strictfp    ❌ (for methods)
```

---

### 7.3 `final` Methods

Cannot be **overridden** by child classes.

```java
class Parent {
    final void login() {
        System.out.println("Secure login — cannot be changed");
    }
}

class Child extends Parent {
    // void login() { ... }  // ❌ Compile error: cannot override final method
}
```

---

### 7.4 `native` Modifier

- Applicable **only for methods**.
- Method is implemented in **non-Java** language (C/C++).
- Declaration ends with semicolon (no body in Java).

```java
class NativeDemo {
    static {
        System.loadLibrary("mylib");   // Step 1: load native library
    }

    public native void fastCompute(); // Step 2: declare native method
}

class Client {
    public static void main(String[] args) {
        NativeDemo n = new NativeDemo();
        n.fastCompute();              // Step 3: call it
    }
}
```

**Main objectives:**
- Improve performance
- Reuse existing non-Java (legacy) code
- Machine-level memory access

> ⚠️ **Disadvantage:** Breaks Java's platform-independent nature.

---

### 7.5 `synchronized` Modifier

- Applicable for **methods and blocks** (not variables or classes).
- Only **one thread** can execute the synchronized method/block at a time on a given object.

```java
class Counter {
    private int count = 0;

    synchronized void increment() {  // only one thread at a time
        count++;
    }
}
```

> ✅ Solves data inconsistency in multithreading.  
> ❌ Increases thread waiting time — avoid unless needed.

---

### 7.6 `strictfp` Method

Forces IEEE 754 floating-point standard for that method only.

```java
class Demo {
    strictfp double calculate(double a, double b) {
        return a / b;   // platform-independent result
    }
}
```

> 📌 As of **Java 17**, IEEE 754 is the default — `strictfp` has no effect but is still valid syntax.

---

## 8. Other Modifiers

### 8.1 `transient` Modifier

- Only for **variables** (not methods or classes).
- During **serialization**, transient variable values are **not saved** — JVM saves the default value instead.

```java
import java.io.*;

class User implements Serializable {
    String username = "alice";
    transient String password = "secret123";  // won't be serialized
}
// After deserialization: username = "alice", password = null
```

**Key notes:**
- `static` variables are not part of object state — serialization doesn't apply → `transient static` is useless.
- `final` variables participate by their value directly → `transient final` has no impact.

---

### 8.2 `volatile` Modifier

- Only for **variables**.
- When multiple threads read/write a variable, `volatile` ensures the **master copy** is always used (no thread-local caching).

```java
class SharedFlag {
    volatile boolean running = true;  // all threads see latest value

    void stop() { running = false; }
}
```

> ⚠️ `final volatile` is **illegal** — `final` means value never changes, `volatile` means it keeps changing.  
> 📌 `volatile` is largely outdated — prefer `java.util.concurrent` classes in modern Java (Java 5+).

---

## 9. Illegal Combinations

### For Methods:

| Combination | Reason |
|---|---|
| `abstract` + `final` | abstract must be overridden; final prevents it |
| `abstract` + `static` | static needs body; abstract has none |
| `abstract` + `private` | private hides from child; abstract needs child |
| `abstract` + `native` | native has external body; abstract has none |
| `abstract` + `synchronized` | synchronized needs body |
| `abstract` + `strictfp` | strictfp needs body |
| `native` + `strictfp` | no guarantee old language supports IEEE 754 |
| `final volatile` | final = never changes; volatile = always changing |

### For Top-Level Classes:
Only allowed: `public`, `default`, `final`, `abstract`, `strictfp`

```java
private class Test { }    // ❌ — not allowed for top-level class
protected class Test { }  // ❌ — not allowed for top-level class
```

> Inner/nested classes can additionally use: `private`, `protected`, `static`

---

## 10. Modifier Summary Table

| Modifier | Classes | Methods | Variables | Blocks |
|---|:---:|:---:|:---:|:---:|
| `public` | ✅ | ✅ | ✅ | ❌ |
| `default` | ✅ | ✅ | ✅ | ❌ |
| `private` | ❌ (top-level) | ✅ | ✅ | ❌ |
| `protected` | ❌ (top-level) | ✅ | ✅ | ❌ |
| `final` | ✅ | ✅ | ✅ | ❌ |
| `abstract` | ✅ | ✅ | ❌ | ❌ |
| `static` | ❌ (top-level) | ✅ | ✅ | ✅ |
| `strictfp` | ✅ | ✅ | ❌ | ❌ |
| `synchronized` | ❌ | ✅ | ❌ | ✅ |
| `native` | ❌ | ✅ | ❌ | ❌ |
| `transient` | ❌ | ❌ | ✅ | ❌ |
| `volatile` | ❌ | ❌ | ✅ | ❌ |

---

## 🔖 Quick Cheat Sheet

```
Top-Level Class  →  public | default | final | abstract | strictfp
Inner Class      →  above + private | protected | static

Variable         →  private (recommended)
Method           →  public (recommended)

final class      →  cannot extend
final method     →  cannot override
final variable   →  cannot reassign

abstract class   →  cannot instantiate
abstract method  →  no body, child must implement

static           →  class-level, no object needed
transient        →  skip during serialization
volatile         →  always read from main memory (multithreading)
```

---

> 📘 **References:** Java SE Documentation — [docs.oracle.com/javase](https://docs.oracle.com/en/java/index.html)  
> Covers Java 8 through Java 21+
