# Java Lambda Expressions 

## Quick Reference Guide

---

## 1. Lambda Expressions Basics

### What is a Lambda?
- Anonymous function (no name, no return type, no access modifiers)
- Concise way to represent functional interfaces
- Enables functional programming in Java

### Basic Syntax
```java
(parameters) -> expression
(parameters) -> { statements; }
```

### Key Components
1. **Parameter list:** `(a, b)` or `a` (single param)
2. **Arrow token:** `->`
3. **Body:** Expression or block

### Syntax Rules Quick Reference

| Rule | Valid | Invalid |
|------|-------|---------|
| No params | `() -> 42` | `-> 42` |
| Single param | `x -> x * 2` or `(x) -> x * 2` | - |
| Multiple params | `(x, y) -> x + y` | `x, y -> x + y` |
| Type inference | `(a, b) -> a + b` | `(int a, b) -> a + b` (mixed) |
| Expression body | `x -> x * 2` | - |
| Block body | `x -> { return x * 2; }` | `x -> { x * 2 }` (no return) |

---

## 2. Functional Interfaces

### Definition
- Interface with **exactly one abstract method** (SAM)
- Can have default and static methods
- Required for lambda expressions

### @FunctionalInterface Annotation
```java
@FunctionalInterface
interface MyInterface {
    void doSomething();  // Only one abstract method
    default void m2() { }  // OK - default method
    static void m3() { }   // OK - static method
}
```

### Inheritance Rules
✅ **Valid:**
```java
@FunctionalInterface
interface A { void m1(); }

@FunctionalInterface
interface B extends A { }  // No new abstract method
```

❌ **Invalid:**
```java
@FunctionalInterface
interface B extends A { 
    void m2();  // ERROR - two abstract methods
}
```

---

## 3. Built-in Functional Interfaces

### Main Categories

| Interface | Method | Input | Output | Use Case |
|-----------|--------|-------|--------|----------|
| `Supplier<T>` | `get()` | None | T | Supply values |
| `Consumer<T>` | `accept(T)` | T | void | Consume values |
| `Function<T,R>` | `apply(T)` | T | R | Transform |
| `Predicate<T>` | `test(T)` | T | boolean | Test condition |
| `UnaryOperator<T>` | `apply(T)` | T | T | Same type transform |
| `BinaryOperator<T>` | `apply(T,T)` | T, T | T | Combine values |

### 1. Predicate<T> - Test Conditions
```java
Predicate<Integer> isEven = n -> n % 2 == 0;
isEven.test(4);  // true
```

**Combining Predicates:**
```java
Predicate<Integer> p1 = n -> n > 10;
Predicate<Integer> p2 = n -> n % 2 == 0;

p1.and(p2)     // Logical AND
p1.or(p2)      // Logical OR
p1.negate()    // Logical NOT
```

### 2. Function<T, R> - Transform Values
```java
Function<String, Integer> length = s -> s.length();
length.apply("Hello");  // 5
```

**Chaining Functions:**
```java
Function<Integer, Integer> f1 = x -> x * 2;
Function<Integer, Integer> f2 = x -> x + 10;

f1.andThen(f2)  // (x * 2) + 10 - Left to right
f1.compose(f2)  // (x + 10) * 2 - Right to left
```

### 3. Consumer<T> - Consume Values
```java
Consumer<String> print = s -> System.out.println(s);
print.accept("Hello");
```

**Chaining Consumers:**
```java
Consumer<String> c1 = s -> System.out.println("A: " + s);
Consumer<String> c2 = s -> System.out.println("B: " + s);
c1.andThen(c2).accept("Test");
```

### 4. Supplier<T> - Supply Values
```java
Supplier<Double> random = () -> Math.random();
random.get();  // Random number
```

### 5. UnaryOperator<T> - Same Type Transform
```java
UnaryOperator<Integer> square = x -> x * x;
square.apply(5);  // 25
```

### 6. BinaryOperator<T> - Combine Two Values
```java
BinaryOperator<Integer> add = (a, b) -> a + b;
add.apply(5, 3);  // 8

BinaryOperator<Integer> max = BinaryOperator.maxBy(Integer::compareTo);
max.apply(10, 20);  // 20
```

### Bi-Variants (Two Parameters)

| Interface | Method | Example |
|-----------|--------|---------|
| `BiConsumer<T,U>` | `accept(T,U)` | `(k,v) -> map.put(k,v)` |
| `BiFunction<T,U,R>` | `apply(T,U)` | `(a,b) -> a + b` |
| `BiPredicate<T,U>` | `test(T,U)` | `(s1,s2) -> s1.equals(s2)` |

