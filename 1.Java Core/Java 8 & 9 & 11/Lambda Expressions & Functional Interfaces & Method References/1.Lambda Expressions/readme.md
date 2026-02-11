
# Java Lambda Expressions

## Table of Contents
1. [Introduction](#introduction)
2. [What is a Lambda Expression?](#what-is-a-lambda-expression)
3. [Why Lambda Expressions?](#why-lambda-expressions)
4. [Basic Syntax](#basic-syntax)
5. [Syntax Variations](#syntax-variations)
6. [Lambda Syntax Rules](#lambda-syntax-rules)
7. [Lambda vs Anonymous Inner Class](#lambda-vs-anonymous-inner-class)
8. [Benefits](#benefits)

---

## Introduction

Lambda expressions in Java are a concise way to express instances of single-method interfaces (functional interfaces). They provide a way to write anonymous methods in a functional style, reducing boilerplate code and improving readability.

**Definition:** A lambda expression is an anonymous (nameless) function that does not have a name, return type, or access modifiers.

---

## What is a Lambda Expression?

A lambda expression is a concise way to represent an anonymous function. It was introduced in Java 8 as part of the move toward functional programming.

**Simple Definition:** A lambda is a block of code that can be passed around as if it were data.

### Key Characteristics
- **Anonymous:** No name, defined inline
- **Function:** Has parameters, body, and return type (can be inferred)
- **Passed around:** Can be passed as argument or assigned to variable
- **Concise:** Reduces boilerplate code significantly

---

## Why Lambda Expressions?

**Before Java 8 (Verbose):**
```java
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello from thread");
    }
});
```

**With Java 8 Lambda (Concise):**
```java
Thread thread = new Thread(() -> System.out.println("Hello from thread"));
```

---

## Basic Syntax

### Structure
```
(parameters) -> expression
```
or
```
(parameters) -> { statements; }
```

### Components
- **Parameter list:** Input parameters in parentheses
- **Arrow token:** `->` separates parameters from body
- **Body:** Expression or block of statements

### Transformation Example

**Traditional Method:**
```java
public void m1() {
    System.out.println("Hello");
}
```

**Lambda Expression:**
```java
() -> System.out.println("Hello");
```

---

## Syntax Variations

### 1. No Parameters
```java
// Expression body
Runnable r = () -> System.out.println("Hello");

// Block body
Runnable r2 = () -> {
    System.out.println("Line 1");
    System.out.println("Line 2");
};
```

### 2. One Parameter
```java
// Parentheses optional
Consumer<String> print1 = s -> System.out.println(s);

// With parentheses
Consumer<String> print2 = (s) -> System.out.println(s);

// With type (parentheses required)
Consumer<String> print3 = (String s) -> System.out.println(s);
```

### 3. Multiple Parameters
```java
// Parentheses required
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;

// With types
BiFunction<Integer, Integer, Integer> multiply = 
    (Integer a, Integer b) -> a * b;
```

### 4. Type Inference
```java
// Compiler infers types
List<String> names = Arrays.asList("Alice", "Bob");
names.forEach(name -> System.out.println(name));

// Explicit types (verbose)
names.forEach((String name) -> System.out.println(name));
```

### 5. Expression vs Block Body
```java
// Expression body - implicit return
Function<Integer, Integer> square1 = x -> x * x;

// Block body - explicit return required
Function<Integer, Integer> square2 = x -> {
    return x * x;
};
```

---

## Lambda Syntax Rules

| Rule | Example | Valid? |
|------|---------|--------|
| No params need `()` | `() -> 42` | ✅ |
| Single param no `()` | `x -> x * 2` | ✅ |
| Single param with `()` | `(x) -> x * 2` | ✅ |
| Multiple params need `()` | `(x, y) -> x + y` | ✅ |
| Block needs `{}` | `x -> { return x * 2; }` | ✅ |
| Expression no `{}` | `x -> x * 2` | ✅ |
| Block needs return | `x -> { x * 2 }` | ❌ |
| Expression has implicit return | `x -> x * 2` | ✅ |
| All types or no types | `(int x, int y) -> x + y` | ✅ |
| Mixed types | `(int x, y) -> x + y` | ❌ |

### Key Points
1. Zero or more parameters are allowed
2. Type inference is supported
3. Multiple parameters must be comma-separated
4. Empty `()` required for no parameters
5. Single parameter can omit `()` (if type not specified)
6. Multiple statements require `{}`
7. Requires functional interfaces (exactly one abstract method)

---

## Lambda vs Anonymous Inner Class

| Feature | Lambda Expression | Anonymous Inner Class |
|---------|------------------|----------------------|
| Syntax | Concise | Verbose |
| `this` reference | Enclosing class | Anonymous class |
| Compilation | `invokedynamic` | New class file |
| Performance | Better | Slower |
| Scope | Lexical scope | Own scope |
| Variables | Only effectively final | Can have instance variables |
| Restrictions | Only functional interfaces | Any interface/abstract class |

### Example
```java
public class LambdaVsAnonymous {
    private String message = "Outer class";
    
    public void demo() {
        // Anonymous inner class
        Runnable r1 = new Runnable() {
            private String message = "Inner class";
            public void run() {
                System.out.println(this.message);  // "Inner class"
            }
        };
        
        // Lambda expression
        Runnable r2 = () -> {
            System.out.println(this.message);  // "Outer class"
        };
    }
}
```

---

## Benefits

1. **Reduced Boilerplate:** Less code to write and maintain
2. **Improved Readability:** Intent is clearer
3. **Functional Programming:** Enables functional style in Java
4. **Better Performance:** `invokedynamic` is more efficient
5. **Parallel Processing:** Works seamlessly with Streams API
6. **Better API Design:** Encourages behavior parameterization

### Example: Functional Interface
```java
@FunctionalInterface
interface MyFunctionalInterface {
    void display();
}

public class LambdaDemo {
    public static void main(String[] args) {
        MyFunctionalInterface obj = () -> System.out.println("Hello, Lambda!");
        obj.display();
    }
}
```

---
