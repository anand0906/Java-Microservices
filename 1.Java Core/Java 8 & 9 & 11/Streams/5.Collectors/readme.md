# Collectors in Java Streams

## Table of Contents

1. [Understanding Collectors](#1-understanding-collectors)
2. [Collection Collectors](#2-collection-collectors)
3. [joining() - String Operations](#3-joining---string-operations)
4. [counting() - Counting Elements](#4-counting---counting-elements)
5. [Numeric Aggregations - Sum](#5-numeric-aggregations---sum)
6. [Numeric Aggregations - Average](#6-numeric-aggregations---average)
7. [Numeric Aggregations - Statistics](#7-numeric-aggregations---statistics)
8. [groupingBy() - Grouping Elements](#8-groupingby---grouping-elements)
9. [partitioningBy() - Binary Classification](#9-partitioningby---binary-classification)
10. [toMap() - Creating Maps](#10-tomap---creating-maps)
11. [Downstream Collectors](#11-downstream-collectors)
12. [Quick Revision](#quick-revision)

---

## 1. Understanding Collectors

### What is a Collector?

**Definition:**
- A recipe that describes how to accumulate stream elements into a final result
- Used with the `collect()` terminal operation
- Encapsulates the accumulation logic

### Four Core Functions

**Every Collector Contains:**
1. **Supplier:** Creates a new result container
2. **Accumulator:** Adds an element to the container
3. **Combiner:** Merges two containers (for parallel processing)
4. **Finisher:** Transforms the container to final result (optional)

### Key Characteristics

**Properties:**
- **Mutable reduction:** Builds result by modifying a container
- **Thread-safe:** Can work with parallel streams
- **Composable:** Can combine multiple collectors
- **Reusable:** Same collector instance can be used multiple times
- **Efficient:** Optimized for common operations

### Types of Collectors

**Categories:**
- **Collecting into Collections:** `toList()`, `toSet()`, `toMap()`
- **String Operations:** `joining()`
- **Numeric Aggregations:** `summingInt()`, `averagingInt()`, `summarizingInt()`
- **Reduction:** `reducing()`
- **Grouping:** `groupingBy()`
- **Partitioning:** `partitioningBy()`
- **Downstream Collectors:** Used as second argument to `groupingBy`/`partitioningBy`

---

## 2. Collection Collectors

### toList(), toSet(), toCollection()

**Method Signatures:**
- `Collector<T, ?, List<T>> toList()`
- `Collector<T, ?, Set<T>> toSet()`
- `Collector<T, ?, C> toCollection(Supplier<C> collectionFactory)`

### Key Points

**Characteristics:**
- `toList()`: Returns an ArrayList (implementation not guaranteed)
- `toSet()`: Returns a HashSet (implementation not guaranteed)
- `toCollection()`: Allows specifying exact collection type
- All return mutable collections
- No guarantees about thread-safety or serializability

### When to Use Which

| Use Case | Method | Reason |
|----------|--------|--------|
| Ordered collection with duplicates | `toList()` | Maintains order, allows duplicates |
| Unique elements | `toSet()` | Automatically removes duplicates |
| Specific collection type | `toCollection()` | Full control over implementation |

### Examples

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Example 1: To List (most common)
List<Integer> list = numbers.stream()
    .collect(Collectors.toList());
// Output: [1, 2, 3, 4, 5]

// Example 2: To Set (removes duplicates)
Set<Integer> set = Arrays.asList(1, 2, 2, 3, 3, 3).stream()
    .collect(Collectors.toSet());
// Output: [1, 2, 3]

// Example 3: To specific collection (ArrayList)
ArrayList<Integer> arrayList = numbers.stream()
    .collect(Collectors.toCollection(ArrayList::new));
// Guaranteed to be ArrayList

// Example 4: To LinkedList
LinkedList<Integer> linkedList = numbers.stream()
    .collect(Collectors.toCollection(LinkedList::new));

// Example 5: To TreeSet (sorted set)
TreeSet<String> treeSet = Stream.of("C", "A", "B", "A")
    .collect(Collectors.toCollection(TreeSet::new));
// Output: [A, B, C] (sorted, no duplicates)

// Example 6: To unmodifiable list (Java 10+)
List<Integer> immutableList = numbers.stream()
    .collect(Collectors.toUnmodifiableList());
// Cannot be modified

// Example 7: Custom collection
class CustomList<E> extends ArrayList<E> {
    // Custom implementation
}

CustomList<Integer> custom = numbers.stream()
    .collect(Collectors.toCollection(CustomList::new));
```

### Performance Note

```java
// These are functionally equivalent
List<String> list1 = stream.collect(Collectors.toList());
List<String> list2 = stream.collect(ArrayList::new, ArrayList::add, ArrayList::addAll);

// But toList() is more readable and potentially optimized
```

---

## 3. joining() - String Operations

**Method Signatures:**
- `Collector<CharSequence, ?, String> joining()`
- `Collector<CharSequence, ?, String> joining(CharSequence delimiter)`
- `Collector<CharSequence, ?, String> joining(CharSequence delimiter, CharSequence prefix, CharSequence suffix)`

### Key Points

**Characteristics:**
- Works only with CharSequence (String, StringBuilder, etc.)
- Uses StringBuilder internally for efficiency
- Can add delimiter between elements
- Can add prefix and suffix to result
- Empty stream produces empty string (or just prefix+suffix)

### Common Use Cases

- Creating CSV strings
- Building SQL queries
- Formatting output
- Creating comma-separated lists

### Examples

```java
List<String> words = Arrays.asList("Java", "is", "awesome");

// Example 1: Simple join (no delimiter)
String sentence1 = words.stream()
    .collect(Collectors.joining());
// Output: "Javaisawesome"

// Example 2: Join with delimiter (most common)
String sentence2 = words.stream()
    .collect(Collectors.joining(" "));
// Output: "Java is awesome"

// Example 3: Join with delimiter, prefix, and suffix
String sentence3 = words.stream()
    .collect(Collectors.joining(", ", "[", "]"));
// Output: "[Java, is, awesome]"

// Example 4: CSV format
List<String> data = Arrays.asList("John", "30", "Engineer");
String csv = data.stream()
    .collect(Collectors.joining(","));
// Output: "John,30,Engineer"

// Example 5: SQL IN clause
List<Integer> ids = Arrays.asList(1, 2, 3, 4, 5);
String sqlIn = ids.stream()
    .map(String::valueOf)
    .collect(Collectors.joining(",", "WHERE id IN (", ")"));
// Output: "WHERE id IN (1,2,3,4,5)"

// Example 6: HTML list
List<String> items = Arrays.asList("Apple", "Banana", "Cherry");
String html = items.stream()
    .collect(Collectors.joining("</li><li>", "<ul><li>", "</li></ul>"));
// Output: "<ul><li>Apple</li><li>Banana</li><li>Cherry</li></ul>"

// Example 7: Join after transformation
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
String joined = numbers.stream()
    .map(n -> "Item" + n)
    .collect(Collectors.joining(", "));
// Output: "Item1, Item2, Item3, Item4, Item5"

// Example 8: Empty stream
String empty = Stream.<String>empty()
    .collect(Collectors.joining(", ", "[", "]"));
// Output: "[]"
```

### Performance Comparison

```java
// SLOW - creates many intermediate String objects
String result = "";
for (String s : list) {
    result += s + ",";
}

// FAST - uses StringBuilder internally
String result = list.stream()
    .collect(Collectors.joining(","));
```

---

## 4. counting() - Counting Elements

**Method Signature:** `Collector<T, ?, Long> counting()`

### Key Points

**Characteristics:**
- Returns Long (not int)
- Equivalent to `stream.count()` but more composable
- Useful in grouping/partitioning operations
- Can be used as downstream collector

### Examples

```java
// Example 1: Simple count
List<String> words = Arrays.asList("A", "B", "C", "D");
Long count = words.stream()
    .collect(Collectors.counting());
// Output: 4

// Example 2: Count with filter
Long evenCount = Arrays.asList(1, 2, 3, 4, 5).stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.counting());
// Output: 2

// Example 3: As downstream collector (more useful)
Map<Integer, Long> countByLength = Stream.of("A", "BB", "CCC", "DD")
    .collect(Collectors.groupingBy(
        String::length,
        Collectors.counting()
    ));
// Output: {1=1, 2=2, 3=1}
```

### When to Use counting() vs count()

```java
// Use count() for simple counting
long count1 = stream.count();

// Use counting() as downstream collector
Map<K, Long> counts = stream.collect(
    Collectors.groupingBy(classifier, Collectors.counting())
);
```

---

## 5. Numeric Aggregations - Sum

### summingInt(), summingLong(), summingDouble()

**Method Signatures:**
- `Collector<T, ?, Integer> summingInt(ToIntFunction<? super T> mapper)`
- `Collector<T, ?, Long> summingLong(ToLongFunction<? super T> mapper)`
- `Collector<T, ?, Double> summingDouble(ToDoubleFunction<? super T> mapper)`

### Key Points

**Characteristics:**
- Maps elements to numeric values and sums them
- Returns primitive wrapper (Integer, Long, Double)
- More composable than `reduce()` for summing
- Can be used as downstream collector
- Handles overflow differently based on type

### Examples

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Example 1: Sum integers
Integer sum = numbers.stream()
    .collect(Collectors.summingInt(Integer::intValue));
// Output: 15

// Example 2: Sum of string lengths
List<String> words = Arrays.asList("Hi", "Hello", "Hey");
Integer totalLength = words.stream()
    .collect(Collectors.summingInt(String::length));
// Output: 10

// Example 3: Sum with custom extractor
class Product {
    String name;
    int quantity;
    Product(String name, int quantity) {
        this.name = name;
        this.quantity = quantity;
    }
}

List<Product> products = Arrays.asList(
    new Product("Apple", 10),
    new Product("Banana", 15),
    new Product("Orange", 8)
);

Integer totalQuantity = products.stream()
    .collect(Collectors.summingInt(p -> p.quantity));
// Output: 33

// Example 4: Sum of doubles
class Item {
    double price;
    Item(double price) { this.price = price; }
}

List<Item> items = Arrays.asList(
    new Item(10.5),
    new Item(20.3),
    new Item(15.7)
);

Double totalPrice = items.stream()
    .collect(Collectors.summingDouble(i -> i.price));
// Output: 46.5
```

---

## 6. Numeric Aggregations - Average

### averagingInt(), averagingLong(), averagingDouble()

**Method Signatures:**
- `Collector<T, ?, Double> averagingInt(ToIntFunction<? super T> mapper)`
- `Collector<T, ?, Double> averagingLong(ToLongFunction<? super T> mapper)`
- `Collector<T, ?, Double> averagingDouble(ToDoubleFunction<? super T> mapper)`

### Key Points

**Characteristics:**
- Maps elements to numeric values and calculates average
- Always returns Double (even for Int/Long versions)
- Returns 0.0 for empty stream
- Handles precision better than manual calculation

### Examples

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Example 1: Average of integers
Double average = numbers.stream()
    .collect(Collectors.averagingInt(Integer::intValue));
// Output: 3.0

// Example 2: Average string length
List<String> words = Arrays.asList("Hi", "Hello", "Hey", "World");
Double avgLength = words.stream()
    .collect(Collectors.averagingInt(String::length));
// Output: 4.0

// Example 3: Average with custom extractor
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

Double avgScore = students.stream()
    .collect(Collectors.averagingInt(s -> s.score));
// Output: 85.0

// Example 4: Empty stream
Double avgEmpty = Stream.<Integer>empty()
    .collect(Collectors.averagingInt(Integer::intValue));
// Output: 0.0
```

---

## 7. Numeric Aggregations - Statistics

### summarizingInt(), summarizingLong(), summarizingDouble()

**Method Signature:** `Collector<T, ?, IntSummaryStatistics> summarizingInt(ToIntFunction<? super T> mapper)`

### Key Points

**Characteristics:**
- Calculates multiple statistics in single pass (efficient!)
- Returns SummaryStatistics object with getters
- Available methods: `getCount()`, `getSum()`, `getMin()`, `getAverage()`, `getMax()`
- Min/Max return extreme values for empty stream

### Statistics Available

- **count:** Number of elements
- **sum:** Sum of all values
- **min:** Minimum value
- **max:** Maximum value
- **average:** Average value

### Examples

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Example 1: Get all statistics at once
IntSummaryStatistics stats = numbers.stream()
    .collect(Collectors.summarizingInt(Integer::intValue));

System.out.println("Count: " + stats.getCount());       // 5
System.out.println("Sum: " + stats.getSum());           // 15
System.out.println("Min: " + stats.getMin());           // 1
System.out.println("Average: " + stats.getAverage());   // 3.0
System.out.println("Max: " + stats.getMax());           // 5

// Example 2: Statistics for custom objects
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
    new Employee("Bob", 80000),
    new Employee("Charlie", 90000)
);

IntSummaryStatistics salaryStats = employees.stream()
    .collect(Collectors.summarizingInt(e -> e.salary));

System.out.println("Average Salary: " + salaryStats.getAverage());  // 80000.0
System.out.println("Highest Salary: " + salaryStats.getMax());      // 90000
System.out.println("Lowest Salary: " + salaryStats.getMin());       // 70000

// Example 3: DoubleSummaryStatistics for prices
class Product {
    String name;
    double price;
    Product(String name, double price) {
        this.name = name;
        this.price = price;
    }
}

List<Product> products = Arrays.asList(
    new Product("Laptop", 1200.50),
    new Product("Mouse", 25.99),
    new Product("Keyboard", 75.00)
);

DoubleSummaryStatistics priceStats = products.stream()
    .collect(Collectors.summarizingDouble(p -> p.price));

System.out.println("Average Price: $" + priceStats.getAverage());
System.out.println("Price Range: $" + priceStats.getMin() + 
                   " - $" + priceStats.getMax());
```

### Why Use summarizing?

```java
// INEFFICIENT - multiple passes over data (ERROR!)
long count = stream.count();
int sum = stream.mapToInt(i -> i).sum();  // Stream already consumed!
OptionalInt min = stream.mapToInt(i -> i).min();

// EFFICIENT - single pass
IntSummaryStatistics stats = stream
    .collect(Collectors.summarizingInt(Integer::intValue));
// All stats available from one pass
```

---

## 8. groupingBy() - Grouping Elements

**Method Signatures:**
- `groupingBy(Function<? super T, ? extends K> classifier)`
- `groupingBy(Function classifier, Collector<? super T, A, D> downstream)`
- `groupingBy(Function classifier, Supplier<M> mapFactory, Collector downstream)`

### Key Points

**Characteristics:**
- Most powerful collector for complex aggregations
- Creates `Map<K, List<T>>` by default
- Key: Result of classifier function
- Value: List of elements with that key (customizable with downstream collector)
- Can nest `groupingBy` for multi-level grouping
- MapFactory allows custom Map implementation (LinkedHashMap, TreeMap, etc.)

### Common Patterns

- **Group by property:** `groupingBy(Person::getCity)`
- **Count by group:** `groupingBy(classifier, counting())`
- **Sum by group:** `groupingBy(classifier, summingInt(mapper))`
- **Nested grouping:** `groupingBy(classifier1, groupingBy(classifier2))`

### Examples

```java
// Example 1: Group by length (simple)
List<String> words = Arrays.asList("Hi", "Hello", "Hey", "Yes", "No");
Map<Integer, List<String>> byLength = words.stream()
    .collect(Collectors.groupingBy(String::length));
// Output: {2=[Hi, No], 3=[Hey, Yes], 5=[Hello]}

// Example 2: Group even and odd numbers
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
Map<String, List<Integer>> evenOdd = numbers.stream()
    .collect(Collectors.groupingBy(
        n -> n % 2 == 0 ? "Even" : "Odd"
    ));
// Output: {Odd=[1, 3, 5], Even=[2, 4, 6]}

// Example 3: Count by group (downstream collector)
Map<Integer, Long> countByLength = words.stream()
    .collect(Collectors.groupingBy(
        String::length,
        Collectors.counting()
    ));
// Output: {2=2, 3=2, 5=1}

// Example 4: Sum by group
class Sale {
    String product;
    int quantity;
    Sale(String product, int quantity) {
        this.product = product;
        this.quantity = quantity;
    }
}

List<Sale> sales = Arrays.asList(
    new Sale("Apple", 10),
    new Sale("Banana", 15),
    new Sale("Apple", 5),
    new Sale("Banana", 10)
);

Map<String, Integer> totalByProduct = sales.stream()
    .collect(Collectors.groupingBy(
        s -> s.product,
        Collectors.summingInt(s -> s.quantity)
    ));
// Output: {Apple=15, Banana=25}

// Example 5: Collect to Set instead of List
Map<Integer, Set<String>> setsByLength = words.stream()
    .collect(Collectors.groupingBy(
        String::length,
        Collectors.toSet()
    ));
// Output: {2=[Hi, No], 3=[Hey, Yes], 5=[Hello]}

// Example 6: Multi-level grouping (nested)
class Person {
    String city;
    String gender;
    int age;
    Person(String city, String gender, int age) {
        this.city = city;
        this.gender = gender;
        this.age = age;
    }
}

List<Person> people = Arrays.asList(
    new Person("NYC", "M", 25),
    new Person("NYC", "F", 30),
    new Person("LA", "M", 28),
    new Person("LA", "F", 22)
);

Map<String, Map<String, List<Person>>> grouped = people.stream()
    .collect(Collectors.groupingBy(
        p -> p.city,
        Collectors.groupingBy(p -> p.gender)
    ));
// Output: {NYC={M=[Person(25)], F=[Person(30)]}, LA={M=[Person(28)], F=[Person(22)]}}

// Example 7: Use TreeMap for sorted keys
Map<Integer, List<String>> sortedByLength = words.stream()
    .collect(Collectors.groupingBy(
        String::length,
        TreeMap::new,
        Collectors.toList()
    ));
// Keys will be in sorted order

// Example 8: Average by group
class Student {
    String course;
    int score;
    Student(String course, int score) {
        this.course = course;
        this.score = score;
    }
}

List<Student> students = Arrays.asList(
    new Student("Math", 85),
    new Student("Math", 92),
    new Student("Science", 78),
    new Student("Science", 88)
);

Map<String, Double> avgByCourse = students.stream()
    .collect(Collectors.groupingBy(
        s -> s.course,
        Collectors.averagingInt(s -> s.score)
    ));
// Output: {Math=88.5, Science=83.0}

// Example 9: Mapping values in groups
Map<Integer, List<String>> upperByLength = words.stream()
    .collect(Collectors.groupingBy(
        String::length,
        Collectors.mapping(
            String::toUpperCase,
            Collectors.toList()
        )
    ));
// Output: {2=[HI, NO], 3=[HEY, YES], 5=[HELLO]}
```

### Advanced Patterns

```java
// Pattern: Group and get first element
Map<K, Optional<V>> firstByGroup = stream.collect(
    Collectors.groupingBy(
        classifier,
        Collectors.reducing((a, b) -> a)  // Keep first
    )
);

// Pattern: Group and join strings
Map<K, String> joinedByGroup = stream.collect(
    Collectors.groupingBy(
        classifier,
        Collectors.mapping(
            mapper,
            Collectors.joining(", ")
        )
    )
);
```

---

## 9. partitioningBy() - Binary Classification

**Method Signatures:**
- `partitioningBy(Predicate<? super T> predicate)`
- `partitioningBy(Predicate<? super T> predicate, Collector<? super T, A, D> downstream)`

### Key Points

**Characteristics:**
- Special case of `groupingBy` with exactly two groups
- Returns `Map<Boolean, List<T>>`
- Key is always Boolean (true or false)
- More efficient than `groupingBy` for binary classification
- Both keys always present in result (even if one group is empty)

### partitioningBy vs groupingBy

| Criterion | Use partitioningBy | Use groupingBy |
|-----------|-------------------|----------------|
| Categories | Binary (yes/no, pass/fail) | Multiple categories |
| Keys | Always Boolean | Any type |
| Efficiency | More efficient for 2 groups | General purpose |

### Examples

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);

// Example 1: Partition by even/odd
Map<Boolean, List<Integer>> partitioned = numbers.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));
// Output: {false=[1, 3, 5, 7], true=[2, 4, 6, 8]}

List<Integer> evens = partitioned.get(true);
List<Integer> odds = partitioned.get(false);

// Example 2: Partition with counting
Map<Boolean, Long> countPartition = numbers.stream()
    .collect(Collectors.partitioningBy(
        n -> n % 2 == 0,
        Collectors.counting()
    ));
// Output: {false=4, true=4}

// Example 3: Partition students by pass/fail
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
    new Student("Bob", 55),
    new Student("Charlie", 92),
    new Student("David", 45)
);

Map<Boolean, List<Student>> passFail = students.stream()
    .collect(Collectors.partitioningBy(s -> s.score >= 60));
// true = passed, false = failed

// Example 4: Partition with average
Map<Boolean, Double> avgScoreByPass = students.stream()
    .collect(Collectors.partitioningBy(
        s -> s.score >= 60,
        Collectors.averagingInt(s -> s.score)
    ));
// Output: {false=50.0, true=88.5}

// Example 5: Partition and map
Map<Boolean, List<String>> namesByPass = students.stream()
    .collect(Collectors.partitioningBy(
        s -> s.score >= 60,
        Collectors.mapping(
            s -> s.name,
            Collectors.toList()
        )
    ));
// Output: {false=[Bob, David], true=[Alice, Charlie]}

// Example 6: Empty partitions still present
Map<Boolean, List<Integer>> allPositive = Arrays.asList(1, 2, 3).stream()
    .collect(Collectors.partitioningBy(n -> n < 0));
// Output: {false=[1, 2, 3], true=[]}
// Note: true key is present with empty list
```

---

## 10. toMap() - Creating Maps

**Method Signatures:**
- `toMap(Function<? super T, ? extends K> keyMapper, Function<? super T, ? extends U> valueMapper)`
- `toMap(Function keyMapper, Function valueMapper, BinaryOperator<U> mergeFunction)`
- `toMap(Function keyMapper, Function valueMapper, BinaryOperator<U> mergeFunction, Supplier<M> mapSupplier)`

### Key Points

**Characteristics:**
- Converts stream elements to key-value pairs
- Throws `IllegalStateException` if duplicate keys (unless merge function provided)
- Merge function handles duplicate keys
- Map supplier allows custom Map implementation
- Useful for lookups and indexing

### Examples

```java
// Example 1: Simple map (word -> length)
List<String> words = Arrays.asList("A", "BB", "CCC");
Map<String, Integer> map1 = words.stream()
    .collect(Collectors.toMap(
        word -> word,           // key
        word -> word.length()   // value
    ));
// Output: {A=1, BB=2, CCC=3}

// Using Function.identity() for key
Map<String, Integer> map2 = words.stream()
    .collect(Collectors.toMap(
        Function.identity(),
        String::length
    ));

// Example 2: Handle duplicates with merge function
List<String> words2 = Arrays.asList("A", "B", "A", "C");
Map<String, Integer> map3 = words2.stream()
    .collect(Collectors.toMap(
        word -> word,
        word -> word.length(),
        (existing, replacement) -> existing  // Keep existing
    ));
// Output: {A=1, B=1, C=1}

// Example 3: Sum duplicate values
class Sale {
    String product;
    int amount;
    Sale(String product, int amount) {
        this.product = product;
        this.amount = amount;
    }
}

List<Sale> sales = Arrays.asList(
    new Sale("Apple", 10),
    new Sale("Banana", 15),
    new Sale("Apple", 5)
);

Map<String, Integer> totalSales = sales.stream()
    .collect(Collectors.toMap(
        s -> s.product,
        s -> s.amount,
        Integer::sum  // Sum duplicate keys
    ));
// Output: {Apple=15, Banana=15}

// Example 4: Object to Map (entity indexing)
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

Map<Integer, Person> personById = people.stream()
    .collect(Collectors.toMap(
        p -> p.id,
        Function.identity()
    ));
// Output: {1=Person(Alice), 2=Person(Bob), 3=Person(Charlie)}

// Example 5: Use TreeMap for sorted keys
Map<String, Integer> sortedMap = words.stream()
    .collect(Collectors.toMap(
        Function.identity(),
        String::length,
        (a, b) -> a,  // Merge function (required even if no duplicates)
        TreeMap::new  // Sorted map
    ));

// Example 6: Invert map (swap keys and values)
Map<String, Integer> original = Map.of("A", 1, "B", 2, "C", 3);
Map<Integer, String> inverted = original.entrySet().stream()
    .collect(Collectors.toMap(
        Map.Entry::getValue,  // Value becomes key
        Map.Entry::getKey     // Key becomes value
    ));
// Output: {1=A, 2=B, 3=C}
```

---

## 11. Downstream Collectors

### What are Downstream Collectors?

**Definition:**
- Collectors used as the second argument to `groupingBy`, `partitioningBy`, or other collectors
- Allow complex nested aggregations
- Transform grouped data before final collection

### Common Downstream Collectors

**Available Options:**
- `counting()` - Count elements in each group
- `summingInt()` - Sum values in each group
- `averagingInt()` - Average values in each group
- `mapping()` - Transform elements before collecting
- `filtering()` - Filter elements before collecting (Java 9+)
- `flatMapping()` - Flatten elements before collecting (Java 9+)
- `reducing()` - Reduce elements in each group
- `toList()`, `toSet()` - Collect to different collection type
- `maxBy()`, `minBy()` - Find extreme in each group

### Examples

```java
class Transaction {
    String category;
    double amount;
    String merchant;
    Transaction(String category, double amount, String merchant) {
        this.category = category;
        this.amount = amount;
        this.merchant = merchant;
    }
}

List<Transaction> transactions = Arrays.asList(
    new Transaction("Food", 25.0, "Restaurant A"),
    new Transaction("Food", 40.0, "Restaurant B"),
    new Transaction("Transport", 15.0, "Taxi"),
    new Transaction("Transport", 20.0, "Bus")
);

// Example 1: mapping - transform before collecting
Map<String, List<String>> merchantsByCategory = transactions.stream()
    .collect(Collectors.groupingBy(
        t -> t.category,
        Collectors.mapping(
            t -> t.merchant,
            Collectors.toList()
        )
    ));
// Output: {Food=[Restaurant A, Restaurant B], Transport=[Taxi, Bus]}

// Example 2: filtering - filter in each group (Java 9+)
Map<String, List<Transaction>> largeTransactions = transactions.stream()
    .collect(Collectors.groupingBy(
        t -> t.category,
        Collectors.filtering(
            t -> t.amount > 20,
            Collectors.toList()
        )
    ));
// Output: {Food=[Transaction(40.0)], Transport=[]}

// Example 3: Average in each group
Map<String, Double> avgAmountByCategory = transactions.stream()
    .collect(Collectors.groupingBy(
        t -> t.category,
        Collectors.averagingDouble(t -> t.amount)
    ));
// Output: {Food=32.5, Transport=17.5}

// Example 4: maxBy/minBy - find extreme in each group
Map<String, Optional<Transaction>> maxByCategory = transactions.stream()
    .collect(Collectors.groupingBy(
        t -> t.category,
        Collectors.maxBy(Comparator.comparing(t -> t.amount))
    ));
// Output: {Food=Optional[Transaction(40.0)], Transport=Optional[Transaction(20.0)]}

// Example 5: collectingAndThen - transform result
Map<String, Long> result = transactions.stream()
    .collect(Collectors.groupingBy(
        t -> t.category,
        Collectors.collectingAndThen(
            Collectors.counting(),
            count -> count * 2  // Transform the result
        )
    ));
```

---

## Quick Revision

### Summary of Collectors

| Collector | Purpose | Return Type |
|-----------|---------|-------------|
| `toList()` | Collect to List | `List<T>` |
| `toSet()` | Collect to Set | `Set<T>` |
| `toMap()` | Collect to Map | `Map<K,V>` |
| `joining()` | Join strings | `String` |
| `counting()` | Count elements | `Long` |
| `summingInt()` | Sum values | `Integer` |
| `averagingInt()` | Average values | `Double` |
| `summarizingInt()` | Statistics | `IntSummaryStatistics` |
| `groupingBy()` | Group elements | `Map<K,List<V>>` |
| `partitioningBy()` | Partition by predicate | `Map<Boolean,List<T>>` |
| `reducing()` | Reduce to single value | `T` or `Optional<T>` |

### Collector Categories

**1. Collection Collectors:**
- `toList()` - Most common, maintains order
- `toSet()` - Removes duplicates
- `toCollection()` - Specify exact type

**2. String Operations:**
- `joining()` - Concatenate with delimiter
- `joining(delimiter, prefix, suffix)` - Full control

**3. Numeric Operations:**
- `summingInt/Long/Double()` - Sum values
- `averagingInt/Long/Double()` - Calculate average
- `summarizingInt/Long/Double()` - All stats in one pass

**4. Grouping Operations:**
- `groupingBy()` - Group by classifier
- `partitioningBy()` - Binary classification (true/false)

**5. Map Operations:**
- `toMap()` - Create maps from streams
- Handle duplicates with merge function

### When to Use Which Collector

| Scenario | Collector | Example |
|----------|-----------|---------|
| Need ordered list | `toList()` | `collect(Collectors.toList())` |
| Need unique elements | `toSet()` | `collect(Collectors.toSet())` |
| Join strings | `joining()` | `collect(Collectors.joining(", "))` |
| Count elements | `counting()` | `collect(Collectors.counting())` |
| Sum numbers | `summingInt()` | `collect(Collectors.summingInt(mapper))` |
| Get statistics | `summarizingInt()` | `collect(Collectors.summarizingInt(mapper))` |
| Group by property | `groupingBy()` | `collect(Collectors.groupingBy(classifier))` |
| Binary split | `partitioningBy()` | `collect(Collectors.partitioningBy(predicate))` |
| Create lookup map | `toMap()` | `collect(Collectors.toMap(k, v))` |

### Common Patterns

**Pattern 1: Count by Category**
```java
Map<Category, Long> counts = stream
    .collect(Collectors.groupingBy(
        Item::getCategory,
        Collectors.counting()
    ));
```

**Pattern 2: Sum by Group**
```java
Map<String, Integer> totals = stream
    .collect(Collectors.groupingBy(
        Item::getType,
        Collectors.summingInt(Item::getValue)
    ));
```

**Pattern 3: Nested Grouping**
```java
Map<String, Map<String, List<Item>>> nested = stream
    .collect(Collectors.groupingBy(
        Item::getCategory,
        Collectors.groupingBy(Item::getType)
    ));
```

**Pattern 4: Partition and Count**
```java
Map<Boolean, Long> partition = stream
    .collect(Collectors.partitioningBy(
        item -> item.getValue() > 100,
        Collectors.counting()
    ));
```

**Pattern 5: Create Lookup Map**
```java
Map<Integer, Person> lookup = stream
    .collect(Collectors.toMap(
        Person::getId,
        Function.identity()
    ));
```

### Key Principles

1. **Choose the right collector for your use case**
   - Don't use `toList()` when you need unique elements
   - Don't use `groupingBy()` for binary classification

2. **Combine collectors with downstream collectors**
   - `groupingBy` with `counting()` for frequency maps
   - `partitioningBy` with `averagingInt()` for split statistics

3. **Use primitive-specialized collectors appropriately**
   - `summingInt` vs `summingLong` vs `summingDouble`
   - Match the type to your data

4. **Handle duplicate keys in toMap**
   - Always provide merge function if duplicates possible
   - Use `Integer::sum` for summing duplicates

5. **groupingBy and partitioningBy are powerful**
   - Use for multi-level aggregations
   - Nest collectors for complex operations

### Performance Tips

**✅ Efficient:**
```java
// Single pass with summarizingInt
IntSummaryStatistics stats = stream
    .collect(Collectors.summarizingInt(mapper));

// Use joining() for strings
String result = stream.collect(Collectors.joining(","));
```

**❌ Inefficient:**
```java
// Multiple passes (stream consumed after first)
long count = stream.count();
int sum = stream.mapToInt(i -> i).sum();  // ERROR!

// String concatenation in loop
String result = "";
for (String s : list) result += s + ",";
```

### Common Pitfalls to Avoid

- ❌ Not handling duplicate keys in `toMap()`
- ❌ Using `groupingBy` when `partitioningBy` is more appropriate
- ❌ Multiple passes over stream instead of using `summarizingInt()`
- ❌ String concatenation instead of `joining()`
- ❌ Forgetting that `counting()` returns `Long`, not `int`
- ❌ Not providing map factory when order matters (`TreeMap`, `LinkedHashMap`)
- ❌ Using wrong numeric type (`summingInt` when you need `summingDouble`)

### Best Practices

1. ✅ Use `Function.identity()` instead of `x -> x`
2. ✅ Use method references when possible
3. ✅ Provide merge functions for `toMap` when duplicates expected
4. ✅ Use `summarizingInt()` for multiple statistics in one pass
5. ✅ Choose `partitioningBy` for binary classification
6. ✅ Specify map factory when order matters
7. ✅ Use downstream collectors for complex aggregations
8. ✅ Prefer `joining()` over manual string concatenation