### Primitive Specializations
Avoid boxing/unboxing overhead:

```java
IntSupplier        // () -> int
IntConsumer        // int -> void
IntFunction<R>     // int -> R
ToIntFunction<T>   // T -> int
IntUnaryOperator   // int -> int
IntBinaryOperator  // (int, int) -> int
IntPredicate       // int -> boolean

// Same pattern for Long and Double
```

---

## 4. Method References

### Types of Method References

| Type | Syntax | Lambda Equivalent | Example |
|------|--------|-------------------|---------|
| **Static** | `Class::staticMethod` | `(args) -> Class.staticMethod(args)` | `Integer::parseInt` |
| **Instance (bound)** | `instance::method` | `(args) -> instance.method(args)` | `System.out::println` |
| **Instance (unbound)** | `Class::instanceMethod` | `(obj, args) -> obj.instanceMethod(args)` | `String::toUpperCase` |
| **Constructor** | `Class::new` | `(args) -> new Class(args)` | `ArrayList::new` |
| **Array Constructor** | `Type[]::new` | `size -> new Type[size]` | `String[]::new` |

### Examples

```java
// Static method reference
Function<String, Integer> parser = Integer::parseInt;

// Instance method reference (bound)
Consumer<String> printer = System.out::println;

// Instance method reference (unbound)
Function<String, String> upper = String::toUpperCase;

// Constructor reference
Supplier<List<String>> listCreator = ArrayList::new;

// Array constructor reference
IntFunction<String[]> arrayCreator = String[]::new;
```

### When to Use
✅ **Use Method Reference:**
- Lambda only calls one method
- Method signature matches
- Improves readability

❌ **Use Lambda:**
- Need to manipulate arguments
- Multiple operations
- Method reference less clear

---

## 5. Variable Capture and Scope

### Effectively Final Rule
Variables used in lambdas must be **effectively final** (never reassigned).

```java
// ✅ Valid
String prefix = "Hello";
Consumer<String> c = s -> System.out.println(prefix + s);

// ❌ Invalid
String prefix = "Hello";
Consumer<String> c = s -> System.out.println(prefix + s);
prefix = "Hi";  // ERROR - not effectively final
```

### What Can Be Accessed?

| Variable Type | Can Access? | Can Modify? |
|---------------|-------------|-------------|
| Local variable | ✅ Yes | ❌ No (must be effectively final) |
| Parameter | ✅ Yes | ❌ No (must be effectively final) |
| Instance variable | ✅ Yes | ✅ Yes |
| Static variable | ✅ Yes | ✅ Yes |

### 'this' Reference

```java
class MyClass {
    private String name = "MyClass";
    
    public void demo() {
        // Lambda: 'this' refers to MyClass instance
        Runnable r1 = () -> System.out.println(this.name);
        
        // Anonymous class: 'this' refers to anonymous class
        Runnable r2 = new Runnable() {
            private String name = "Anonymous";
            public void run() {
                System.out.println(this.name);  // "Anonymous"
            }
        };
    }
}
```

### Workarounds for Mutable State

```java
// ❌ Won't work - not effectively final
int counter = 0;
list.forEach(item -> counter++);  // ERROR

// ✅ Solution 1: Atomic variable
AtomicInteger counter = new AtomicInteger(0);
list.forEach(item -> counter.incrementAndGet());

// ✅ Solution 2: Mutable object
class Counter { int value = 0; }
Counter counter = new Counter();
list.forEach(item -> counter.value++);

// ✅ Solution 3: Use streams (best)
int sum = list.stream().mapToInt(Integer::intValue).sum();
```

---

## 6. Lambda vs Anonymous Inner Class

| Feature | Lambda | Anonymous Class |
|---------|--------|-----------------|
| Syntax | Concise | Verbose |
| 'this' | Enclosing class | Anonymous class itself |
| Compilation | `invokedynamic` | New class file |
| Performance | Better | Slower (class loading) |
| Scope | Lexical | Own scope |
| Variables | Only effectively final | Can have instance variables |
| Usage | Functional interfaces only | Any interface/abstract class |

---

## 7. Advanced Topics - Quick Notes

### Lambda Compilation
- Uses `invokedynamic` (not anonymous classes)
- Better performance, smaller memory footprint
- JVM optimizes aggressively

### Serialization
```java
interface SerializableFunction<T,R> 
    extends Function<T,R>, Serializable { }

SerializableFunction<String, Integer> f = String::length;
// Can serialize now
```

