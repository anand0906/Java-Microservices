# ☕ Java — Flow Control

> **Java Version Coverage:** Java 1.0+ · forEach (1.5v) · String in switch (1.7v)  
> **Topics:** if-else · switch · while · do-while · for · for-each · break · continue · labeled statements

---

## 📑 Index

| # | Topic |
|---|-------|
| 1 | [Introduction](#1-introduction) |
| 2 | [Selection Statements — if-else](#2-selection-statements--if-else) |
| 3 | [Selection Statements — switch](#3-selection-statements--switch) |
| 4 | [Iterative Statements — while](#4-iterative-statements--while) |
| 5 | [Iterative Statements — do-while](#5-iterative-statements--do-while) |
| 6 | [Iterative Statements — for Loop](#6-iterative-statements--for-loop) |
| 7 | [Iterative Statements — for-each (Enhanced for)](#7-iterative-statements--for-each-enhanced-for) |
| 8 | [Transfer Statements — break](#8-transfer-statements--break) |
| 9 | [Transfer Statements — continue](#9-transfer-statements--continue) |
| 10 | [Labeled break and continue](#10-labeled-break-and-continue) |
| 11 | [do-while vs continue — The Dangerous Combination](#11-do-while-vs-continue--the-dangerous-combination) |
| 12 | [Unreachable Statements — if-else vs loops](#12-unreachable-statements--if-else-vs-loops) |

---

## 1. Introduction

**Flow control** describes the order in which all statements will be executed at runtime.

```
                        Flow Control
                            │
          ┌─────────────────┼──────────────────┐
          │                 │                  │
  1. Selection          2. Iterative      3. Transfer
  Statements            Statements        Statements
  ├─ if-else            ├─ while          ├─ break
  └─ switch             ├─ do-while       ├─ continue
                        ├─ for()          ├─ return
                        └─ forEach() 1.5v ├─ try-catch-finally
                                          └─ assert (1.4v)
```

---

## 2. Selection Statements — if-else

### 📌 Syntax

```java
if (b)           // b must be boolean
{
    // action if b is true
} else {
    // action if b is false
}
```

### 📌 Key Rules

**Rule 1 — Argument to `if` must be boolean type.**  
If any other type is provided → CE: incompatible types.

```java
int x = 0;
if (x)                  // ❌ CE: incompatible types — found: int, required: boolean
if (x = 20)             // ❌ CE: incompatible types — found: int, required: boolean
if (x == 20)            // ✅  boolean result
if (b = true)           // ✅  assignment of boolean is itself boolean → Output: Hello
if (b == true)          // ✅  comparison → Output: Hi (if b is false)
```

**Rule 2 — `else` part and curly braces are both optional.**

**Rule 3 — Without curly braces, only ONE statement allowed under `if`.**  
That one statement must NOT be a declarative statement.

```java
if (true)
    System.out.println("hello");  // ✅  Output: Hello

if (true);                        // ✅  empty statement — No output

if (true)
    int x = 10;                   // ❌ CE: '.class' expected — declarative not allowed without {}

if (true) {
    int x = 10;                   // ✅  works fine inside curly braces
}
```

**Rule 4 — Without curly braces, only the immediately next statement is dependent on `if`.**

```java
if (true)
    System.out.println("hello");  // dependent on if
    System.out.println("hi");     // independent — always executes
// Output: Hello  Hi
```

> 💡 Semicolon (`;`) is a valid Java statement called an **empty statement** — it won't produce any output.

> ⚠️ The compiler does **NOT** check unreachability in `if-else` — only in loops.

---

## 3. Selection Statements — switch

### 📌 When to use
If several options are available, it is **not recommended** to use if-else — use `switch` instead. It improves **readability** of the code.

### 📌 Syntax

```java
switch (x) {
    case 1:
        action1;
    case 2:
        action2;
    ...
    default:
        default action;
}
```

### 📌 Allowed Argument Types

| Java Version | Allowed Types |
|:---:|---|
| Up to 1.4 | `byte`, `short`, `char`, `int` |
| From 1.5 | + `Byte`, `Short`, `Character`, `Integer` (wrapper classes) + `enum` |
| From 1.7 | + `String` |

```
switch(x)
    ├── byte   → Byte
    ├── short  → Short
    ├── int    → Integer    +  String (1.7v)
    └── char   → Character
                    +
                  enum (1.5v)
```

### 📌 Key Rules

**Rule 1 — Curly braces are mandatory** (unlike all other statements where they are optional).

**Rule 2 — Both `case` and `default` are optional.**

**Rule 3 — Every statement inside `switch` must be under some `case` or `default`.** Independent statements are NOT allowed.

```java
int x = 10;
switch (x) {
    System.out.println("hello");  // ❌ CE: case, default, or '}' expected
}
```

**Rule 4 — Every case label must be a compile-time constant.**  
Using a variable as a case label → CE: constant expression required.

```java
int x = 10, y = 20;
switch (x) {
    case 10: System.out.println("10");
    case y:  System.out.println("20");  // ❌ CE: constant expression required
}

// If y is declared final → ✅ no error
final int y = 20;
case y:   // ✅ final variable is replaced by its value at compile time
```

> 💡 Switch argument and case label can be **expressions**, but case label must be a **constant expression**.

```java
switch (x + 1) {           // ✅  expression as switch argument
    case 10:               // ✅
    case 10 + 20:          // ✅  constant expression
    case 10 + 20 + 30:     // ✅
}
```

**Rule 5 — Every case label must be within the range of the switch argument type.**

```java
byte b = 10;
switch (b) {
    case 10:   System.out.println("10");   // ✅
    case 100:  System.out.println("100");  // ✅
    case 1000: System.out.println("1000"); // ❌ CE: possible loss of precision — 1000 exceeds byte range
}
```

> ⚠️ When switch argument is `b+1`, the type becomes `int` — so `case 1000` would be allowed for `byte b`.

**Rule 6 — Duplicate case labels are NOT allowed.**

```java
int x = 10;
switch (x) {
    case 97:  System.out.println("97");
    case 99:  System.out.println("99");
    case 'a': System.out.println("a");  // ❌ CE: duplicate case label ('a' = 97)
}
```

---

### 📌 Case Label Summary

```
case label must:
    ├── 1. Be a compile-time constant
    ├── 2. Expression allowed — but must be a constant expression
    ├── 3. Value within range of switch argument type
    └── 4. No duplicate case labels
```

---

### 📌 Fall-Through Inside switch

Within the switch statement, if any case is matched, **from that case onwards ALL statements will be executed** until the end of switch OR a `break` is encountered. This is called **fall-through**.

> 💡 The main advantage of fall-through: we can define **common action for multiple cases**.

```java
int x = 0;
switch (x) {
    case 0: System.out.println("0");
    case 1: System.out.println("1"); break;
    case 2: System.out.println("2");
    default: System.out.println("default");
}
```

| x value | Output |
|:-------:|--------|
| 0 | 0, 1 (falls through to case 1, then break stops) |
| 1 | 1 |
| 2 | 2, default |
| 3 | default |

---

### 📌 default case

- We can write `default` only **once** inside a switch.
- `default` executes **only if no other case matches**.
- `default` can be placed **anywhere** inside switch — but convention is to keep it last.
- When `default` is NOT last: if default is matched, it falls through to subsequent cases too.

```java
int x = 0;
switch (x) {
    default: System.out.println("default");
    case 0:  System.out.println("0"); break;
    case 1:  System.out.println("1");
    case 2:  System.out.println("2");
}
```

| x value | Output |
|:-------:|--------|
| 0 | 0 (case 0 matched, break stops) |
| 1 | 1, 2 (fall-through) |
| 2 | 2 |
| 3 | default, 0 (default falls through to case 0, break stops) |

---

## 4. Iterative Statements — while

### 📌 When to use
Use `while` loop when the **number of iterations is not known in advance**.

```java
while (rs.next())          { ... }  // JDBC ResultSet
while (e.hasMoreElements()){ ... }  // Enumeration
while (itr.hasNext())      { ... }  // Iterator
```

### 📌 Syntax

```java
while (boolean_expression) {
    // body
}
```

### 📌 Key Rules

**Rule 1 — Argument must be boolean type.** Any other type → CE.

```java
while (1)     // ❌ CE: incompatible types — found: int, required: boolean
while (true)  // ✅
```

**Rule 2 — Curly braces are optional.** Without them, only ONE statement is allowed, and it must NOT be a declarative statement.

```java
while (true)
    System.out.println("hello");  // ✅  Output: Hello (infinite times)

while (true);                     // ✅  No output (empty loop)

while (true)
    int x = 10;                   // ❌ CE: '.class' expected

while (true) {
    int x = 10;                   // ✅  No output (infinite loop, nothing printed)
}
```

---

### 📌 Unreachable Statement in while

```java
// Case 1: while(true) → code after is unreachable
while (true) {
    System.out.println("hello");
}
System.out.println("hi");  // ❌ CE: unreachable statement

// Case 2: while(false) → loop body itself is unreachable
while (false) {
    System.out.println("hello");  // ❌ CE: unreachable statement
}

// Case 3: variables (non-final) → JVM evaluates at runtime → no CE
int a = 10, b = 20;
while (a < b) {
    System.out.println("hello");
}
System.out.println("hi");  // ✅  compiles fine → hello (infinite), then hi never reached at runtime

// Case 4: final variables → compiler replaces with values → treats as constant → CE
final int a = 10, b = 20;
while (a < b) {              // compiler sees while(true)
    System.out.println("hello");
}
System.out.println("hi");   // ❌ CE: unreachable statement
```

### 📌 Compiler Rules for Unreachability

| Condition | Compiler Behavior |
|-----------|-------------------|
| `final` variable | Replaced with actual value at compile time |
| Only constants in operation | Compiler evaluates the operation |
| At least one non-final variable | JVM evaluates at runtime — no CE |

---

## 5. Iterative Statements — do-while

### 📌 When to use
Use `do-while` when the **loop body must execute at least once**, regardless of the condition.

### 📌 Syntax

```java
do {
    // body
} while (b);   // ← semicolon is MANDATORY
```

### 📌 Key Rules

**Rule 1 — Curly braces are optional.** Without them, only ONE statement allowed between `do` and `while`, and it must NOT be declarative.

```java
do
    System.out.println("hello");
while (true);           // ✅  Hello (infinite times)

do;
while (true);           // ✅  compiles fine (empty statement)

do
    int x = 10;
while (true);           // ❌ CE: '.class' expected

do {
    int x = 10;
} while (true);         // ✅  compiles fine
```

**Rule 2 — Nesting do-while (tricky case)**

```java
do
    while (true)
        System.out.println("hello");
while (true);           // ✅  Hello (infinite times) — inner while is the single statement of outer do
```

```java
do
while (true);           // ❌ CE: while expected — do has no body statement
```

---

### 📌 Unreachable Statement in do-while

```java
// while(true) → code after is unreachable
do {
    System.out.println("hello");
} while (true);
System.out.println("hi");   // ❌ CE: unreachable statement

// while(false) → body executes once, code after is reachable
do {
    System.out.println("hello");
} while (false);
System.out.println("hi");   // ✅  Output: Hello  Hi

// Non-final variables → compiles fine
int a = 10, b = 20;
do {
    System.out.println("hello");  // Output: Hello (infinite)
} while (a < b);
System.out.println("hi");         // never reached

// Final variables → compiler evaluates → CE if unreachable
final int a = 10, b = 20;
do {
    System.out.println("hello");
} while (a < b);                  // compiler sees while(true)
System.out.println("hi");         // ❌ CE: unreachable statement

final int a = 10, b = 20;
do {
    System.out.println("hello");
} while (a > b);                  // compiler sees while(false) → executes once
System.out.println("hi");         // ✅  Output: Hello  Hi
```

---

## 6. Iterative Statements — for Loop

### 📌 When to use
The **most commonly used loop**. Best suitable when the **number of iterations is known in advance**.

### 📌 Syntax and Execution Order

```
for (①initialization ; ②⑤⑧ condition ; ④⑦ increment/decrement)
{
    body ③⑥⑨
}
```

**Execution order:** `①` init → `②` check → `③` body → `④` update → `⑤` check → `⑥` body → `⑦` update → ...

### 📌 Curly Braces
Optional. Without them, only ONE statement is allowed, and it must NOT be declarative.

---

### 📌 Initialization Section

- Executed **only once** at the beginning.
- Used to declare and initialize loop variables.
- Can declare **multiple variables** — but all must be of the **same type**.
- Can also hold any valid Java statement (including `System.out.println`).

```java
int i = 0, j = 0;          // ✅  same type
int i = 0; boolean b = true; // ❌ invalid — different types
int i = 0, int j = 0;      // ❌ invalid — cannot repeat type keyword

// s.o.p in initialization section — executes only once
for (System.out.println("hello"); i < 3; i++) {
    System.out.println("no boss");
}
// Output: hello  no boss  no boss  no boss
```

---

### 📌 Conditional Check Section

- Can be any Java expression — but must be of **boolean type**.
- **Optional** — if omitted, compiler inserts `true`.

---

### 📌 Increment / Decrement Section

- Can hold any valid Java statement (including `System.out.println`).

```java
for (System.out.println("hello"); i < 3; System.out.println("hi")) {
    i++;
}
// Output: hello  hi  hi  hi
```

---

### 📌 All 3 Sections are Optional

```java
for (;;) {
    System.out.println("hello");  // Output: Hello (infinite times)
}
```

---

### 📌 Unreachable Statement in for Loop

```java
// Condition true → after-loop code unreachable
for (int i = 0; true; i++) {
    System.out.println("hello");
}
System.out.println("hi");   // ❌ CE: unreachable statement

// Condition false → loop body unreachable
for (int i = 0; false; i++) {
    System.out.println("hello");  // ❌ CE: unreachable statement
}
System.out.println("hi");   // ✅

// No condition → same as true → after-loop code unreachable
for (int i = 0;; i++) {
    System.out.println("hello");
}
System.out.println("hi");   // ❌ CE: unreachable statement

// Non-final variable → JVM evaluates → no CE
int a = 10, b = 20;
for (int i = 0; a < b; i++) {
    System.out.println("hello");
}
System.out.println("hi");   // ✅  compiles, hello prints infinite, hi never reached

// Final variable → compiler evaluates → CE
final int a = 10, b = 20;
for (int i = 0; a < b; i++) {
    System.out.println("hello");
}
System.out.println("hi");   // ❌ CE: unreachable statement
```

---

## 7. Iterative Statements — for-each (Enhanced for)

### 📌 Introduction
- Introduced in **Java 1.5**.
- Best suitable to **retrieve elements of arrays and collections**.
- NOT a general purpose loop.

### 📌 Syntax

```java
for (each_item : target) {   // target must be Iterable (array or Collection)
    // body
}
```

### 📌 Example — 1D Array

```java
int[] a = {10, 20, 30, 40, 50};

// Normal for loop
for (int i = 0; i < a.length; i++) {
    System.out.println(a[i]);
}

// Enhanced for loop
for (int x : a) {
    System.out.println(x);
}
// Output: 10  20  30  40  50
```

### 📌 Example — 2D Array

```java
int[][] a = {{10, 20, 30}, {40, 50}};

// Normal for loop
for (int i = 0; i < a.length; i++) {
    for (int j = 0; j < a[i].length; j++) {
        System.out.println(a[i][j]);
    }
}

// Enhanced for loop
for (int[] x : a) {
    for (int y : x) {
        System.out.println(y);
    }
}
```

### 📌 Limitations of for-each

- We **cannot write equivalent for-each** for a loop that prints elements from right to left.
- Normal `for` loop can print elements **left to right OR right to left**.
- `for-each` always prints elements **only left to right**.
- It is **NOT a general purpose loop** — only for element retrieval.

```java
// Cannot convert this to for-each:
for (int i = 0; i < 10; i++) {
    System.out.println("hello");
}
```

---

### 📌 Iterable vs Iterator

The target element in for-each must be an **Iterable** object — i.e., its class must implement `java.lang.Iterable`.

| Feature | `Iterable` | `Iterator` |
|---------|-----------|-----------|
| Related to | `forEach` loop | Collection |
| Purpose | Target of forEach loop | Get objects one by one from collection |
| Package | `java.lang` | `java.util` |
| Methods | `iterator()` — 1 method | `hasNext()`, `next()`, `remove()` — 3 methods |
| Introduced | Java 1.5 | Java 1.2 |

> 💡 Every **array class** and **Collection interface** already implements `Iterable`.

---

## 8. Transfer Statements — break

### 📌 Valid places to use `break`

**1. Inside `switch`** — to stop fall-through

```java
switch (x) {
    case 0: System.out.println("hello"); break;  // stops fall-through
    case 1: System.out.println("hi");
}
// Output: hello (only case 0 runs)
```

**2. Inside loops** — to break out of the loop based on a condition

```java
for (int i = 0; i < 10; i++) {
    if (i == 5) break;
    System.out.println(i);
}
// Output: 0  1  2  3  4
```

**3. Inside labeled blocks** — to exit the block based on a condition

```java
int x = 10;
l1: {
    System.out.println("begin");
    if (x == 10) break l1;
    System.out.println("end");   // skipped
}
System.out.println("hello");
// Output: begin  hello
```

### ⚠️ `break` outside these 3 places → CE

```java
int x = 10;
if (x == 10)
    break;   // ❌ CE: break outside switch or loop
```

---

## 9. Transfer Statements — continue

### 📌 Definition
Used to **skip the current iteration** and continue with the next iteration of the loop.

```java
for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) continue;
    System.out.println(i);
}
// Output: 1  3  5  7  9
```

### ⚠️ `continue` is valid ONLY inside loops

Using `continue` anywhere else → CE: continue outside of loop.

```java
int x = 10;
if (x == 10);
continue;              // ❌ CE: continue outside of loop
System.out.println("hello");
```

---

## 10. Labeled break and continue

### 📌 When to use
In **nested loops**, to break or continue a **specific (outer) loop** — not just the innermost one — use labeled break/continue.

### 📌 Syntax

```java
l1:
for (;;) {
    l2:
    for (;;) {
        l3:
        for (;;) {
            break l1;    // breaks outermost loop
            break l2;    // breaks middle loop
            break l3;    // breaks innermost loop
        }
    }
}
```

### 📌 Example

```java
l1:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (i == j) break;         // or: break l1 / continue / continue l1
        System.out.println(i + "........." + j);
    }
}
```

| Statement | Output |
|-----------|--------|
| `break` | 1.........0 &nbsp; 2.........0 &nbsp; 2.........1 |
| `break l1` | No output (breaks outer loop immediately) |
| `continue` | 0.........1 &nbsp; 0.........2 &nbsp; 1.........0 &nbsp; 1.........2 &nbsp; 2.........0 &nbsp; 2.........1 |
| `continue l1` | 1.........0 &nbsp; 2.........0 &nbsp; 2.........1 |

---

## 11. do-while vs continue — The Dangerous Combination

### 📌 Why dangerous?
In a `do-while` loop, `continue` skips to the **while condition check** — NOT to the top of the loop body. This means any statements **between `continue` and the `while` condition** are skipped.

```java
int x = 0;
do {
    ++x;
    System.out.println(x);    // prints x
    if (++x < 5)
        continue;             // jumps to while(++x < 10) — skips next two lines
    ++x;
    System.out.println(x);    // skipped when continue triggers
} while (++x < 10);
```

**Trace:**

| Iteration | x before | Actions | Output |
|:---------:|:--------:|---------|--------|
| 1 | 0 | ++x→1, print 1, ++x→2, 2<5→continue, while: ++x→3, 3<10→loop | 1 |
| 2 | 3 | ++x→4, print 4, ++x→5, 5<5 false→no continue, ++x→6, print 6, while: ++x→7, 7<10→loop | 4, 6 (but only 4 printed before continue check) |

> **Output: 1  4  6  8  10**

---

## 12. Unreachable Statements — if-else vs loops

### 📌 Key Difference
The **compiler checks unreachability only in loops** — NOT in `if-else`.

```java
// ✅ while(true) → CE: unreachable statement after loop
while (true) {
    System.out.println("hello");
}
System.out.println("hi");   // ❌ CE: unreachable statement

// ✅ if(true) → NO CE — compiler doesn't check unreachability in if-else
if (true) {
    System.out.println("hello");
} else {
    System.out.println("hi");   // NOT flagged as unreachable — Output: Hello
}
```

---

## 📝 Quick Recap

| Topic | Key Rule |
|-------|----------|
| `if` argument | Must be **boolean** — no other type |
| Without `{}` in if/while/for | Only **one statement** allowed — NOT declarative |
| switch curly braces | **Mandatory** (unlike other statements) |
| switch argument types | byte/short/char/int (1.4v) + wrappers+enum (1.5v) + String (1.7v) |
| case label | Must be **compile-time constant**, within range, no duplicates |
| Fall-through | Executes all cases from matched case until `break` or end |
| `default` | Optional, anywhere in switch, executes only if no case matches |
| while | Use when **iterations not known in advance** |
| do-while | Use when **loop body must execute at least once** |
| for | Use when **iterations known in advance** — all 3 parts optional |
| for-each | Introduced 1.5v — arrays/collections only — left-to-right only |
| Iterable | `java.lang`, 1 method `iterator()`, target of for-each |
| Iterator | `java.util`, 3 methods `hasNext()` `next()` `remove()` |
| `break` | Valid in: switch, loops, labeled blocks only |
| `continue` | Valid in: loops only |
| Labeled break/continue | Used to control **specific loop** in nested loops |
| do-while + continue | `continue` jumps to **while condition**, skips rest of loop body |
| Unreachability | Compiler checks only in **loops** — NOT in if-else |
| final variables | Replaced with values at compile time — may cause unreachable CE |
| Non-final variables | JVM evaluates at runtime — compiler never flags as unreachable |
