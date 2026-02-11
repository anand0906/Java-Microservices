# Java Functional Interfaces

## Table of Contents
1. [Introduction](#introduction)
2. [What is a Functional Interface?](#what-is-a-functional-interface)
3. [The @FunctionalInterface Annotation](#the-functionalinterface-annotation)
4. [Functional Interface with Inheritance](#functional-interface-with-inheritance)
5. [Functional Interface vs Lambda Expressions](#functional-interface-vs-lambda-expressions)
6. [Built-in Functional Interfaces](#built-in-functional-interfaces)
   - [Predicate](#1-predicatet)
   - [Function](#2-functiont-r)
   - [Consumer](#3-consumert)
   - [Supplier](#4-suppliert)
   - [UnaryOperator](#5-unaryoperatort)
   - [BinaryOperator](#6-binaryoperatort)
7. [Method and Constructor References](#method-and-constructor-references)
8. [Summary Tables](#summary-tables)

---

## Introduction

A **functional interface** is an interface that contains exactly **one abstract method** (also called **Single Abstract Method - SAM**). Functional interfaces enable the use of lambda expressions and method references in Java.

---

## What is a Functional Interface?

A functional interface can have:
- **Exactly one abstract method** (required)
- **Any number of default methods** (optional)
- **Any number of static methods** (optional)

### Common Examples

| Interface | Abstract Method |
|-----------|----------------|
| `Runnable` | `run()` |
| `Comparable` | `compareTo()` |
| `ActionListener` | `actionPerformed()` |
| `Callable` | `call()` |

### Example with Default Method
```java
interface Interf {
    void m1();  // Abstract method
    
    default void m2() {
        System.out.println("Default method");
    }
}
```

---

## The @FunctionalInterface Annotation

The `@FunctionalInterface` annotation indicates that an interface is intended to be a functional interface. It helps the compiler enforce the single abstract method rule.

### Valid Functional Interface
```java
@FunctionalInterface
interface Interf {
    void m1();  // Valid - exactly one abstract method
}
```

### Invalid Examples
```java
// ❌ Multiple abstract methods
@FunctionalInterface
interface Interf {
    void m1();
    void m2();  // Compilation Error
}

// ❌ No abstract methods
@FunctionalInterface
interface Interf {
    // Compilation Error
}
```

---

## Functional Interface with Inheritance

### Valid Inheritance

**Case 1: No new abstract methods**
```java
@FunctionalInterface
interface A {
    void methodOne();
}

@FunctionalInterface
interface B extends A {
    // Still a functional interface
}
```

**Case 2: Same abstract method**
```java
@FunctionalInterface
interface A {
    void methodOne();
}

@FunctionalInterface
interface B extends A {
    void methodOne();  // Same method - Valid
}
```

### Invalid Inheritance

```java
@FunctionalInterface
interface A {
    void methodOne();
}

@FunctionalInterface  // ❌ Compilation Error
interface B extends A {
    void methodTwo();  // New abstract method
}
```

**Note:** Without `@FunctionalInterface`, the above is a valid normal interface.

---

## Functional Interface vs Lambda Expressions

Functional interfaces are required to use lambda expressions.

### Without Lambda Expression
```java
interface Interf {
    void sum(int a, int b);
}

class Demo implements Interf {
    public void sum(int a, int b) {
        System.out.println("Sum: " + (a + b));
    }
}

public class Test {
    public static void main(String[] args) {
        Interf i = new Demo();
        i.sum(5, 10);
    }
}
```

### With Lambda Expression
```java
interface Interf {
    void sum(int a, int b);
}

public class Test {
    public static void main(String[] args) {
        Interf i = (a, b) -> System.out.println("Sum: " + (a + b));
        i.sum(5, 10);
    }
}
```

---

## Built-in Functional Interfaces

Java provides built-in functional interfaces in the `java.util.function` package.

### Overview of Main Categories

| Category | Input | Output | Purpose |
|----------|-------|--------|---------|
| `Supplier` | None | T | Supply values |
| `Consumer` | T | void | Consume values |
| `Function` | T | R | Transform values |
| `Predicate` | T | boolean | Test conditions |
| `UnaryOperator` | T | T | Transform (same type) |
| `BinaryOperator` | T, T | T | Combine two values |

---

## 1. Predicate\<T>

**Purpose:** Tests a condition and returns a boolean value.

**Method:** `boolean test(T t)`

### Basic Example
```java
import java.util.function.Predicate;

Predicate<Integer> isEven = n -> n % 2 == 0;
System.out.println(isEven.test(4));  // true
System.out.println(isEven.test(3));  // false
```

### Predicate Joining

Predicates can be combined using:
- `and()` - Logical AND (&&)
- `or()` - Logical OR (||)
- `negate()` - Logical NOT (!)

```java
Predicate<Integer> greaterThan10 = i -> i > 10;
Predicate<Integer> isEven = i -> i % 2 == 0;

// Combining predicates
Predicate<Integer> greaterThan10AndEven = greaterThan10.and(isEven);
Predicate<Integer> greaterThan10OrEven = greaterThan10.or(isEven);
Predicate<Integer> notGreaterThan10 = greaterThan10.negate();

System.out.println(greaterThan10AndEven.test(12));  // true
System.out.println(greaterThan10OrEven.test(5));    // false
System.out.println(notGreaterThan10.test(5));       // true
```

### Practical Example
```java
public static void filterNumbers(Predicate<Integer> p, int[] numbers) {
    for (int num : numbers) {
        if (p.test(num)) {
            System.out.println(num);
        }
    }
}

// Usage
int[] arr = {0, 5, 10, 15, 20};
filterNumbers(i -> i > 10, arr);  // Prints: 15, 20
```

---

## 2. Function\<T, R>

**Purpose:** Transforms input of type T to output of type R.

**Method:** `R apply(T t)`

### Basic Example
```java
import java.util.function.Function;

Function<String, Integer> length = s -> s.length();
System.out.println(length.apply("Hello"));  // 5
```

### Function Chaining

**andThen() - Left to Right**
```java
Function<Integer, Integer> multiply = x -> x * 2;
Function<Integer, Integer> add = x -> x + 10;

Function<Integer, Integer> combined = multiply.andThen(add);
System.out.println(combined.apply(5));  // (5 * 2) + 10 = 20
```

**compose() - Right to Left**
```java
Function<Integer, Integer> multiply = x -> x * 2;
Function<Integer, Integer> add = x -> x + 10;

Function<Integer, Integer> composed = multiply.compose(add);
System.out.println(composed.apply(5));  // (5 + 10) * 2 = 30
```

### BiFunction Example
```java
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
System.out.println(add.apply(5, 3));  // 8
```

---

## 3. Consumer\<T>

**Purpose:** Consumes a value and returns nothing (void).

**Method:** `void accept(T t)`

### Basic Example
```java
import java.util.function.Consumer;

Consumer<String> print = s -> System.out.println(s);
print.accept("Hello World");
```

### Chaining with andThen()
```java
Consumer<String> c1 = s -> System.out.println("First: " + s);
Consumer<String> c2 = s -> System.out.println("Second: " + s);

Consumer<String> combined = c1.andThen(c2);
combined.accept("Test");
// Output:
// First: Test
// Second: Test
```

### BiConsumer Example
```java
BiConsumer<String, Integer> printNameAge = (name, age) -> 
    System.out.println(name + " is " + age + " years old");
    
printNameAge.accept("Alice", 25);  // Alice is 25 years old
```

---

## 4. Supplier\<T>

**Purpose:** Supplies values without taking any input.

**Method:** `T get()`

### Basic Example
```java
import java.util.function.Supplier;

Supplier<Double> randomSupplier = () -> Math.random();
System.out.println(randomSupplier.get());  // Random number
System.out.println(randomSupplier.get());  // Different random number
```

### Practical Examples
```java
// Date supplier
Supplier<LocalDateTime> dateSupplier = () -> LocalDateTime.now();
System.out.println(dateSupplier.get());

// Object creation
Supplier<List<String>> listSupplier = () -> new ArrayList<>();
List<String> list = listSupplier.get();
```

---

## 5. UnaryOperator\<T>

**Purpose:** Special case of Function where input and output types are the same.

**Extends:** `Function<T, T>`

**Method:** `T apply(T t)`

### Basic Example
```java
UnaryOperator<Integer> square = x -> x * x;
System.out.println(square.apply(5));  // 25

UnaryOperator<String> toUpperCase = s -> s.toUpperCase();
System.out.println(toUpperCase.apply("hello"));  // HELLO
```

### List Transformation
```java
List<String> words = new ArrayList<>(Arrays.asList("apple", "banana"));
words.replaceAll(s -> s.toUpperCase());  // UnaryOperator
System.out.println(words);  // [APPLE, BANANA]
```

---

## 6. BinaryOperator\<T>

**Purpose:** Special case of BiFunction where both inputs and output are the same type.

**Extends:** `BiFunction<T, T, T>`

**Method:** `T apply(T t1, T t2)`

### Basic Examples
```java
BinaryOperator<Integer> add = (a, b) -> a + b;
BinaryOperator<Integer> max = (a, b) -> a > b ? a : b;

System.out.println(add.apply(5, 3));   // 8
System.out.println(max.apply(10, 20)); // 20
```

### Using maxBy() and minBy()
```java
BinaryOperator<String> maxByLength = BinaryOperator.maxBy(
    Comparator.comparing(String::length)
);

System.out.println(maxByLength.apply("Hi", "Hello"));  // Hello
```

### Reduce Operation
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
BinaryOperator<Integer> sum = (a, b) -> a + b;

int total = numbers.stream().reduce(0, sum);
System.out.println(total);  // 15
```

---

## Method and Constructor References

Method references provide a cleaner alternative to lambda expressions when calling existing methods.

### Method Reference Syntax

**Static Method:** `ClassName::methodName`

**Instance Method:** `objectReference::methodName`

### Static Method Reference
```java
class Test {
    public static void m1() {
        for (int i = 0; i <= 10; i++) {
            System.out.println("Child Thread");
        }
    }

    public static void main(String[] args) {
        Runnable r = Test::m1;  // Method reference
        Thread t = new Thread(r);
        t.start();
    }
}
```

### Instance Method Reference
```java
interface Interf {
    void display(int i);
}

class Test {
    public void show(int i) {
        System.out.println("Value: " + i);
    }

    public static void main(String[] args) {
        Test obj = new Test();
        Interf i = obj::show;  // Instance method reference
        i.display(20);
    }
}
```

### Constructor Reference

**Syntax:** `ClassName::new`

```java
class Sample {
    private String s;

    Sample(String s) {
        this.s = s;
        System.out.println("Constructor: " + s);
    }
}

interface Interf {
    Sample get(String s);
}

class Test {
    public static void main(String[] args) {
        Interf f = Sample::new;  // Constructor reference
        f.get("Hello");
    }
}
```

---

## Summary Tables

### Main Functional Interfaces

| Interface | Method | Input | Output | Use Case |
|-----------|--------|-------|--------|----------|
| `Supplier<T>` | `get()` | None | T | Supply values |
| `Consumer<T>` | `accept(T)` | T | void | Consume values |
| `Function<T,R>` | `apply(T)` | T | R | Transform |
| `Predicate<T>` | `test(T)` | T | boolean | Test condition |
| `UnaryOperator<T>` | `apply(T)` | T | T | Transform (same type) |
| `BinaryOperator<T>` | `apply(T,T)` | T, T | T | Combine values |

### Bi-Variants

| Interface | Method | Inputs | Output |
|-----------|--------|--------|--------|
| `BiConsumer<T,U>` | `accept(T,U)` | T, U | void |
| `BiFunction<T,U,R>` | `apply(T,U)` | T, U | R |
| `BiPredicate<T,U>` | `test(T,U)` | T, U | boolean |

### Primitive Specializations

Java provides primitive versions to avoid boxing/unboxing overhead:

| Pattern | Examples |
|---------|----------|
| `IntXxx` | `IntSupplier`, `IntConsumer`, `IntFunction<R>` |
| `LongXxx` | `LongSupplier`, `LongConsumer`, `LongFunction<R>` |
| `DoubleXxx` | `DoubleSupplier`, `DoubleConsumer`, `DoubleFunction<R>` |
| `ToIntXxx` | `ToIntFunction<T>`, `ToIntBiFunction<T,U>` |

```java
IntSupplier randomInt = () -> (int)(Math.random() * 100);
IntPredicate isEven = n -> n % 2 == 0;
IntUnaryOperator square = n -> n * n;
```

---
