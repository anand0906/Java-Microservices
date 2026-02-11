# Stream Creation in Java

## Table of Contents

1. [Understanding Stream Sources](#1-understanding-stream-sources)
2. [From Collections](#2-from-collections)
3. [From Arrays](#3-from-arrays)
4. [Using Stream.of()](#4-using-streamof)
5. [Using Stream.builder()](#5-using-streambuilder)
6. [Infinite Streams](#6-infinite-streams)
7. [Empty Streams](#7-empty-streams)
8. [Primitive Streams](#8-primitive-streams)
9. [From Files (I/O)](#9-from-files-io)
10. [From Strings](#10-from-strings)
11. [Concatenating Streams](#11-concatenating-streams)
12. [Quick Revision](#quick-revision)

---

## 1. Understanding Stream Sources

**What is a Stream Source?**
- Any entity that can supply elements to a stream
- Common sources include:
  - Collections (List, Set, Queue)
  - Arrays
  - Static factory methods
  - Builder pattern
  - Files and I/O
  - Generator functions

---

## 2. From Collections

**Key Points:**
- All Collection implementations provide the `stream()` method
- Calling `stream()` creates a view over the collection (no data copying)
- The original collection remains unchanged

**Basic Examples:**

```java
// From List
List<String> fruits = Arrays.asList("Apple", "Banana", "Cherry");
Stream<String> stream1 = fruits.stream();

// From Set
Set<Integer> numbers = new HashSet<>(Arrays.asList(1, 2, 3, 4, 5));
Stream<Integer> stream2 = numbers.stream();

// From Map entries
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);
Stream<Map.Entry<String, Integer>> stream3 = map.entrySet().stream();
```

**Working with Maps:**
- Maps have three stream views:
  - `map.keySet().stream()` → Stream of keys
  - `map.values().stream()` → Stream of values
  - `map.entrySet().stream()` → Stream of key-value pairs (most common)

```java
Map<String, Integer> scores = new HashMap<>();
scores.put("Alice", 95);
scores.put("Bob", 87);
scores.put("Charlie", 92);

// Stream and filter entries
scores.entrySet().stream()
    .filter(entry -> entry.getValue() > 90)
    .forEach(entry -> System.out.println(entry.getKey() + ": " + entry.getValue()));
```

---

## 3. From Arrays

**Key Points:**
- Use `Arrays.stream()` to convert arrays to streams
- Works with both object arrays and primitive arrays
- For primitive arrays, specialized streams (IntStream, LongStream, DoubleStream) are created
- Avoids boxing overhead for primitives

**Examples:**

```java
// Integer array
Integer[] nums = {10, 20, 30, 40, 50};
Stream<Integer> stream4 = Arrays.stream(nums);

// String array
String[] names = {"John", "Jane", "Bob"};
Stream<String> stream5 = Arrays.stream(names);

// Primitive int array (creates IntStream)
int[] primitiveNums = {1, 2, 3, 4, 5};
IntStream intStream = Arrays.stream(primitiveNums);

// Partial array (from index 1 to 3, exclusive end)
String[] words = {"A", "B", "C", "D", "E"};
Stream<String> partial = Arrays.stream(words, 1, 4);  // B, C, D
```

---

## 4. Using Stream.of()

**Key Points:**
- Static factory method for creating streams from specified values
- Convenient for small, known sets of elements
- Under the hood: creates an array, then creates a stream from it

**Basic Examples:**

```java
// Direct values
Stream<String> colors = Stream.of("Red", "Green", "Blue");

// Numbers
Stream<Integer> numbers = Stream.of(1, 2, 3, 4, 5);

// Single element
Stream<String> single = Stream.of("OnlyOne");

// From array (alternative to Arrays.stream())
String[] array = {"A", "B", "C"};
Stream<String> fromArray = Stream.of(array);
```

**Handling Null Values:**
- `Stream.of(null)` → Creates stream with one null element (size = 1)
- `Stream.ofNullable(null)` → Creates empty stream (size = 0)
- Use `ofNullable()` for potentially null values

```java
String nullableValue = getValue();  // might return null
Stream<String> safeStream = Stream.ofNullable(nullableValue);
```

---

## 5. Using Stream.builder()

**Key Points:**
- Allows adding elements one at a time programmatically
- Uses a buffer to collect elements
- `build()` creates the stream (builder cannot be reused after this)
- Useful when elements aren't known upfront

**Examples:**

```java
// Incremental building
Stream.Builder<String> builder = Stream.builder();
builder.add("A");
builder.add("B");
builder.add("C");
Stream<String> stream = builder.build();

// Fluent style
Stream<String> fluentStream = Stream.<String>builder()
    .add("A")
    .add("B")
    .add("C")
    .build();

// Conditional building
Stream.Builder<Integer> numberBuilder = Stream.builder();
for (int i = 1; i <= 10; i++) {
    if (i % 2 == 0) {
        numberBuilder.add(i);
    }
}
Stream<Integer> evenNumbers = numberBuilder.build();
```

---

## 6. Infinite Streams

**Key Points:**
- Streams with no defined end
- Created using `generate()` or `iterate()`
- **Must use `limit()` to avoid infinite loops**

### Stream.generate()

**Characteristics:**
- Takes a Supplier (function with no arguments)
- Generates elements independently
- No relationship between successive elements

**Use Cases:**
- Random values
- Constant values
- Computed values from external state

```java
// Random values
Stream<Double> randomStream = Stream.generate(Math::random);
randomStream.limit(5).forEach(System.out::println);

// Constant value
Stream<String> constants = Stream.generate(() -> "Hello");
constants.limit(3).forEach(System.out::println);  // Hello, Hello, Hello

// Stateful supplier
class Counter {
    private int count = 0;
    public int getNext() { return count++; }
}
Counter counter = new Counter();
Stream<Integer> numbers = Stream.generate(counter::getNext);
numbers.limit(5).forEach(System.out::println);  // 0, 1, 2, 3, 4
```

### Stream.iterate()

**Characteristics:**
- Each element computed from the previous one
- Takes a seed value and a UnaryOperator
- Creates sequential dependency between elements

**Use Cases:**
- Arithmetic sequences
- Fibonacci numbers
- Iterative calculations

```java
// Even numbers
Stream<Integer> evenNumbers = Stream.iterate(0, n -> n + 2);
evenNumbers.limit(10).forEach(System.out::println);

// Powers of 2
Stream<Integer> powersOf2 = Stream.iterate(1, n -> n * 2);
powersOf2.limit(10).forEach(System.out::println);  
// Output: 1, 2, 4, 8, 16, 32, 64, 128, 256, 512

// Fibonacci sequence
Stream.iterate(new int[]{0, 1}, fib -> new int[]{fib[1], fib[0] + fib[1]})
    .limit(10)
    .map(fib -> fib[0])
    .forEach(System.out::println);  
// Output: 0, 1, 1, 2, 3, 5, 8, 13, 21, 34

// With predicate (Java 9+) - stops when predicate is false
Stream.iterate(0, n -> n < 100, n -> n + 10)
    .forEach(System.out::println);  
// Output: 0, 10, 20, 30, 40, 50, 60, 70, 80, 90
```

**Comparison:**

| Feature | generate() | iterate() |
|---------|-----------|-----------|
| Element relationship | Independent | Depends on previous element |
| Sequence | Unpredictable | Predictable based on function |

---

## 7. Empty Streams

**Key Points:**
- Contains no elements
- Optimized (doesn't allocate unnecessary resources)
- Use instead of returning `null` to maintain stream API contract

**Examples:**

```java
// Create empty stream
Stream<String> emptyStream = Stream.empty();

// Return empty stream instead of null
public Stream<String> getResults() {
    if (noResults) {
        return Stream.empty();  // Better than returning null
    }
    return actualResults.stream();
}

// Count is always 0
long count = Stream.empty().count();  // 0
```

---

## 8. Primitive Streams

**Key Points:**
- Avoid boxing/unboxing overhead
- Three specialized types: `IntStream`, `LongStream`, `DoubleStream`
- Work directly with primitive values
- Provide specialized operations: `sum()`, `average()`, `max()`, `min()`

**Why Use Primitive Streams?**
- `Stream<Integer>` boxes each int into an Integer object (expensive)
- Primitive streams work with raw primitives (faster, more memory-efficient)

**Examples:**

```java
// IntStream
IntStream intStream1 = IntStream.range(1, 10);        // 1 to 9
IntStream intStream2 = IntStream.rangeClosed(1, 10);  // 1 to 10
IntStream intStream3 = IntStream.of(1, 2, 3, 4, 5);

// LongStream
LongStream longStream1 = LongStream.range(1L, 1000000L);
LongStream longStream2 = LongStream.of(10L, 20L, 30L);

// DoubleStream
DoubleStream doubleStream = DoubleStream.of(1.1, 2.2, 3.3);

// Converting between streams
Stream<Integer> boxedStream = intStream1.boxed();  // IntStream → Stream<Integer>
IntStream primitiveStream = Stream.of(1, 2, 3).mapToInt(Integer::intValue);

// Specialized operations
int sum = IntStream.range(1, 101).sum();                      // 5050
double avg = IntStream.range(1, 101).average().getAsDouble(); // 50.5
int max = IntStream.of(5, 3, 9, 1).max().getAsInt();         // 9
```

**Performance Comparison:**

```java
// Slower - boxing overhead
int sum1 = Stream.of(1, 2, 3, 4, 5)
    .reduce(0, Integer::sum);

// Faster - no boxing
int sum2 = IntStream.of(1, 2, 3, 4, 5).sum();
```

---

## 9. From Files (I/O)

**Key Points:**
- Files are read lazily (on-demand as stream is processed)
- Memory-efficient for large files (entire file not loaded into memory)
- **Always use try-with-resources** to ensure files are closed properly

**Examples:**

```java
import java.nio.file.*;
import java.io.IOException;

// Stream of lines from file
try (Stream<String> lines = Files.lines(Paths.get("data.txt"))) {
    lines.filter(line -> line.startsWith("ERROR"))
        .forEach(System.out::println);
} catch (IOException e) {
    e.printStackTrace();
}

// List all files in directory
try (Stream<Path> paths = Files.list(Paths.get("."))) {
    paths.filter(Files::isRegularFile)
        .forEach(System.out::println);
} catch (IOException e) {
    e.printStackTrace();
}

// Walk directory tree
try (Stream<Path> paths = Files.walk(Paths.get("src"))) {
    paths.filter(p -> p.toString().endsWith(".java"))
        .forEach(System.out::println);
} catch (IOException e) {
    e.printStackTrace();
}
```

---

## 10. From Strings

**Key Points:**
- Strings can create streams of characters or tokens
- `chars()` returns `IntStream` of character codes
- `codePoints()` handles Unicode properly
- Use `split()` or `Pattern.splitAsStream()` for tokenization

**Examples:**

```java
// Stream of characters
String text = "Hello";
IntStream charStream = text.chars();
charStream.forEach(c -> System.out.print((char) c + " "));  // H e l l o

// Stream of code points (proper Unicode handling)
Stream<String> codePoints = text.codePoints()
    .mapToObj(cp -> new String(Character.toChars(cp)));

// Split string into stream
String sentence = "Java Streams are powerful";
Stream<String> words = Arrays.stream(sentence.split(" "));
words.forEach(System.out::println);  // Java, Streams, are, powerful

// Stream from Pattern
import java.util.regex.Pattern;
Pattern pattern = Pattern.compile(",");
Stream<String> tokens = pattern.splitAsStream("A,B,C,D");
```

---

## 11. Concatenating Streams

**Key Points:**
- `Stream.concat()` combines two streams
- Elements of first stream followed by elements of second stream
- For multiple streams, nest `concat()` calls or use `flatMap`

**Examples:**

```java
// Concatenate two streams
Stream<String> stream1 = Stream.of("A", "B", "C");
Stream<String> stream2 = Stream.of("D", "E", "F");
Stream<String> combined = Stream.concat(stream1, stream2);
// Result: A, B, C, D, E, F

// Concatenate multiple streams (nested)
Stream<String> s1 = Stream.of("1");
Stream<String> s2 = Stream.of("2");
Stream<String> s3 = Stream.of("3");
Stream<String> all = Stream.concat(Stream.concat(s1, s2), s3);

// Better approach for multiple streams using flatMap
Stream<String> multiConcat = Stream.of(
    Stream.of("A", "B"),
    Stream.of("C", "D"),
    Stream.of("E", "F")
).flatMap(s -> s);
// Result: A, B, C, D, E, F
```

---

## Quick Revision

### Stream Creation Methods Summary

| Scenario | Method | Example |
|----------|--------|---------|
| From existing collection | `.stream()` | `list.stream()` |
| Few known values | `Stream.of()` | `Stream.of(1, 2, 3)` |
| From array | `Arrays.stream()` | `Arrays.stream(arr)` |
| Build dynamically | `Stream.builder()` | `builder.add().build()` |
| Infinite sequence | `Stream.iterate()` | `iterate(0, n -> n + 1)` |
| Random/generated | `Stream.generate()` | `generate(Math::random)` |
| No elements | `Stream.empty()` | `Stream.empty()` |
| Primitives | `IntStream.range()` | `IntStream.range(1, 10)` |
| From file | `Files.lines()` | `Files.lines(path)` |
| Combine streams | `Stream.concat()` | `concat(s1, s2)` |

### Best Practices

1. **Use primitive streams** for numeric operations to avoid boxing overhead
2. **Prefer `.stream()`** over converting to array then streaming
3. **Use `Stream.empty()`** instead of returning null
4. **Always limit infinite streams** before terminal operations
5. **Close file streams** using try-with-resources
6. **Use `ofNullable()`** for potentially null single values
7. **Choose the right creation method** based on your data source and requirements

### Key Differences

**generate() vs iterate():**
- `generate()`: Elements are independent, no predictable sequence
- `iterate()`: Each element depends on previous, predictable sequence

**Stream.of(null) vs Stream.ofNullable(null):**
- `Stream.of(null)`: Creates stream with 1 null element
- `Stream.ofNullable(null)`: Creates empty stream (size 0)

**Object Streams vs Primitive Streams:**
- Object streams (e.g., `Stream<Integer>`): Box primitives, more memory overhead
- Primitive streams (e.g., `IntStream`): Work with raw primitives, better performance

### Common Pitfalls to Avoid

- ❌ Forgetting to limit infinite streams
- ❌ Not closing file streams (memory leaks)
- ❌ Using `Stream<Integer>` for large numeric operations (use `IntStream`)
- ❌ Returning `null` instead of `Stream.empty()`
- ❌ Reusing a builder after calling `build()`
- ❌ Not using try-with-resources for file streams
- ❌ Calling terminal operations multiple times on the same stream
