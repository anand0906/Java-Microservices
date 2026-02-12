# Interface Methods in Java

## Table of Contents

1. [Evolution of Interface Methods](#1-evolution-of-interface-methods)
2. [Default Methods (Java 8)](#2-default-methods-java-8)
3. [Static Methods (Java 8)](#3-static-methods-java-8)
4. [Private Methods (Java 9)](#4-private-methods-java-9)
5. [Comparison Tables](#5-comparison-tables)
6. [Interview Questions & Answers](#6-interview-questions--answers)
7. [Quick Revision](#7-quick-revision)

---

## 1. Evolution of Interface Methods

### Timeline

| Java Version | Interface Features |
|--------------|-------------------|
| **Java 7 and before** | Only abstract methods and constants |
| **Java 8** | Added default methods and static methods |
| **Java 9** | Added private instance and static methods |

### Java 7 Interface (Traditional)

```java
interface Java7Interface {
    // Only public static final variables (constants)
    int MAX_VALUE = 100;
    
    // Only public abstract methods
    void m1();
    void m2();
}
```

### Modern Interface (Java 9+)

```java
interface ModernInterface {
    // Constants
    int MAX_VALUE = 100;
    
    // Abstract methods
    void abstractMethod();
    
    // Default method (Java 8)
    default void defaultMethod() {
        System.out.println("Default method");
    }
    
    // Static method (Java 8)
    static void staticMethod() {
        System.out.println("Static method");
    }
    
    // Private method (Java 9)
    private void privateMethod() {
        System.out.println("Private method");
    }
    
    // Private static method (Java 9)
    private static void privateStaticMethod() {
        System.out.println("Private static method");
    }
}
```

---

## 2. Default Methods (Java 8)

### What are Default Methods?

**Definition:**
- Methods with concrete implementation in interfaces
- Also called **defender methods** or **virtual extension methods**
- Introduced in Java 8 for backward compatibility

### Syntax

```java
interface MyInterface {
    default void methodName() {
        // Method implementation
    }
}
```

### Basic Example

```java
interface Interf {
    default void m1() {
        System.out.println("Default Method");
    }
}

class Test implements Interf {
    public static void main(String[] args) {
        Test t = new Test();
        t.m1();  // Calls inherited default method
    }
}
```

**Output:**
```
Default Method
```

### Key Characteristics

**Properties:**
- Provide concrete implementation in interface
- Inherited by all implementing classes
- Can be overridden by implementing classes
- Allow adding new methods without breaking existing code
- **Cannot override Object class methods** (hashCode, equals, toString)

### Why Default Methods?

**Problem (Before Java 8):**
```java
// Interface with 100 implementing classes
interface Collection {
    void add();
    void remove();
    // Want to add new method - breaks all 100 classes!
}
```

**Solution (Java 8):**
```java
interface Collection {
    void add();
    void remove();
    
    // New method with default implementation
    default void forEach() {
        // Default implementation
    }
}
// Existing 100 classes still work without changes!
```

### Overriding Default Methods

```java
interface MyInterface {
    default void display() {
        System.out.println("Interface implementation");
    }
}

class MyClass implements MyInterface {
    // Override default method
    @Override
    public void display() {
        System.out.println("Class implementation");
    }
}

class Test {
    public static void main(String[] args) {
        MyClass obj = new MyClass();
        obj.display();  // Calls overridden method
    }
}
```

**Output:**
```
Class implementation
```

### Restriction: Cannot Override Object Methods

```java
interface MyInterface {
    // ❌ Compilation Error - Cannot override Object methods
    default int hashCode() {
        return 10;
    }
    
    // ❌ Compilation Error
    default boolean equals(Object obj) {
        return false;
    }
    
    // ❌ Compilation Error
    default String toString() {
        return "MyInterface";
    }
}
```

**Why?** Every class inherits from Object. Allowing interfaces to override Object methods would create ambiguity.

### Diamond Problem (Multiple Inheritance Ambiguity)

**Problem:**
```java
interface Left {
    default void m1() {
        System.out.println("Left Default Method");
    }
}

interface Right {
    default void m1() {
        System.out.println("Right Default Method");
    }
}

// ❌ Compilation Error - Ambiguous!
class Test implements Left, Right {
    // Which m1() to inherit?
}
```

**Solution 1: Override the Method**
```java
class Test implements Left, Right {
    @Override
    public void m1() {
        System.out.println("Test Class Method");
    }
}
```

**Solution 2: Call Specific Interface Method**
```java
class Test implements Left, Right {
    @Override
    public void m1() {
        Left.super.m1();  // Call Left's implementation
        // OR
        Right.super.m1(); // Call Right's implementation
    }
}
```

**Solution 3: Call Both**
```java
class Test implements Left, Right {
    @Override
    public void m1() {
        System.out.println("Test implementation");
        Left.super.m1();   // Also call Left
        Right.super.m1();  // Also call Right
    }
}
```

### Default Methods vs Abstract Class

| Feature | Interface with Default Methods | Abstract Class |
|---------|-------------------------------|----------------|
| Variables | Only `public static final` | Can have instance variables |
| State | Cannot maintain object state | Can maintain object state |
| Constructors | ❌ No constructors | ✅ Can have constructors |
| Blocks | ❌ No instance/static blocks | ✅ Can have blocks |
| Lambda | ✅ Can be functional interface | ❌ Cannot use lambdas |
| Object Methods | ❌ Cannot override | ✅ Can override |
| Multiple Inheritance | ✅ Supports multiple | ❌ Single inheritance only |

**Conclusion:** Interface with default methods ≠ Abstract class

---

## 3. Static Methods (Java 8)

### What are Static Methods in Interfaces?

**Definition:**
- Methods with implementation that belong to the interface itself
- Used for utility functions
- **Not inherited** by implementing classes
- Must be called using interface name

### Syntax

```java
interface MyInterface {
    static void methodName() {
        // Method implementation
    }
}
```

### Basic Example

```java
interface Calculator {
    static void sum(int a, int b) {
        System.out.println("Sum: " + (a + b));
    }
}

class Test implements Calculator {
    public static void main(String[] args) {
        // ❌ Wrong ways
        Test t = new Test();
        t.sum(10, 20);        // Compilation Error
        Test.sum(10, 20);     // Compilation Error
        
        // ✅ Correct way
        Calculator.sum(10, 20);  // Must use interface name
    }
}
```

**Output:**
```
Sum: 30
```

### Key Characteristics

**Properties:**
- Belong to interface, not implementing class
- **Cannot be inherited** by implementing classes
- Must be invoked using **interface name**
- Cannot be overridden
- Available from Java 8 onwards
- Can access only static members of interface

### Not Inherited (Cannot Override)

```java
interface Interf {
    static void m1() {
        System.out.println("Interface static method");
    }
}

// Valid - but NOT overriding
class Test implements Interf {
    static void m1() {
        System.out.println("Class static method");
    }
    
    public static void main(String[] args) {
        Interf.m1();  // Interface static method
        Test.m1();    // Class static method
    }
}
```

**Output:**
```
Interface static method
Class static method
```

### Instance Method with Same Name (Also Valid)

```java
interface Interf {
    static void m1() {
        System.out.println("Interface static method");
    }
}

class Test implements Interf {
    // Valid - instance method with same name
    public void m1() {
        System.out.println("Instance method");
    }
    
    public static void main(String[] args) {
        Interf.m1();     // Interface static method
        Test t = new Test();
        t.m1();          // Instance method
    }
}
```

### Utility Methods Example

```java
interface MathUtils {
    static int max(int a, int b) {
        return a > b ? a : b;
    }
    
    static int min(int a, int b) {
        return a < b ? a : b;
    }
    
    static int square(int a) {
        return a * a;
    }
}

class Test {
    public static void main(String[] args) {
        System.out.println(MathUtils.max(10, 20));      // 20
        System.out.println(MathUtils.min(10, 20));      // 10
        System.out.println(MathUtils.square(5));        // 25
    }
}
```

### Running Interface Directly (Java 8+)

```java
interface Interf {
    public static void main(String[] args) {
        System.out.println("Interface Main Method");
    }
}
```

**Command Line:**
```bash
javac Interf.java
java Interf
```

**Output:**
```
Interface Main Method
```

---

## 4. Private Methods (Java 9)

### Why Private Methods?

**Problem (Before Java 9):**
```java
interface Java8DBLogging {
    default void logInfo(String message) {
        // Step 1: Connect to Database - DUPLICATE
        // Step 2: Log Info Message
        // Step 3: Close Database Connection - DUPLICATE
    }
    
    default void logWarn(String message) {
        // Step 1: Connect to Database - DUPLICATE
        // Step 2: Log Warn Message
        // Step 3: Close Database Connection - DUPLICATE
    }
    
    default void logError(String message) {
        // Step 1: Connect to Database - DUPLICATE
        // Step 2: Log Error Message
        // Step 3: Close Database Connection - DUPLICATE
    }
}
```

**Problems:**
- Code duplication
- Hard to maintain
- Reduced readability

**Solution (Java 9 - Private Methods):**
```java
interface Java9DBLogging {
    default void logInfo(String message) {
        log(message, "INFO");
    }
    
    default void logWarn(String message) {
        log(message, "WARN");
    }
    
    default void logError(String message) {
        log(message, "ERROR");
    }
    
    // Private method - reusable, not visible to implementing classes
    private void log(String msg, String logLevel) {
        // Step 1: Connect to Database
        System.out.println("Connecting to database...");
        // Step 2: Log Message
        System.out.println(logLevel + ": " + msg);
        // Step 3: Close Connection
        System.out.println("Closing database connection...");
    }
}
```

### Private Instance Methods

**Definition:**
- Used to share code between default methods
- Not accessible from implementing classes
- Can access instance members

**Example:**
```java
interface Java9Interf {
    default void m1() {
        m3();  // Call private method
    }
    
    default void m2() {
        m3();  // Call private method
    }
    
    // Private instance method
    private void m3() {
        System.out.println("Common functionality of m1 & m2");
    }
}

class Test implements Java9Interf {
    public static void main(String[] args) {
        Test t = new Test();
        t.m1();
        t.m2();
        // t.m3();  // ❌ Compilation Error - private method not accessible
    }
}
```

**Output:**
```
Common functionality of m1 & m2
Common functionality of m1 & m2
```

### Private Static Methods

**Definition:**
- Used to share code between static methods
- Not accessible from implementing classes
- Can only access static members

**Example:**
```java
interface Java9Interf {
    static void m1() {
        m3();  // Call private static method
    }
    
    static void m2() {
        m3();  // Call private static method
    }
    
    // Private static method
    private static void m3() {
        System.out.println("Common functionality of m1 & m2");
    }
}

class Test {
    public static void main(String[] args) {
        Java9Interf.m1();
        Java9Interf.m2();
        // Java9Interf.m3();  // ❌ Compilation Error - private
    }
}
```

**Output:**
```
Common functionality of m1 & m2
Common functionality of m1 & m2
```

### Private Method Rules

**Key Points:**
1. Private methods **must have a body** (cannot be abstract)
2. Can be **instance** or **static**
3. **Not inherited** by implementing classes
4. Used for **code reusability** within the interface
5. Provide **encapsulation** - hide implementation details

### Complete Example

```java
interface PaymentProcessor {
    // Abstract method
    void processPayment(double amount);
    
    // Default methods
    default void processCreditCard(double amount) {
        validateAmount(amount);
        log("Processing credit card payment: $" + amount);
    }
    
    default void processDebitCard(double amount) {
        validateAmount(amount);
        log("Processing debit card payment: $" + amount);
    }
    
    // Static method
    static void displayInfo() {
        System.out.println("Payment Processor v1.0");
    }
    
    // Private instance method
    private void validateAmount(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Invalid amount");
        }
    }
    
    // Private static method
    private void log(String message) {
        System.out.println("[LOG] " + message);
    }
}
```

### Advantages of Private Methods

**Benefits:**
1. **Code Reusability** - Eliminates duplicate code
2. **Encapsulation** - Hides implementation details
3. **Maintainability** - Changes in one place
4. **Clean API** - Only expose intended methods

---

## 5. Comparison Tables

### JDK Evolution

| Feature | JDK 7 | JDK 8 | JDK 9 |
|---------|-------|-------|-------|
| Public Static Final Variables | ✅ | ✅ | ✅ |
| Public Abstract Methods | ✅ | ✅ | ✅ |
| Default Methods | ❌ | ✅ | ✅ |
| Public Static Methods | ❌ | ✅ | ✅ |
| Private Instance Methods | ❌ | ❌ | ✅ |
| Private Static Methods | ❌ | ❌ | ✅ |

### Method Types Comparison

| Aspect | Default Method | Static Method | Private Instance | Private Static |
|--------|---------------|---------------|------------------|----------------|
| **Java Version** | 8+ | 8+ | 9+ | 9+ |
| **Keyword** | `default` | `static` | `private` | `private static` |
| **Inherited?** | ✅ Yes | ❌ No | ❌ No | ❌ No |
| **Can Override?** | ✅ Yes | ❌ No | ❌ No | ❌ No |
| **Access via** | Object | Interface name | Within interface | Within interface |
| **Purpose** | Add functionality | Utility methods | Helper for default | Helper for static |
| **Can be abstract?** | ❌ No | ❌ No | ❌ No | ❌ No |

### Access Modifiers in Interface

| Method Type | Allowed Modifiers |
|-------------|------------------|
| Abstract methods | `public` (implicit) |
| Default methods | `public` (implicit) |
| Static methods | `public` (implicit) |
| Private instance methods | `private` only |
| Private static methods | `private static` only |

---

## 6. Interview Questions & Answers

### Q1: What are default methods in interfaces?

**Answer:** Default methods are methods with concrete implementation in interfaces, introduced in Java 8. They allow interfaces to provide method implementations without breaking existing implementing classes. They are also called defender methods or virtual extension methods.

### Q2: Why were default methods introduced?

**Answer:** Default methods were introduced for backward compatibility. They allow adding new methods to interfaces without forcing all implementing classes to implement them. For example, Java 8 added the `forEach()` method to the Collection interface as a default method, so thousands of existing classes didn't break.

### Q3: Can we override default methods?

**Answer:** Yes, implementing classes can override default methods. If not overridden, the class inherits the default implementation.

```java
interface MyInterface {
    default void display() {
        System.out.println("Default");
    }
}

class MyClass implements MyInterface {
    @Override
    public void display() {
        System.out.println("Overridden");
    }
}
```

### Q4: What is the diamond problem with default methods?

**Answer:** The diamond problem occurs when a class implements two interfaces with the same default method signature. This creates ambiguity about which method to inherit.

**Solution:** The implementing class must override the method or explicitly call a specific interface method using `InterfaceName.super.methodName()`.

### Q5: Can default methods override Object class methods?

**Answer:** No, default methods cannot override Object class methods (hashCode, equals, toString). This would result in a compilation error because every class inherits from Object, creating ambiguity.

### Q6: What is the difference between static methods in class and interface?

**Answer:**
- **Class static methods:** Inherited by subclasses
- **Interface static methods:** NOT inherited by implementing classes, must be called using interface name

### Q7: Can we call interface static methods from implementing class?

**Answer:** No, interface static methods cannot be called using implementing class name or object. They must be called using the interface name.

```java
// ❌ Wrong: Test.method()
// ✅ Correct: InterfaceName.method()
```

### Q8: Why were private methods added in Java 9?

**Answer:** Private methods were added to eliminate code duplication in default and static methods within interfaces. They provide code reusability and encapsulation without exposing helper methods to implementing classes.

### Q9: What's the difference between private and private static methods in interfaces?

**Answer:**
- **Private instance methods:** Used by default methods, can access instance context
- **Private static methods:** Used by static methods, can only access static members

### Q10: Can an interface have a main method?

**Answer:** Yes, from Java 8 onwards, interfaces can have a static main method and can be run directly from the command line.

```java
interface Test {
    static void main(String[] args) {
        System.out.println("Interface main method");
    }
}
```

### Q11: How to resolve diamond problem?

**Answer:** Three ways:
1. Override the method in implementing class
2. Call specific interface method: `Left.super.m1()`
3. Call both: Override and call both interface methods inside

### Q12: Can private methods be abstract?

**Answer:** No, private methods must have a body. They cannot be abstract because they are meant for code reuse within the interface itself.

### Q13: Difference between abstract class and interface with default methods?

**Answer:** Key differences:
- Interface cannot have instance variables or constructors
- Interface cannot maintain object state
- Interface supports multiple inheritance
- Interface can be used as functional interface with lambdas
- Abstract class can override Object methods

---

## 7. Quick Revision

### Default Methods (Java 8)

**Key Points:**
- Declared with `default` keyword
- Have concrete implementation
- Inherited by implementing classes
- Can be overridden
- Cannot override Object methods
- Solve diamond problem by explicit override

**Syntax:**
```java
interface MyInterface {
    default void method() {
        // implementation
    }
}
```

### Static Methods (Java 8)

**Key Points:**
- Declared with `static` keyword
- NOT inherited by implementing classes
- Must call using interface name
- Cannot be overridden
- Used for utility functions
- Can have main() method

**Syntax:**
```java
interface MyInterface {
    static void method() {
        // implementation
    }
}

// Call: MyInterface.method()
```

### Private Methods (Java 9)

**Two Types:**

**1. Private Instance Methods:**
```java
interface MyInterface {
    default void publicMethod() {
        privateHelper();
    }
    
    private void privateHelper() {
        // helper code
    }
}
```

**2. Private Static Methods:**
```java
interface MyInterface {
    static void publicMethod() {
        privateStaticHelper();
    }
    
    private static void privateStaticHelper() {
        // helper code
    }
}
```

### Diamond Problem Resolution

```java
interface A {
    default void m() { }
}

interface B {
    default void m() { }
}

class C implements A, B {
    // Solution 1: Override
    public void m() {
        System.out.println("C's implementation");
    }
    
    // Solution 2: Call specific
    public void m() {
        A.super.m();  // or B.super.m()
    }
}
```

### Evolution Summary

**Java 7:**
- Only abstract methods and constants

**Java 8:**
- ✅ Default methods (backward compatibility)
- ✅ Static methods (utility functions)

**Java 9:**
- ✅ Private instance methods (code reuse for default)
- ✅ Private static methods (code reuse for static)

### Common Mistakes to Avoid

**❌ Don't:**
```java
// Cannot override Object methods
default int hashCode() { return 10; }

// Cannot call static method via object
obj.staticMethod();

// Cannot access private method from implementing class
class Test implements Interf {
    void method() {
        privateMethod();  // Error!
    }
}
```

**✅ Do:**
```java
// Override default methods when needed
@Override
public void defaultMethod() { }

// Call static methods via interface name
InterfaceName.staticMethod();

// Use private methods within interface only
private void helper() { }  // Only for internal use
```

### Best Practices

1. ✅ Use default methods for backward compatibility
2. ✅ Use static methods for utility functions
3. ✅ Use private methods to eliminate code duplication
4. ✅ Always override when implementing multiple interfaces with same default method
5. ✅ Prefer composition over multiple inheritance when possible
6. ✅ Document default method behavior clearly
7. ✅ Keep interfaces focused (Single Responsibility Principle)

### Memory Points

**Remember:**
- Default = Inherited, Can Override
- Static = NOT Inherited, Interface Name
- Private = NOT Inherited, Internal Use Only
- Java 8 = Default + Static
- Java 9 = Private + Private Static

### Quick Test

**What's wrong here?**
```java
interface Test {
    default String toString() {  // ❌ Error!
        return "Test";
    }
    
    static void m1() { }
}

class A implements Test {
    void method() {
        m1();  // ❌ Error!
    }
}
```

**Answers:**
1. Cannot override Object methods in interface
2. Static methods must be called using interface name: `Test.m1()`

---

## Summary Cheat Sheet

### Interface Method Types

| Type | Keyword | Since | Inherited | Override | Access |
|------|---------|-------|-----------|----------|--------|
| Abstract | none | All | ✅ | Must | - |
| Default | `default` | Java 8 | ✅ | Can | Anywhere |
| Static | `static` | Java 8 | ❌ | No | Interface name |
| Private | `private` | Java 9 | ❌ | No | Within interface |
| Private Static | `private static` | Java 9 | ❌ | No | Within interface |

### When to Use What

- **Abstract methods:** Contract that implementing classes must fulfill
- **Default methods:** Provide common implementation, allow interface evolution
- **Static methods:** Utility methods related to interface
- **Private methods:** Eliminate code duplication within interface
- **Private static methods:** Helper methods for static methods

**Master these concepts and you'll ace any interface-related interview question!** 🎯
