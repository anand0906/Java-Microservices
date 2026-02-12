# Terminal Operations in Java Streams

## Table of Contents

1. [Understanding Terminal Operations](#1-understanding-terminal-operations)
2. [forEach() - Side Effects](#2-foreach---side-effects)
3. [collect() - Accumulating Elements](#3-collect---accumulating-elements)
4. [count() - Counting Elements](#4-count---counting-elements)
5. [reduce() - Reducing to Single Value](#5-reduce---reducing-to-single-value)
6. [min() and max() - Finding Extremes](#6-min-and-max---finding-extremes)
7. [Matching Operations](#7-matching-operations)
8. [Finding Operations](#8-finding-operations)
9. [toArray() - Converting to Array](#9-toarray---converting-to-array)
10. [Quick Revision](#quick-revision)

---

## 1. Understanding Terminal Operations

### What Makes an Operation Terminal?

**Defining Characteristics:**
- Produces a non-stream result (value, collection, or side-effect)
- Triggers execution of all intermediate operations in the pipeline
- Consumes the stream - stream cannot be used after
- Eager evaluation - executes immediately
- **Required** - a stream pipeline without a terminal operation does nothing

### Stream Lifecycle

```
Creation → Configuration (intermediate ops) → Execution (terminal op) → Result
```

### Important Concepts

**Key Ideas:**
- **Reduction:** Many terminal operations combine stream elements into a single result
- **Short-Circuiting:** Some operations (`findFirst`, `anyMatch`) can stop early
- **Side Effects:** Some operations (`forEach`) are executed for their effects
- **Collectors:** `collect()` uses Collector to accumulate elements in various ways

### Classification of Terminal Operations

**By Return Type:**
- **Value-producing:** `count()`, `reduce()`, `min()`, `max()`
- **Collection-producing:** `collect()`, `toArray()`
- **Boolean-producing:** `anyMatch()`, `allMatch()`, `noneMatch()`
- **Optional-producing:** `findFirst()`, `findAny()`, `min()`, `max()`
- **Void (side-effect):** `forEach()`, `forEachOrdered()`

**By Processing Style:**
- **Short-circuiting:** `findFirst()`, `findAny()`, `anyMatch()`, `allMatch()`, `noneMatch()`
- **Full processing:** `count()`, `collect()`, `reduce()`, `forEach()`

---

## 2. forEach() - Side Effects

**Method Signature:** `void forEach(Consumer<? super T> action)`

### Key Characteristics

**Properties:**
- Terminal operation that returns void
- Used for side effects (printing, updating external state, etc.)
- Processes all elements (not short-circuiting)
- For parallel streams, order is not guaranteed
- Use `forEachOrdered()` for ordered processing in parallel streams
- Does NOT return a value

### When to Use

**Good Use Cases:**
- ✅ Printing/logging elements
- ✅ Triggering actions on each element
- ✅ When you don't need the result, just the effect

**Bad Use Cases:**
- ❌ Updating external state (use `collect()` instead)
- ❌ Building collections (use `collect()` instead)

### Examples

```java
// Basic usage with method reference
List<String> names = Arrays.asList("John", "Jane", "Bob");
names.stream().forEach(System.out::println);

// forEachOrdered (maintains order even for parallel streams)
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
numbers.parallelStream()
    .forEachOrdered(System.out::println);
// Output: 1, 2, 3, 4, 5 (in order)
```

### forEach vs collect

```java
// ❌ BAD - modifying external state
List<String> result = new ArrayList<>();
stream.forEach(x -> result.add(transform(x)));

// ✅ GOOD - functional approach
List<String> result = stream
    .map(x -> transform(x))
    .collect(Collectors.toList());
```

---

## 3. collect() - Accumulating Elements

**Method Signatures:**
- `<R, A> R collect(Collector<? super T, A, R> collector)`
- `<R> R collect(Supplier<R> supplier, BiConsumer<R, ? super T> accumulator, BiConsumer<R, R> combiner)`

### Key Characteristics

**Properties:**
- Mutable reduction: Accumulates into a mutable result container
- Most commonly used with Collectors utility class
- Highly optimized - can work efficiently in parallel
- Flexible - can collect into List, Set, Map, String, custom objects
- Can perform complex aggregations (grouping, partitioning, statistics)

### Three-Argument collect() Explained

**Parameters:**
1. **Supplier:** Creates the result container (e.g., `ArrayList::new`)
2. **Accumulator:** Adds element to container (e.g., `List::add`)
3. **Combiner:** Combines containers (for parallel streams)

### Examples

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// To List (most common)
List<Integer> evenList = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
// Output: [2, 4]

// To Set (removes duplicates)
Set<String> uniqueNames = Arrays.asList("A", "B", "A", "C").stream()
    .collect(Collectors.toSet());
// Output: [A, B, C]

// To Map
List<String> words = Arrays.asList("Hi", "Hello", "Hey");
Map<String, Integer> wordLengths = words.stream()
    .collect(Collectors.toMap(
        word -> word,           // key
        word -> word.length()   // value
    ));
// Output: {Hi=2, Hello=5, Hey=3}

// Three-argument collect (manual)
ArrayList<String> result = words.stream()
    .collect(
        ArrayList::new,     // Create container
        ArrayList::add,     // Add element
        ArrayList::addAll   // Combine containers (for parallel)
    );
```

### Why collect() Instead of forEach()?

```java
// ✅ Functional, thread-safe, recommended
List<String> result = stream
    .map(String::toUpperCase)
    .collect(Collectors.toList());

// ❌ Imperative, not thread-safe, avoid
List<String> result = new ArrayList<>();
stream.map(String::toUpperCase)
    .forEach(result::add);  // Not safe for parallel streams
```

---

## 4. count() - Counting Elements

**Method Signature:** `long count()`

### Key Characteristics

**Properties:**
- Terminal operation returning long
- For infinite streams, never completes (don't use!)
- For sized streams (from collections), can be optimized
- Equivalent to `collect(Collectors.counting())` but more efficient
- Does NOT trigger filter/map operations if not necessary

### Examples

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Count all elements
long count = numbers.stream().count();
// Output: 5

// Count after filter
long evenCount = numbers.stream()
    .filter(n -> n % 2 == 0)
    .count();
// Output: 2

// Count distinct elements
long uniqueCount = Arrays.asList(1, 2, 2, 3, 3, 3).stream()
    .distinct()
    .count();
// Output: 3
```

### Performance Note

```java
// ❌ INEFFICIENT - creates intermediate collection
long count1 = stream.filter(predicate)
    .collect(Collectors.toList())
    .size();

// ✅ EFFICIENT - just counts
long count2 = stream.filter(predicate)
    .count();
```

---

## 5. reduce() - Reducing to Single Value

**Method Signatures:**
- `Optional<T> reduce(BinaryOperator<T> accumulator)`
- `T reduce(T identity, BinaryOperator<T> accumulator)`
- `<U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner)`

### Key Characteristics

**Properties:**
- General-purpose reduction - can implement many operations
- Requires an associative function: `(a op b) op c = a op (b op c)`
- Identity value: Value that doesn't change the result (e.g., 0 for addition, 1 for multiplication)
- Three forms for different use cases
- Can work efficiently in parallel

### How reduce() Works

```
Elements: [1, 2, 3, 4, 5]
Operation: sum
Process: ((((0 + 1) + 2) + 3) + 4) + 5 = 15
```

### Examples

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Sum with identity
int sum = numbers.stream()
    .reduce(0, Integer::sum);
// Output: 15

// Product
int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);
// Output: 120

// Find maximum (returns Optional)
Optional<Integer> max = numbers.stream()
    .reduce(Integer::max);
// Output: Optional[5]

// Concatenate strings
List<String> words = Arrays.asList("Hello", "World", "Java");
String sentence = words.stream()
    .reduce("", (a, b) -> a + " " + b).trim();
// Output: "Hello World Java"

// Complex reduction with different type
int totalChars = words.stream()
    .reduce(0, 
        (count, word) -> count + word.length(),  // Accumulator
        Integer::sum);                            // Combiner (for parallel)
// Output: 14
```

### Understanding the Three Forms

**Form 1: No identity - Returns Optional**
```java
Optional<T> reduce(BinaryOperator<T> accumulator)
```
- Might be empty if stream is empty

**Form 2: With identity - Returns actual value**
```java
T reduce(T identity, BinaryOperator<T> accumulator)
```
- Never empty, returns at least the identity

**Form 3: With identity and combiner**
```java
<U> U reduce(U identity, BiFunction<U,T,U> accumulator, BinaryOperator<U> combiner)
```
- For different types and parallel processing

### Common reduce() Patterns

```java
// Sum
.reduce(0, Integer::sum)

// Product
.reduce(1, (a, b) -> a * b)

// Max
.reduce(Integer::max)  // Returns Optional

// Min
.reduce(Integer::min)  // Returns Optional

// Concatenation
.reduce("", String::concat)

// Count
.reduce(0, (count, element) -> count + 1, Integer::sum)
```

---

## 6. min() and max() - Finding Extremes

**Method Signatures:**
- `Optional<T> min(Comparator<? super T> comparator)`
- `Optional<T> max(Comparator<? super T> comparator)`

### Key Characteristics

**Properties:**
- Returns Optional (stream might be empty)
- Requires a Comparator to determine ordering
- For natural ordering, use `Comparator.naturalOrder()`
- Implemented using `reduce()` internally
- Full stream traversal required (O(n))

### Examples

```java
List<Integer> numbers = Arrays.asList(5, 3, 8, 1, 9);

// Find min
Optional<Integer> min = numbers.stream()
    .min(Comparator.naturalOrder());
// Output: Optional[1]

// Find max
Optional<Integer> max = numbers.stream()
    .max(Comparator.naturalOrder());
// Output: Optional[9]

// Find longest string
List<String> words = Arrays.asList("Hi", "Hello", "Hey");
Optional<String> longest = words.stream()
    .max(Comparator.comparing(String::length));
// Output: Optional[Hello]

// Custom object comparison
class Person {
    String name;
    int age;
    Person(String name, int age) { this.name = name; this.age = age; }
    int getAge() { return age; }
}

List<Person> people = Arrays.asList(
    new Person("Alice", 30),
    new Person("Bob", 25),
    new Person("Charlie", 35)
);

Optional<Person> youngest = people.stream()
    .min(Comparator.comparing(Person::getAge));
// Output: Optional[Bob(25)]
```

### min/max vs reduce

```java
// ✅ Using min/max (cleaner)
Optional<Integer> max1 = stream.max(Integer::compare);

// ❌ Using reduce (more verbose)
Optional<Integer> max2 = stream.reduce((a, b) -> a > b ? a : b);
```

---

## 7. Matching Operations

### anyMatch(), allMatch(), noneMatch()

**Method Signatures:**
- `boolean anyMatch(Predicate<? super T> predicate)`
- `boolean allMatch(Predicate<? super T> predicate)`
- `boolean noneMatch(Predicate<? super T> predicate)`

### Key Characteristics

**Properties:**
- Return boolean (not Optional)
- Short-circuiting: Can stop processing early
- **anyMatch:** Returns true if AT LEAST ONE element matches
- **allMatch:** Returns true if ALL elements match (returns true for empty stream!)
- **noneMatch:** Returns true if NO elements match (returns true for empty stream!)
- Very efficient for large streams

### Logical Relationships

```java
!anyMatch(p) == noneMatch(p)
!allMatch(p) == anyMatch(not p)
```

### Empty Stream Behavior

- `anyMatch()` → false (no elements match)
- `allMatch()` → true (vacuous truth - all zero elements match)
- `noneMatch()` → true (no elements, so none match)

### Examples

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// anyMatch - at least one matches
boolean hasEven = numbers.stream()
    .anyMatch(n -> n % 2 == 0);
// Output: true (stops at 2, short-circuits)

// allMatch - all elements match
boolean allPositive = numbers.stream()
    .allMatch(n -> n > 0);
// Output: true

boolean allEven = numbers.stream()
    .allMatch(n -> n % 2 == 0);
// Output: false (stops at 1, short-circuits)

// noneMatch - no elements match
boolean noNegative = numbers.stream()
    .noneMatch(n -> n < 0);
// Output: true

// With custom objects
class Student {
    String name;
    int score;
    Student(String name, int score) {
        this.name = name;
        this.score = score;
    }
}

List<Student> students = Arrays.asList(
    new Student("Alice", 85),
    new Student("Bob", 92),
    new Student("Charlie", 78)
);

// Any student scored above 90?
boolean hasHighScorer = students.stream()
    .anyMatch(s -> s.score > 90);
// Output: true

// All students passed (score >= 60)?
boolean allPassed = students.stream()
    .allMatch(s -> s.score >= 60);
// Output: true
```

### Short-Circuiting in Action

```java
boolean result = Stream.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
    .peek(n -> System.out.println("Checking: " + n))
    .anyMatch(n -> n > 5);

// Output:
// Checking: 1
// Checking: 2
// Checking: 3
// Checking: 4
// Checking: 5
// Checking: 6
// (stops here - found match)
```

### Practical Usage Patterns

```java
// Validation
boolean allEmailsValid = emails.stream()
    .allMatch(email -> email.contains("@"));

// Search
boolean hasTarget = list.stream()
    .anyMatch(item -> item.equals(target));

// Constraint checking
boolean noNulls = list.stream()
    .noneMatch(Objects::isNull);
```

---

## 8. Finding Operations

### findFirst() and findAny()

**Method Signatures:**
- `Optional<T> findFirst()`
- `Optional<T> findAny()`

### Key Characteristics

**Properties:**
- Return `Optional<T>` (stream might be empty)
- Short-circuiting: Stop after finding an element
- **findFirst():** Returns first element in encounter order
- **findAny():** Returns any element (useful for parallel streams)
- For sequential streams, `findAny()` typically returns first element
- For parallel streams, `findAny()` returns whichever element is found first

### findFirst() vs findAny()

| Criterion | Use findFirst() | Use findAny() |
|-----------|----------------|---------------|
| Order matters | ✅ Yes | ❌ No |
| Sequential streams | Either works | Either works |
| Parallel streams | Slower (must maintain order) | Faster (any thread can win) |

### Examples

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Find first element
Optional<Integer> first = numbers.stream()
    .findFirst();
// Output: Optional[1]

// Find first even number
Optional<Integer> firstEven = numbers.stream()
    .filter(n -> n % 2 == 0)
    .findFirst();
// Output: Optional[2]

// Find any (useful in parallel streams)
Optional<Integer> anyParallel = numbers.parallelStream()
    .filter(n -> n > 2)
    .findAny();
// Output: Could be 3, 4, or 5 (whichever thread finds first)

// Short-circuiting demonstration
Optional<Integer> found = Stream.iterate(1, n -> n + 1)
    .filter(n -> n > 5)
    .findFirst();
// Stops at 6

// With custom objects
class Product {
    String name;
    double price;
    Product(String name, double price) {
        this.name = name;
        this.price = price;
    }
}

List<Product> products = Arrays.asList(
    new Product("Laptop", 1200),
    new Product("Mouse", 25),
    new Product("Keyboard", 75)
);

Optional<Product> affordable = products.stream()
    .filter(p -> p.price < 100)
    .findFirst();
// Output: Optional[Mouse]
```

### Practical Usage

```java
// Search for element
Optional<String> result = list.stream()
    .filter(s -> s.startsWith("A"))
    .findFirst();

// Get value or default
String value = result.orElse("Not Found");

// Execute action if found
result.ifPresent(s -> System.out.println("Found: " + s));

// Throw exception if not found
String value2 = result.orElseThrow(() -> 
    new NoSuchElementException("Element not found"));
```

---

## 9. toArray() - Converting to Array

**Method Signatures:**
- `Object[] toArray()`
- `<A> A[] toArray(IntFunction<A[]> generator)`

### Key Characteristics

**Properties:**
- Terminal operation that returns an array
- First form returns `Object[]` (type-unsafe)
- Second form returns typed array (type-safe, recommended)
- Generator function creates array of correct size
- Cannot return primitive arrays directly (use toArray on primitive streams)

### Examples

```java
List<String> words = Arrays.asList("A", "B", "C");

// To Object array (type-unsafe)
Object[] array1 = words.stream().toArray();

// To typed array (recommended)
String[] array2 = words.stream().toArray(String[]::new);
// Output: ["A", "B", "C"]

// After transformations
String[] upperArray = words.stream()
    .map(String::toUpperCase)
    .toArray(String[]::new);
// Output: ["A", "B", "C"]

// Primitive array from IntStream
int[] primitiveArray = IntStream.range(1, 6)
    .toArray();
// Output: [1, 2, 3, 4, 5]
```

### Why Use Typed Arrays

```java
// ❌ BAD - requires casting
Object[] array1 = stream.toArray();
String first = (String) array1[0];  // Cast required

// ✅ GOOD - no casting needed
String[] array2 = stream.toArray(String[]::new);
String first = array2[0];  // Type-safe
```

---

## Quick Revision

### Summary of Terminal Operations

| Operation | Return Type | Short-Circuit | Purpose |
|-----------|-------------|---------------|---------|
| `forEach()` | void | No | Side effects |
| `forEachOrdered()` | void | No | Ordered side effects |
| `collect()` | R | No | Collect to collection |
| `toArray()` | Array | No | Convert to array |
| `reduce()` | T/Optional\<T\> | No | Reduce to single value |
| `count()` | long | No | Count elements |
| `min()` | Optional\<T\> | No | Find minimum |
| `max()` | Optional\<T\> | No | Find maximum |
| `anyMatch()` | boolean | Yes | Check if any match |
| `allMatch()` | boolean | Yes | Check if all match |
| `noneMatch()` | boolean | Yes | Check if none match |
| `findFirst()` | Optional\<T\> | Yes | Find first element |
| `findAny()` | Optional\<T\> | Yes | Find any element |

### Operation Categories

**1. Collecting Operations:**
- `collect()` - Most versatile, accumulate into collections
- `toArray()` - Convert to array
- Both preserve stream elements

**2. Reduction Operations:**
- `reduce()` - General-purpose reduction
- `count()` - Count elements
- `min()`, `max()` - Find extremes
- All produce single value from multiple elements

**3. Matching Operations:**
- `anyMatch()` - At least one matches
- `allMatch()` - All match
- `noneMatch()` - None match
- All return boolean, all short-circuit

**4. Finding Operations:**
- `findFirst()` - First in order
- `findAny()` - Any element (faster in parallel)
- Both return Optional, both short-circuit

**5. Side Effect Operations:**
- `forEach()` - Unordered iteration
- `forEachOrdered()` - Ordered iteration
- Use sparingly, prefer collect()

### Short-Circuiting Operations

**What are they?**
- Operations that can complete without processing entire stream
- Stop as soon as result is determined
- Critical for performance with large/infinite streams

**Short-Circuiting Operations:**
- `findFirst()` - Stops after finding first match
- `findAny()` - Stops after finding any match
- `anyMatch()` - Stops after finding first true
- `allMatch()` - Stops after finding first false
- `noneMatch()` - Stops after finding first true

### When to Use Which Operation

| Scenario | Operation | Example |
|----------|-----------|---------|
| Need elements in collection | `collect()` | `collect(Collectors.toList())` |
| Need typed array | `toArray()` | `toArray(String[]::new)` |
| Print/log each element | `forEach()` | `forEach(System.out::println)` |
| Count elements | `count()` | `stream.count()` |
| Sum/product/combine | `reduce()` | `reduce(0, Integer::sum)` |
| Find smallest/largest | `min()`, `max()` | `max(Comparator.naturalOrder())` |
| Check if any match | `anyMatch()` | `anyMatch(n -> n > 10)` |
| Check if all match | `allMatch()` | `allMatch(s -> s.length() > 0)` |
| Check if none match | `noneMatch()` | `noneMatch(Objects::isNull)` |
| Find first match | `findFirst()` | `filter(predicate).findFirst()` |
| Find any match (parallel) | `findAny()` | `filter(predicate).findAny()` |

### Common Patterns

**Pattern 1: Collect to List**
```java
List<T> result = stream
    .filter(predicate)
    .map(mapper)
    .collect(Collectors.toList());
```

**Pattern 2: Count Matches**
```java
long count = stream
    .filter(predicate)
    .count();
```

**Pattern 3: Sum Values**
```java
int sum = stream
    .mapToInt(mapper)
    .sum();
// Or
int sum = stream
    .reduce(0, Integer::sum);
```

**Pattern 4: Find First Match**
```java
Optional<T> result = stream
    .filter(predicate)
    .findFirst();

T value = result.orElse(defaultValue);
```

**Pattern 5: Check Conditions**
```java
boolean valid = stream.allMatch(predicate);
boolean hasMatch = stream.anyMatch(predicate);
boolean noMatch = stream.noneMatch(predicate);
```

**Pattern 6: Find Extreme**
```java
Optional<T> max = stream
    .max(Comparator.comparing(T::getValue));

T maxValue = max.orElseThrow();
```

### Key Principles

1. **Terminal operations are required**
   - Intermediate operations are lazy
   - Nothing happens without a terminal operation

2. **Streams are consumed after terminal operation**
   - Cannot reuse a stream
   - Must create new stream for additional operations

3. **Choose appropriate operation**
   - Don't use `forEach()` when `collect()` is better
   - Don't use `collect().size()` when `count()` is available

4. **Short-circuiting improves performance**
   - Use for large streams
   - Essential for infinite streams
   - `findFirst()`, `anyMatch()`, etc.

5. **Handle Optional properly**
   - `min()`, `max()`, `findFirst()`, `findAny()` return Optional
   - Use `orElse()`, `orElseGet()`, `orElseThrow()`
   - Never call `get()` without checking `isPresent()`

### Performance Tips

**✅ Efficient:**
```java
// Use count() directly
long count = stream.filter(predicate).count();

// Use short-circuiting
boolean hasMatch = stream.anyMatch(predicate);

// Use collect() for building collections
List<T> list = stream.collect(Collectors.toList());

// Use primitive streams for numeric operations
int sum = stream.mapToInt(mapper).sum();
```

**❌ Inefficient:**
```java
// Don't collect then count
long count = stream.filter(predicate)
    .collect(Collectors.toList()).size();

// Don't use forEach to build collections
List<T> list = new ArrayList<>();
stream.forEach(list::add);  // Not thread-safe!

// Don't process all when short-circuit available
boolean hasMatch = stream.filter(predicate).count() > 0;
// Use anyMatch() instead
```

### Common Pitfalls to Avoid

- ❌ Reusing consumed streams
- ❌ Using `forEach()` to build collections (use `collect()`)
- ❌ Calling `Optional.get()` without checking `isPresent()`
- ❌ Using `count() > 0` instead of `anyMatch()`
- ❌ Collecting to list just to get size (use `count()`)
- ❌ Not handling empty streams with Optional methods
- ❌ Using `reduce()` when specialized methods exist (`sum()`, `max()`)
- ❌ Forgetting `forEachOrdered()` for parallel streams when order matters

### Best Practices

1. ✅ Use `collect()` for building collections, not `forEach()`
2. ✅ Use `count()` instead of `collect().size()`
3. ✅ Use short-circuiting operations when possible
4. ✅ Handle Optional with `orElse()`, `orElseGet()`, `orElseThrow()`
5. ✅ Use method references when possible
6. ✅ Use specialized methods (`sum()`, `max()`) over `reduce()`
7. ✅ Use `findAny()` for parallel streams when order doesn't matter
8. ✅ Use `forEachOrdered()` for parallel streams when order matters
9. ✅ Remember terminal operations consume the stream
10. ✅ Choose the right operation for the job

### reduce() Quick Reference

**Three Forms:**
```java
// Form 1: No identity, returns Optional
Optional<T> reduce(BinaryOperator<T> accumulator)

// Form 2: With identity, returns value
T reduce(T identity, BinaryOperator<T> accumulator)

// Form 3: With combiner (for different types/parallel)
U reduce(U identity, BiFunction<U,T,U> accumulator, BinaryOperator<U> combiner)
```

**Common Operations:**
- Sum: `.reduce(0, Integer::sum)`
- Product: `.reduce(1, (a, b) -> a * b)`
- Max: `.reduce(Integer::max)`
- Min: `.reduce(Integer::min)`
- Concatenation: `.reduce("", String::concat)`

### Optional Handling

**Safe Ways to Handle Optional:**
```java
// Provide default value
T value = optional.orElse(defaultValue);

// Compute default if needed
T value = optional.orElseGet(() -> computeDefault());

// Throw exception if empty
T value = optional.orElseThrow(() -> new NoSuchElementException());

// Execute action if present
optional.ifPresent(value -> doSomething(value));

// Transform if present
Optional<U> mapped = optional.map(mapper);
```

**Never Do:**
```java
// ❌ Don't call get() without checking
T value = optional.get();  // Throws if empty!

// ✅ Always check or use orElse variants
if (optional.isPresent()) {
    T value = optional.get();
}
```
