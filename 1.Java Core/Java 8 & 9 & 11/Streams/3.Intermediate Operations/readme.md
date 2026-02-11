# Intermediate Operations in Java Streams

## Table of Contents

1. [Understanding Intermediate Operations](#1-understanding-intermediate-operations)
2. [filter() - Filtering Elements](#2-filter---filtering-elements)
3. [map() - Transforming Elements](#3-map---transforming-elements)
4. [flatMap() - Flattening Structures](#4-flatmap---flattening-structures)
5. [distinct() - Removing Duplicates](#5-distinct---removing-duplicates)
6. [sorted() - Sorting Elements](#6-sorted---sorting-elements)
7. [peek() - Debugging Streams](#7-peek---debugging-streams)
8. [limit() - Limiting Size](#8-limit---limiting-size)
9. [skip() - Skipping Elements](#9-skip---skipping-elements)
10. [Primitive Mapping Operations](#10-primitive-mapping-operations)
11. [Chaining Operations](#11-chaining-operations)
12. [Quick Revision](#quick-revision)

---

## 1. Understanding Intermediate Operations

### What are Intermediate Operations?

- Return a Stream, allowing method chaining
- **Lazy evaluation** - not executed until a terminal operation is called
- Form the backbone of stream pipelines
- Enable powerful data transformations

### Lazy Evaluation

**How it works:**
- Intermediate operations don't execute when called
- They build up a pipeline of operations
- Pipeline executes only when a terminal operation is invoked

**Operation Fusion:**
- Stream API intelligently combines multiple operations
- Minimizes iterations over the data
- Elements flow through all operations before the next element is processed

```java
// What you write:
list.stream()
    .filter(n -> n > 5)
    .map(n -> n * 2)
    .collect(Collectors.toList());

// How it's conceptually executed:
// For each element:
//   1. Check if > 5 (filter)
//   2. If yes, multiply by 2 (map)
//   3. Add to result (collect)
// NOT: filter all → then map all → then collect all
```

### Stateless vs Stateful Operations

**Stateless Operations:**
- Each element processed independently
- Operation on one element doesn't affect others
- Examples: `filter()`, `map()`, `flatMap()`, `peek()`
- Advantages:
  - Can be easily parallelized
  - O(1) memory per operation
  - Better performance

**Stateful Operations:**
- Require knowledge of or affect other elements
- May need to buffer elements
- Examples: `sorted()`, `distinct()`, `limit()`, `skip()`
- Considerations:
  - May impact performance, especially with infinite streams
  - O(n) memory possible
  - Less suitable for parallelization

---

## 2. filter() - Filtering Elements

**Method Signature:** `Stream<T> filter(Predicate<? super T> predicate)`

**Characteristics:**
- Filters elements based on a predicate (boolean condition)
- Returns stream with only elements that match the predicate
- Stateless operation - each element tested independently
- Short-circuiting with terminal operations like `findFirst()`
- One of the most commonly used operations

**Mental Model:** Think of filter as a gate - only elements that pass the test (predicate returns true) go through.

**Examples:**

```java
// Filter even numbers
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
// Output: [2, 4, 6, 8, 10]

// Filter strings by length
List<String> words = Arrays.asList("Hi", "Hello", "Hey", "Goodbye");
List<String> longWords = words.stream()
    .filter(word -> word.length() > 3)
    .collect(Collectors.toList());
// Output: [Hello, Goodbye]

// Filter null values
List<String> items = Arrays.asList("A", null, "B", "C", null);
List<String> nonNull = items.stream()
    .filter(Objects::nonNull)  // Better way using Objects utility
    .collect(Collectors.toList());
// Output: [A, B, C]

// Multiple filters (chained)
List<Integer> result = numbers.stream()
    .filter(n -> n > 3)        // First filter
    .filter(n -> n < 8)        // Second filter
    .filter(n -> n % 2 == 0)   // Third filter
    .collect(Collectors.toList());
// Output: [4, 6]
```

**Performance Note:**
- Multiple filter calls are fused by the stream implementation
- `filter(a).filter(b)` is as efficient as `filter(a && b)`
- Use whichever approach is more readable

---

## 3. map() - Transforming Elements

**Method Signature:** `<R> Stream<R> map(Function<? super T, ? extends R> mapper)`

**Characteristics:**
- Transforms each element using a function
- One-to-one transformation
- Each input element produces exactly one output element
- Output type can be different from input type
- Stateless operation
- Does NOT change the size of the stream

**Mental Model:** Think of map as a transformation machine - each item goes in, gets transformed, and comes out as a new item (possibly of a different type).

**Examples:**

```java
// Convert to uppercase
List<String> names = Arrays.asList("john", "jane", "bob");
List<String> upperNames = names.stream()
    .map(String::toUpperCase)  // Using method reference
    .collect(Collectors.toList());
// Output: [JOHN, JANE, BOB]

// Get string lengths
List<String> words = Arrays.asList("Hi", "Hello", "Hey");
List<Integer> lengths = words.stream()
    .map(String::length)
    .collect(Collectors.toList());
// Output: [2, 5, 3]

// Square numbers
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> squares = numbers.stream()
    .map(n -> n * n)
    .collect(Collectors.toList());
// Output: [1, 4, 9, 16, 25]

// Extract property from objects
class Person {
    String name;
    int age;
    Person(String name, int age) { this.name = name; this.age = age; }
    String getName() { return name; }
}

List<Person> people = Arrays.asList(
    new Person("Alice", 30),
    new Person("Bob", 25)
);

List<String> names = people.stream()
    .map(Person::getName)
    .collect(Collectors.toList());
// Output: [Alice, Bob]

// Type transformation
List<String> numberStrings = Arrays.asList("1", "2", "3");
List<Integer> integers = numberStrings.stream()
    .map(Integer::parseInt)
    .collect(Collectors.toList());
// Output: [1, 2, 3]
```

**Common Patterns:**
- **Property extraction:** `objects.stream().map(Object::getProperty)`
- **Type conversion:** `strings.stream().map(Integer::parseInt)`
- **Transformation:** `values.stream().map(v -> transform(v))`

---

## 4. flatMap() - Flattening Structures

**Method Signature:** `<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)`

**Characteristics:**
- Flattens nested structures into a single stream
- One-to-many transformation
- Takes a Function that returns a Stream for each element
- Flattens all resulting streams into one stream
- Each input element can produce 0, 1, or many output elements
- Essential for working with nested collections
- Stateless operation

**Mental Model:** Think of flatMap as "unwrapping" nested structures. Each element is transformed into a stream, then all those streams are concatenated into one stream.

**map vs flatMap:**
- **map:** T → R (one-to-one)
- **flatMap:** T → Stream<R> (one-to-many, then flattened)

**Examples:**

```java
// Flatten list of lists
List<List<Integer>> listOfLists = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4),
    Arrays.asList(5, 6)
);
List<Integer> flattened = listOfLists.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList());
// Output: [1, 2, 3, 4, 5, 6]

// Split sentences into words
List<String> sentences = Arrays.asList("Hello World", "Java Streams");
List<String> words = sentences.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
    .collect(Collectors.toList());
// Output: [Hello, World, Java, Streams]

// Get all characters from strings
List<String> words2 = Arrays.asList("Hi", "Bye");
List<String> chars = words2.stream()
    .flatMap(word -> word.chars().mapToObj(c -> String.valueOf((char) c)))
    .collect(Collectors.toList());
// Output: [H, i, B, y, e]

// Flatten optional values (Java 9+)
List<Optional<String>> optionals = Arrays.asList(
    Optional.of("A"),
    Optional.empty(),
    Optional.of("B"),
    Optional.empty(),
    Optional.of("C")
);

List<String> values = optionals.stream()
    .flatMap(Optional::stream)
    .collect(Collectors.toList());
// Output: [A, B, C]

// Get all items from nested structure
class Order {
    List<String> items;
    Order(List<String> items) { this.items = items; }
    List<String> getItems() { return items; }
}

List<Order> orders = Arrays.asList(
    new Order(Arrays.asList("Book", "Pen")),
    new Order(Arrays.asList("Laptop")),
    new Order(Arrays.asList("Mouse", "Keyboard", "Monitor"))
);

List<String> allItems = orders.stream()
    .flatMap(order -> order.getItems().stream())
    .collect(Collectors.toList());
// Output: [Book, Pen, Laptop, Mouse, Keyboard, Monitor]
```

**Visualization:**

```java
// map: Stream<List<String>> → Stream<List<String>>
// Result is still nested
List<List<String>> result1 = lists.stream()
    .map(list -> list)
    .collect(Collectors.toList());

// flatMap: Stream<List<String>> → Stream<String>
// Result is flattened
List<String> result2 = lists.stream()
    .flatMap(list -> list.stream())
    .collect(Collectors.toList());
```

---

## 5. distinct() - Removing Duplicates

**Method Signature:** `Stream<T> distinct()`

**Characteristics:**
- Removes duplicate elements based on `equals()` method
- Uses a Set internally to track seen elements
- Preserves encounter order (first occurrence kept)
- Stateful operation - must remember all seen elements
- Memory usage: O(n) in worst case (all unique)
- Uses `hashCode()` and `equals()` for comparison

**Performance Consideration:** For large streams, `distinct()` can be memory-intensive as it needs to store all unique elements seen so far.

**Examples:**

```java
// Remove duplicate numbers
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 3, 4, 5, 5);
List<Integer> unique = numbers.stream()
    .distinct()
    .collect(Collectors.toList());
// Output: [1, 2, 3, 4, 5]

// Remove duplicate strings
List<String> words = Arrays.asList("A", "B", "A", "C", "B");
List<String> uniqueWords = words.stream()
    .distinct()
    .collect(Collectors.toList());
// Output: [A, B, C]

// Distinct with custom objects (requires proper equals/hashCode)
class Person {
    String name;
    Person(String name) { this.name = name; }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}

List<Person> people = Arrays.asList(
    new Person("Alice"),
    new Person("Bob"),
    new Person("Alice")
);

List<Person> uniquePeople = people.stream()
    .distinct()
    .collect(Collectors.toList());
// Output: [Alice, Bob] (one Alice removed)

// Distinct by property (using custom approach)
class Employee {
    String name;
    String department;
    Employee(String name, String department) {
        this.name = name;
        this.department = department;
    }
}

List<Employee> employees = Arrays.asList(
    new Employee("John", "IT"),
    new Employee("Jane", "HR"),
    new Employee("Bob", "IT")
);

// Get unique departments
List<String> uniqueDepts = employees.stream()
    .map(e -> e.department)
    .distinct()
    .collect(Collectors.toList());
// Output: [IT, HR]
```

---

## 6. sorted() - Sorting Elements

**Method Signature:**
- `Stream<T> sorted()` - natural order
- `Stream<T> sorted(Comparator<? super T> comparator)` - custom order

**Characteristics:**
- Sorts elements in natural order or using a Comparator
- Stateful operation - must see all elements to sort them
- Memory usage: O(n) - needs to buffer all elements
- Time complexity: O(n log n)
- Stable sort (preserves order of equal elements)
- **For infinite streams, never completes (avoid!)**

**Natural Order Requirements:** Elements must implement Comparable, or you must provide a Comparator.

**Examples:**

```java
// Sort numbers (natural order)
List<Integer> numbers = Arrays.asList(5, 3, 8, 1, 9, 2);
List<Integer> sorted = numbers.stream()
    .sorted()
    .collect(Collectors.toList());
// Output: [1, 2, 3, 5, 8, 9]

// Sort strings (alphabetically)
List<String> names = Arrays.asList("Charlie", "Alice", "Bob");
List<String> sortedNames = names.stream()
    .sorted()
    .collect(Collectors.toList());
// Output: [Alice, Bob, Charlie]

// Sort in reverse order
List<Integer> numbers2 = Arrays.asList(5, 3, 8, 1, 9);
List<Integer> reverseSorted = numbers2.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());
// Output: [9, 8, 5, 3, 1]

// Sort by string length
List<String> words = Arrays.asList("Hi", "Hello", "Hey", "A");
List<String> byLength = words.stream()
    .sorted(Comparator.comparing(String::length))
    .collect(Collectors.toList());
// Output: [A, Hi, Hey, Hello]

// Sort with multiple criteria
class Person {
    String name;
    int age;
    Person(String name, int age) { this.name = name; this.age = age; }
    String getName() { return name; }
    int getAge() { return age; }
}

List<Person> people = Arrays.asList(
    new Person("Alice", 30),
    new Person("Bob", 25),
    new Person("Alice", 25)
);

// Sort by name, then by age
List<Person> sorted = people.stream()
    .sorted(Comparator.comparing(Person::getName)
                      .thenComparing(Person::getAge))
    .collect(Collectors.toList());
// Output: Alice(25), Alice(30), Bob(25)

// Reverse order with comparator
List<Person> reverseAge = people.stream()
    .sorted(Comparator.comparing(Person::getAge).reversed())
    .collect(Collectors.toList());

// Null-safe sorting
List<String> withNulls = Arrays.asList("C", null, "A", "B");
List<String> sortedSafe = withNulls.stream()
    .sorted(Comparator.nullsFirst(Comparator.naturalOrder()))
    .collect(Collectors.toList());
// Output: [null, A, B, C]
```

**Comparator Utility Methods:**
- `Comparator.naturalOrder()` - natural ordering
- `Comparator.reverseOrder()` - reverse natural ordering
- `Comparator.comparing(Function)` - extract and compare by property
- `Comparator.nullsFirst(Comparator)` - nulls before non-nulls
- `Comparator.nullsLast(Comparator)` - nulls after non-nulls
- `.thenComparing(Function)` - secondary sort criteria
- `.reversed()` - reverse the comparator

---

## 7. peek() - Debugging Streams

**Method Signature:** `Stream<T> peek(Consumer<? super T> action)`

**Characteristics:**
- Performs an action on each element without modifying the stream
- Primarily used for debugging
- Stateless operation
- Does NOT modify elements (returns same stream)
- Side-effect operation (generally avoid side effects in streams)
- Elements flow through unchanged

**Important:** `peek()` is an intermediate operation, so it only executes when a terminal operation is called. Don't rely on it for side effects in production code.

**Examples:**

```java
// Debug stream processing
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> result = numbers.stream()
    .peek(n -> System.out.println("Original: " + n))
    .map(n -> n * 2)
    .peek(n -> System.out.println("After map: " + n))
    .collect(Collectors.toList());

// Output shows each element flows through completely before next element:
// Original: 1
// After map: 2
// Original: 2
// After map: 4
// ...

// Logging for debugging
List<String> names = Arrays.asList("John", "Jane", "Bob");
List<String> processed = names.stream()
    .peek(name -> System.out.println("Processing: " + name))
    .filter(name -> name.length() > 3)
    .peek(name -> System.out.println("Filtered: " + name))
    .map(String::toUpperCase)
    .peek(name -> System.out.println("Mapped: " + name))
    .collect(Collectors.toList());

// Counting elements at each stage
AtomicInteger counter1 = new AtomicInteger(0);
AtomicInteger counter2 = new AtomicInteger(0);

List<Integer> result = numbers.stream()
    .peek(n -> counter1.incrementAndGet())
    .filter(n -> n > 2)
    .peek(n -> counter2.incrementAndGet())
    .collect(Collectors.toList());

System.out.println("Total processed: " + counter1.get());
System.out.println("After filter: " + counter2.get());
```

**When to Use peek():**
- ✅ Debugging during development
- ✅ Logging for troubleshooting
- ✅ Temporary code to understand stream flow
- ❌ Production logic (use map or forEach instead)
- ❌ Side effects that must happen (use forEach after collecting)

---

## 8. limit() - Limiting Size

**Method Signature:** `Stream<T> limit(long maxSize)`

**Characteristics:**
- Limits the stream to the first n elements
- Short-circuiting operation - stops processing after n elements
- Stateful operation for ordered streams (must count elements)
- Essential for working with infinite streams
- For unordered streams (like from Set), any n elements may be selected
- Size-reducing operation

**Examples:**

```java
// Get first 3 elements
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);
List<Integer> first3 = numbers.stream()
    .limit(3)
    .collect(Collectors.toList());
// Output: [1, 2, 3]

// Limit infinite stream
List<Integer> first10Even = Stream.iterate(0, n -> n + 2)
    .limit(10)
    .collect(Collectors.toList());
// Output: [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

// Top N after sorting
List<Integer> top5 = Arrays.asList(10, 5, 8, 20, 15, 3, 25, 12).stream()
    .sorted(Comparator.reverseOrder())
    .limit(5)
    .collect(Collectors.toList());
// Output: [25, 20, 15, 12, 10]

// Short-circuit expensive operations
List<Integer> result = Stream.iterate(1, n -> n + 1)
    .filter(n -> {
        System.out.println("Checking: " + n);
        return n % 2 == 0;
    })
    .limit(3)  // Stops after finding 3 even numbers
    .collect(Collectors.toList());
// Only checks: 1, 2, 3, 4, 5, 6 (stops at 6)
// Output: [2, 4, 6]
```

---

## 9. skip() - Skipping Elements

**Method Signature:** `Stream<T> skip(long n)`

**Characteristics:**
- Skips the first n elements of the stream
- Stateful operation - must count elements to skip
- Opposite of `limit()`
- Returns empty stream if n >= stream size
- Useful for pagination
- Complementary to `limit()` for range selection

**Examples:**

```java
// Skip first 3 elements
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
List<Integer> afterSkip = numbers.stream()
    .skip(3)
    .collect(Collectors.toList());
// Output: [4, 5, 6]

// Pagination (skip + limit)
List<Integer> numbers2 = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
int pageSize = 3;
int pageNumber = 2;  // 0-indexed

List<Integer> page = numbers2.stream()
    .skip(pageNumber * pageSize)  // Skip first pages
    .limit(pageSize)               // Take one page
    .collect(Collectors.toList());
// Output: [7, 8, 9] (third page)

// Remove first element
List<String> words = Arrays.asList("A", "B", "C", "D");
List<String> withoutFirst = words.stream()
    .skip(1)
    .collect(Collectors.toList());
// Output: [B, C, D]

// Process data after header row
List<String> csvLines = Arrays.asList(
    "Name,Age,City",  // Header
    "John,30,NYC",
    "Jane,25,LA"
);

List<String> dataOnly = csvLines.stream()
    .skip(1)  // Skip header
    .collect(Collectors.toList());
// Output: [John,30,NYC, Jane,25,LA]
```

**Pagination Pattern:**

```java
// Generic pagination method
public <T> List<T> getPage(List<T> data, int page, int size) {
    return data.stream()
        .skip((long) page * size)
        .limit(size)
        .collect(Collectors.toList());
}
```

---

## 10. Primitive Mapping Operations

**Methods:**
- `mapToInt(ToIntFunction<? super T> mapper)`
- `mapToLong(ToLongFunction<? super T> mapper)`
- `mapToDouble(ToDoubleFunction<? super T> mapper)`

**Why Use Primitive Streams?**
- Convert object stream to primitive stream
- Avoid boxing/unboxing performance penalty
- Access specialized operations (`sum`, `average`, `max`, `min`)
- More memory efficient for numeric operations

**Examples:**

```java
// Sum using mapToInt
List<String> numbers = Arrays.asList("1", "2", "3", "4", "5");
int sum = numbers.stream()
    .mapToInt(Integer::parseInt)
    .sum();
// Output: 15

// Get lengths as IntStream
List<String> words = Arrays.asList("Hi", "Hello", "Hey");
IntStream lengths = words.stream()
    .mapToInt(String::length);

// Average age
class Person {
    int age;
    Person(int age) { this.age = age; }
    int getAge() { return age; }
}

List<Person> people = Arrays.asList(
    new Person(25),
    new Person(30),
    new Person(35)
);

double avgAge = people.stream()
    .mapToInt(Person::getAge)
    .average()
    .orElse(0.0);
// Output: 30.0
```

**Performance Comparison:**

```java
// Slower - boxing overhead
List<Integer> numbers = IntStream.range(0, 1000000)
    .boxed()
    .collect(Collectors.toList());
long sum1 = numbers.stream()
    .reduce(0, Integer::sum);

// Faster - no boxing
long sum2 = IntStream.range(0, 1000000).sum();
```

---

## 11. Chaining Operations

**Complex Pipeline Example:**

```java
// Demonstrating operation chaining
List<String> words = Arrays.asList(
    "Hello", "world", "Java", "Streams", "are", "powerful", "hello"
);

List<String> result = words.stream()
    .filter(w -> w.length() > 3)      // Remove short words
    .map(String::toLowerCase)         // Convert to lowercase
    .distinct()                        // Remove duplicates
    .sorted()                          // Sort alphabetically
    .limit(3)                          // Take first 3
    .collect(Collectors.toList());
    
// Output: [hello, java, powerful]
```

---

## Quick Revision

### Summary of Intermediate Operations

| Operation | Type | Purpose | Output Stream Size |
|-----------|------|---------|-------------------|
| `filter()` | Stateless | Keep matching elements | ≤ input |
| `map()` | Stateless | Transform elements | = input |
| `flatMap()` | Stateless | Flatten nested structures | ≥ input |
| `distinct()` | Stateful | Remove duplicates | ≤ input |
| `sorted()` | Stateful | Sort elements | = input |
| `peek()` | Stateless | Debug/log elements | = input |
| `limit()` | Stateful | Take first n | ≤ n |
| `skip()` | Stateful | Skip first n | ≤ input - n |
| `mapToInt/Long/Double()` | Stateless | Convert to primitive stream | = input |

### Key Characteristics

**Stateless Operations:**
- `filter()`, `map()`, `flatMap()`, `peek()`
- Process each element independently
- O(1) memory per operation
- Can be easily parallelized
- Better performance

**Stateful Operations:**
- `distinct()`, `sorted()`, `limit()`, `skip()`
- Require knowledge of other elements
- May buffer elements
- O(n) memory possible
- Less suitable for parallelization

### Operation Comparisons

**filter() vs map():**
- `filter()`: Keeps or removes elements (boolean test)
- `map()`: Transforms elements (one-to-one)

**map() vs flatMap():**
- `map()`: T → R (one-to-one transformation)
- `flatMap()`: T → Stream<R> (one-to-many, then flattened)

**limit() vs skip():**
- `limit(n)`: Take first n elements
- `skip(n)`: Skip first n elements
- Combined: Pagination pattern

### Best Practices

1. **Filter early, transform later**
   - Place `filter()` operations early in the pipeline
   - Reduces number of elements for subsequent operations

2. **Use method references when possible**
   - `map(String::toUpperCase)` instead of `map(s -> s.toUpperCase())`
   - More concise and readable

3. **Chain filters vs compound conditions**
   - `filter(a).filter(b)` is as efficient as `filter(a && b)`
   - Choose based on readability

4. **Avoid peek() in production**
   - Use only for debugging
   - For side effects, use `forEach()` after collecting

5. **Use primitive streams for numeric operations**
   - `mapToInt()`, `mapToLong()`, `mapToDouble()`
   - Avoid boxing overhead
   - Access specialized operations

6. **Be cautious with stateful operations**
   - `sorted()`, `distinct()` require buffering all elements
   - May impact performance and memory
   - Avoid with infinite streams

7. **Understand lazy evaluation**
   - Intermediate operations don't execute until terminal operation
   - Build pipelines efficiently
   - Operations are fused for optimization

### Common Patterns

**Pagination:**
```java
data.stream()
    .skip(page * pageSize)
    .limit(pageSize)
    .collect(Collectors.toList());
```

**Property Extraction:**
```java
objects.stream()
    .map(Object::getProperty)
    .collect(Collectors.toList());
```

**Flattening Nested Collections:**
```java
nestedLists.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList());
```

**Top N Elements:**
```java
stream
    .sorted(Comparator.reverseOrder())
    .limit(n)
    .collect(Collectors.toList());
```

**Removing Nulls and Duplicates:**
```java
stream
    .filter(Objects::nonNull)
    .distinct()
    .collect(Collectors.toList());
```

### Common Pitfalls to Avoid

- ❌ Using `sorted()` on infinite streams (never completes)
- ❌ Relying on `peek()` for critical side effects
- ❌ Using object streams (`Stream<Integer>`) for large numeric operations
- ❌ Forgetting that intermediate operations are lazy
- ❌ Using `distinct()` without proper `equals()`/`hashCode()` implementation
- ❌ Placing expensive operations before filters
- ❌ Chaining too many operations without considering readability

### Memory and Performance Tips

- **Stateless operations:** O(1) memory - prefer when possible
- **Stateful operations:** O(n) memory - use judiciously
- **Operation fusion:** Multiple operations combined in single pass
- **Short-circuiting:** `limit()` with `filter()` stops early
- **Primitive streams:** Avoid boxing for better performance
- **Filter early:** Reduce elements before expensive operations