### Exception Handling
```java
// Standard interfaces don't allow checked exceptions
// Solution: Wrap in unchecked exception
files.forEach(file -> {
    try {
        // code that throws IOException
    } catch (IOException e) {
        throw new UncheckedIOException(e);
    }
});
```

### Currying
```java
// Transform multi-argument function to chain of single-argument functions
Function<Integer, Function<Integer, Integer>> curriedAdd = 
    a -> b -> a + b;

Function<Integer, Integer> add5 = curriedAdd.apply(5);
add5.apply(3);  // 8
```

### Higher-Order Functions
```java
// Function that returns function
public static Function<Integer, Integer> createMultiplier(int n) {
    return x -> x * n;
}

// Function that takes function as parameter
public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    return list.stream().filter(p).collect(Collectors.toList());
}
```

---

## 8. Best Practices - Quick Checklist

### ✅ DO

1. **Keep lambdas short** (3 lines max)
   ```java
   list.forEach(System.out::println);
   ```

2. **Use method references when appropriate**
   ```java
   list.stream().map(String::toUpperCase)
   ```

3. **Use descriptive names**
   ```java
   filter(name -> name.length() > 5)  // Not: filter(x -> x.length() > 5)
   ```

4. **Avoid side effects**
   ```java
   // Good: Use collect
   List<Integer> result = stream.map(n -> n * 2).collect(Collectors.toList());
   ```

5. **Choose right functional interface**
   ```java
   Predicate<String> validator = s -> s.length() > 5;  // Not Function<String, Boolean>
   ```

6. **Handle nulls**
   ```java
   Function<String, Integer> length = s -> s == null ? 0 : s.length();
   ```

7. **Use primitive specializations**
   ```java
   IntUnaryOperator square = n -> n * n;  // Not Function<Integer, Integer>
   ```

### ❌ DON'T

1. **Complex logic in lambdas**
   ```java
   // Bad - extract to method
   list.filter(s -> { /* 10 lines of code */ });
   ```

2. **Unnecessary lambdas**
   ```java
   // Bad
   list.forEach(s -> System.out.println(s));
   
   // Good
   list.forEach(System.out::println);
   ```

3. **Side effects with captured variables**
   ```java
   // Bad
   List<Integer> results = new ArrayList<>();
   stream.forEach(n -> results.add(n));  // Side effect
   ```

4. **Overly clever code**
   ```java
   // Bad - hard to read
   Function<Integer, Function<Integer, Integer>> add = a -> b -> a + b;
   
   // Good
   BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
   ```

---

## 9. Common Patterns

### Filtering
```java
List<String> filtered = list.stream()
    .filter(s -> s.length() > 5)
    .collect(Collectors.toList());
```

### Mapping
```java
List<Integer> lengths = list.stream()
    .map(String::length)
    .collect(Collectors.toList());
```

### Reducing
```java
int sum = list.stream()
    .reduce(0, (a, b) -> a + b);

// Or use method reference
int sum = list.stream()
    .reduce(0, Integer::sum);
```

### Sorting
```java
list.sort((s1, s2) -> s1.compareTo(s2));
// Or
list.sort(String::compareTo);
```

### ForEach
```java
list.forEach(System.out::println);
```

### Grouping
```java
Map<Integer, List<String>> grouped = list.stream()
    .collect(Collectors.groupingBy(String::length));
```

---

## 10. Quick Syntax Reference

### Lambda Expressions
```java
// No parameters
() -> System.out.println("Hello")
() -> { return 42; }

// One parameter
x -> x * 2
(x) -> x * 2
(Integer x) -> x * 2

// Multiple parameters
(a, b) -> a + b
(int a, int b) -> a + b
(a, b) -> { return a + b; }
```

### Functional Interfaces
```java
Supplier<T>           // () -> T
Consumer<T>           // T -> void
Function<T, R>        // T -> R
Predicate<T>          // T -> boolean
UnaryOperator<T>      // T -> T
BinaryOperator<T>     // (T, T) -> T
```

### Method References
```java
Class::staticMethod          // Static method
instance::method             // Instance method (bound)
Class::instanceMethod        // Instance method (unbound)
Class::new                   // Constructor
Type[]::new                  // Array constructor
```

---

## 11. Interview Quick Tips

### Key Points to Remember

1. **Functional Interface = One abstract method**
   - Can have default and static methods
   - `@FunctionalInterface` annotation enforces this

2. **Effectively Final Rule**
   - Local variables used in lambdas cannot be reassigned
   - Instance/static variables can be modified

