# 🔌 Java Interfaces — Complete Notes

> **Java Versions covered:** Java 8, 9, 11, 17, 21+

---

## 📑 Index

1. [What is an Interface?](#1-what-is-an-interface)
2. [Declaring and Implementing an Interface](#2-declaring-and-implementing-an-interface)
3. [extends vs implements](#3-extends-vs-implements)
4. [Interface Methods](#4-interface-methods)
5. [Interface Variables](#5-interface-variables)
6. [Naming Conflicts](#6-naming-conflicts)
   - [Method Naming Conflicts](#61-method-naming-conflicts)
   - [Variable Naming Conflicts](#62-variable-naming-conflicts)
7. [Marker Interface](#7-marker-interface)
8. [Adapter Class](#8-adapter-class)
9. [Interface vs Abstract Class vs Concrete Class](#9-interface-vs-abstract-class-vs-concrete-class)
10. [Modern Java Interface Features (Java 8+)](#10-modern-java-interface-features-java-8)
11. [Quick Reference](#11-quick-reference)

---

## 1. What is an Interface?

An interface can be understood from **3 angles**:

**Definition 1 — Service Requirement Specification (SRS)**
> The client defines what services are needed. The service provider delivers implementation.

Real-world example: Sun defined the **JDBC API** (interface). Database vendors like Oracle, MySQL provide the actual driver (implementation).

```
Sun (defines spec)
      │
   JDBC API  ◄─── interface
    /   |   \
Oracle  DB2  MySQL  ◄─── implementations
driver driver driver
```

**Definition 2 — Contract between client and service provider**
> ATM screen = contract between the bank (provider) and the customer (client).
> The screen shows what services are available. The bank implements them.

**Definition 3 — 100% Pure Abstract Class**
> Every method inside an interface is implicitly `public abstract`.
> It defines *what* to do, not *how* to do it.

**Summary:** An interface = SRS + contract + 100% pure abstract class.

---

## 2. Declaring and Implementing an Interface

```java
// Declaring an interface
interface Drawable {
    void draw();       // implicitly public abstract
    void resize();     // implicitly public abstract
}
```

```java
// Implementing the interface
class Circle implements Drawable {
    public void draw() {
        System.out.println("Drawing circle");
    }
    public void resize() {
        System.out.println("Resizing circle");
    }
}
```

> 💡 When implementing an interface, ALL methods must be declared `public` — otherwise compile error.

**What if you don't implement all methods?**
Declare the class as `abstract` — the next concrete child must implement the rest.

```java
abstract class PartialImpl implements Drawable {
    public void draw() {
        System.out.println("Drawing");
    }
    // resize() not implemented here → child must implement it
}

class Square extends PartialImpl {
    public void resize() {
        System.out.println("Resizing square");
    }
}
```

---

## 3. extends vs implements

| Scenario | Keyword | Rule |
|---|---|---|
| Class inherits a class | `extends` | only ONE class at a time |
| Class implements an interface | `implements` | any NUMBER of interfaces |
| Class extends + implements | both | `extends` must come BEFORE `implements` |
| Interface extends an interface | `extends` | any NUMBER of interfaces |

```java
// ✅ A class can implement multiple interfaces
interface Flyable { void fly(); }
interface Swimmable { void swim(); }

class Duck implements Flyable, Swimmable {
    public void fly()  { System.out.println("Duck flying"); }
    public void swim() { System.out.println("Duck swimming"); }
}
```

```java
// ✅ Class extends class AND implements interface
class Animal { }
interface Talkable { void talk(); }

class Human extends Animal implements Talkable {
    public void talk() { System.out.println("Hello!"); }
}
```

```java
// ✅ Interface can extend multiple interfaces
interface A { void m1(); }
interface B { void m2(); }
interface C extends A, B { }    // C inherits both m1() and m2()
```

```java
// ❌ Wrong order — implements before extends
class Three implements One extends Two { }  // Compile error!
// ✅ Correct order
class Three extends Two implements One { }
```

**Quick rules for `X extends Y` and `X implements Y`:**

```
X extends Y, Z      → X, Y, Z must all be interfaces
X extends Y         → both classes OR both interfaces
X implements Y, Z   → X is class; Y, Z are interfaces
X extends Y implements Z → X, Y are classes; Z is interface
```

---

## 4. Interface Methods

Every interface method is **implicitly** `public abstract` — whether you write it or not.

```java
interface Payment {
    void pay(double amount);                  // same as:
    public void pay(double amount);           // same as:
    abstract void pay(double amount);         // same as:
    public abstract void pay(double amount);  // all EQUAL
}
```

**Why public?** So every implementation class can access it.  
**Why abstract?** The implementation class is responsible for providing the body.

### ❌ Modifiers NOT allowed on interface methods (pre-Java 8):

`private`, `protected`, `final`, `static`, `synchronized`, `native`, `strictfp`

### ✅ Modern Java — New Method Types in Interfaces

**Java 8** introduced two new method types:

```java
interface Greeting {

    // 1. default method — has a body, inherited by impl classes
    default void greet() {
        System.out.println("Hello!");
    }

    // 2. static method — called on interface name directly
    static void info() {
        System.out.println("Greeting interface");
    }
}

class FormalGreeting implements Greeting {
    // greet() is inherited — no need to override (but can)
}

// Usage:
FormalGreeting f = new FormalGreeting();
f.greet();             // Output: Hello!
Greeting.info();       // Output: Greeting interface
```

**Java 9** added:

```java
interface Helper {
    // private method — helper for default/static methods only
    private void log(String msg) {
        System.out.println("Log: " + msg);
    }

    default void doWork() {
        log("Working...");      // reusing private helper
    }
}
```

---

## 5. Interface Variables

Every interface variable is implicitly `public static final` — whether you write it or not.

```java
interface Config {
    int MAX_USERS = 100;                      // same as:
    public static final int MAX_USERS = 100;  // EQUAL
}
```

**Why public?** Available to all implementation classes.  
**Why static?** Accessible without creating an object.  
**Why final?** Implementation class can read but NOT modify it.

```java
class App implements Config {
    public static void main(String[] args) {
        System.out.println(MAX_USERS);     // ✅ Output: 100
        // MAX_USERS = 200;               // ❌ Cannot assign to final variable
    }
}
```

**Must initialize at declaration:**
```java
interface Bad {
    int x;       // ❌ Compile error: = expected (must initialize)
}

interface Good {
    int x = 10;  // ✅
}
```

**Modifiers NOT allowed on interface variables:**
`private`, `protected`, `transient`, `volatile`

---

## 6. Naming Conflicts

### 6.1 Method Naming Conflicts

**Case 1: Same name, same signature, same return type** → Only ONE implementation needed ✅

```java
interface Left  { void show(); }
interface Right { void show(); }

class Test implements Left, Right {
    public void show() {            // handles both
        System.out.println("show");
    }
}
```

**Case 2: Same name, different arguments** → Acts as overloading, implement both ✅

```java
interface Left  { void show(); }
interface Right { void show(int x); }

class Test implements Left, Right {
    public void show()       { System.out.println("no args"); }
    public void show(int x)  { System.out.println("with: " + x); }
}
```

**Case 3: Same name + same signature, different return types** → IMPOSSIBLE to implement both ❌

```java
interface Left  { void show(); }
interface Right { int show(); }     // same signature, different return type

// class Test implements Left, Right { }  // ❌ Impossible — compile error
```

> 💡 A class can implement any number of interfaces **except** when two interfaces have a method with the same signature but different return types.

---

### 6.2 Variable Naming Conflicts

Two interfaces can both have a variable with the same name. Resolve using the **interface name**:

```java
interface Left  { int x = 888; }
interface Right { int x = 999; }

class Test implements Left, Right {
    public static void main(String[] args) {
        // System.out.println(x);       // ❌ Ambiguous
        System.out.println(Left.x);     // ✅ Output: 888
        System.out.println(Right.x);    // ✅ Output: 999
    }
}
```

---

## 7. Marker Interface

An interface with **no methods and no variables** is called a **Marker Interface** (also: Tag Interface / Ability Interface).

By implementing such an interface, objects gain a special ability — provided internally by the JVM.

```java
// These are all built-in marker interfaces:
Serializable      // ability to save/send objects over network
Cloneable         // ability to clone objects
RandomAccess      // ability for fast random access in lists
```

```java
import java.io.*;

class Student implements Serializable {    // marker interface
    String name;
    int rollno;
}
// Now Student objects can be written to files or sent over network
```

**Why no methods?** JVM internally handles the ability. This reduces code complexity.

**Can you create custom marker interfaces?** Yes, but JVM customization is required to give them special meaning.

---

## 8. Adapter Class

An **Adapter Class** is a class that implements an interface with **empty bodies for all methods**.

**Problem without adapter:**
```java
interface WindowListener {
    void windowOpened();
    void windowClosed();
    void windowMinimized();
    void windowMaximized();
    // ... 7 methods total
}

// If I only care about windowClosed(), I still must implement ALL 7:
class MyListener implements WindowListener {
    public void windowOpened()    {}    // don't need this
    public void windowClosed()    { System.out.println("Closed!"); }
    public void windowMinimized() {}    // don't need this
    public void windowMaximized() {}    // don't need this
    // ... forced to write 6 useless methods
}
```

**Solution — Adapter Class:**
```java
// Adapter provides empty implementation for ALL methods
abstract class WindowAdapter implements WindowListener {
    public void windowOpened()    {}
    public void windowClosed()    {}
    public void windowMinimized() {}
    public void windowMaximized() {}
}

// Now extend adapter and override ONLY what you need:
class MyListener extends WindowAdapter {
    public void windowClosed() {
        System.out.println("Closed!");   // only override what matters
    }
}
```

> ✅ Adapter class reduces code length and improves readability.

**Real Java example:** `GenericServlet` is the adapter class for the `Servlet` interface.

```
Servlet (Interface)
      ↓
GenericServlet (Abstract Class — Adapter)
      ↓
HttpServlet (Abstract Class)
      ↓
YourServlet (Concrete Class — extends HttpServlet)
```

---

## 9. Interface vs Abstract Class vs Concrete Class

### When to use what:

| Situation | Use |
|---|---|
| Only requirement spec, no implementation idea | **Interface** |
| Partial implementation (some methods done) | **Abstract Class** |
| Full implementation, ready to use | **Concrete Class** |

### Interface vs Abstract Class — Detailed Comparison:

| Feature | Interface | Abstract Class |
|---|---|---|
| Methods | Always `public abstract` (pre-Java 8) | Can be any modifier |
| Variables | Always `public static final` | No restrictions |
| Constructor | ❌ Not allowed | ✅ Allowed |
| Static block | ❌ Not allowed | ✅ Allowed |
| Instance block | ❌ Not allowed | ✅ Allowed |
| Variable init | Must init at declaration | Can init later |
| Extending other class | ❌ Cannot | ✅ Can extend one class |
| Performance | Faster (object creation ~2 sec) | Slower (~20 sec) |

```java
// Interface — everything abstract
interface Shape {
    double area();
    double perimeter();
}

// Abstract class — partial implementation
abstract class RegularShape implements Shape {
    String color;
    RegularShape(String color) { this.color = color; }

    public String getColor() { return color; }  // concrete method
    // area() and perimeter() still abstract
}

// Concrete class — full implementation
class Circle extends RegularShape {
    double radius;
    Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }
    public double area()      { return Math.PI * radius * radius; }
    public double perimeter() { return 2 * Math.PI * radius; }
}
```

### Why not just use Abstract Class instead of Interface?

```
abstract class X { ... }
class Test extends X { }   ← Test can't extend any other class now

interface X { ... }
class Test implements X { }  ← Test is still free to extend another class
```

> 💡 If everything is abstract, **prefer interface** — better performance + keeps inheritance open.

### Why does Abstract Class have a constructor when you can't instantiate it?

The constructor initializes **instance variables** for child objects.  
Interface has no instance variables (all are `static final`), so no constructor is needed.

```java
abstract class Animal {
    String name;
    Animal(String name) { this.name = name; }  // initializes child's 'name'
}

class Dog extends Animal {
    Dog(String name) { super(name); }   // calls Animal constructor
    void bark() { System.out.println(name + " says Woof!"); }
}

Dog d = new Dog("Rex");
d.bark();   // Output: Rex says Woof!
```

---

## 10. Modern Java Interface Features (Java 8+)

| Feature | Java Version | Description |
|---|---|---|
| `default` methods | Java 8 | Methods with a body in interface |
| `static` methods | Java 8 | Called on interface name, not inherited |
| Functional Interface | Java 8 | Interface with exactly one abstract method (used with lambdas) |
| `private` methods | Java 9 | Helper methods inside interface |
| `private static` methods | Java 9 | Static helper methods |

```java
// Functional Interface (Java 8)
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);   // exactly one abstract method
}

// Using lambda instead of anonymous class:
Calculator add = (a, b) -> a + b;
Calculator mul = (a, b) -> a * b;

System.out.println(add.calculate(3, 4));  // Output: 7
System.out.println(mul.calculate(3, 4));  // Output: 12
```

> 📌 Built-in functional interfaces: `Runnable`, `Callable`, `Comparator`, `Predicate`, `Function`, `Consumer`, `Supplier` — all in `java.util.function` package.

---

## 11. Quick Reference

```
Interface rules:
✅ Methods       → implicitly public abstract (pre-Java 8)
✅ Variables     → implicitly public static final
✅ Must init     → variables must be initialized at declaration
✅ Implements    → class can implement any number of interfaces
✅ Extends       → interface can extend multiple interfaces
❌ Constructor   → not allowed
❌ Static block  → not allowed
❌ Instance var  → not allowed (all vars are static)

extends vs implements order:
class X extends Y implements Z { }   ✅ (extends FIRST)
class X implements Z extends Y { }   ❌ (WRONG order)

Conflict resolution:
Same name + same sig + same return  → one impl is enough
Same name + different args          → overloading, implement both
Same name + same sig + diff return  → IMPOSSIBLE to implement both
Same variable name                  → use InterfaceName.variable

Choose:
  No implementation idea    → Interface
  Partial implementation    → Abstract Class
  Full implementation       → Concrete Class
```

---

> 📘 **References:** [Java SE Documentation](https://docs.oracle.com/en/java/index.html)  
> Covers Java 8 through Java 21+
