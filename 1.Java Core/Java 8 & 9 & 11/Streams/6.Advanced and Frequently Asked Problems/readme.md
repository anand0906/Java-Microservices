# Advanced Stream Topics & Interview Programs

## Table of Contents

1. [Parallel Streams](#1-parallel-streams)
2. [Primitive Streams](#2-primitive-streams)
3. [Optional Class](#3-optional-class)
4. [Method References](#4-method-references)
5. [Stream Chaining](#5-stream-chaining)
6. [FlatMap with Objects](#6-flatmap-with-objects)
7. [Frequently Asked Interview Programs](#7-frequently-asked-interview-programs)
8. [Quick Revision](#quick-revision)

---

## 1. Parallel Streams

### What are Parallel Streams?

**Definition:**
- Divide data into multiple chunks
- Process chunks in parallel using multiple threads
- Leverage multi-core processors for performance

### Key Characteristics

**How It Works:**
- Uses ForkJoin framework internally
- Splits data into segments
- Processes segments concurrently
- Combines results

### Examples

```java
// Create parallel stream
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);
int sum = numbers.parallelStream()
    .mapToInt(Integer::intValue)
    .sum();

// Convert sequential to parallel
List<String> words = Arrays.asList("A", "B", "C", "D");
long count = words.stream()
    .parallel()
    .count();

// Check if parallel
boolean isParallel = numbers.parallelStream().isParallel();
// Output: true
```

### When to Use Parallel Streams

**✅ Good Use Cases:**
- Large datasets (thousands of elements)
- Computationally intensive operations
- Stateless operations
- No order dependency
- Independent element processing

**❌ Avoid When:**
- Small datasets (overhead not worth it)
- I/O operations (blocking operations)
- Operations requiring ordering
- Shared mutable state
- Quick operations (overhead > benefit)

### Performance Considerations

```java
// Overhead example - parallel not worth it
List<Integer> small = Arrays.asList(1, 2, 3, 4, 5);
small.parallelStream().forEach(System.out::println);  // Overhead > benefit

// Good use case - large computation
List<Integer> large = IntStream.range(0, 1000000).boxed().collect(Collectors.toList());
long sum = large.parallelStream()
    .mapToLong(i -> expensiveOperation(i))
    .sum();  // Parallel worth it here
```

---

## 2. Primitive Streams

### IntStream, LongStream, DoubleStream

**Why Use Primitive Streams?**
- Avoid boxing/unboxing overhead
- Specialized operations: `sum()`, `average()`, `max()`, `min()`
- Better performance for numeric operations

### Examples

```java
// IntStream range (exclusive end)
IntStream range = IntStream.range(1, 5);  // 1, 2, 3, 4
range.forEach(System.out::println);

// IntStream rangeClosed (inclusive end)
IntStream rangeClosed = IntStream.rangeClosed(1, 5);  // 1, 2, 3, 4, 5

// Sum using IntStream
int sum = IntStream.of(1, 2, 3, 4, 5).sum();
// Output: 15

// Convert to IntStream
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
int total = numbers.stream()
    .mapToInt(Integer::intValue)
    .sum();

// Statistics
IntSummaryStatistics stats = IntStream.of(1, 2, 3, 4, 5)
    .summaryStatistics();
System.out.println("Average: " + stats.getAverage());
System.out.println("Max: " + stats.getMax());
System.out.println("Min: " + stats.getMin());
System.out.println("Sum: " + stats.getSum());
System.out.println("Count: " + stats.getCount());
```

---

## 3. Optional Class

### What is Optional?

**Definition:**
- Container that may or may not contain a value
- Helps avoid NullPointerException
- Forces explicit handling of absence

### Creating Optional

```java
// Create Optional with value
Optional<String> optional1 = Optional.of("Hello");

// Create Optional that may be null
Optional<String> optional2 = Optional.ofNullable(null);

// Create empty Optional
Optional<String> optional3 = Optional.empty();
```

### Using Optional

```java
// Check if present
if (optional1.isPresent()) {
    System.out.println(optional1.get());
}

// ifPresent with action
optional1.ifPresent(value -> System.out.println(value));

// orElse (default value)
String value1 = optional2.orElse("Default");

// orElseGet (lazy default - only computed if needed)
String value2 = optional2.orElseGet(() -> "Computed Default");

// orElseThrow
String value3 = optional1.orElseThrow(() -> 
    new IllegalArgumentException("Value not present"));

// map (transform value if present)
Optional<Integer> length = optional1.map(String::length);
// Output: Optional[5]

// filter (keep value if matches predicate)
Optional<String> filtered = optional1.filter(s -> s.length() > 3);
// Output: Optional[Hello]
```

---

## 4. Method References

### Types of Method References

**Four Types:**
1. Static method reference
2. Instance method of particular object
3. Instance method of arbitrary object
4. Constructor reference

### Examples

```java
List<String> words = Arrays.asList("apple", "banana", "cherry");

// 1. Static method reference
// Lambda: word -> String.valueOf(word)
words.stream().map(String::valueOf);

// 2. Instance method of particular object
// Lambda: word -> System.out.println(word)
words.stream().forEach(System.out::println);

// 3. Instance method of arbitrary object
// Lambda: word -> word.toUpperCase()
words.stream().map(String::toUpperCase);

// 4. Constructor reference
// Lambda: word -> new String(word)
words.stream().map(String::new);
```

---

## 5. Stream Chaining

### Complex Pipeline Example

```java
List<String> names = Arrays.asList(
    "John", "Jane", "Bob", "Alice", "Charlie", "john"
);

List<String> result = names.stream()
    .filter(name -> name.length() > 3)      // Keep names longer than 3
    .map(String::toLowerCase)                // Convert to lowercase
    .distinct()                              // Remove duplicates
    .sorted()                                // Sort alphabetically
    .limit(3)                                // Take first 3
    .collect(Collectors.toList());

// Output: [alice, charlie, jane]
```

---

## 6. FlatMap with Objects

### Working with Nested Collections

```java
class Student {
    String name;
    List<String> courses;
    
    Student(String name, List<String> courses) {
        this.name = name;
        this.courses = courses;
    }
    
    public List<String> getCourses() {
        return courses;
    }
}

List<Student> students = Arrays.asList(
    new Student("John", Arrays.asList("Math", "Science")),
    new Student("Jane", Arrays.asList("Math", "English")),
    new Student("Bob", Arrays.asList("Science", "History"))
);

// Get all unique courses
Set<String> allCourses = students.stream()
    .flatMap(student -> student.getCourses().stream())
    .collect(Collectors.toSet());
// Output: [Math, Science, English, History]
```

---

## 7. Frequently Asked Interview Programs

### 1. Find Even and Odd Numbers

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Even numbers
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
// Output: [2, 4, 6, 8, 10]

// Odd numbers
List<Integer> oddNumbers = numbers.stream()
    .filter(n -> n % 2 != 0)
    .collect(Collectors.toList());
// Output: [1, 3, 5, 7, 9]
```

### 2. Find Numbers Starting with 1

```java
List<Integer> numbers = Arrays.asList(10, 15, 8, 49, 25, 98, 100, 32);

List<Integer> startsWithOne = numbers.stream()
    .map(String::valueOf)
    .filter(s -> s.startsWith("1"))
    .map(Integer::valueOf)
    .collect(Collectors.toList());
// Output: [10, 15, 100]
```

### 3. Find Duplicate Elements

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 2, 4, 5, 6, 5, 7);

// Using Set to track seen elements
Set<Integer> seen = new HashSet<>();
Set<Integer> duplicates = numbers.stream()
    .filter(n -> !seen.add(n))
    .collect(Collectors.toSet());
// Output: [2, 5]

// Using frequency map
List<Integer> duplicates2 = numbers.stream()
    .collect(Collectors.groupingBy(n -> n, Collectors.counting()))
    .entrySet().stream()
    .filter(entry -> entry.getValue() > 1)
    .map(Map.Entry::getKey)
    .collect(Collectors.toList());
// Output: [2, 5]
```

### 4. Find First Element

```java
List<Integer> numbers = Arrays.asList(5, 3, 8, 1, 9);

Optional<Integer> first = numbers.stream()
    .findFirst();
// Output: Optional[5]

// With filter
Optional<Integer> firstEven = numbers.stream()
    .filter(n -> n % 2 == 0)
    .findFirst();
// Output: Optional[8]
```

### 5. Find Total Number of Elements

```java
List<String> words = Arrays.asList("apple", "banana", "cherry");

long count = words.stream().count();
// Output: 3

// With filter
long longWords = words.stream()
    .filter(w -> w.length() > 5)
    .count();
// Output: 2
```

### 6. Find Maximum and Minimum

```java
List<Integer> numbers = Arrays.asList(5, 3, 8, 1, 9, 2);

// Maximum
Optional<Integer> max = numbers.stream()
    .max(Integer::compare);
// Output: Optional[9]

// Minimum
Optional<Integer> min = numbers.stream()
    .min(Integer::compare);
// Output: Optional[1]
```

### 7. Sort in Ascending and Descending Order

```java
List<Integer> numbers = Arrays.asList(5, 3, 8, 1, 9, 2);

// Ascending order
List<Integer> ascending = numbers.stream()
    .sorted()
    .collect(Collectors.toList());
// Output: [1, 2, 3, 5, 8, 9]

// Descending order
List<Integer> descending = numbers.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());
// Output: [9, 8, 5, 3, 2, 1]
```

### 8. Sort Strings by Length

```java
List<String> words = Arrays.asList("apple", "pie", "banana", "cat");

List<String> sortedByLength = words.stream()
    .sorted(Comparator.comparing(String::length))
    .collect(Collectors.toList());
// Output: [pie, cat, apple, banana]
```

### 9. Find Second Highest Number

```java
List<Integer> numbers = Arrays.asList(5, 3, 8, 1, 9, 2);

Optional<Integer> secondHighest = numbers.stream()
    .distinct()
    .sorted(Comparator.reverseOrder())
    .skip(1)
    .findFirst();
// Output: Optional[8]
```

### 10. Find Second Lowest Number

```java
List<Integer> numbers = Arrays.asList(5, 3, 8, 1, 9, 2);

Optional<Integer> secondLowest = numbers.stream()
    .distinct()
    .sorted()
    .skip(1)
    .findFirst();
// Output: Optional[2]
```

### 11. Sum of All Elements

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Using reduce
int sum = numbers.stream()
    .reduce(0, Integer::sum);
// Output: 15

// Using mapToInt and sum
int sum2 = numbers.stream()
    .mapToInt(Integer::intValue)
    .sum();
// Output: 15
```

### 12. Average of All Elements

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

double average = numbers.stream()
    .mapToInt(Integer::intValue)
    .average()
    .orElse(0.0);
// Output: 3.0
```

### 13. Square Each Element

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

List<Integer> squares = numbers.stream()
    .map(n -> n * n)
    .collect(Collectors.toList());
// Output: [1, 4, 9, 16, 25]
```

### 14. Cube and Filter > 50

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

List<Integer> result = numbers.stream()
    .map(n -> n * n * n)
    .filter(n -> n > 50)
    .collect(Collectors.toList());
// Output: [64, 125]
```

### 15. Convert List to Map

```java
// Simple list to map
List<String> words = Arrays.asList("apple", "banana", "cherry");

Map<String, Integer> wordLengthMap = words.stream()
    .collect(Collectors.toMap(
        word -> word,
        word -> word.length()
    ));
// Output: {apple=5, banana=6, cherry=6}

// List of objects to map
class Person {
    int id;
    String name;
    Person(int id, String name) {
        this.id = id;
        this.name = name;
    }
}

List<Person> people = Arrays.asList(
    new Person(1, "Alice"),
    new Person(2, "Bob"),
    new Person(3, "Charlie")
);

Map<Integer, String> idNameMap = people.stream()
    .collect(Collectors.toMap(
        p -> p.id,
        p -> p.name
    ));
// Output: {1=Alice, 2=Bob, 3=Charlie}
```

### 16. Count Occurrence of Each Character

```java
String input = "hello world";

Map<Character, Long> charCount = input.chars()
    .mapToObj(c -> (char) c)
    .filter(c -> c != ' ')
    .collect(Collectors.groupingBy(
        c -> c,
        Collectors.counting()
    ));
// Output: {d=1, e=1, h=1, l=3, o=2, r=1, w=1}
```

### 17. Count Occurrence of Each Word

```java
String sentence = "apple banana apple cherry banana apple";

Map<String, Long> wordCount = Arrays.stream(sentence.split(" "))
    .collect(Collectors.groupingBy(
        word -> word,
        Collectors.counting()
    ));
// Output: {apple=3, banana=2, cherry=1}
```

### 18. Remove Duplicates from List

```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 4, 5, 5);

List<Integer> unique = numbers.stream()
    .distinct()
    .collect(Collectors.toList());
// Output: [1, 2, 3, 4, 5]
```

### 19. Join Strings with Delimiter

```java
List<String> words = Arrays.asList("apple", "banana", "cherry");

String result = words.stream()
    .collect(Collectors.joining(", "));
// Output: "apple, banana, cherry"

// With prefix and suffix
String result2 = words.stream()
    .collect(Collectors.joining(", ", "[", "]"));
// Output: "[apple, banana, cherry]"
```

### 20. Partition Even and Odd Numbers

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);

Map<Boolean, List<Integer>> partitioned = numbers.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));

List<Integer> evens = partitioned.get(true);    // [2, 4, 6, 8]
List<Integer> odds = partitioned.get(false);     // [1, 3, 5, 7]
```

### 21. Group by Property

```java
class Employee {
    String name;
    String department;
    int salary;
    
    Employee(String name, String department, int salary) {
        this.name = name;
        this.department = department;
        this.salary = salary;
    }
}

List<Employee> employees = Arrays.asList(
    new Employee("Alice", "IT", 70000),
    new Employee("Bob", "HR", 60000),
    new Employee("Charlie", "IT", 80000),
    new Employee("David", "HR", 65000)
);

// Group by department
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(e -> e.department));
// Output: {IT=[Alice, Charlie], HR=[Bob, David]}

// Count by department
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(
        e -> e.department,
        Collectors.counting()
    ));
// Output: {IT=2, HR=2}

// Average salary by department
Map<String, Double> avgSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        e -> e.department,
        Collectors.averagingInt(e -> e.salary)
    ));
// Output: {IT=75000.0, HR=62500.0}
```

### 22. Find Common Elements Between Two Lists

```java
List<Integer> list1 = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> list2 = Arrays.asList(4, 5, 6, 7, 8);

List<Integer> common = list1.stream()
    .filter(list2::contains)
    .collect(Collectors.toList());
// Output: [4, 5]
```

### 23. Reverse a String

```java
String input = "hello";

String reversed = Stream.of(input)
    .map(str -> new StringBuilder(str).reverse())
    .collect(Collectors.joining());
// Output: "olleh"

// Character by character
String reversed2 = input.chars()
    .mapToObj(c -> (char) c)
    .reduce("", (s, c) -> c + s, (s1, s2) -> s2 + s1);
// Output: "olleh"
```

### 24. Check if String is Palindrome

```java
String input = "racecar";

boolean isPalindrome = input.equals(
    new StringBuilder(input).reverse().toString()
);
// Output: true

// Using streams
boolean isPalindrome2 = IntStream.range(0, input.length() / 2)
    .allMatch(i -> input.charAt(i) == input.charAt(input.length() - 1 - i));
// Output: true
```

### 25. Find All Prime Numbers in Range

```java
int start = 1;
int end = 20;

List<Integer> primes = IntStream.rangeClosed(start, end)
    .filter(n -> n > 1 && IntStream.rangeClosed(2, (int) Math.sqrt(n))
        .noneMatch(i -> n % i == 0))
    .boxed()
    .collect(Collectors.toList());
// Output: [2, 3, 5, 7, 11, 13, 17, 19]
```

### 26. Fibonacci Series

```java
// First 10 Fibonacci numbers
List<Integer> fibonacci = Stream.iterate(new int[]{0, 1}, 
        f -> new int[]{f[1], f[0] + f[1]})
    .limit(10)
    .map(f -> f[0])
    .collect(Collectors.toList());
// Output: [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

### 27. Factorial

```java
int n = 5;

int factorial = IntStream.rangeClosed(1, n)
    .reduce(1, (a, b) -> a * b);
// Output: 120
```

### 28. Anagram Check

```java
String str1 = "listen";
String str2 = "silent";

boolean isAnagram = str1.chars().sorted().boxed().collect(Collectors.toList())
    .equals(str2.chars().sorted().boxed().collect(Collectors.toList()));
// Output: true

// Cleaner approach
String sorted1 = str1.chars().sorted()
    .collect(StringBuilder::new, StringBuilder::appendCodePoint, StringBuilder::append)
    .toString();
String sorted2 = str2.chars().sorted()
    .collect(StringBuilder::new, StringBuilder::appendCodePoint, StringBuilder::append)
    .toString();
boolean isAnagram2 = sorted1.equals(sorted2);
// Output: true
```

### 29. Remove Null Values from List

```java
List<String> words = Arrays.asList("apple", null, "banana", null, "cherry");

List<String> nonNull = words.stream()
    .filter(Objects::nonNull)
    .collect(Collectors.toList());
// Output: [apple, banana, cherry]
```

### 30. Concatenate Two Streams

```java
List<Integer> list1 = Arrays.asList(1, 2, 3);
List<Integer> list2 = Arrays.asList(4, 5, 6);

List<Integer> combined = Stream.concat(
    list1.stream(),
    list2.stream()
).collect(Collectors.toList());
// Output: [1, 2, 3, 4, 5, 6]
```

### 31. Skip and Limit (Pagination)

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

int pageNumber = 2;  // 0-indexed
int pageSize = 3;

List<Integer> page = numbers.stream()
    .skip(pageNumber * pageSize)
    .limit(pageSize)
    .collect(Collectors.toList());
// Output: [7, 8, 9]
```

### 32. Find Longest String

```java
List<String> words = Arrays.asList("apple", "pie", "banana", "cat");

Optional<String> longest = words.stream()
    .max(Comparator.comparing(String::length));
// Output: Optional[banana]
```

### 33. Convert String Array to Uppercase

```java
String[] words = {"apple", "banana", "cherry"};

String[] uppercase = Arrays.stream(words)
    .map(String::toUpperCase)
    .toArray(String[]::new);
// Output: [APPLE, BANANA, CHERRY]
```

### 34. Filter and Transform

```java
List<String> words = Arrays.asList("apple", "pie", "banana", "cat");

List<String> result = words.stream()
    .filter(w -> w.length() > 3)
    .map(String::toUpperCase)
    .collect(Collectors.toList());
// Output: [APPLE, BANANA]
```

### 35. Top N Salaries

```java
class Employee {
    String name;
    int salary;
    Employee(String name, int salary) {
        this.name = name;
        this.salary = salary;
    }
}

List<Employee> employees = Arrays.asList(
    new Employee("Alice", 70000),
    new Employee("Bob", 60000),
    new Employee("Charlie", 80000),
    new Employee("David", 65000),
    new Employee("Eve", 90000)
);

// Top 3 salaries
List<Employee> top3 = employees.stream()
    .sorted(Comparator.comparing((Employee e) -> e.salary).reversed())
    .limit(3)
    .collect(Collectors.toList());
// Output: [Eve(90000), Charlie(80000), Alice(70000)]
```

---

## Quick Revision

### Advanced Topics Summary

**Parallel Streams:**
- Use for large datasets and computationally intensive operations
- Avoid for I/O operations and small datasets
- Check with `isParallel()`, convert with `parallel()`

**Primitive Streams:**
- `IntStream`, `LongStream`, `DoubleStream`
- Avoid boxing overhead
- Use `range()`, `rangeClosed()`, `sum()`, `average()`

**Optional:**
- Container for nullable values
- Methods: `of()`, `ofNullable()`, `empty()`
- Operations: `orElse()`, `orElseGet()`, `orElseThrow()`
- Transformations: `map()`, `filter()`, `flatMap()`

**Method References:**
- Static: `ClassName::methodName`
- Instance: `object::methodName`
- Arbitrary object: `ClassName::instanceMethod`
- Constructor: `ClassName::new`

### Common Interview Program Patterns

**Finding Elements:**
- Even/Odd: `.filter(n -> n % 2 == 0)`
- Max/Min: `.max()`, `.min()`
- First/Any: `.findFirst()`, `.findAny()`
- Nth element: `.skip(n-1).findFirst()`

**Transformations:**
- Square: `.map(n -> n * n)`
- Uppercase: `.map(String::toUpperCase)`
- String to Int: `.map(Integer::parseInt)`

**Aggregations:**
- Sum: `.reduce(0, Integer::sum)` or `.mapToInt().sum()`
- Count: `.count()`
- Average: `.mapToInt().average()`
- Statistics: `.summaryStatistics()`

**Grouping:**
- By property: `.collect(Collectors.groupingBy(classifier))`
- With counting: `.collect(Collectors.groupingBy(classifier, counting()))`
- Partitioning: `.collect(Collectors.partitioningBy(predicate))`

**Sorting:**
- Ascending: `.sorted()`
- Descending: `.sorted(Comparator.reverseOrder())`
- By property: `.sorted(Comparator.comparing(getter))`

**String Operations:**
- Join: `.collect(Collectors.joining(", "))`
- Reverse: `new StringBuilder(str).reverse()`
- Character frequency: `.chars().collect(Collectors.groupingBy(...))`

### Performance Tips

**✅ Do:**
- Use primitive streams for numeric operations
- Use parallel streams for large datasets
- Use method references for readability
- Chain operations efficiently
- Use `distinct()` to remove duplicates

**❌ Avoid:**
- Parallel streams for small data
- Modifying external state in streams
- Calling `get()` on Optional without checking
- Boxing/unboxing when unnecessary
- Using streams for simple iterations

### Best Practices

1. ✅ Use appropriate stream type (sequential vs parallel, object vs primitive)
2. ✅ Handle Optional safely with `orElse()` or `orElseThrow()`
3. ✅ Use method references for cleaner code
4. ✅ Chain operations logically (filter early, transform later)
5. ✅ Use collectors for complex aggregations
6. ✅ Test parallel streams for actual performance benefit
7. ✅ Keep stream operations stateless and side-effect-free
8. ✅ Use `distinct()` before `count()` for unique counts
9. ✅ Leverage specialized collectors (`joining`, `groupingBy`, `partitioningBy`)
10. ✅ Remember streams can only be consumed once