3. **'this' Reference**
   - In lambda: refers to enclosing class
   - In anonymous class: refers to anonymous class itself

4. **Performance**
   - Lambdas use `invokedynamic` (better than anonymous classes)
   - No class loading overhead
   - JVM optimizes well

5. **Method References**
   - Shorthand for lambdas that just call a method
   - Four types: static, instance bound, instance unbound, constructor

6. **Exception Handling**
   - Standard interfaces don't allow checked exceptions
   - Must wrap in unchecked exceptions or create custom interface

7. **Common Mistakes**
   - Modifying captured local variables
   - Using wrong functional interface
   - Too complex logic in lambdas
   - Not using method references when appropriate

---

## 12. Code Examples - Must Know

### Example 1: Filtering and Mapping
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

List<String> result = names.stream()
    .filter(name -> name.length() > 3)
    .map(String::toUpperCase)
    .collect(Collectors.toList());
// Result: [ALICE, CHARLIE]
```

### Example 2: Predicate Chaining
```java
Predicate<Integer> greaterThan10 = n -> n > 10;
Predicate<Integer> lessThan20 = n -> n < 20;
Predicate<Integer> between10And20 = greaterThan10.and(lessThan20);

System.out.println(between10And20.test(15));  // true
```

### Example 3: Custom Sorting
```java
List<Person> people = Arrays.asList(
    new Person("Alice", 25),
    new Person("Bob", 30)
);

// Sort by age
people.sort((p1, p2) -> Integer.compare(p1.age, p2.age));
// Or
people.sort(Comparator.comparing(Person::getAge));
```

### Example 4: GroupingBy
```java
Map<Integer, List<String>> grouped = words.stream()
    .collect(Collectors.groupingBy(String::length));
// Groups words by their length
```

### Example 5: Reducing
```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);
// sum = 15

Optional<Integer> max = numbers.stream()
    .reduce(Integer::max);
```

---

## 13. Comparison Table

### Lambda vs Method Reference vs Anonymous Class

```java
// Anonymous Inner Class
Runnable r1 = new Runnable() {
    public void run() {
        System.out.println("Hello");
    }
};

// Lambda Expression
Runnable r2 = () -> System.out.println("Hello");

// Method Reference
Runnable r3 = System.out::println;  // If println takes no args
```

### Function Chaining Methods

```java
Function<Integer, Integer> f1 = x -> x * 2;
Function<Integer, Integer> f2 = x -> x + 10;

// andThen: f1 first, then f2
f1.andThen(f2).apply(5);  // (5 * 2) + 10 = 20

// compose: f2 first, then f1
f1.compose(f2).apply(5);  // (5 + 10) * 2 = 30
```

---

## 14. Memory Checklist

Before exam/interview, verify you know:

- [ ] Lambda syntax (all variations)
- [ ] What is a functional interface
- [ ] All 6 main built-in functional interfaces
- [ ] Predicate combining (and, or, negate)
- [ ] Function chaining (andThen, compose)
- [ ] 4 types of method references
- [ ] Effectively final rule
- [ ] 'this' reference behavior
- [ ] Difference between lambda and anonymous class
- [ ] How to handle exceptions in lambdas
- [ ] Common stream operations with lambdas
- [ ] Best practices (3-line rule, descriptive names, etc.)
- [ ] Primitive specializations (IntConsumer, etc.)
- [ ] When to use method reference vs lambda

---

## 15. Quick Quiz - Test Yourself

**Q1:** What's wrong with this code?
```java
int count = 0;
list.forEach(item -> count++);
```
**A:** `count` is not effectively final (being modified).

**Q2:** What does this print?
```java
Function<Integer, Integer> f = x -> x * 2;
System.out.println(f.andThen(x -> x + 1).apply(5));
```
**A:** `11` - (5 * 2) + 1

**Q3:** Which is correct?
```java
// Option A
Predicate<String> p = s -> s.length() > 5;

// Option B
Function<String, Boolean> p = s -> s.length() > 5;
```
**A:** Option A - Use `Predicate` for boolean tests

**Q4:** What's the output?
```java
List<String> list = Arrays.asList("a", "bb", "ccc");
list.stream().map(String::length).forEach(System.out::print);
```
**A:** `123` - prints lengths: 1, 2, 3

**Q5:** Is this valid?
```java
@FunctionalInterface
interface MyInterface {
    void m1();
    default void m2() { }
}
```
**A:** Yes - functional interface can have default methods

---

**End of Revision Notes**

*Remember: Practice is key! Code examples yourself to solidify understanding.*
