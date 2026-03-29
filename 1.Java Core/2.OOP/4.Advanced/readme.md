# ☕ Java OOP 

> **Based on:** Core Java by Durga Sir (Durgasoft)
> **Updated for:** Java 8 → Java 21+ (LTS)

---

## 📑 Index

| # | Topic |
|---|-------|
| 1 | [Singleton Classes](#1-singleton-classes) |
| 2 | [Factory Methods](#2-factory-methods) |
| 3 | [Static Control Flow](#3-static-control-flow) |
| 4 | [Instance Control Flow](#4-instance-control-flow) |
| 5 | [Type Casting](#5-type-casting) |
| 6 | [Coupling](#6-coupling) |
| 7 | [Cohesion](#7-cohesion) |

---

## 1. Singleton Classes

### What is it?
A class for which **only one object can be created** throughout the entire application is called a **Singleton class**.

```
Multiple references → All point to the SAME single object

  ref1 ──┐
  ref2 ──┼──► [ Single Object ]
  ref3 ──┘
```

### Real Examples in Java

| Class | How to get instance |
|-------|-------------------|
| `Runtime` | `Runtime.getRuntime()` |
| `ActionServlet` | Internal servlet container |
| `ServiceLocator` | Design pattern |
| `BusinessDelegate` | Design pattern |

```java
Runtime r1 = Runtime.getRuntime();
Runtime r2 = Runtime.getRuntime();
Runtime r3 = Runtime.getRuntime();

System.out.println(r1 == r2); // true
System.out.println(r1 == r3); // true — all point to same object
```

### Advantages

- **Better Performance** — object created only once, reused every time
- **Better Memory Utilization** — no duplicate objects in heap
- **Shared State** — same config/resource shared across the app

---

### Creating Your Own Singleton Class

You need **3 things**:
1. `private` constructor — prevents `new ClassName()` from outside
2. `private static` variable — holds the single instance
3. `public static` factory method — returns the instance

```java
class Database {
    private static Database instance = null; // step 2

    private Database() {}                    // step 1 — prevents new Database()

    public static Database getInstance() {   // step 3 — factory method
        if (instance == null) {
            instance = new Database();
        }
        return instance;
    }
}

class Main {
    public static void main(String[] args) {
        Database d1 = Database.getInstance();
        Database d2 = Database.getInstance();
        Database d3 = Database.getInstance();

        System.out.println(d1 == d2); // true
        System.out.println(d1 == d3); // true — same hashCode for all
    }
}
```

---

### Thread-Safe Singleton (Java 5+)

The basic version above has a problem in multithreaded apps — two threads might create two objects simultaneously. Fix options:

**Option 1 — Synchronized method**
```java
class Database {
    private static Database instance = null;
    private Database() {}

    public static synchronized Database getInstance() {
        if (instance == null) {
            instance = new Database();
        }
        return instance;
    }
}
```

**Option 2 — Double-Checked Locking (Java 5+, recommended)**
```java
class Database {
    private static volatile Database instance = null; // volatile is crucial

    private Database() {}

    public static Database getInstance() {
        if (instance == null) {                       // first check (no lock)
            synchronized (Database.class) {
                if (instance == null) {               // second check (with lock)
                    instance = new Database();
                }
            }
        }
        return instance;
    }
}
```

**Option 3 — Enum Singleton (Java 5+, Best Practice)**
```java
enum AppConfig {
    INSTANCE;

    public void showConfig() {
        System.out.println("App is running");
    }
}

// Usage
AppConfig.INSTANCE.showConfig();
```

> ✅ Enum singleton is thread-safe by default and handles serialization automatically. Recommended by Joshua Bloch in *Effective Java*.

---

### Doubleton / N-ton (Multiple Fixed Instances)

If you want **exactly 2 objects** (Doubleton):

```java
class Connection {
    private static Connection c1 = null;
    private static Connection c2 = null;

    private Connection() {}

    public static Connection getInstance() {
        if (c1 == null) {
            c1 = new Connection(); return c1;
        } else if (c2 == null) {
            c2 = new Connection(); return c2;
        } else {
            return (Math.random() < 0.5) ? c1 : c2; // randomly return one
        }
    }
}
```

---

### Interview Question — Prevent Subclassing Without `final`?

**Q: How to prevent child class creation without marking the class `final`?**

**A: Make every constructor `private`.**

```java
class Parent {
    private Parent() {} // child cannot call super() — compile error
}

// class Child extends Parent {} // ❌ Compile Error
```

> **Note:** When a child object is created, the parent constructor *executes* but a separate parent object is **NOT** created. Both share the same object — `this.hashCode()` is the same in both constructors.

```java
class Parent {
    Parent() { System.out.println(this.hashCode()); } // e.g. 12345
}
class Child extends Parent {
    Child() { System.out.println(this.hashCode()); }  // same 12345
}
// Only ONE object is created — the Child object
```

---

### Constructor Quick Facts (True / False)

| Statement | Answer |
|-----------|--------|
| Constructor name and class name need not be same | ❌ False |
| Constructor can have return type void | ❌ False |
| Any modifier can be used for constructor | ✅ True |
| Compiler always generates default constructor | ❌ False (only if none defined) |
| Default constructor modifier is always `default` | ❌ False (matches class modifier) |
| First line of constructor is always `super` | ❌ False (can be `this()`) |
| Constructor overloading is not applicable | ❌ False (it is applicable) |
| Constructors can be inherited or overridden | ❌ False |
| Abstract class cannot have constructor | ❌ False (it can) |
| Interface can have constructor | ❌ False |
| Recursive constructor call is runtime exception | ❌ False (compile-time error) |
| Singleton class needs private constructor | ✅ True |

---

## 2. Factory Methods

### What is it?
When you call a method using the **class name** and it **returns an object of that same class**, it is called a **Factory Method**.

```
ClassName.methodName()  →  returns ClassName object
```

### Examples in Java

```java
// Runtime.getRuntime() — returns Runtime object
Runtime r = Runtime.getRuntime();

// DateFormat.getInstance() — returns DateFormat object
DateFormat df = DateFormat.getInstance();

// Calendar.getInstance() — returns Calendar object
Calendar cal = Calendar.getInstance();

// Java 8+ — LocalDate
LocalDate today = LocalDate.now();
LocalDate date  = LocalDate.of(2025, 3, 29);

// Java 8+ — Optional
Optional<String> opt = Optional.of("Hello");

// Java 9+ — List.of(), Map.of()
List<String> names = List.of("Alice", "Bob", "Charlie");
Map<String, Integer> scores = Map.of("Alice", 95, "Bob", 87);
```

### When to Use Factory Methods?

> If object creation needs to happen **under some constraint** (allow only one object, return cached object, validate before creating), use a factory method instead of a constructor.

```java
class Temperature {
    private double value;
    private String unit;

    private Temperature(double value, String unit) {
        this.value = value;
        this.unit = unit;
    }

    // Factory methods — clearly named, controlled creation
    public static Temperature ofCelsius(double value) {
        return new Temperature(value, "C");
    }

    public static Temperature ofFahrenheit(double value) {
        return new Temperature(value, "F");
    }
}

Temperature t1 = Temperature.ofCelsius(100);
Temperature t2 = Temperature.ofFahrenheit(212);
```

---

## 3. Static Control Flow

### What happens when a class is loaded?

When JVM loads a class, it follows this **3-step sequence automatically**:

```
Step 1 → Identify all static members (top to bottom)
Step 2 → Execute static variable assignments & static blocks (top to bottom)
Step 3 → Execute main() method
```

> Static control flow happens **only once** — at the time of class loading.

---

### Simple Example

```java
class Demo {
    static int x = 10;

    static {
        System.out.println("Static Block 1, x = " + x); // 10
    }

    public static void main(String[] args) {
        System.out.println("main(), x = " + x);          // 10
    }

    static {
        System.out.println("Static Block 2");             // after block 1
    }
}
```

**Output:**
```
Static Block 1, x = 10
Static Block 2
main(), x = 10
```

---

### RIWO State — Read Indirectly Write Only

When a static variable is **declared but not yet assigned** (still in identification phase), it is in **RIWO** state.

| State | Meaning |
|-------|---------|
| RIWO | Identified, holds default value (0/null/false), direct read NOT allowed |
| R&W | Fully initialized, can be used anywhere |

```
Static variable lifecycle:
  RIWO  →  (assignment line executes)  →  R&W
```

**Direct read in RIWO → Compile Error**
```java
class Test {
    static {
        System.out.println(x); // ❌ illegal forward reference
    }
    static int x = 10;
}
```

**Indirect read in RIWO → Allowed, but prints default value**
```java
class Test {
    static {
        show();            // indirect read via method — allowed
    }
    static void show() {
        System.out.println(x); // prints 0 (default), not 10
    }
    static int x = 10;
}
// Output: 0
```

---

### Static Control Flow in Parent → Child

When you **run a Child class**, JVM does this:

```
1. Identify static members: Parent first, then Child
2. Execute static assignments & blocks: Parent first, then Child
3. Execute Child's main() method
```

```java
class Parent {
    static int a = 10;
    static { System.out.println("Parent static block"); }

    public static void main(String[] args) {
        System.out.println("Parent main");
    }
}

class Child extends Parent {
    static int b = 20;
    static { System.out.println("Child static block"); }

    public static void main(String[] args) {
        System.out.println("Child main");
    }
}
```

**Running `java Child`:**
```
Parent static block
Child static block
Child main
```

**Running `java Parent`:**
```
Parent static block
Parent main
(Child is NOT loaded — parent loading doesn't trigger child loading)
```

> **Key Rule:** Loading Child → loads Parent automatically. Loading Parent does NOT load Child.

---

### Static Block Use Cases

Static blocks run at the time of class loading. Use them for:

**1. Loading native libraries**
```java
class MyLib {
    static {
        System.loadLibrary("myNativeLib");
    }
}
```

**2. JDBC driver registration** (happens automatically inside driver class)
```java
class OracleDriver {
    static {
        // registers this driver with DriverManager internally
    }
}
```

**3. Can we print without `main()`?**

```java
// Yes — using static block (valid ONLY up to Java 1.6)
class Test {
    static {
        System.out.println("Hello without main!");
        System.exit(0);
    }
}
// Java 1.7+ → ❌ main() is mandatory
```

**Can we print without `System.out.println()`?**
```java
public static void main(String[] args) {
    System.err.println("This also prints to console"); // ✅
}
```

---

## 4. Instance Control Flow

### What happens when an object is created?

Every time `new ClassName()` is called, JVM follows this **3-step sequence**:

```
Step 1 → Identify instance members (top to bottom)
Step 2 → Execute instance variable assignments & instance blocks (top to bottom)
Step 3 → Execute constructor
```

> Instance control flow runs **every time** a new object is created (unlike static which runs only once).

---

### Simple Example

```java
class Student {
    int marks = 90;

    {
        System.out.println("Instance Block 1, marks = " + marks);
    }

    Student() {
        System.out.println("Constructor called");
    }

    {
        System.out.println("Instance Block 2");
    }

    public static void main(String[] args) {
        Student s1 = new Student();
        System.out.println("---");
        Student s2 = new Student(); // entire flow repeats for each object
    }
}
```

**Output:**
```
Instance Block 1, marks = 90
Instance Block 2
Constructor called
---
Instance Block 1, marks = 90
Instance Block 2
Constructor called
```

---

### Instance Control Flow in Parent → Child

When a **Child object is created**:

```
1. Identify instance members: Parent first, then Child
2. Execute instance assignments & blocks of Parent
3. Execute Parent constructor
4. Execute instance assignments & blocks of Child
5. Execute Child constructor
```

```java
class Animal {
    { System.out.println("Animal instance block"); }
    Animal() { System.out.println("Animal constructor"); }
}

class Dog extends Animal {
    { System.out.println("Dog instance block"); }
    Dog() { System.out.println("Dog constructor"); }

    public static void main(String[] args) {
        new Dog();
    }
}
```

**Output:**
```
Animal instance block
Animal constructor
Dog instance block
Dog constructor
```

---

### Static vs Instance Control Flow Comparison

| Feature | Static Control Flow | Instance Control Flow |
|---------|--------------------|-----------------------|
| When runs | Once at class loading | Every time `new` is called |
| Members involved | Static variables, static blocks | Instance variables, instance blocks |
| Execution order | Top to bottom | Top to bottom |
| Inheritance | Parent then Child | Parent then Child |

---

### Cannot Access Instance Variables from Static Area

```java
class Test {
    int x = 10; // instance variable

    public static void main(String[] args) {
        System.out.println(x); // ❌ Compile Error
        // non-static variable x cannot be referenced from a static context
    }
}
```

Why? When `main()` (static) runs, no object exists yet — instance variable `x` doesn't exist in memory.

**Fix:**
```java
public static void main(String[] args) {
    Test t = new Test();
    System.out.println(t.x); // ✅ 10
}
```

**Rules:**
- From **static area** → can access only static members directly
- From **instance area** → can access both static and instance members directly
- Static members can be accessed from anywhere directly (identified at class loading)

> **Note:** Object creation is the most costly operation in Java. Avoid unnecessary object creation.

---

## 5. Type Casting

### What is it?
Assigning one type's reference to another type's variable. We are **not creating a new object** — just giving the existing object a different reference type.

```
Type Casting = Changing the reference type, NOT the object itself
```

### Syntax

```
A  b = (C) d;
│      │   │
│      │   └── object / reference being cast
│      └────── type you are casting TO
└───────────── type of new reference variable
```

### Upcasting (Child → Parent) — Always Safe

```java
Animal a = new Dog();   // implicit upcasting — no cast needed
a.eat();                // can call Animal methods
// a.bark();            // ❌ cannot call Dog-specific methods via Animal ref
```

### Downcasting (Parent → Child) — Needs Explicit Cast

```java
Animal a = new Dog();
Dog d = (Dog) a;        // explicit downcast — safe here because actual object IS a Dog
d.bark();               // now Dog methods are accessible
```

---

### Two Levels of Checking

**Level 1 — Compile Time**

Rule 1: The types of `d` (object) and `C` (cast type) must have a relationship. If no relationship → `inconvertible types` error.

```java
String s = "hello";
StringBuffer sb = (StringBuffer) s; // ❌ Compile Error — no relationship
```

Rule 2: Cast type `C` must be same as or a subtype of reference variable type `A`.

```java
Object o = new String("hi");
StringBuffer sb = (StringBuffer) o; // ✅ compiles — Object and StringBuffer related
```

```java
Object o = new String("hi");
StringBuffer sb = (String) o; // ❌ Compile Error — String is not subtype of StringBuffer
```

**Level 2 — Runtime**

The actual object at runtime must be the same type or subtype of cast type `C`, otherwise → `ClassCastException`.

```java
Object o = new String("hello");
StringBuffer sb = (StringBuffer) o;
// ❌ Runtime: ClassCastException — actual object is String, not StringBuffer
```

---

### Hierarchy Example

```
           Object
          /      \
       Base1    Base2
       /    \   /    \
   Der1   Der2 Der3  Der4
```

```java
Base1 b = new Der2();              // ✅ valid — Der2 IS-A Base1
Object o = (Base1) b;              // ✅ valid — Base1 IS-A Object
Object o1 = (Base2) o;             // ❌ invalid at runtime — object is Der2, not Base2
Base2 b2 = (Base2)(new Der3());    // ✅ valid — Der3 IS-A Base2
Base2 b3 = (Base2)(new Der1());    // ❌ invalid — Der1 is not related to Base2
```

> Type casting just converts the **reference type**, not the object itself. No new object is created.

---

### Method vs Variable Resolution

```
Instance method resolution  → based on RUNTIME OBJECT  (overriding / polymorphism)
Static method resolution    → based on REFERENCE TYPE   (method hiding)
Variable resolution         → ALWAYS based on REFERENCE TYPE
```

```java
class A { int x = 100; }
class B extends A { int x = 200; }
class C extends B { int x = 300; }

C c = new C();
System.out.println(c.x);              // 300 — C's x (reference type = C)
System.out.println(((B) c).x);        // 200 — B's x (reference type = B)
System.out.println(((A)((B) c)).x);   // 100 — A's x (reference type = A)
```

### Safe Casting with `instanceof` (Java 16+ Pattern Matching)

```java
// Old way (before Java 16)
if (animal instanceof Dog) {
    Dog d = (Dog) animal;
    d.bark();
}

// New way — Pattern Matching instanceof (Java 16+)
if (animal instanceof Dog d) {
    d.bark(); // no explicit cast needed — cleaner and safer
}
```

---

## 6. Coupling

### What is it?
**Coupling** = the degree of dependency between components (classes/modules).

```
Tight Coupling  →  A changes → B breaks → C breaks  ❌
Loose Coupling  →  A changes → only A is affected   ✅
```

### Tight Coupling — Bad Practice

```java
class D { static int value = 10; }
class C { static int getVal() { return D.value; } } // C depends on D
class B { static int data = C.getVal(); }            // B depends on C
class A { static int info = B.data; }                // A depends on B
// A → B → C → D — all tightly coupled
```

**Problems:**
- Cannot modify one component without breaking others
- Low maintainability
- Low reusability

### Loose Coupling — Good Practice

Use **interfaces** to remove direct dependencies:

```java
interface PaymentGateway {
    void pay(double amount);
}

class RazorpayGateway implements PaymentGateway {
    public void pay(double amount) {
        System.out.println("Paid via Razorpay: ₹" + amount);
    }
}

class Order {
    private PaymentGateway gateway; // depends on interface, not implementation

    Order(PaymentGateway gateway) {
        this.gateway = gateway;
    }

    void checkout(double amount) {
        gateway.pay(amount);
    }
}

// Can switch payment gateway without changing Order class at all
Order o = new Order(new RazorpayGateway());
o.checkout(500);
```

> **Modern Java (Spring Boot):** Loose coupling is achieved through **Dependency Injection** using `@Autowired`, `@Bean`, etc.

---

## 7. Cohesion

### What is it?
**Cohesion** = how focused and well-defined a component's responsibility is.

```
Low Cohesion  →  One class does EVERYTHING  ❌
High Cohesion →  Each class does ONE thing well  ✅
```

### Low Cohesion — Bad Practice

```java
class TotalServlet {
    void login()    { /* login logic */ }
    void validate() { /* validation logic */ }
    void inbox()    { /* inbox logic */ }
    void compose()  { /* compose logic */ }
    void logout()   { /* logout logic */ }
    // hard to maintain, hard to reuse
}
```

### High Cohesion — Good Practice

```java
class LoginServlet    { void login()    { /* only login */ } }
class ValidateServlet { void validate() { /* only validate */ } }
class InboxServlet    { void inbox()    { /* only inbox */ } }
class ComposeServlet  { void compose()  { /* only compose */ } }
```

```
Low Cohesion                    High Cohesion
┌─────────────────┐             ┌──────────┐   ┌──────────┐
│  TotalServlet   │             │  Login   │   │ Validate │
│  - login        │    ──►      └──────────┘   └──────────┘
│  - validate     │             ┌──────────┐   ┌──────────┐
│  - inbox        │             │  Inbox   │   │ Compose  │
│  - compose      │             └──────────┘   └──────────┘
└─────────────────┘
```

**Advantages of High Cohesion:**
- Easy to modify one component without touching others
- Better maintainability
- Better reusability — e.g., `ValidateServlet` can be reused everywhere validation is needed

---

### The Golden Rule

```
Always aim for:
  ✅ Loosely Coupled  +  ✅ Highly Cohesive  components
```

> **Modern Java / Spring Boot** enforces this naturally:
> - **Single Responsibility Principle (SRP)** → High Cohesion
> - **Dependency Injection** → Loose Coupling
> - **Interfaces / Abstractions** → both

---

## Quick Reference Cheat Sheet

### Singleton Pattern Variants

| Variant | Thread Safe | Lazy Init | Recommended |
|---------|-------------|-----------|-------------|
| Basic (no sync) | ❌ | ✅ | ❌ |
| Synchronized method | ✅ | ✅ | ⚠️ slow |
| Double-Checked Locking | ✅ | ✅ | ✅ |
| Enum Singleton | ✅ | ❌ | ✅ Best |
| Static Inner Class | ✅ | ✅ | ✅ |

### Type Casting Rules

| Check | When | Error thrown |
|-------|------|--------------|
| No relationship between types | Compile time | `inconvertible types` |
| Cast type not a subtype of target | Compile time | `incompatible types` |
| Actual object is wrong type at runtime | Runtime | `ClassCastException` |

### Control Flow Execution Order

| Flow | Sequence |
|------|----------|
| Static (single class) | Identify → Assign/Static blocks → `main()` |
| Instance (single class) | Identify → Assign/Instance blocks → Constructor |
| Static (Parent → Child) | Identify all → Parent execute → Child execute → Child `main()` |
| Instance (Parent → Child) | Identify all → Parent blocks → Parent constructor → Child blocks → Child constructor |

---

> **Java Version Note:** From **Java 1.7 onwards**, `main()` method is mandatory to run a Java program. Running code via only static blocks (without `main()`) works only up to Java 1.6.
