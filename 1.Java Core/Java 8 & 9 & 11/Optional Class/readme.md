# Optional Class in Java

## Table of Contents

### Part 1: Fundamentals
- [1.1 What is Optional?](#11-what-is-optional)
- [1.2 Why Optional Exists](#12-why-optional-exists)
- [1.3 Core Concepts](#13-core-concepts)
- [1.4 When to Use Optional](#14-when-to-use-optional)

### Part 2: Creating Optional Objects
- [2.1 Optional.of()](#21-optionalof)
- [2.2 Optional.ofNullable()](#22-optionalofnullable)
- [2.3 Optional.empty()](#23-optionalempty)
- [2.4 Choosing the Right Method](#24-choosing-the-right-method)

### Part 3: Working with Optional
- [3.1 Checking for Values](#31-checking-for-values)
- [3.2 Retrieving Values](#32-retrieving-values)
- [3.3 Transforming Values](#33-transforming-values)
- [3.4 Filtering Values](#34-filtering-values)

### Part 4: Advanced Usage
- [4.1 Optional with Streams](#41-optional-with-streams)
- [4.2 Chaining Operations](#42-chaining-operations)
- [4.3 Primitive Optionals](#43-primitive-optionals)

### Part 5: Best Practices
- [5.1 Do's and Don'ts](#51-dos-and-donts)
- [5.2 Common Patterns](#52-common-patterns)
- [5.3 Anti-Patterns to Avoid](#53-anti-patterns-to-avoid)

### Part 6: Quick Reference
- [6.1 Method Summary](#61-method-summary)
- [6.2 Decision Tree](#62-decision-tree)
- [6.3 Key Takeaways](#63-key-takeaways)

---

# Part 1: Fundamentals

## 1.1 What is Optional?

### Definition

**Optional<T>** is a container object introduced in Java 8 that may or may not contain a non-null value.

```java
public final class Optional<T> {
    // Contains either a value or is empty (never null itself)
}
```

### Three Possible States

| State | Description | Example |
|-------|-------------|---------|
| **Present** | Contains a non-null value | `Optional[Alice]` |
| **Empty** | Contains no value | `Optional.empty` |
| **Invalid** | Optional itself is null | ❌ Anti-pattern |

### Basic Example

```java
// Creating Optional
Optional<String> presentOptional = Optional.of("Alice");
Optional<String> emptyOptional = Optional.empty();

// Checking presence
System.out.println(presentOptional.isPresent());  // true
System.out.println(emptyOptional.isPresent());    // false
```

---

## 1.2 Why Optional Exists

### The Billion-Dollar Mistake

**Tony Hoare** (inventor of null references) called null his **"billion-dollar mistake"** due to countless bugs, crashes, and security vulnerabilities it has caused.

### Problems with null

#### 1. NullPointerException (NPE)
Most common runtime exception in Java.

```java
User user = findUser(id);  // might return null
String email = user.getEmail();  // NPE if user is null!
```

#### 2. No Compile-Time Safety
Compiler cannot help prevent NPEs.

```java
// Compiler doesn't warn you
public String processUser(User user) {
    return user.getName();  // What if user is null?
}
```

#### 3. Unclear Semantics
Is null a valid value or an error?

```java
// What does null mean here?
String result = findUserName(id);  // Not found? Error? Bug?
```

#### 4. Defensive Programming Hell

```java
// Nested null checks everywhere
String city = null;
if (user != null) {
    Address address = user.getAddress();
    if (address != null) {
        city = address.getCity();
        if (city != null) {
            return city.toUpperCase();
        }
    }
}
return "UNKNOWN";
```

### The Optional Solution

```java
// Clean, explicit, functional
return Optional.ofNullable(user)
    .map(User::getAddress)
    .map(Address::getCity)
    .map(String::toUpperCase)
    .orElse("UNKNOWN");
```

### Benefits of Optional

| Benefit | Description |
|---------|-------------|
| **Explicit Intent** | Method signature clearly indicates value might be absent |
| **Compile-Time Safety** | Forces you to handle absence case |
| **Functional Style** | Enables chaining and composition |
| **Self-Documenting** | Code clearly shows null handling strategy |
| **Fewer Bugs** | Prevents forgotten null checks |

---

## 1.3 Core Concepts

### Type-Level Indication

Optional makes the **possibility of absence explicit** in the type system.

```java
// Implicit - not obvious value can be null
public String findUserName(Long id) { }

// Explicit - clearly states value might be absent
public Optional<String> findUserName(Long id) { }
```

### Container Metaphor

Think of Optional as a **box**:
- Box can contain one item (present)
- Box can be empty (absent)
- Box itself is never null
- You can transform what's in the box
- You can provide defaults if box is empty

```
┌─────────────────┐         ┌─────────────────┐
│ Optional        │         │ Optional        │
│                 │         │                 │
│  ┌─────────┐   │         │                 │
│  │ "Alice" │   │         │    (empty)      │
│  └─────────┘   │         │                 │
│                 │         │                 │
└─────────────────┘         └─────────────────┘
   Present                      Empty
```

### Immutability

Optional is **immutable**. Operations return new Optionals.

```java
Optional<String> original = Optional.of("hello");
Optional<String> transformed = original.map(String::toUpperCase);

// original is unchanged
System.out.println(original.get());     // "hello"
System.out.println(transformed.get());  // "HELLO"
```

---

## 1.4 When to Use Optional

### ✅ DO Use Optional For

| Use Case | Example |
|----------|---------|
| **Method return types** | `Optional<User> findById(Long id)` |
| **Optional configuration** | `Optional<String> getConfig(String key)` |
| **Repository find operations** | `Optional<Order> findByOrderNumber(String num)` |
| **Chain of transformations** | `.map(User::getAddress).map(Address::getCity)` |

### ❌ DON'T Use Optional For

| Use Case | Why Not | Alternative |
|----------|---------|-------------|
| **Method parameters** | Caller might pass null | Method overloading |
| **Class fields** | Memory overhead, serialization issues | Use null |
| **Collections** | Collections have natural empty state | `Collections.emptyList()` |
| **Primary keys** | Required values shouldn't be Optional | Plain type |
| **Replacing every null** | Adds overhead | Use judiciously |

### Example: Good vs Bad

```java
// ✅ GOOD - Return type
public Optional<User> findUserById(Long id) {
    return Optional.ofNullable(database.get(id));
}

// ❌ BAD - Parameter
public void processUser(Optional<User> user) { }

// ✅ BETTER - Method overloading
public void processUser(User user) { }
public void processUser() { }

// ❌ BAD - Field
class Person {
    private Optional<String> email;
}

// ✅ BETTER - Null field, Optional getter
class Person {
    private String email;
    public Optional<String> getEmail() {
        return Optional.ofNullable(email);
    }
}
```

---

# Part 2: Creating Optional Objects

## 2.1 Optional.of()

### Purpose
Creates an Optional containing a **non-null** value.

### Signature
```java
public static <T> Optional<T> of(T value)
```

### Behavior
- ✅ Value is non-null → Returns `Optional[value]`
- ❌ Value is null → Throws `NullPointerException`

### When to Use
- You **guarantee** the value is not null
- Working with constants
- After explicit null checks
- From constructors (which never return null)

### Example

```java
// Creating Optional with guaranteed non-null value
Optional<String> name = Optional.of("Alice");
System.out.println(name);  // Optional[Alice]

// From object creation
User user = new User("Bob");
Optional<User> optUser = Optional.of(user);

// ❌ WRONG - throws NullPointerException
String nullValue = null;
// Optional<String> bad = Optional.of(nullValue);  // NPE!
```

### Key Points
- Use when you're **certain** value is not null
- Acts as a programmer assertion
- Fails fast if assumption is wrong

---

## 2.2 Optional.ofNullable()

### Purpose
Creates an Optional that **safely handles null** values.

### Signature
```java
public static <T> Optional<T> ofNullable(T value)
```

### Behavior
- ✅ Value is non-null → Returns `Optional[value]`
- ✅ Value is null → Returns `Optional.empty`

### When to Use
- Value **might be null** (most common case)
- Wrapping method calls that might return null
- Database/Map lookups
- **Default choice** when uncertain

### Example

```java
// Safe with any value
Optional<String> opt1 = Optional.ofNullable("Alice");  // Optional[Alice]
Optional<String> opt2 = Optional.ofNullable(null);     // Optional.empty

// Typical usage: repository pattern
public Optional<User> findById(Long id) {
    User user = database.get(id);  // might return null
    return Optional.ofNullable(user);
}

// Wrapping Map.get()
Map<String, String> map = new HashMap<>();
Optional<String> value = Optional.ofNullable(map.get("key"));
```

### Key Points
- **Most commonly used** creation method
- Always safe - never throws
- Use as default choice

---

## 2.3 Optional.empty()

### Purpose
Creates an **explicitly empty** Optional.

### Signature
```java
public static <T> Optional<T> empty()
```

### Behavior
- Returns a singleton empty Optional instance
- No memory overhead (reuses same instance)
- Type-safe representation of "no value"

### When to Use
- Explicit "no value" return
- Invalid input cases
- Default/fallback cases
- Search/find operations that don't find anything

### Example

```java
// Creating empty Optional
Optional<String> empty = Optional.empty();
System.out.println(empty.isPresent());  // false

// Early return for invalid input
public Optional<User> findUser(String username) {
    if (username == null || username.isEmpty()) {
        return Optional.empty();  // Invalid input
    }
    return Optional.ofNullable(database.find(username));
}

// Default case
public Optional<String> getStatus(int code) {
    switch (code) {
        case 200: return Optional.of("OK");
        case 404: return Optional.of("Not Found");
        default: return Optional.empty();
    }
}
```

### Key Points
- Better than returning null
- Use for "not found" scenarios
- Singleton instance (efficient)

---

## 2.4 Choosing the Right Method

### Decision Tree

```
Is the value definitely NOT null?
├─ YES → Optional.of(value)
└─ NO
   ├─ Might be null → Optional.ofNullable(value)
   └─ Explicitly no value → Optional.empty()
```

### Comparison Table

| Method | Input | Result if null | Use When |
|--------|-------|----------------|----------|
| `of(value)` | Non-null required | NPE | Guaranteed non-null |
| `ofNullable(value)` | Any value | `Optional.empty` | Might be null |
| `empty()` | No input | `Optional.empty` | Explicit absence |

### Example: Repository Pattern

```java
public class UserRepository {
    private Map<Long, User> users = new HashMap<>();
    
    public Optional<User> findById(Long id) {
        // ofNullable - map.get() might return null
        return Optional.ofNullable(users.get(id));
    }
    
    public Optional<User> createUser(String name) {
        if (name == null || name.isEmpty()) {
            // empty - validation failed
            return Optional.empty();
        }
        User user = new User(name);
        users.put(user.getId(), user);
        // of - we just created it, definitely not null
        return Optional.of(user);
    }
}
```

---

# Part 3: Working with Optional

## 3.1 Checking for Values

### Overview

Two approaches to check if value exists:
1. **Imperative** - Check then act (`isPresent()`, `isEmpty()`)
2. **Functional** - Act if present (`ifPresent()`, `ifPresentOrElse()`)

**Recommendation**: Prefer functional approach (more concise, less error-prone)

### Methods

| Method | Return Type | Java Version | Style |
|--------|-------------|--------------|-------|
| `isPresent()` | boolean | 8+ | Imperative |
| `isEmpty()` | boolean | 11+ | Imperative |
| `ifPresent(Consumer)` | void | 8+ | Functional |
| `ifPresentOrElse(Consumer, Runnable)` | void | 9+ | Functional |

---

### isPresent()

**Returns**: `true` if value exists, `false` otherwise

**Signature**: `public boolean isPresent()`

```java
Optional<String> name = Optional.of("Alice");
if (name.isPresent()) {
    System.out.println(name.get());
}
```

⚠️ **Warning**: Using `isPresent() + get()` is an anti-pattern. Use functional methods instead.

---

### isEmpty()

**Returns**: `true` if empty, `false` if value present (opposite of `isPresent()`)

**Signature**: `public boolean isEmpty()`

**Available**: Java 11+

```java
Optional<String> opt = Optional.empty();
if (opt.isEmpty()) {
    System.out.println("No value present");
}

// More readable than
if (!opt.isPresent()) { }
```

---

### ifPresent(Consumer)

**Purpose**: Execute action **only if** value is present (functional approach)

**Signature**: `public void ifPresent(Consumer<? super T> action)`

```java
Optional<String> name = Optional.of("Alice");
name.ifPresent(n -> System.out.println("Hello, " + n));
// Prints: Hello, Alice

// With method reference
name.ifPresent(System.out::println);

// Empty Optional - action not executed
Optional.empty().ifPresent(n -> System.out.println("Won't print"));
```

**Prefer this over**:
```java
// ❌ Imperative
if (opt.isPresent()) {
    System.out.println(opt.get());
}

// ✅ Functional
opt.ifPresent(System.out::println);
```

---

### ifPresentOrElse(Consumer, Runnable)

**Purpose**: Execute one action if present, another if empty

**Signature**: `public void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)`

**Available**: Java 9+

```java
Optional<String> name = Optional.ofNullable(null);
name.ifPresentOrElse(
    value -> System.out.println("Found: " + value),
    () -> System.out.println("Not found")
);
// Prints: Not found

// With present value
Optional.of("Alice").ifPresentOrElse(
    value -> System.out.println("Found: " + value),
    () -> System.out.println("Not found")
);
// Prints: Found: Alice
```

---

## 3.2 Retrieving Values

### Overview

Five methods to extract values with different **safety guarantees**:

| Method | Safety | Evaluation | Best For |
|--------|--------|------------|----------|
| `get()` | ❌ Unsafe | N/A | Almost never use |
| `orElse(T)` | ✅ Safe | Eager | Constants |
| `orElseGet(Supplier)` | ✅ Safe | Lazy | Computed values |
| `orElseThrow()` | ✅ Safe | N/A | Error conditions |
| `or(Supplier)` | ✅ Safe | Lazy | Alternative Optionals |

---

### get()

**Purpose**: Returns value if present, **throws exception** if empty

**Signature**: `public T get()`

**Throws**: `NoSuchElementException` if empty

```java
Optional<String> name = Optional.of("Alice");
String value = name.get();  // "Alice"

// ❌ DANGER - throws exception
Optional<String> empty = Optional.empty();
// String bad = empty.get();  // NoSuchElementException!
```

⚠️ **Critical Rule**: Almost **never use** `get()`. Use safer alternatives.

---

### orElse(T)

**Purpose**: Returns value if present, otherwise returns **default**

**Signature**: `public T orElse(T other)`

**Evaluation**: **Eager** (default always evaluated)

```java
Optional<String> name = Optional.empty();
String result = name.orElse("Unknown");  // "Unknown"

// With present value
Optional<String> present = Optional.of("Alice");
String result2 = present.orElse("Unknown");  // "Alice"
```

⚠️ **Performance Warning**:
```java
// Default is ALWAYS evaluated, even if not needed
String result = opt.orElse(expensiveMethod());  // Always calls expensiveMethod()!
```

**Use for**: Constants and cheap values

---

### orElseGet(Supplier)

**Purpose**: Returns value if present, otherwise **computes** default

**Signature**: `public T orElseGet(Supplier<? extends T> supplier)`

**Evaluation**: **Lazy** (supplier called only if empty)

```java
Optional<String> name = Optional.empty();
String result = name.orElseGet(() -> "Unknown");

// Supplier only called if Optional is empty
String result2 = name.orElseGet(() -> expensiveComputation());
```

**Performance Comparison**:
```java
Optional<String> opt = Optional.of("exists");

// ❌ Inefficient - method always called
String v1 = opt.orElse(expensiveMethod());

// ✅ Efficient - method NOT called
String v2 = opt.orElseGet(() -> expensiveMethod());
```

**Use for**: Expensive operations, object creation, method calls

---

### orElseThrow()

**Purpose**: Returns value if present, otherwise **throws exception**

**Signatures**: 
```java
public T orElseThrow()  // Java 10+ - throws NoSuchElementException
public <X> T orElseThrow(Supplier<X> exceptionSupplier)  // Custom exception
```

```java
// Default exception (Java 10+)
Optional<String> name = Optional.of("Alice");
String value = name.orElseThrow();  // "Alice"

// Custom exception
Optional<User> user = findUser(id);
User actual = user.orElseThrow(() -> 
    new UserNotFoundException("User " + id + " not found")
);
```

**Use for**: When absence should be an error

---

### or(Supplier)

**Purpose**: Returns this Optional if present, otherwise returns **alternative Optional**

**Signature**: `public Optional<T> or(Supplier<? extends Optional<T>> supplier)`

**Available**: Java 9+

```java
Optional<String> primary = Optional.empty();
Optional<String> secondary = Optional.of("Backup");

Optional<String> result = primary.or(() -> secondary);
System.out.println(result.get());  // "Backup"

// Multiple fallbacks
Optional<String> config = getLocalConfig()
    .or(() -> getEnvConfig())
    .or(() -> getDefaultConfig())
    .orElse("hardcoded");
```

**Use for**: Chaining multiple Optional sources

---

## 3.3 Transforming Values

### Overview

Two transformation methods:

| Method | Input Function Returns | Output | Use When |
|--------|----------------------|--------|----------|
| `map()` | Plain value (U) | `Optional<U>` | Simple transformation |
| `flatMap()` | Optional (Optional<U>) | `Optional<U>` | Chaining Optionals |

Both are:
- **Lazy** - Only execute if value present
- **Immutable** - Return new Optional
- **Null-safe** - Handle null returns gracefully

---

### map(Function)

**Purpose**: Transform value **if present**

**Signature**: `public <U> Optional<U> map(Function<? super T, ? extends U> mapper)`

**Behavior**:
- If empty → Returns `Optional.empty`
- If present → Applies function, wraps result in Optional
- If function returns null → Returns `Optional.empty`

```java
// String to Integer
Optional<String> str = Optional.of("123");
Optional<Integer> num = str.map(Integer::parseInt);
System.out.println(num.get());  // 123

// Property extraction
class Person {
    String name;
    String getName() { return name; }
}
Optional<Person> person = Optional.of(new Person("Alice"));
Optional<String> name = person.map(Person::getName);

// Chaining transformations
Optional<String> result = Optional.of("  hello  ")
    .map(String::trim)
    .map(String::toUpperCase);
System.out.println(result.get());  // "HELLO"
```

**Use for**: Type conversion, property extraction, simple transformations

---

### flatMap(Function)

**Purpose**: Transform value **when function returns Optional** (avoids nesting)

**Signature**: `public <U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper)`

**Behavior**:
- If empty → Returns `Optional.empty`
- If present → Applies function (which returns Optional), returns that Optional
- **Flattens** the result

```java
class Person {
    Optional<String> email;
    Person(String email) { 
        this.email = Optional.ofNullable(email); 
    }
    Optional<String> getEmail() { return email; }
}

Optional<Person> person = Optional.of(new Person("test@mail.com"));

// ❌ Using map - creates nested Optional<Optional<String>>
Optional<Optional<String>> nested = person.map(Person::getEmail);

// ✅ Using flatMap - flattens to Optional<String>
Optional<String> email = person.flatMap(Person::getEmail);
System.out.println(email.get());  // "test@mail.com"
```

**Visual Difference**:
```
map(f):     Optional<T> → Optional<U>         (f returns U)
flatMap(f): Optional<T> → Optional<U>         (f returns Optional<U>)
```

**Use for**: Chaining methods that return Optional, avoiding nested Optionals

---

## 3.4 Filtering Values

### filter(Predicate)

**Purpose**: Keep value **only if** it matches condition

**Signature**: `public Optional<T> filter(Predicate<? super T> predicate)`

**Behavior**:
- If empty → Returns `Optional.empty`
- If present and matches → Returns same Optional
- If present but doesn't match → Returns `Optional.empty`

```java
// Basic filtering
Optional<Integer> num = Optional.of(42);
Optional<Integer> even = num.filter(n -> n % 2 == 0);  // Optional[42]
Optional<Integer> odd = num.filter(n -> n % 2 != 0);   // Optional.empty

// Validation
Optional<String> email = Optional.of("test@example.com");
Optional<String> valid = email.filter(e -> e.contains("@"));

// Chaining filters
Optional<String> password = Optional.of("Pass123");
Optional<String> validPass = password
    .filter(p -> p.length() >= 8)
    .filter(p -> p.matches(".*[A-Z].*"))
    .filter(p -> p.matches(".*[0-9].*"));
```

**Combining with map**:
```java
Optional<String> result = Optional.of("  hello  ")
    .map(String::trim)           // Transform
    .filter(s -> s.length() > 3) // Validate
    .map(String::toUpperCase);   // Transform again
```

---

# Part 4: Advanced Usage

## 4.1 Optional with Streams

### Stream Operations Returning Optional

Many Stream terminal operations return Optional:

| Operation | Returns | Example |
|-----------|---------|---------|
| `findFirst()` | `Optional<T>` | First element |
| `findAny()` | `Optional<T>` | Any element |
| `min()` | `Optional<T>` | Minimum value |
| `max()` | `Optional<T>` | Maximum value |
| `reduce()` | `Optional<T>` | Reduced value |

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// findFirst
Optional<String> first = names.stream().findFirst();

// max/min
Optional<Integer> max = Stream.of(5, 3, 8).max(Integer::compareTo);

// reduce
Optional<Integer> sum = Stream.of(1, 2, 3).reduce(Integer::sum);
```

### Optional.stream() (Java 9+)

Converts Optional to Stream (0 or 1 elements)

```java
// Filtering empty Optionals from a stream
List<Optional<String>> optionals = Arrays.asList(
    Optional.of("A"),
    Optional.empty(),
    Optional.of("B")
);

List<String> values = optionals.stream()
    .flatMap(Optional::stream)  // 0 or 1 elements per Optional
    .collect(Collectors.toList());
// Result: [A, B]
```

---

## 4.2 Chaining Operations

### Pattern: Transform → Filter → Default

```java
String result = Optional.ofNullable(input)
    .map(String::trim)           // Transform
    .filter(s -> s.length() > 0) // Validate
    .map(String::toUpperCase)    // Transform
    .orElse("DEFAULT");          // Default
```

### Pattern: Multiple Transformations

```java
class User {
    Address address;
    Address getAddress() { return address; }
}

class Address {
    String city;
    String getCity() { return city; }
}

// Null-safe property access
String city = Optional.ofNullable(user)
    .map(User::getAddress)
    .map(Address::getCity)
    .orElse("Unknown");
```

---

## 4.3 Primitive Optionals

### Why Primitive Optionals?

Avoid boxing/unboxing overhead for primitives.

| Type | Wrapper Optional | Primitive Optional |
|------|-----------------|-------------------|
| int | `Optional<Integer>` | `OptionalInt` |
| long | `Optional<Long>` | `OptionalLong` |
| double | `Optional<Double>` | `OptionalDouble` |

```java
// ❌ Boxing overhead
Optional<Integer> count = Optional.of(5);

// ✅ No boxing
OptionalInt count = OptionalInt.of(5);

// Specialized methods
OptionalInt.of(5).getAsInt();      // Returns int
OptionalDouble.of(3.14).orElse(0.0);
```

---

# Part 5: Best Practices

## 5.1 Do's and Don'ts

### ✅ DO

| Practice | Example |
|----------|---------|
| Use as return type | `public Optional<User> findById(Long id)` |
| Use functional methods | `opt.map(f).orElse(default)` |
| Chain operations | `.map().filter().orElse()` |
| Return `Optional.empty()` | Never return null |
| Use `orElseGet()` for expensive defaults | `opt.orElseGet(() -> compute())` |

### ❌ DON'T

| Practice | Why Not | Alternative |
|----------|---------|-------------|
| Use as parameter | Caller might pass null | Overloading |
| Use as field | Memory overhead | Null field, Optional getter |
| Call `get()` without check | Throws exception | Use `orElse()`/`orElseGet()` |
| Use with collections | Collections have empty state | `Collections.emptyList()` |
| Return null | Defeats purpose | Return `Optional.empty()` |

---

## 5.2 Common Patterns

### Pattern 1: Repository Find

```java
public class UserRepository {
    public Optional<User> findById(Long id) {
        return Optional.ofNullable(database.get(id));
    }
}

// Usage
User user = userRepository.findById(id)
    .orElseThrow(() -> new UserNotFoundException(id));
```

### Pattern 2: Configuration with Defaults

```java
public String getConfig(String key) {
    return configMap.get(key)
        .filter(v -> !v.isEmpty())
        .orElse("default-value");
}
```

### Pattern 3: Null-Safe Navigation

```java
String city = Optional.ofNullable(user)
    .map(User::getAddress)
    .map(Address::getCity)
    .orElse("Unknown");
```

### Pattern 4: Conditional Processing

```java
findUser(id)
    .filter(User::isActive)
    .ifPresent(this::sendEmail);
```

### Pattern 5: Multiple Fallbacks

```java
String value = getLocalValue()
    .or(() -> getRemoteValue())
    .or(() -> getDefaultValue())
    .orElse("hardcoded");
```

---

## 5.3 Anti-Patterns to Avoid

### ❌ Anti-Pattern 1: isPresent() + get()

```java
// WRONG
if (opt.isPresent()) {
    process(opt.get());
}

// RIGHT
opt.ifPresent(this::process);
```

### ❌ Anti-Pattern 2: Optional.of(null)

```java
// WRONG - throws NPE
Optional<String> opt = Optional.of(null);

// RIGHT
Optional<String> opt = Optional.ofNullable(null);
// or
Optional<String> opt = Optional.empty();
```

### ❌ Anti-Pattern 3: Returning null

```java
// WRONG
public Optional<User> findUser(Long id) {
    return null;  // Defeats entire purpose!
}

// RIGHT
public Optional<User> findUser(Long id) {
    return Optional.empty();
}
```

### ❌ Anti-Pattern 4: Optional in Collections

```java
// WRONG
List<Optional<String>> list;

// RIGHT
List<String> list;  // Use null or filter them out
```

### ❌ Anti-Pattern 5: Optional as Field

```java
// WRONG
class User {
    private Optional<String> email;
}

// RIGHT
class User {
    private String email;
    public Optional<String> getEmail() {
        return Optional.ofNullable(email);
    }
}
```

---

# Part 6: Quick Reference

## 6.1 Method Summary

### Creation Methods

| Method | Null Safe? | Use Case |
|--------|-----------|----------|
| `of(value)` | No (throws NPE) | Guaranteed non-null |
| `ofNullable(value)` | Yes | Might be null |
| `empty()` | N/A | Explicit absence |

### Checking Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `isPresent()` | boolean | Has value? |
| `isEmpty()` | boolean | Is empty? (Java 11+) |
| `ifPresent(Consumer)` | void | Execute if present |
| `ifPresentOrElse(Consumer, Runnable)` | void | Execute based on presence (Java 9+) |

### Retrieval Methods

| Method | Throws? | Evaluation | Use For |
|--------|---------|------------|---------|
| `get()` | Yes | N/A | ⚠️ Avoid |
| `orElse(T)` | No | Eager | Constants |
| `orElseGet(Supplier)` | No | Lazy | Computed values |
| `orElseThrow()` | Yes | N/A | Error conditions |
| `or(Supplier)` | No | Lazy | Alternative Optionals (Java 9+) |

### Transformation Methods

| Method | Input Type | Output Type |
|--------|-----------|-------------|
| `map(Function)` | T → U | Optional<U> |
| `flatMap(Function)` | T → Optional<U> | Optional<U> |
| `filter(Predicate)` | T → boolean | Optional<T> |

---

## 6.2 Decision Tree

### Creating Optional
```
Do you have a value?
├─ YES
│  ├─ Is it definitely not null?
│  │  └─ YES → Optional.of(value)
│  └─ Might it be null?
│     └─ YES → Optional.ofNullable(value)
└─ NO → Optional.empty()
```

### Retrieving Value
```
What if Optional is empty?
├─ Use default constant → orElse(default)
├─ Compute default → orElseGet(() -> compute())
├─ Throw exception → orElseThrow(() -> new Exception())
└─ Try another Optional → or(() -> alternative)
```

---

## 6.3 Key Takeaways

### Core Principles

1. **Optional makes absence explicit** in method signatures
2. **Use functional methods** over imperative style
3. **Never call get() without checking** - use orElse/orElseGet/orElseThrow
4. **Never return null** from Optional-returning methods
5. **Use only for return types** - not parameters or fields

### When to Use

✅ Method return types where absence is valid  
✅ Repository/DAO find operations  
✅ Configuration lookups  
✅ Chaining transformations  

❌ Method parameters  
❌ Class fields  
❌ Collections  
❌ Replacing every null  

### Mental Model

Think of Optional as a **box**:
- Contains 0 or 1 item
- Never null itself
- Can transform contents
- Can provide defaults
- Makes absence explicit

---

**Use Optional to write safer, more expressive Java code! 🛡️**
