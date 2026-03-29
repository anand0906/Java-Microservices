# ☕ Java OOP Notes

> **Java Versions Referenced:** Java 8, Java 11, Java 17 (LTS), Java 21 (LTS)

---

## 📑 Index

1. [Data Hiding](#1-data-hiding)
2. [Abstraction](#2-abstraction)
3. [Encapsulation](#3-encapsulation)
4. [Tightly Encapsulated Class](#4-tightly-encapsulated-class)
5. [IS-A Relationship (Inheritance)](#5-is-a-relationship-inheritance)
6. [Multiple Inheritance](#6-multiple-inheritance)
7. [Cyclic Inheritance](#7-cyclic-inheritance)
8. [HAS-A Relationship](#8-has-a-relationship)
9. [Composition vs Aggregation](#9-composition-vs-aggregation)

---

## 1. Data Hiding

> **"Internal data should not go out directly."**

- Outside code should **not** access internal data directly.
- Use the `private` modifier to implement data hiding.
- ✅ Recommended modifier for data members: **`private`**
- Main advantage: **Security**

```java
class BankAccount {
    private double balance;  // hidden from outside

    public double getBalance(String password) {
        if (isValid(password)) return balance;
        throw new SecurityException("Invalid credentials");
    }
}
```

```
Outside World ❌──────────────────────────────────> balance (private)
Outside World ✅──> getBalance(password) ──> validate ──> balance
```

> 💡 **Java 17+ Records** make data hiding simpler. Record fields are `private final` by default:
> ```java
> record Point(int x, int y) {}  // x and y are private final automatically
> ```

---

## 2. Abstraction

> **"Show WHAT you can do, hide HOW you do it."**

- Hide internal implementation; expose only the services.
- Implemented using **abstract classes** and **interfaces**.
- Main advantage: **Security + Flexibility + Maintainability**

```
ATM Screen (User sees)          Internal Bank System (Hidden)
┌────────────────────┐          ┌────────────────────────────┐
│  [ Check Balance ] │ ──────>  │  DB query, auth, logging   │
│  [ Withdraw Cash ] │ ──────>  │  Transaction processing    │
│  [ Transfer ]      │ ──────>  │  Encryption, fraud check   │
└────────────────────┘          └────────────────────────────┘
      Interface                      Implementation (hidden)
```

### Advantages of Abstraction

| # | Advantage |
|---|-----------|
| 1 | Security — internal implementation stays hidden |
| 2 | Easy enhancement — change internals without affecting users |
| 3 | More flexibility for end users |
| 4 | Improves maintainability |
| 5 | Improves modularity |
| 6 | Easier to use the system |

### Example — Abstract Class

```java
abstract class Shape {
    abstract double area();   // what — no implementation

    void display() {
        System.out.println("Area = " + area());  // how — defined
    }
}

class Circle extends Shape {
    double r;
    Circle(double r) { this.r = r; }

    @Override
    double area() { return Math.PI * r * r; }
}
```

### Example — Interface (Java 8+)

```java
interface Payment {
    void pay(double amount);           // abstract by default

    default void receipt() {           // default method (Java 8+)
        System.out.println("Payment done.");
    }

    static void info() {               // static method (Java 8+)
        System.out.println("UPI / Card / NetBanking");
    }
}
```

> 💡 **Java 9+** added `private` methods inside interfaces for internal helper logic.

---

## 3. Encapsulation

> **Encapsulation = Data Hiding + Abstraction**

- Binding **data** and **methods** into a single unit (class).
- A class that follows both data hiding and abstraction is called an **encapsulated class**.

```
┌───────────────────────────────────────────────┐
│                  class Account                │
│                                               │
│   private double balance;  ◄── Data Hiding   │
│                                               │
│   public double getBalance() { ... }  ◄──┐   │
│   public void setBalance(double b) { }    │   │
│                                        Abstraction
└───────────────────────────────────────────────┘
                   Encapsulation
```

```java
class Account {
    private double balance;   // data hiding

    public double getBalance() {   // abstraction (service exposed)
        // validate user
        return balance;
    }

    public void setBalance(double balance) {
        // validate user
        this.balance = balance;
    }
}
```

### Rules
- Every data member → **`private`**
- For every data member → provide **getter and setter** methods

### Advantages

| # | Advantage |
|---|-----------|
| 1 | Security |
| 2 | Easy enhancement |
| 3 | Better maintainability and modularity |
| 4 | Flexibility |

### Disadvantage
- Increases code length and slightly slows execution.

> 💡 **Java 16+ Records** give you encapsulation out of the box for immutable data:
> ```java
> record Employee(String name, double salary) {}
> // name and salary are private + auto getter generated
> ```

---

## 4. Tightly Encapsulated Class

> A class is **tightly encapsulated** if and only if **every variable** is declared `private` — regardless of whether getter/setter methods exist or are public.

### Rule
```
ALL variables must be private → Tightly Encapsulated ✅
Even ONE non-private variable → NOT Tightly Encapsulated ❌
```

### Example

```java
class A {
    private int x = 10;   // ✅ private
}
// A is tightly encapsulated

class B extends A {
    int y = 20;           // ❌ not private
}
// B is NOT tightly encapsulated

class C extends A {
    private int z = 30;   // ✅ private
}
// C is tightly encapsulated (A is also tightly encapsulated)
```

### Important Rule about Inheritance

```
class A { int x = 10; }          // NOT tightly encapsulated
class B extends A { private int y; }   // NOT tightly encapsulated
class C extends B { private int z; }   // NOT tightly encapsulated
```

> ⚠️ **If the parent class is NOT tightly encapsulated, no child class can be tightly encapsulated.**

---

## 5. IS-A Relationship (Inheritance)

> Also known as **Inheritance** — the child "is a" type of parent.

- Implemented using the **`extends`** keyword.
- Main advantage: **Reusability**

```
        Animal
           │
    ┌──────┴──────┐
   Dog           Cat
(IS-A Animal)  (IS-A Animal)
```

```java
class Animal {
    void eat() { System.out.println("eating..."); }
}

class Dog extends Animal {
    void bark() { System.out.println("barking..."); }
}

// Usage
Dog d = new Dog();
d.eat();    // ✅ inherited
d.bark();   // ✅ own method

Animal a = new Dog();   // ✅ parent ref can hold child object
a.eat();    // ✅
// a.bark(); ❌ compile error — parent ref can't call child-specific methods

// Dog d2 = new Animal(); ❌ child ref can't hold parent object
```

### Key Rules

| Reference Type | Object Type | Can Call |
|---|---|---|
| Parent ref | Parent obj | Parent methods only |
| Child ref | Child obj | Both parent and child methods |
| Parent ref | Child obj | Parent methods only (child-specific ❌) |
| Child ref | Parent obj | ❌ Compile Error |

### Multilevel Inheritance

```
Object
  └── B
       └── A     ✅ Valid (multilevel)
```

```java
class B {}
class A extends B {}  // A → B → Object (multilevel)
```

### Object Class — Root of All

```
                    Object
                   /      \
              String    Number    ...    Throwable
                        /  \               /    \
                    Integer Long      Exception  Error
```

- Every class in Java extends `Object` either directly or indirectly.
- `Object` class acts as the **root** for all Java classes.
- `Throwable` class is the root for the exception hierarchy.

> 💡 **Java 21** introduced **Sealed Classes** (finalized) — you can control which classes can extend a class:
> ```java
> sealed class Shape permits Circle, Rectangle {}
> final class Circle extends Shape {}
> final class Rectangle extends Shape {}
> ```

---

## 6. Multiple Inheritance

> Having **more than one parent class** at the same level.

```
Parent1    Parent2
    \       /
      Child
```

### Java does NOT support multiple class inheritance ❌

```java
class A {}
class B {}
class C extends A, B {}  // ❌ Compile Error
```

**Why?** — **Ambiguity Problem**

```
Parent1 ──> methodOne()
                 ↑
             c.methodOne()  // Which one? Ambiguous!
                 ↑
Parent2 ──> methodOne()
```

### Java DOES support multiple inheritance via Interfaces ✅

```java
interface A { void methodOne(); }
interface B { void methodOne(); }

interface C extends A, B {}  // ✅ valid

class Test implements C {
    public void methodOne() {
        System.out.println("methodOne");  // one implementation, no ambiguity
    }
}
```

**Why no ambiguity in interfaces?**
- Interfaces only have **declarations** (no implementation), so there's nothing to be ambiguous about.
- The implementing class provides the single implementation.

> 💡 **Java 8 Default Methods** — if two interfaces have the same default method, the implementing class **must override** it to resolve ambiguity:
> ```java
> interface A { default void greet() { System.out.println("A"); } }
> interface B { default void greet() { System.out.println("B"); } }
>
> class C implements A, B {
>     public void greet() { A.super.greet(); }  // must override and pick one
> }
> ```

---

## 7. Cyclic Inheritance

> **Cyclic inheritance is NOT allowed in Java.**

```java
class A extends B {}
class B extends A {}   // ❌ Compile Error: cyclic inheritance involving A
```

```java
class A extends A {}   // ❌ Compile Error: cyclic inheritance involving A
```

```
A ──extends──> B
↑              │
└──extends─────┘
      ❌ Not allowed
```

---

## 8. HAS-A Relationship

> Also known as **Composition** or **Aggregation** — one class "has" another class as a member.

- No specific keyword; typically uses the **`new`** operator.
- Main advantage: **Reusability**
- Main disadvantage: **Increases dependency** between components (maintenance problems).

```java
class Engine {
    void start() { System.out.println("Engine started"); }
}

class Car {
    Engine e = new Engine();  // Car HAS-A Engine

    void drive() {
        e.start();
        System.out.println("Car is moving");
    }
}
```

```
Car ──HAS-A──> Engine
```

---

## 9. Composition vs Aggregation

Both are types of HAS-A relationship, but differ in **how tightly the objects are bound**.

### Composition — Strong Association

> Without the container object, the contained object **cannot exist**.

```
University ─────────────────────────┐
│  ┌───────────┐ ┌───────────┐     │
│  │ Dept - CS │ │ Dept - IT │     │  Container Object
│  └───────────┘ └───────────┘     │
│  ┌───────────┐ ┌───────────┐     │
│  │ Dept - EC │ │ Dept - ME │     │
│  └───────────┘ └───────────┘     │
└───────────────────────────────────┘
      Contained Objects (Departments)

University destroyed → All Departments destroyed ✅
```

```java
class Department {
    String name;
    Department(String name) { this.name = name; }
}

class University {
    // Departments are created INSIDE University
    Department d1 = new Department("CS");
    Department d2 = new Department("IT");
    // If University object dies, d1 and d2 also die
}
```

### Aggregation — Weak Association

> Without the container object, the contained object **can still exist**.

```
Department ─────────────────────────┐
│  ── Professor p1                  │  Container Object
│  ── Professor p2                  │
│  ── Professor p3                  │
└───────────────────────────────────┘
      Contained Objects (Professors)

Department closed → Professors still exist elsewhere ✅
```

```java
class Professor {
    String name;
    Professor(String name) { this.name = name; }
}

class Department {
    Professor p;  // holds a REFERENCE to an existing Professor
    Department(Professor p) { this.p = p; }
}

// Professor exists independently
Professor prof = new Professor("Dr. Smith");
Department dept = new Department(prof);
// dept closed → prof still exists
```

### Comparison Table

| Feature | Composition | Aggregation |
|---|---|---|
| Association strength | Strong | Weak |
| Contained object lifecycle | Depends on container | Independent |
| Container holds | Direct object | Reference to object |
| Example | University → Department | Department → Professor |
| Relationship | "part-of" | "has-a" |

---

## 🔁 Quick Summary

```
┌──────────────────────────────────────────────────────────┐
│                   OOP Pillars in Java                    │
├──────────────┬───────────────────────────────────────────┤
│ Data Hiding  │ private modifier, security                │
│ Abstraction  │ abstract class / interface, hide impl.    │
│ Encapsulation│ = Data Hiding + Abstraction               │
│ Inheritance  │ extends keyword, IS-A, reusability        │
│ HAS-A        │ new keyword, composition/aggregation      │
└──────────────┴───────────────────────────────────────────┘
```

### Java Version Quick Ref

| Feature | Introduced In |
|---|---|
| Abstract classes, interfaces | Java 1.0 |
| Generics | Java 5 |
| Default & static methods in interfaces | Java 8 |
| Private methods in interfaces | Java 9 |
| Records (immutable data classes) | Java 16 |
| Sealed classes (controlled inheritance) | Java 17 |
| Pattern matching for switch | Java 21 |
