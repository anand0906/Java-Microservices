# Java Streams 

## Table of Contents

1. [Stream Basics](#1-stream-basics)
2. [Stream Creation](#2-stream-creation)
3. [Intermediate Operations](#3-intermediate-operations)
4. [Terminal Operations](#4-terminal-operations)
5. [Collectors](#5-collectors)
6. [Advanced Topics](#6-advanced-topics)
7. [Interview Programs Cheat Sheet](#7-interview-programs-cheat-sheet)
8. [Quick Reference](#8-quick-reference)

---

## 1. Stream Basics

### What is a Stream?

**Definition:**
- Sequence of elements supporting sequential and parallel aggregate operations
- NOT a data structure (doesn't store data)
- Supports functional-style operations on collections

### Stream Characteristics

**Key Properties:**
- **Lazy evaluation:** Intermediate operations are not executed until terminal operation
- **Consumable:** Can only be used once
- **Pipelined:** Operations return streams for chaining
- **Internal iteration:** Stream handles iteration internally

### Stream Pipeline

```
Source → Intermediate Operations → Terminal Operation → Result
```

**Example:**
```java
list.stream()           // Source
    .filter()           // Intermediate
    .map()              // Intermediate
    .collect()          // Terminal → Result
```

---

## 2. Stream Creation

### Creation Methods Summary

| Method | Example | Use Case |
|--------|---------|----------|
| `.stream()` | `list.stream()` | From collection |
| `Stream.of()` | `Stream.of(1,2,3)` | Few known values |
| `Arrays.stream()` | `Arrays.stream(arr)` | From array |
| `Stream.builder()` | `builder.add().build()` | Dynamic building |
| `Stream.iterate()` | `iterate(0, n->n+1)` | Infinite sequence |
| `Stream.generate()` | `generate(Math::random)` | Random/generated |
| `Stream.empty()` | `Stream.empty()` | No elements |
| `IntStream.range()` | `range(1, 10)` | Primitive int |
| `Files.lines()` | `Files.lines(path)` | From file |

### Important Notes

**Collection to Stream:**
```java
List<String> list = Arrays.asList("A", "B", "C");
Stream<String> stream = list.stream();
```

**Primitive Streams (avoid boxing):**
```java
IntStream intStream = IntStream.range(1, 10);      // 1 to 9
IntStream closed = IntStream.rangeClosed(1, 10);   // 1 to 10
```

**Infinite Streams (must use limit):**
```java
Stream.iterate(0, n -> n + 2).limit(10)     // Even numbers
Stream.generate(Math::random).limit(5)       // Random values
```

---

## 3. Intermediate Operations

### Stateless Operations

**Characteristics:** Each element processed independently

| Operation | Purpose | Example |
|-----------|---------|---------|
| `filter()` | Keep matching elements | `.filter(n -> n > 5)` |
| `map()` | Transform elements (1-to-1) | `.map(String::toUpperCase)` |
| `flatMap()` | Flatten nested structures | `.flatMap(Collection::stream)` |
| `peek()` | Debug/side-effect | `.peek(System.out::println)` |

### Stateful Operations

**Characteristics:** May need to see all elements

| Operation | Purpose | Example |
|-----------|---------|---------|
| `distinct()` | Remove duplicates | `.distinct()` |
| `sorted()` | Sort elements | `.sorted(Comparator.naturalOrder())` |
| `limit()` | Take first n | `.limit(10)` |
| `skip()` | Skip first n | `.skip(5)` |

### Key Examples

```java
// filter - keep elements matching condition
stream.filter(n -> n % 2 == 0)

// map - transform each element
stream.map(String::toUpperCase)

// flatMap - flatten nested collections
listOfLists.stream().flatMap(Collection::stream)

// distinct - remove duplicates
stream.distinct()

// sorted - sort elements
stream.sorted()
stream.sorted(Comparator.reverseOrder())

// limit - take first n
stream.limit(5)

// skip - skip first n
stream.skip(3)
```

---

## 4. Terminal Operations

### Categories

**Value-Producing:**
- `count()` - Returns long
- `reduce()` - Reduces to single value
- `min()`, `max()` - Find extremes

**Collection-Producing:**
- `collect()` - Accumulate to collection
- `toArray()` - Convert to array

**Boolean-Producing:**
- `anyMatch()` - At least one matches
- `allMatch()` - All match
- `noneMatch()` - None match

**Optional-Producing:**
- `findFirst()` - First element
- `findAny()` - Any element

**Side-Effect:**
- `forEach()` - Iterate with action

### Short-Circuiting Operations

**Stop early without processing entire stream:**
- `findFirst()`
- `findAny()`
- `anyMatch()`
- `allMatch()`
- `noneMatch()`
- `limit()`

### Key Examples

```java
// collect - to list
List<T> list = stream.collect(Collectors.toList());

// count
long count = stream.count();

// reduce - sum
int sum = stream.reduce(0, Integer::sum);

// min/max
Optional<T> min = stream.min(Comparator.naturalOrder());

// anyMatch/allMatch/noneMatch
boolean hasEven = stream.anyMatch(n -> n % 2 == 0);
boolean allPositive = stream.allMatch(n -> n > 0);
boolean noNegative = stream.noneMatch(n -> n < 0);

// findFirst/findAny
Optional<T> first = stream.findFirst();

// forEach
stream.forEach(System.out::println);

// toArray
String[] array = stream.toArray(String[]::new);
```

---

## 5. Collectors

### Basic Collectors

| Collector | Purpose | Example |
|-----------|---------|---------|
| `toList()` | Collect to List | `collect(Collectors.toList())` |
| `toSet()` | Collect to Set | `collect(Collectors.toSet())` |
| `toMap()` | Collect to Map | `collect(Collectors.toMap(k, v))` |
| `joining()` | Join strings | `collect(Collectors.joining(", "))` |

### Numeric Collectors

| Collector | Return Type | Example |
|-----------|-------------|---------|
| `counting()` | Long | `collect(Collectors.counting())` |
| `summingInt()` | Integer | `collect(Collectors.summingInt(mapper))` |
| `averagingInt()` | Double | `collect(Collectors.averagingInt(mapper))` |
| `summarizingInt()` | IntSummaryStatistics | `collect(Collectors.summarizingInt(mapper))` |

### Grouping Collectors

```java
// Group by property
Map<K, List<V>> grouped = stream
    .collect(Collectors.groupingBy(classifier));

// Group and count
Map<K, Long> counts = stream
    .collect(Collectors.groupingBy(classifier, Collectors.counting()));

// Partition (binary grouping)
Map<Boolean, List<T>> partitioned = stream
    .collect(Collectors.partitioningBy(predicate));
```

### Common Patterns

```java
// To List
.collect(Collectors.toList())

// To Set (remove duplicates)
.collect(Collectors.toSet())

// To Map
.collect(Collectors.toMap(keyMapper, valueMapper))

// Join strings
.collect(Collectors.joining(", "))

// Group by
.collect(Collectors.groupingBy(Person::getCity))

// Partition
.collect(Collectors.partitioningBy(n -> n % 2 == 0))

// Count by group
.collect(Collectors.groupingBy(classifier, Collectors.counting()))

// Sum by group
.collect(Collectors.groupingBy(classifier, Collectors.summingInt(mapper)))
```

---

## 6. Advanced Topics

### Parallel Streams

**When to Use:**
- ✅ Large datasets (10,000+ elements)
- ✅ Computationally intensive operations
- ✅ Stateless operations
- ✅ No order dependency

**When NOT to Use:**
- ❌ Small datasets
- ❌ I/O operations
- ❌ Shared mutable state
- ❌ Order-dependent operations

```java
// Create parallel stream
list.parallelStream()

// Convert to parallel
stream.parallel()

// Check if parallel
stream.isParallel()
```

### Primitive Streams

**Benefits:**
- Avoid boxing/unboxing overhead
- Specialized operations (`sum()`, `average()`)
- Better performance

```java
// IntStream
IntStream.range(1, 10)           // 1 to 9
IntStream.rangeClosed(1, 10)     // 1 to 10
IntStream.of(1, 2, 3, 4, 5)

// Operations
intStream.sum()
intStream.average()
intStream.max()
intStream.min()
intStream.summaryStatistics()

// Convert
stream.mapToInt(Integer::intValue)
intStream.boxed()
```

### Optional

**Creating:**
```java
Optional.of(value)              // Non-null value
Optional.ofNullable(value)      // May be null
Optional.empty()                 // Empty optional
```

**Using:**
```java
optional.isPresent()            // Check if present
optional.get()                  // Get value (unsafe!)
optional.orElse(default)        // Default if empty
optional.orElseGet(supplier)    // Lazy default
optional.orElseThrow()          // Throw if empty
optional.ifPresent(consumer)    // Action if present
optional.map(mapper)            // Transform if present
optional.filter(predicate)      // Filter if present
```

### Method References

**Four Types:**

```java
// 1. Static method
ClassName::staticMethod
String::valueOf

// 2. Instance method of particular object
object::instanceMethod
System.out::println

// 3. Instance method of arbitrary object
ClassName::instanceMethod
String::toUpperCase

// 4. Constructor
ClassName::new
String::new
```

---

## 7. Interview Programs Cheat Sheet

### Finding Elements

```java
// Even numbers
.filter(n -> n % 2 == 0)

// Numbers starting with 1
.map(String::valueOf).filter(s -> s.startsWith("1"))

// First element
.findFirst()

// Max/Min
.max(Comparator.naturalOrder())
.min(Comparator.naturalOrder())

// Second highest
.distinct().sorted(Comparator.reverseOrder()).skip(1).findFirst()
```

### Duplicates

```java
// Find duplicates
Set<T> seen = new HashSet<>();
stream.filter(n -> !seen.add(n)).collect(Collectors.toSet())

// Remove duplicates
.distinct()

// Count duplicates
.collect(Collectors.groupingBy(Function.identity(), Collectors.counting()))
```

### Transformations

```java
// Square
.map(n -> n * n)

// Cube
.map(n -> n * n * n)

// Uppercase
.map(String::toUpperCase)

// String to Integer
.map(Integer::parseInt)
```

### Aggregations

```java
// Sum
.reduce(0, Integer::sum)
.mapToInt(Integer::intValue).sum()

// Average
.mapToInt(Integer::intValue).average()

// Count
.count()

// Statistics
.mapToInt(Integer::intValue).summaryStatistics()
```

### Sorting

```java
// Ascending
.sorted()

// Descending
.sorted(Comparator.reverseOrder())

// By property
.sorted(Comparator.comparing(Person::getAge))

// By length
.sorted(Comparator.comparing(String::length))
```

### Grouping

```java
// Group by property
.collect(Collectors.groupingBy(Employee::getDepartment))

// Count by group
.collect(Collectors.groupingBy(classifier, Collectors.counting()))

// Sum by group
.collect(Collectors.groupingBy(classifier, Collectors.summingInt(mapper)))

// Partition (even/odd)
.collect(Collectors.partitioningBy(n -> n % 2 == 0))
```

### String Operations

```java
// Join with delimiter
.collect(Collectors.joining(", "))

// Character frequency
str.chars().mapToObj(c -> (char)c)
   .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()))

// Word frequency
Arrays.stream(str.split(" "))
   .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()))

// Reverse string
new StringBuilder(str).reverse().toString()
```

### List Operations

```java
// Common elements
list1.stream().filter(list2::contains)

// Remove nulls
.filter(Objects::nonNull)

// Concatenate
Stream.concat(stream1, stream2)

// Pagination
.skip(page * size).limit(size)

// Top N
.sorted(comparator).limit(n)
```

### Special Patterns

```java
// Fibonacci
Stream.iterate(new int[]{0,1}, f -> new int[]{f[1], f[0]+f[1]})
   .limit(10).map(f -> f[0])

// Factorial
IntStream.rangeClosed(1, n).reduce(1, (a,b) -> a*b)

// Prime numbers
IntStream.rangeClosed(2, n)
   .filter(num -> IntStream.rangeClosed(2, (int)Math.sqrt(num))
      .noneMatch(i -> num % i == 0))

// Anagram check
str1.chars().sorted().equals(str2.chars().sorted())
```

---

## 8. Quick Reference

### Stream Pipeline Pattern

```java
collection.stream()              // Source
    .filter(predicate)           // Intermediate: filter
    .map(function)               // Intermediate: transform
    .sorted()                    // Intermediate: sort
    .limit(n)                    // Intermediate: limit
    .collect(Collectors.toList()) // Terminal: collect
```

### Common Operation Chains

```java
// Filter and collect
stream.filter(predicate).collect(Collectors.toList())

// Map and collect
stream.map(mapper).collect(Collectors.toList())

// Filter, map, and collect
stream.filter(predicate).map(mapper).collect(Collectors.toList())

// Sort and limit (Top N)
stream.sorted(comparator).limit(n).collect(Collectors.toList())

// Distinct and count
stream.distinct().count()

// Group and count
stream.collect(Collectors.groupingBy(classifier, Collectors.counting()))
```

### Performance Best Practices

**✅ DO:**
- Use primitive streams for numeric operations
- Filter early in the pipeline
- Use method references for readability
- Use parallel streams for large datasets
- Chain operations efficiently
- Use appropriate collectors

**❌ DON'T:**
- Use `forEach()` to build collections (use `collect()`)
- Call `collect().size()` when `count()` available
- Use `count() > 0` instead of `anyMatch()`
- Modify external state in stream operations
- Call `Optional.get()` without checking
- Reuse consumed streams
- Use parallel for small data or I/O

### Common Mistakes to Avoid

```java
// ❌ WRONG - forEach to build collection
List<T> result = new ArrayList<>();
stream.forEach(result::add);

// ✅ CORRECT - use collect
List<T> result = stream.collect(Collectors.toList());

// ❌ WRONG - collect then size
long count = stream.collect(Collectors.toList()).size();

// ✅ CORRECT - count directly
long count = stream.count();

// ❌ WRONG - unsafe Optional.get()
T value = optional.get();

// ✅ CORRECT - safe handling
T value = optional.orElse(defaultValue);
```

### Stateless vs Stateful

**Stateless (Preferred):**
- `filter()`
- `map()`
- `flatMap()`
- `peek()`

**Stateful (Use Carefully):**
- `distinct()`
- `sorted()`
- `limit()`
- `skip()`

### Memory Considerations

**O(1) Memory:**
- `filter()`
- `map()`
- `flatMap()`
- `peek()`

**O(n) Memory:**
- `distinct()` - Stores unique elements
- `sorted()` - Buffers all elements
- `collect()` - Accumulates results

### Collector Comparison

| Need | Collector |
|------|-----------|
| List with order | `toList()` |
| Unique elements | `toSet()` |
| Key-value pairs | `toMap()` |
| Concatenated string | `joining()` |
| Count | `counting()` |
| Sum | `summingInt()` |
| Average | `averagingInt()` |
| All statistics | `summarizingInt()` |
| Group by category | `groupingBy()` |
| Split true/false | `partitioningBy()` |

### Terminal Operation Comparison

| Need | Operation |
|------|-----------|
| Build collection | `collect()` |
| Get array | `toArray()` |
| Count elements | `count()` |
| Reduce to one | `reduce()` |
| Find min/max | `min()`, `max()` |
| Check condition | `anyMatch()`, `allMatch()`, `noneMatch()` |
| Find element | `findFirst()`, `findAny()` |
| Iterate | `forEach()` |

### reduce() Forms

```java
// Form 1: No identity (returns Optional)
Optional<T> reduce(BinaryOperator<T> accumulator)

// Form 2: With identity (returns value)
T reduce(T identity, BinaryOperator<T> accumulator)

// Form 3: Different type (for parallel)
U reduce(U identity, BiFunction<U,T,U> accumulator, BinaryOperator<U> combiner)

// Examples
stream.reduce(Integer::sum)              // Optional
stream.reduce(0, Integer::sum)           // int
stream.reduce(0, Integer::sum, Integer::sum) // parallel
```

### Comparator Utilities

```java
// Natural order
Comparator.naturalOrder()
Comparator.reverseOrder()

// By property
Comparator.comparing(Person::getAge)
Comparator.comparing(Person::getAge).reversed()

// Multiple criteria
Comparator.comparing(Person::getLastName)
          .thenComparing(Person::getFirstName)

// Null handling
Comparator.nullsFirst(Comparator.naturalOrder())
Comparator.nullsLast(Comparator.naturalOrder())
```

### Stream Creation Quick Guide

```java
// From collection
list.stream()

// From values
Stream.of(1, 2, 3)

// From array
Arrays.stream(array)

// Empty
Stream.empty()

// Infinite
Stream.iterate(0, n -> n + 1)
Stream.generate(Math::random)

// Primitive
IntStream.range(1, 10)
IntStream.rangeClosed(1, 10)

// From file
Files.lines(Paths.get("file.txt"))

// Concatenate
Stream.concat(stream1, stream2)
```

### Interview Quick Tips

**Most Common Questions:**
1. Find even/odd numbers → `.filter(n -> n % 2 == 0)`
2. Find duplicates → Use Set with `!seen.add(n)`
3. Group by property → `groupingBy(classifier)`
4. Sort by property → `sorted(Comparator.comparing(getter))`
5. Top N elements → `sorted().limit(n)`
6. String operations → `joining()`, `chars()`, `split()`
7. Count frequency → `groupingBy(identity(), counting())`
8. Remove nulls → `filter(Objects::nonNull)`
9. Flatten nested → `flatMap(Collection::stream)`
10. Convert to map → `toMap(keyMapper, valueMapper)`

**Time Complexity:**
- Most operations: O(n)
- `sorted()`: O(n log n)
- `distinct()`: O(n)
- `count()`: O(n) or O(1) for sized streams

**Remember:**
- Streams are lazy (only execute on terminal operation)
- Streams can only be used once
- Use primitive streams to avoid boxing
- Handle Optional safely
- Short-circuiting operations stop early
- Parallel streams aren't always faster

---

## Summary Cheat Sheet

### 5 Stream Creation Methods
1. `list.stream()` - From collection
2. `Stream.of(...)` - From values
3. `Arrays.stream(arr)` - From array
4. `IntStream.range(1,10)` - Primitive range
5. `Stream.iterate()/generate()` - Infinite

### 8 Key Intermediate Operations
1. `filter()` - Keep matching
2. `map()` - Transform
3. `flatMap()` - Flatten
4. `distinct()` - Remove duplicates
5. `sorted()` - Sort
6. `limit()` - Take first n
7. `skip()` - Skip first n
8. `peek()` - Debug

### 10 Essential Terminal Operations
1. `collect()` - Accumulate
2. `forEach()` - Iterate
3. `count()` - Count
4. `reduce()` - Reduce
5. `min()/max()` - Extremes
6. `anyMatch()` - Any true
7. `allMatch()` - All true
8. `noneMatch()` - None true
9. `findFirst()` - First element
10. `toArray()` - To array

### 6 Must-Know Collectors
1. `toList()` - To list
2. `toSet()` - To set
3. `toMap()` - To map
4. `joining()` - Join strings
5. `groupingBy()` - Group elements
6. `partitioningBy()` - Split by boolean

### 3 Advanced Concepts
1. Parallel Streams - For large data
2. Primitive Streams - Avoid boxing
3. Optional - Handle nulls safely

---

**Remember: Practice makes perfect! Try implementing these patterns with different scenarios.**
