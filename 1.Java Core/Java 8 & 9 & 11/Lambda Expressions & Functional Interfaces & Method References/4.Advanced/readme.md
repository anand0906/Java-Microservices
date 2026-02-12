# Lambda Variable Capture, Scope & Advanced Topics

## Table of Contents
1. [Variable Capture and Scope](#variable-capture-and-scope)
   - [Effectively Final Variables](#effectively-final-variables)
   - [Why Effectively Final?](#why-must-variables-be-effectively-final)
   - [Workarounds for Modifiable Variables](#workarounds-for-modifiable-variables)
2. [Scope and 'this' Reference](#scope-and-this-reference)
3. [Static vs Instance Context](#static-vs-instance-context)
4. [Advanced Topics](#advanced-topics)
   - [Lambda Compilation](#1-lambda-expression-compilation)
   - [Serialization](#2-serialization-of-lambdas)
   - [Exception Handling](#3-exception-handling-in-lambdas)
   - [Type Inference Limitations](#4-type-inference-limitations)
   - [Performance Considerations](#5-lambda-performance-considerations)
   - [Recursive Lambdas](#6-recursive-lambdas)
   - [Currying and Partial Application](#7-currying-and-partial-application)
   - [Higher-Order Functions](#8-higher-order-functions)
5. [Best Practices](#best-practices)

---

## Variable Capture and Scope

### Effectively Final Variables

Lambda expressions can access variables from their enclosing scope, but these variables must be **effectively final**.

**Effectively Final:** A variable whose value is never changed after initialization.

#### Examples

```java
// ✅ Valid - effectively final variable
String prefix = "Hello";

Consumer<String> greeter = name -> 
    System.out.println(prefix + " " + name);

greeter.accept("Alice");  // "Hello Alice"

// ❌ Invalid - modifying captured variable
// prefix = "Hi";  // Compilation error!
```

```java
// ✅ Captured variable cannot be modified
int multiplier = 10;
Function<Integer, Integer> multiply = n -> n * multiplier;
System.out.println(multiply.apply(5));  // 50

// ❌ Cannot reassign
// multiplier = 20;  // ERROR: Variable used in lambda must be final
```

```java
// ✅ Parameters are effectively final
public void process(String input) {
    Function<String, String> processor = s -> input + s;
    // input = "changed";  // Would cause error
}
```

```java
// ✅ Instance variables CAN be modified
class MyClass {
    private int count = 0;
    
    public void increment() {
        Runnable r = () -> {
            count++;  // OK - instance variable
            System.out.println(count);
        };
        r.run();
    }
}
```

---

### Why Must Variables Be Effectively Final?

**Reason:** Lambda expressions are closures that capture variables from the enclosing scope. For thread safety and clarity, these must be effectively final.

**Technical Explanation:** When a lambda captures a variable, it captures the variable's **value**, not a reference to the variable. If the variable could change, the lambda would have an outdated value.

```java
// This would be problematic if allowed:
int value = 10;
Runnable r = () -> System.out.println(value);

value = 20;  // If allowed...
r.run();     // Would print 10 or 20? Confusing!
```

---

### Workarounds for Modifiable Variables

#### 1. Use Array (Not Recommended)

```java
int[] counter = {0};  // Array is final, contents can change

Runnable increment = () -> {
    counter[0]++;
    System.out.println(counter[0]);
};

increment.run();  // 1
increment.run();  // 2
```

#### 2. Use Atomic Variables (Better)

```java
import java.util.concurrent.atomic.AtomicInteger;

AtomicInteger counter = new AtomicInteger(0);

Runnable increment = () -> {
    int value = counter.incrementAndGet();
    System.out.println(value);
};

increment.run();  // 1
increment.run();  // 2
```

#### 3. Use Mutable Object (Best for Complex State)

```java
class Counter {
    int value = 0;
    void increment() { value++; }
}

Counter counter = new Counter();  // Reference is final

Runnable increment = () -> {
    counter.increment();
    System.out.println(counter.value);
};

increment.run();  // 1
increment.run();  // 2
```

---

## Scope and 'this' Reference

```java
public class ScopeExample {
    private String className = "ScopeExample";
    
    public void demonstrateScope() {
        String localVar = "local";
        
        // Lambda - 'this' refers to enclosing class
        Runnable lambda = () -> {
            System.out.println(this.className);  // ScopeExample
            System.out.println(localVar);
        };
        
        // Anonymous inner class - 'this' refers to anonymous class
        Runnable anonymous = new Runnable() {
            private String className = "Anonymous";
            
            @Override
            public void run() {
                System.out.println(this.className);  // Anonymous
                System.out.println(localVar);
            }
        };
        
        lambda.run();     // ScopeExample, local
        anonymous.run();  // Anonymous, local
    }
}
```

---

## Static vs Instance Context

```java
public class ContextExample {
    private int instanceVar = 10;
    private static int staticVar = 20;
    
    // Instance method
    public void instanceMethod() {
        Runnable r = () -> {
            System.out.println(instanceVar);  // ✅ OK
            System.out.println(staticVar);    // ✅ OK
            System.out.println(this);         // ✅ OK
        };
    }
    
    // Static method
    public static void staticMethod() {
        Runnable r = () -> {
            // System.out.println(instanceVar);  // ❌ ERROR
            System.out.println(staticVar);       // ✅ OK
            // System.out.println(this);         // ❌ ERROR
        };
    }
}
```

---

## Advanced Topics

### 1. Lambda Expression Compilation

**How Lambdas Work Under the Hood:**

Java uses `invokedynamic` (added in Java 7) instead of anonymous inner classes.

**Compilation Process:**
1. Lambda expression is analyzed
2. Compiler generates a method with lambda body
3. At runtime, `invokedynamic` creates implementation
4. First call may create class (cached for subsequent calls)

**Benefits:**
- Better performance (no class loading overhead)
- Smaller memory footprint
- More optimization opportunities for JVM

```java
// Your code
Function<Integer, Integer> square = x -> x * x;

// Compiler generates (simplified):
// private static int lambda$1(int x) { return x * x; }
// Then uses invokedynamic to link it to Function interface
```

---

### 2. Serialization of Lambdas

Lambdas can be serialized if the target type implements `Serializable`.

```java
import java.io.*;
import java.util.function.Function;

// Serializable functional interface
interface SerializableFunction<T, R> 
    extends Function<T, R>, Serializable {
}

public class LambdaSerialization {
    public static void main(String[] args) throws Exception {
        SerializableFunction<String, Integer> lengthFunc = 
            (SerializableFunction<String, Integer> & Serializable) 
            String::length;
        
        // Serialize
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(baos);
        oos.writeObject(lengthFunc);
        
        // Deserialize
        ByteArrayInputStream bais = 
            new ByteArrayInputStream(baos.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bais);
        SerializableFunction<String, Integer> deserialized = 
            (SerializableFunction<String, Integer>) ois.readObject();
        
        System.out.println(deserialized.apply("Hello"));  // 5
    }
}
```

**Warning:** Captured variables must also be serializable!

---

### 3. Exception Handling in Lambdas

Standard functional interfaces don't declare checked exceptions.

#### Problem
```java
List<String> files = Arrays.asList("file1.txt", "file2.txt");

// ❌ Won't compile - IOException not handled
// files.forEach(file -> {
//     BufferedReader reader = new BufferedReader(new FileReader(file));
// });
```

#### Solution 1: Wrap in Unchecked Exception
```java
files.forEach(file -> {
    try {
        BufferedReader reader = new BufferedReader(new FileReader(file));
        // process file
    } catch (IOException e) {
        throw new UncheckedIOException(e);
    }
});
```

#### Solution 2: Custom Functional Interface
```java
@FunctionalInterface
interface CheckedFunction<T, R> {
    R apply(T t) throws Exception;
}
```

#### Solution 3: Wrapper Method
```java
public static <T, R> Function<T, R> wrap(CheckedFunction<T, R> func) {
    return t -> {
        try {
            return func.apply(t);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    };
}

// Usage
Function<String, String> readFile = wrap(filename -> {
    return Files.readString(Paths.get(filename));
});
```

---

### 4. Type Inference Limitations

Sometimes the compiler cannot infer types and requires explicit casts.

```java
// Ambiguous - which overload?
// void process(Function<String, String> f) { }
// void process(UnaryOperator<String> f) { }

// Solution 1: Explicit cast
process((Function<String, String>) s -> s.toUpperCase());

// Solution 2: Explicit type in lambda
process((String s) -> s.toUpperCase());
```

---

### 5. Lambda Performance Considerations

**Performance Characteristics:**

| Approach | Speed | Notes |
|----------|-------|-------|
| Lambda | Fast | Similar to method call after JIT |
| Anonymous Class | Slower | Class loading overhead |
| Direct Method | Fastest | No indirection |

**Guidelines:**
- ✅ Use lambdas freely - performance is good
- ✅ JVM optimizes lambdas aggressively
- ⚠️ Very hot paths might benefit from direct methods
- ❌ Don't avoid lambdas for performance without profiling

---

### 6. Recursive Lambdas

```java
// ❌ Simple recursion doesn't work directly
// Function<Integer, Integer> factorial = 
//     n -> n == 0 ? 1 : n * factorial.apply(n - 1);  // Error!

// ✅ Solution: Use array trick
Function<Integer, Integer>[] factorial = new Function[1];
factorial[0] = n -> n == 0 ? 1 : n * factorial[0].apply(n - 1);

System.out.println(factorial[0].apply(5));  // 120
```

---

### 7. Currying and Partial Application

**Currying:** Transform a function with multiple arguments into a sequence of functions with single arguments.

```java
// Traditional two-argument function
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;

// Curried version
Function<Integer, Function<Integer, Integer>> curriedAdd = 
    a -> b -> a + b;

// Usage
Function<Integer, Integer> add5 = curriedAdd.apply(5);
System.out.println(add5.apply(3));   // 8
System.out.println(add5.apply(10));  // 15
```

**Partial Application:**
```java
class MathOps {
    static Function<Integer, Integer> addN(int n) {
        return x -> x + n;
    }
}

Function<Integer, Integer> add10 = MathOps.addN(10);
System.out.println(add10.apply(5));  // 15
```

---

### 8. Higher-Order Functions

Functions that take functions as parameters or return functions.

#### Function that Returns Function
```java
public static Function<Integer, Integer> createMultiplier(int factor) {
    return n -> n * factor;
}

Function<Integer, Integer> double = createMultiplier(2);
System.out.println(double.apply(5));  // 10
```

#### Function that Takes Function as Parameter
```java
public static <T> List<T> filter(List<T> list, Predicate<T> predicate) {
    return list.stream()
        .filter(predicate)
        .collect(Collectors.toList());
}

List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
List<Integer> evens = filter(numbers, n -> n % 2 == 0);
```

#### Function Composition
```java
public static <T, U, V> Function<T, V> compose(
    Function<T, U> f1,
    Function<U, V> f2
) {
    return f1.andThen(f2);
}

Function<Integer, Integer> addOne = x -> x + 1;
Function<Integer, Integer> multiplyTwo = x -> x * 2;
Function<Integer, Integer> combined = compose(addOne, multiplyTwo);

System.out.println(combined.apply(5));  // (5 + 1) * 2 = 12
```

---

## Best Practices

### 1. Keep Lambdas Short and Simple

**Rule of Thumb:** If lambda exceeds 3 lines, consider extracting to a method.

```java
// ✅ GOOD - Short and clear
list.forEach(System.out::println);

// ❌ BAD - Too complex
list.stream().filter(s -> {
    if (s == null || s.isEmpty()) return false;
    if (!s.matches("[a-zA-Z]+")) return false;
    // ... many more lines
    return true;
}).collect(Collectors.toList());

// ✅ BETTER - Extract to method
list.stream().filter(this::isValidName).collect(Collectors.toList());
```

---

### 2. Prefer Method References When Appropriate

```java
// ✅ GOOD
list.forEach(System.out::println);

// ❌ LESS CLEAR
list.forEach(s -> System.out.println(s));
```

---

### 3. Use Descriptive Parameter Names

```java
// ❌ BAD
list.stream().filter(x -> x.length() > 5);

// ✅ GOOD
list.stream().filter(name -> name.length() > 5);
```

---

### 4. Avoid Side Effects

```java
// ❌ BAD - side effect
List<Integer> results = new ArrayList<>();
stream.forEach(n -> results.add(n * 2));

// ✅ GOOD - functional approach
List<Integer> results = stream
    .map(n -> n * 2)
    .collect(Collectors.toList());
```

---

### 5. Choose Right Functional Interface

```java
// ❌ BAD
Function<String, Boolean> validator = s -> s.length() > 5;

// ✅ GOOD
Predicate<String> validator = s -> s.length() > 5;
```

---

### 6. Consider Readability Over Cleverness

```java
// ❌ CLEVER but hard to read
Function<Integer, Function<Integer, Integer>> add = a -> b -> a + b;

// ✅ BETTER - clearer intent
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
```

---

### 7. Handle Nulls Appropriately

```java
// ❌ BAD - NPE risk
Function<String, Integer> length = s -> s.length();

// ✅ GOOD - null-safe
Function<String, Integer> length = s -> s == null ? 0 : s.length();

// ✅ BETTER - use Optional
Function<String, Optional<Integer>> safeLength = 
    s -> Optional.ofNullable(s).map(String::length);
```

---

### 8. Use Local Variables for Complex Expressions

```java
// ❌ BAD - hard to read
list.stream()
    .filter(s -> s != null && !s.isEmpty() && s.length() > 3)
    .collect(Collectors.toList());

// ✅ BETTER
Predicate<String> notEmpty = s -> s != null && !s.isEmpty();
Predicate<String> longEnough = s -> s.length() > 3;

list.stream()
    .filter(notEmpty.and(longEnough))
    .collect(Collectors.toList());
```

---

### 9. Be Careful with Primitive Types

```java
// ❌ INEFFICIENT - boxing/unboxing
Function<Integer, Integer> square = n -> n * n;

// ✅ EFFICIENT - primitive specialization
IntUnaryOperator square = n -> n * n;
```

---

### 10. Document Complex Lambdas

```java
// When lambda isn't obvious, add comment
Function<String, String> complexTransform = input -> {
    // Remove special characters, convert to lowercase,
    // replace spaces with underscores
    return input.replaceAll("[^a-zA-Z0-9\\s]", "")
                .toLowerCase()
                .replace(" ", "_");
};
```

---

### Summary of Key Takeaways

| Topic | Key Point |
|-------|-----------|
| **Variable Capture** | Must be effectively final |
| **'this' Reference** | Refers to enclosing class (not lambda itself) |
| **Instance Variables** | Can be modified (not captured by value) |
| **Performance** | Lambdas are fast (invokedynamic optimization) |
| **Exceptions** | Wrap checked exceptions or use custom interfaces |
| **Best Length** | Keep under 3 lines; extract to method if longer |
| **Readability** | Prefer method references when appropriate |
| **Side Effects** | Avoid them; use functional approaches |

---
