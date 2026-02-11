# Java Method References

## Table of Contents
1. [Introduction](#introduction)
2. [What are Method References?](#what-are-method-references)
3. [Types of Method References](#types-of-method-references)
   - [Static Method Reference](#1-static-method-reference)
   - [Instance Method of Particular Object](#2-instance-method-of-particular-object)
   - [Instance Method of Arbitrary Object](#3-instance-method-of-arbitrary-object-of-particular-type)
   - [Constructor Reference](#4-constructor-reference)
4. [Method Reference Summary](#method-reference-summary)
5. [When to Use Method References vs Lambdas](#when-to-use-method-references-vs-lambdas)

---

## Introduction

Method references are a shorthand notation for lambda expressions that call a specific method. They provide a cleaner and more readable way to refer to existing methods.

---

## What are Method References?

A **method reference** is a simplified form of a lambda expression that refers to an existing method by name.

### Syntax
```
ClassName::methodName
```
or
```
instance::methodName
```

### When to Use
- When lambda just calls a method
- Makes code more readable
- Reduces verbosity

### Key Requirement
The parameter types of the functional interface method and the referenced method **must match**. Return types and method names do not need to match.

---

## Types of Method References

### 1. Static Method Reference

**Syntax:** `ClassName::staticMethodName`

**Lambda Equivalent:** `(args) -> ClassName.staticMethodName(args)`

#### Examples

```java
// Integer.parseInt
Function<String, Integer> parser = Integer::parseInt;
System.out.println(parser.apply("123"));  // 123

// Math.sqrt
Function<Double, Double> sqrt = Math::sqrt;
System.out.println(sqrt.apply(16.0));  // 4.0

// String.valueOf
Function<Integer, String> toString = String::valueOf;
System.out.println(toString.apply(42));  // "42"
```

#### Custom Static Method
```java
class MathUtils {
    public static int add(int a, int b) {
        return a + b;
    }
}

BinaryOperator<Integer> adder = MathUtils::add;
System.out.println(adder.apply(5, 3));  // 8
```

---

### 2. Instance Method of Particular Object

**Syntax:** `instance::instanceMethodName`

**Lambda Equivalent:** `(args) -> instance.instanceMethodName(args)`

This refers to a method on a **specific instance** of an object.

#### Examples

```java
// System.out.println
Consumer<String> printer = System.out::println;
printer.accept("Hello");  // Prints "Hello"

// Specific String instance
String str = "Hello";
Supplier<Integer> lengthGetter = str::length;
System.out.println(lengthGetter.get());  // 5

// StringBuilder.append
StringBuilder sb = new StringBuilder();
Consumer<String> appender = sb::append;
appender.accept("Hello");
appender.accept(" World");
System.out.println(sb.toString());  // "Hello World"
```

#### Custom Object
```java
class Printer {
    public void print(String message) {
        System.out.println("Printing: " + message);
    }
}

Printer printer = new Printer();
Consumer<String> printMethod = printer::print;
printMethod.accept("Test");  // "Printing: Test"
```

---

### 3. Instance Method of Arbitrary Object of Particular Type

**Syntax:** `ClassName::instanceMethodName`

**Lambda Equivalent:** `(obj, args) -> obj.instanceMethodName(args)`

**Important:** The first parameter becomes the instance on which the method is called.

#### Examples

```java
// String.toUpperCase
Function<String, String> upper = String::toUpperCase;
System.out.println(upper.apply("hello"));  // "HELLO"

// String.length
Function<String, Integer> length = String::length;
System.out.println(length.apply("Hello"));  // 5

// String.compareTo (BiFunction)
BiFunction<String, String, Integer> compare = String::compareTo;
System.out.println(compare.apply("apple", "banana"));  // negative number
```

#### Stream Usage
```java
List<String> words = Arrays.asList("apple", "banana", "cherry");

// Using method reference in stream
words.stream()
    .map(String::toUpperCase)
    .forEach(System.out::println);
```

#### Sorting with Comparator
```java
List<String> words = Arrays.asList("banana", "apple", "cherry");

// Method reference version
words.sort(String::compareToIgnoreCase);
System.out.println(words);  // [apple, banana, cherry]
```

#### Custom Class Example
```java
class Person {
    String name;
    int age;
    
    Person(String name, int age) { 
        this.name = name; 
        this.age = age; 
    }
    
    public String getName() { return name; }
}

List<Person> people = Arrays.asList(
    new Person("Alice", 25),
    new Person("Bob", 30)
);

// Extract names using method reference
List<String> names = people.stream()
    .map(Person::getName)
    .collect(Collectors.toList());
```

---

### 4. Constructor Reference

**Syntax:** `ClassName::new`

**Lambda Equivalent:** `(args) -> new ClassName(args)`

#### Examples

```java
// No-arg constructor
Supplier<ArrayList<String>> listCreator = ArrayList::new;
List<String> list = listCreator.get();

// One-arg constructor
Function<Integer, ArrayList<String>> listWithSize = ArrayList::new;
List<String> list2 = listWithSize.apply(10);

// String constructor
Function<String, String> stringCreator = String::new;
```

#### Custom Class Constructors
```java
class Person {
    String name;
    int age;
    
    Person() { 
        this.name = "Unknown"; 
        this.age = 0; 
    }
    
    Person(String name) { 
        this.name = name; 
        this.age = 0; 
    }
    
    Person(String name, int age) { 
        this.name = name; 
        this.age = age; 
    }
}

// No-arg constructor
Supplier<Person> personCreator = Person::new;

// One-arg constructor
Function<String, Person> personFromName = Person::new;

// Two-arg constructor
BiFunction<String, Integer, Person> personCreator2 = Person::new;
Person p = personCreator2.apply("Alice", 25);
```

#### Array Constructor Reference
```java
// Array creation
IntFunction<String[]> arrayCreator = String[]::new;
String[] array = arrayCreator.apply(10);  // Creates String[10]

// Using in stream
List<String> list = Arrays.asList("A", "B", "C");
String[] array2 = list.stream().toArray(String[]::new);
```

#### Stream Constructor Usage
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// Create Person objects using constructor reference
List<Person> people = names.stream()
    .map(Person::new)
    .collect(Collectors.toList());
```

---

## Method Reference Summary

| Type | Syntax | Lambda Equivalent | Example |
|------|--------|-------------------|---------|
| **Static** | `Class::staticMethod` | `(args) -> Class.staticMethod(args)` | `Integer::parseInt` |
| **Instance (bound)** | `instance::method` | `(args) -> instance.method(args)` | `System.out::println` |
| **Instance (unbound)** | `Class::instanceMethod` | `(obj, args) -> obj.instanceMethod(args)` | `String::toUpperCase` |
| **Constructor** | `Class::new` | `(args) -> new Class(args)` | `ArrayList::new` |
| **Array Constructor** | `Type[]::new` | `size -> new Type[size]` | `String[]::new` |

---

## When to Use Method References vs Lambdas

### ✅ Use Method References When:

1. Lambda only calls one method
2. Method signature matches functional interface
3. Improves readability

```java
// GOOD - method reference is clearer
list.forEach(System.out::println);

// LESS CLEAR - lambda adds no value
list.forEach(s -> System.out.println(s));
```

### ❌ Use Lambdas When:

1. Need to do more than call a single method
2. Need to manipulate arguments before passing
3. Method reference would be less clear

```java
// GOOD - lambda necessary for transformation
list.stream()
    .map(s -> s.toUpperCase() + "!")
    .forEach(System.out::println);

// GOOD - lambda needed for condition
list.stream()
    .filter(s -> s.length() > 5 && s.startsWith("A"))
    .forEach(System.out::println);
```

### Comparison Example
```java
// Lambda vs Method Reference
class Test {
    public static void m1() {
        for (int i = 0; i <= 10; i++) {
            System.out.println("Child Thread");
        }
    }

    public static void main(String[] args) {
        // With Lambda
        Runnable r1 = () -> {
            for (int i = 0; i <= 10; i++) {
                System.out.println("Child Thread");
            }
        };
        
        // With Method Reference (cleaner)
        Runnable r2 = Test::m1;
        
        Thread t = new Thread(r2);
        t.start();
    }
}
```

---

### Key Benefits

- **Code Reusability:** Use already existing methods/constructors
- **Conciseness:** More compact than lambda expressions
- **Readability:** Intent is clearer when referring to named methods
- **Maintainability:** Changes to method logic automatically reflected

---
