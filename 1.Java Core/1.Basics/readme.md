# ☕ Core Java — Complete Revision Notes
### Identifiers → Reserved Words → Data Types → Literals → Variables → Arrays → Var-Args → Main Method → JVM → Coding Standards → Operators → Flow Control → Source File Structure → Import → Package → Modifiers → Interfaces

> **Java Versions Covered:** Java 1.0 → Java 21+  
> OCJP / SCJP — All Chapters in One Place

---

## 📑 Master Index

| # | Topic |
|---|-------|
| 1 | [Identifiers](#1-identifiers) |
| 2 | [Reserved Words](#2-reserved-words) |
| 3 | [Data Types](#3-data-types) |
| 4 | [Literals](#4-literals) |
| 5 | [Types of Variables](#5-types-of-variables) |
| 6 | [Arrays](#6-arrays) |
| 7 | [Var-Arg Methods](#7-var-arg-methods-15v) |
| 8 | [main() Method](#8-main-method) |
| 9 | [Java Coding Standards](#9-java-coding-standards) |
| 10 | [JVM Memory Areas](#10-jvm-memory-areas) |
| 11 | [Operators & Assignments](#11-operators--assignments) |
| 12 | [Flow Control](#12-flow-control) |
| 13 | [Java Source File Structure](#13-java-source-file-structure) |
| 14 | [Import Statement](#14-import-statement) |
| 15 | [Static Import](#15-static-import-15v) |
| 16 | [Package Statement](#16-package-statement) |
| 17 | [Modifiers](#17-modifiers) |
| 18 | [Interfaces](#18-interfaces) |

---

## 1. Identifiers

> Any **name** in a Java program — class name, method name, variable name, or label — is called an **Identifier**.

### Rules

| Rule | Valid Example | Invalid Example |
|------|--------------|-----------------|
| Allowed chars: `a-z`, `A-Z`, `0-9`, `_`, `$` | `total_number`, `$price` | `Total#` ❌ |
| Cannot start with a digit | `ABC123` | `123ABC` ❌ |
| Case sensitive | `number` ≠ `Number` ≠ `NUMBER` | — |
| Cannot use reserved keywords | `count` | `int if = 10` ❌ |
| No length limit (≤15 chars recommended) | `i` | — |

```java
class Test {
    int number  = 10;
    int Number  = 20;   // all 4 are different variables ✅
    int NUMBER  = 30;
    int NuMbEr  = 40;
}

int if  = 10;   // ❌ CE: illegal start of expression
int for = 5;    // ❌ CE: illegal start of expression

int String   = 10;  // ✅ compiles (class name used as identifier — bad practice)
int Runnable = 20;  // ✅ compiles (interface name used as identifier — bad practice)
```

---

## 2. Reserved Words

> Words with **pre-defined meaning** in Java. Cannot be used as identifiers.  
> Total: **53 reserved words**

```
Reserved Words (53)
├── Keywords (50)
│   ├── Data Types      → byte short int long float double char boolean
│   ├── Flow Control    → if else switch case default for do while
│   │                     break continue return
│   ├── Modifiers       → public private protected static final abstract
│   │                     synchronized native strictfp transient volatile
│   ├── Exception       → try catch finally throw throws assert
│   ├── Class-related   → class package import extends implements interface
│   ├── Object-related  → new instanceof super this
│   ├── Return type     → void
│   └── Unused          → goto const  ⚠️ (reserved but cannot be used)
│
└── Reserved Literals (3)
    → true   false   null
```

### Version-wise New Keywords

| Keyword | Version | Purpose |
|---------|---------|---------|
| `strictfp` | 1.2 | Strict floating-point |
| `assert` | 1.4 | Testing/debugging |
| `enum` | 1.5 | Named constants |

### ⚠️ Common Traps

| Statement | Valid? | Reason |
|-----------|:------:|--------|
| `final`, `finally`, `finalize` | ❌ | `finalize` is a method, not a keyword |
| `throw`, `throws`, `thrown` | ❌ | `thrown` doesn't exist |
| `break`, `continue`, `return`, `exit` | ❌ | `exit` is not a keyword |
| `byte`, `short`, `Integer`, `long` | ❌ | `Integer` is a class, not a keyword |
| `extends`, `implements`, `imports` | ❌ | `imports` doesn't exist |
| `new`, `delete` | ❌ | `delete` doesn't exist (GC handles cleanup) |
| `public`, `static`, `void` | ✅ | All valid keywords |
| `main`, `String`, `args` | ❌ | Not keywords |

> 💡 All Java keywords are **lowercase only** — `instanceof` not `instanceOf`, `strictfp` not `strictFp`

---

## 3. Data Types

> Java is **strongly typed** — every variable must have a declared type, checked at compile time.

```
Data Types
├── Primitive (8)
│   ├── Numeric
│   │   ├── Integral   → byte · short · int · long
│   │   └── Decimal    → float · double
│   ├── Character      → char
│   └── Boolean        → boolean
│
└── Non-Primitive (Object References)    default value = null
```

> ⚠️ Java is **not pure OOP** — it has primitive types and lacks operator overloading & multiple inheritance.

### Primitive Data Types

| Type | Size | Range | Wrapper | Default |
|------|------|-------|---------|---------|
| `byte` | 1 byte | -2⁷ to 2⁷-1 (-128 to 127) | `Byte` | `0` |
| `short` | 2 bytes | -2¹⁵ to 2¹⁵-1 (-32768 to 32767) | `Short` | `0` |
| `int` | 4 bytes | -2³¹ to 2³¹-1 | `Integer` | `0` |
| `long` | 8 bytes | -2⁶³ to 2⁶³-1 | `Long` | `0` |
| `float` | 4 bytes | -3.4e38 to 3.4e38 | `Float` | `0.0` |
| `double` | 8 bytes | -1.7e308 to 1.7e308 | `Double` | `0.0` |
| `char` | 2 bytes | 0 to 2¹⁶-1 (0 to 65535, unsigned) | `Character` | `0` (blank) |
| `boolean` | JVM-dependent | `true` / `false` only | `Boolean` | `false` |

> `boolean` and `char` are the only **unsigned** types.

### Widening Order (automatic, no data loss)

```
byte → short → int → long → float → double
               ↑
             char
```

### Key Type Rules

```java
// byte
byte b = 127;      // ✅ max
byte b = 130;      // ❌ CE: possible loss of precision
byte b = 10.5;     // ❌ CE: decimal not allowed

// long
long l = 10L;          // ✅ explicit long literal
int  x = 10L;          // ❌ CE: possible loss of precision

// float vs double
double d = 123.456;    // ✅ default is double
float  f = 123.456f;   // ✅ explicit float (suffix f required)
float  f = 123.456;    // ❌ CE: possible loss of precision

// boolean
boolean b = true;   // ✅
boolean b = 0;      // ❌ CE: incompatible types (unlike C/C++)
boolean b = 1;      // ❌ CE: incompatible types

// char (2 bytes, Unicode)
char c = 'A';        // ✅
char c = 97;         // ✅ → 'a'
char c = '\\u0061';  // ✅ → 'a'
char c = 65536;      // ❌ CE: possible loss of precision
```

---

## 4. Literals

> A **literal** is any constant value assigned to a variable. E.g., `int x = 10` → `10` is a literal.

### Integral Literals

| Form | Prefix | Digits | Example | Output |
|------|--------|--------|---------|--------|
| Decimal | none | 0–9 | `int x = 10` | 10 |
| Octal | `0` | 0–7 | `int x = 010` | 8 |
| Hexadecimal | `0x`/`0X` | 0–9, A–F | `int x = 0x10` | 16 |
| Binary (1.7v) | `0b`/`0B` | 0–1 | `int x = 0b111` | 7 |

```java
int x = 0777;     // ✅ valid octal
int x = 0786;     // ❌ CE: 8,9 not valid in octal
int x = 0xFACE;   // ✅ valid hex
int x = 0xBeer;   // ❌ CE: 'r' not a hex digit

long l = 10L;     // ✅
int  x = 10L;     // ❌ CE: possible loss of precision

byte b  = 127;    // ✅
byte b  = 128;    // ❌ CE: possible loss of precision
```

### Floating-Point Literals

```java
double d = 123.456;    // ✅ default double
double d = 123.456D;   // ✅ explicit double
float  f = 123.456f;   // ✅ explicit float
float  f = 10e2F;      // ✅ = 1000.0

float  f = 123.456;    // ❌ CE: possible loss of precision (missing f)
double d = 0x123.456;  // ❌ CE: malformed floating-point literal

double d = 10e2;
System.out.println(d); // 1000.0  (10 × 10²)

int x = 10.0;          // ❌ CE: possible loss of precision
```

### Char Literals (3 ways)

```java
char c = 'a';        // ✅ single char in single quotes
char c = 97;         // ✅ integer (Unicode decimal 0–65535)
char c = '\\u0061';  // ✅ Unicode escape \\uXXXX

char c = a;          // ❌ CE: cannot find symbol
char c = "a";        // ❌ CE: incompatible types
char c = '\\l';      // ❌ CE: illegal escape character
```

### Escape Characters

| Escape | Meaning | Escape | Meaning |
|--------|---------|--------|---------|
| `\n` | New line | `\'` | Single quote |
| `\t` | Horizontal tab | `\"` | Double quote |
| `\r` | Carriage return | `\\` | Backslash |
| `\f` | Form feed | `\b` | Backspace |

### Java 7+ Literal Enhancements

```java
// Binary literals
int x = 0b1010;    // ✅ = 10
int y = 0B111;     // ✅ = 7

// Underscore in numeric literals (removed at compile time)
double d = 1_23_456.7_8_9;  // ✅ → 123456.789
int    n = 1_000_000;        // ✅ → 1000000

double d = _123.456;   // ❌ can't start with _
double d = 123.456_;   // ❌ can't end with _
double d = 123_.456;   // ❌ can't be adjacent to decimal point
```

### Literals Summary

| Literal | Default Type | Example | Suffix to change |
|---------|-------------|---------|-----------------|
| `10` | `int` | `int x = 10` | `10L` → `long` |
| `10.5` | `double` | `double d = 10.5` | `10.5f` → `float` |
| `'a'` | `char` | `char c = 'a'` | — |
| `true`/`false` | `boolean` | `boolean b = true` | — |
| `"text"` | `String` | `String s = "hi"` | — |
| `0b101` (1.7v) | `int` | `int x = 0b101` | `0b101L` → `long` |

---

## 5. Types of Variables

```
Variables
├── Instance Variables   → one copy per object
├── Static Variables     → one copy per class
└── Local Variables      → one copy per method call / thread
```

### Instance Variables

- Declared **inside class, outside** any method/block/constructor
- Stored in the **Heap** (part of the object)
- Created at object creation, destroyed at object destruction
- JVM provides **default values** ✅
- Cannot access directly from static context ❌

```java
class Test {
    int i = 10;

    public static void main(String[] args) {
        // System.out.println(i);  // ❌ CE: non-static variable
        Test t = new Test();
        System.out.println(t.i);   // ✅ 10
    }

    public void methodOne() {
        System.out.println(i);     // ✅ 10 — direct access in instance area
    }
}
```

### Static Variables

- Declared **inside class with `static`** keyword, outside any method/block
- Stored in the **Method Area**
- Created at class loading, destroyed at class unloading — one copy per class
- JVM provides **default values** ✅
- Recommended access via class name

```java
class Test {
    static int i = 10;

    public static void main(String[] args) {
        System.out.println(Test.i); // ✅ recommended (class name)
        System.out.println(i);      // ✅ direct (same class)
    }
}
```

```java
// Instance vs Static — key difference
class Test {
    int x = 10;        // per object
    static int y = 20; // shared

    public static void main(String[] args) {
        Test t1 = new Test();
        t1.x = 888;    // changes only t1's copy
        t1.y = 999;    // changes shared copy

        Test t2 = new Test();
        System.out.println(t2.x + "----" + t2.y);
        // Output: 10----999
    }
}
```

### Local Variables

- Declared **inside** a method, block, or constructor
- Stored in the **Stack**
- JVM does **NOT** provide default values → must initialize before use ❌
- Only modifier allowed: **`final`**

```java
public static void main(String[] args) {
    int x;
    if (args.length > 0) { x = 10; }
    System.out.println(x);   // ❌ CE: variable x might not have been initialized

    // ✅ Safe version:
    int x;
    if (args.length > 0) { x = 10; }
    else { x = 20; }
    System.out.println(x);   // ✅
}

// ❌ Invalid modifiers on local variables:
public int x = 10;    // ❌ CE
private int x = 10;   // ❌ CE
static  int x = 10;   // ❌ CE
final   int x = 10;   // ✅ only valid modifier
```

### Variable Comparison Table

| Feature | Instance | Static | Local |
|---------|----------|--------|-------|
| Where declared | Inside class, outside method | Inside class with `static` | Inside method/block |
| Stored in | Heap | Method Area | Stack |
| Created | Object creation | Class loading | Block execution |
| Destroyed | Object destruction | Class unloading | Block completion |
| JVM default value | ✅ Yes | ✅ Yes | ❌ No |
| One copy per | Object | Class | Thread |
| Thread safe? | ❌ No | ❌ No | ✅ Yes |
| Allowed modifiers | All | All | `final` only |

---

## 6. Arrays

> An array is an **indexed collection** of a **fixed number** of **homogeneous** data elements.

| ✅ Advantage | ❌ Disadvantage |
|------------|----------------|
| Multiple values with one name | Fixed size — can't grow/shrink |
| Better readability | Must know size in advance |

### Declaration

```java
// 1D — 3 valid forms
int[] a;   // ✅ recommended
int []a;   // ✅
int a[];   // ✅

int[5] a;  // ❌ CE: never specify size at declaration

// 2D — 6 valid forms
int[][] a;  int [][]a;  int a[][];
int[] []a;  int[] a[];  int []a[];

// 3D — 10 valid forms
int[][][] a;   int [][][]a;  int a[][][];
int[] [][]a;   int[] a[][];  int[] []a[];
int[][] []a;   int[][] a[];  int []a[][];   int [][]a[];
```

**Tricky mixed declarations:**
```java
int[] a1, b1;    // ✅ a1 = 1D, b1 = 1D
int[] a2[], b2;  // ✅ a2 = 2D, b2 = 1D
int[] []a3, b3;  // ✅ a3 = 2D, b3 = 2D
int[] a, []b;    // ❌ CE
```

### Construction

```java
int[] a = new int[3];   // ✅ size mandatory
int[] a = new int[];    // ❌ CE: array dimension missing
int[] a = new int[0];   // ✅ size zero is valid
int[] a = new int[-3];  // ❌ RE: NegativeArraySizeException

// Allowed size types: byte, short, char, int ONLY
int[] a = new int['a'];  // ✅ char → 97
int[] a = new int[10l];  // ❌ CE: long not allowed
int[] a = new int[10.5]; // ❌ CE: float not allowed

// Max size: Integer.MAX_VALUE
int[] a = new int[2147483647]; // ✅ (may throw OutOfMemoryError)
int[] a = new int[2147483648]; // ❌ CE: integer number too large
```

**Jagged arrays (arrays of arrays):**
```java
int[][] a = new int[2][];
a[0] = new int[3];   // row 0 → 3 elements
a[1] = new int[2];   // row 1 → 2 elements

int[][] a = new int[][4]; // ❌ CE: must specify from left
```

### Initialization — Default Values

| Type | Default |
|------|---------|
| `int`, `byte`, `short`, `long` | `0` |
| `float`, `double` | `0.0` |
| `boolean` | `false` |
| `char` | `0` (blank) |
| Object reference | `null` |

```java
int[] a = new int[4];
a[0] = 10; a[3] = 40;  // ✅
a[4]  = 50;  // ❌ RE: ArrayIndexOutOfBoundsException: 4
a[-4] = 60;  // ❌ RE: ArrayIndexOutOfBoundsException: -4

// 2D — partially constructed
int[][] a = new int[2][];
System.out.println(a[0]);    // null (not constructed)
System.out.println(a[0][0]); // ❌ RE: NullPointerException
```

### Declaration + Construction + Initialization in One Line

```java
char[]   ch = {'a', 'e', 'i', 'o', 'u'};
String[] s  = {"Java", "Python", "C++"};
int[][]  a  = {{10, 20, 30}, {40, 50}};    // jagged 2D

int[][][] a = {{{10,20,30},{40,50}},{{60},{70,80},{90,100,110}}};
System.out.println(a[0][1][1]); // 50
System.out.println(a[1][2][1]); // 100
```

### length vs length()

| | `length` | `length()` |
|---|---------|-----------|
| Type | `final` variable | `final` method |
| Applies to | Arrays only | String only |

```java
int[] x = new int[3];
System.out.println(x.length);    // ✅ 3
System.out.println(x.length());  // ❌ CE

String s = "Java";
System.out.println(s.length());  // ✅ 4
System.out.println(s.length);    // ❌ CE

// Multi-dimensional:
int[][] a = new int[6][3];
System.out.println(a.length);    // 6  (rows only)
System.out.println(a[0].length); // 3  (cols in row 0)
```

### Anonymous Arrays

> Array **without a name** — for one-time use.

```java
new int[]{10, 20, 30, 40};        // ✅ 1D anonymous
new int[][]{{10,20},{30,40}};     // ✅ 2D anonymous
new int[3]{10, 20, 30};           // ❌ CE: no size in anonymous array

// Best use: pass directly to a method
System.out.println(sum(new int[]{10, 20, 30, 40})); // 100
```

### Array Element Assignments

```java
// Primitive arrays — widening allowed
int[] a = new int[5];
a[1] = 'a';    // ✅ char → int
a[4] = 10L;    // ❌ CE: long → int not allowed

// Object arrays — child class objects allowed
Object[] a = new Object[3];
a[0] = new Integer(10);    // ✅
a[1] = new String("Java"); // ✅

Number[] n = new Number[3];
n[0] = new Integer(10);    // ✅
n[2] = new String("Java"); // ❌ CE: incompatible types

// Interface arrays
Runnable[] r = new Runnable[2];
r[0] = new Thread();           // ✅ Thread implements Runnable
r[1] = new String("Java");     // ❌ CE
```

### Array Variable Assignments

```java
// Element widening ≠ array widening
int[]  a  = {10, 20, 30};
char[] ch = {'a', 'b', 'c'};
int[] b = a;    // ✅ same type
int[] c = ch;   // ❌ CE: char[] cannot be assigned to int[]

// Size doesn't matter for reassignment
int[] a = {10, 20, 30, 40, 50};
int[] b = {80, 90};
a = b;  // ✅ reference changes; size irrelevant

// Dimensions must match
int[][] a = new int[3][];
a[0] = new int[4];       // ✅ 1D → 1D slot
a[0] = new int[4][5];    // ❌ CE: 2D into 1D slot

// Object array: child → parent allowed
String[] s = {"A", "B"};
Object[] o = s;           // ✅
```

**Assignment Rules Summary:**

| What must match | Required? |
|----------------|:---------:|
| Type | ✅ Yes |
| Dimensions | ✅ Yes |
| Size | ❌ No |

---

## 7. Var-Arg Methods (1.5v)

> Allows a method to accept **any number of arguments** of the same type.

| Before Java 1.5 | From Java 1.5 |
|----------------|--------------|
| Can't declare variable-arg methods | `int... x` syntax |
| Needed new method for each arg count | One method handles all |

### Syntax & Usage

```java
public static void methodOne(int... x) { }   // ✅ declaration

methodOne();             // 0 args
methodOne(10);           // 1 arg
methodOne(10, 20, 30);   // multiple args
```

**Valid / Invalid syntax:**
```java
methodOne(int... x)    // ✅
methodOne(int ...x)    // ✅
methodOne(int...x)     // ✅
methodOne(int x...)    // ❌ CE: dots must come after type
```

### Internally — 1D Array

> Var-arg is implemented internally as a **1D array** — use index or for-each inside.

```java
public static void sum(int... x) {
    int total = 0;
    for (int x1 : x) total += x1;
    System.out.println("The sum: " + total);
}

sum();                // The sum: 0
sum(10);              // The sum: 10
sum(10, 20, 30, 40);  // The sum: 100
```

### Rules

```java
// Var-arg MUST be the last parameter
methodOne(int a, int... b)      // ✅ general first, var-arg last
methodOne(int... a, int b)      // ❌ CE: var-arg must be last

// Only ONE var-arg per method
methodOne(int... a, int... b)   // ❌ CE: cannot have more than one var-arg

// Can pass array as var-arg argument
methodOne(new int[]{10, 20, 30}); // ✅

// Cannot declare both int[] and int... — ambiguous
public void methodOne(int[] i) { }
public void methodOne(int... i) { }   // ❌ CE: duplicate method
```

### Priority — General > Var-Arg

```java
public static void methodOne(int i) { System.out.println("general"); }
public static void methodOne(int... i) { System.out.println("var-arg"); }

methodOne();        // var-arg   (no general match)
methodOne(10, 20);  // var-arg   (no general match)
methodOne(10);      // general   (exact match → general wins)
```

### Array vs Var-Arg

```java
// Array CAN be replaced with var-arg ✅
public static void main(String[] args) { }
public static void main(String... args) { }  // ✅ valid

// Var-arg var type becomes 2D when it's an array type
// methodOne(int... x)   → x is int[]
// methodOne(int[]... x) → x is int[][]
```

### Summary

| Rule | Detail |
|------|--------|
| Syntax | `type... varName` — dots after type |
| Position | Must be **last** parameter |
| Count | Only **one** var-arg per method |
| Internally | Treated as a **1D array** |
| Priority | **Lowest** — general method wins on exact match |
| Replace array | Array → var-arg ✅ · Var-arg → array ❌ |

---

## 8. main() Method

> Compiler does **not** check for `main()` — JVM checks at **runtime**.  
> No valid `main()` → RE: `NoSuchMethodError: main`

### Required Prototype

```java
public static void main(String[] args)
```

| Part | Reason |
|------|--------|
| `public` | JVM calls from outside |
| `static` | JVM calls without creating an object |
| `void` | JVM expects no return value |
| `String[] args` | Receives command-line arguments |

### Valid Variations

```java
static public void main(String[] args) {}          // ✅ modifier order flexible
public static void main(String args[]) {}          // ✅ array style flexible
public static void main(String... args) {}         // ✅ var-arg allowed
static final synchronized strictfp public void main(String... args) {} // ✅ extra modifiers allowed
```

```java
public static void main(String args) {}   // ❌ not array/var-arg — RE at runtime
public static void Main(String[] args) {} // ❌ uppercase M — RE at runtime
public static int  main(String[] args) {} // ❌ return type not void — RE at runtime
public void main(String[] args) {}        // ❌ missing static — RE at runtime
```

> ⚠️ None of the invalid main() declarations cause a **Compile Error** — they all cause `NoSuchMethodError` at **runtime**.

### Overloading main()

```java
class Test {
    public static void main(String[] args) {
        System.out.println("String[] main");   // JVM calls this ✅
    }
    public static void main(int[] args) {
        System.out.println("int[] main");       // must be called explicitly
    }
}
// Output: String[] main
```

### Inheritance & main()

```java
class Parent {
    public static void main(String[] args) { System.out.println("parent main"); }
}
class Child extends Parent { }
// java Child → parent main  (inherits main from Parent)

// Child with its own main() — METHOD HIDING (not overriding)
class Child extends Parent {
    public static void main(String[] args) { System.out.println("child main"); }
}
// java Parent → parent main
// java Child  → child main
```

### Java 1.7 Enhancements

| | Java ≤ 1.6 | Java 1.7+ |
|-|-----------|-----------|
| No main() | RE: NoSuchMethodError: main | Error: main method not found in class Test, please define as `public static void main(String[] args)` |
| Static block (no main) | Block executes → then error | Error directly — block does NOT execute |

```java
// Both versions same when main() exists:
class Test {
    static { System.out.println("static block"); }
    public static void main(String[] args) { System.out.println("main method"); }
}
// Output: static block → main method
```

### Command Line Arguments

```java
class Test {
    public static void main(String[] args) {
        for (int i = 0; i < args.length; i++) {  // ✅ use < not <=
            System.out.println(args[i]);
        }
    }
}
// java Test x y z  → x, y, z
// java Test        → nothing (args.length = 0)
```

```java
// All args are Strings — + is concatenation, not addition
System.out.println(args[0] + args[1]);
// java Test 10 20  → 1020  (not 30!)

// Argument with space → use double quotes
// java Test "Sai Charan"  → args[0] = "Sai Charan"
```

---

## 9. Java Coding Standards

| Element | Rule | Example |
|---------|------|---------|
| **Classes** | Noun, PascalCase | `Student`, `CustomerAccount` |
| **Interfaces** | Adjective, PascalCase | `Runnable`, `Serializable` |
| **Methods** | Verb/Verb-Noun, camelCase | `print()`, `calculateSalary()` |
| **Variables** | Noun, camelCase | `age`, `mobileNumber` |
| **Constants** | ALL_CAPS with `_` | `MAX_VALUE`, `MIN_PRIORITY` |

### Java Bean Standards

```java
class Student {
    private String name;
    private boolean active;

    // Setter: public, void, has parameter
    public void setName(String name) { this.name = name; }

    // Getter: public, non-void, no parameters
    public String getName() { return name; }

    // Boolean getter: 'is' preferred over 'get'
    public boolean isActive() { return active; }   // ✅ preferred
    public boolean getActive() { return active; }   // ✅ also valid
}
```

### Listener Standards

```java
public void addMyActionListener(MyActionListener l) {}     // ✅ register
public void removeMyActionListener(MyActionListener l) {}  // ✅ unregister
public void registerMyActionListener(...) {}               // ❌ wrong prefix
```

---

## 10. JVM Memory Areas

```
JVM Memory
├── Method Area       → class bytecode + static variables
├── Heap Area         → objects + instance variables
├── Stack (per Thread)→ method calls + local variables
│                       (each entry = Stack Frame / Activation Record)
├── PC Registers      → address of next instruction (per thread)
└── Native Method Stack → native (non-Java) method calls
```

| Memory Area | Stores | Quick Recall |
|-------------|--------|-------------|
| Method Area | Class bytecode, static variables | `static` → Method Area |
| Heap | Objects, instance variables | `instance` → Heap |
| Stack | Method calls, local variables | `local` → Stack |
| PC Register | Next instruction address (per thread) | — |
| Native Method Stack | Native method invocations | — |

---

## 11. Operators & Assignments

### 11.1 Increment & Decrement Operators

```java
// Pre-increment: increment THEN use
int a = 5;
int b = ++a;   // a=6, b=6

// Post-increment: use THEN increment
int a = 5;
int b = a++;   // b=5, a=6
```

```java
// Nested increment — complex cases
int i = 4;
int j = ++i + ++i + ++i;  // 5 + 6 + 7 = 18  (i becomes 7)

int i = 4;
int j = i++ + i++ + i++;  // 4 + 5 + 6 = 15  (i becomes 7)

int i = 4;
int j = ++i + i++ + ++i;  // 5 + 5 + 7 = 17  (i becomes 7)
```

**Special rule — variable used twice in same expression:**

```java
int i = 5;
i = i++;
// i++ → use i(5) then increment to 6
// But JVM stores original i(5) and assigns back → i = 5 (not 6!)
System.out.println(i); // 5

int i = 5;
i = ++i;  // i = 6
```

**Only variables allowed — not literals:**

```java
int x = 10;
++x;   // ✅
10++;  // ❌ CE: unexpected type, variable required
```

**`final` variable — cannot increment/decrement:**

```java
final int x = 10;
x++;  // ❌ CE: cannot assign to final variable
```

---

### 11.2 Arithmetic Operators

```java
// Integer division truncates (no rounding)
System.out.println(10 / 3);    // 3
System.out.println(10 % 3);    // 1

// Infinity for floating point
System.out.println(10.0 / 0);  // Infinity
System.out.println(-10.0 / 0); // -Infinity
System.out.println(0.0 / 0);   // NaN

// Integer division by zero → ArithmeticException
System.out.println(10 / 0);    // RE: ArithmeticException: / by zero
System.out.println(10 % 0);    // RE: ArithmeticException: / by zero

// Modulo with negative numbers: result takes sign of numerator
System.out.println(-10 % 3);  // -1
System.out.println(10 % -3);  // 1
System.out.println(-10 % -3); // -1
```

---

### 11.3 String Concatenation Operator

```java
// Rule: if one operand is String, + is concatenation (left to right)
String a = "string" + 1 + 2;  // "string12"
String b = 1 + 2 + "string";  // "3string"   (1+2=3 first, then concat)

System.out.println("" + 1 + 2 + 3); // "123"
System.out.println(1 + 2 + "" + 3); // "33"

// Concatenation with += always valid if left is String
String s = "str";
s += 1;     // ✅ "str1"
s += true;  // ✅ "str1true"
s += 10.5;  // ✅ "str1true10.5"
```

---

### 11.4 Relational Operators

```java
// Only for primitive types (not String/objects)
System.out.println(10 > 20);   // false
System.out.println('a' > 'A'); // true (97 > 65)
System.out.println('a' > 10);  // true (97 > 10)

// Chaining is not allowed
System.out.println(10 < 20 < 30);  // ❌ CE: bad operand type
// ✅ Use: 10 < 20 && 20 < 30
```

---

### 11.5 Equality Operators

```java
// Primitives: compare values
System.out.println(10 == 20);    // false
System.out.println('a' == 97);   // true
System.out.println(true == true); // true

// Object references: compare addresses (not content)
Integer i = new Integer(5);
Integer j = new Integer(5);
System.out.println(i == j);       // false (different objects)
System.out.println(i.equals(j));  // true  (same value)

// null checks
String s = null;
System.out.println(s == null);    // true

// ❌ Cannot compare incompatible types
String s = "abc";
Thread t = new Thread();
System.out.println(s == t);  // ❌ CE: incompatible types
```

---

### 11.6 instanceof Operator

```java
// obj instanceof ClassName → true if obj IS-A ClassName
// null instanceof anything → always false (no exception)
Thread t = new Thread();
System.out.println(t instanceof Thread);   // true
System.out.println(t instanceof Object);   // true
System.out.println(t instanceof Runnable); // true

System.out.println(null instanceof Thread); // false (no exception)

String s = "Java";
// System.out.println(s instanceof Thread); // ❌ CE: incompatible types
```

---

### 11.7 Bitwise Operators

| Operator | Name | Description |
|----------|------|-------------|
| `&` | AND | 1 only if both are 1 |
| `\|` | OR | 1 if at least one is 1 |
| `^` | XOR | 1 if exactly one is 1 |
| `~` | Bitwise complement | flips all bits |
| `<<` | Left shift | multiply by 2ⁿ |
| `>>` | Right shift (signed) | divide by 2ⁿ |
| `>>>` | Unsigned right shift | fills with 0 |

```java
System.out.println(4 & 5);    // 4    (100 & 101 = 100)
System.out.println(4 | 5);    // 5    (100 | 101 = 101)
System.out.println(4 ^ 5);    // 1    (100 ^ 101 = 001)
System.out.println(~4);       // -5   (two's complement)
System.out.println(4 << 1);   // 8    (4 × 2¹)
System.out.println(8 >> 1);   // 4    (8 ÷ 2¹)
System.out.println(-8 >>> 1); // 2147483644  (fills with 0)
```

---

### 11.8 Short-Circuit Operators

```java
// && : evaluates right side ONLY if left is true
// || : evaluates right side ONLY if left is false

int x = 10;
if (++x < 10 && ++x < 20) { }  // first ++x → 11, 11 < 10 → false → right NOT evaluated
System.out.println(x); // 11

int x = 10;
if (++x < 10 & ++x < 20) { }   // & always evaluates both → x becomes 12
System.out.println(x); // 12

// || short-circuit
int y = 10;
if (++y > 5 || ++y > 3) { }    // first ++y → 11, 11 > 5 → true → right NOT evaluated
System.out.println(y); // 11
```

---

### 11.9 Type Cast Operator

```java
// Widening: automatic (no cast needed)
int i = 100;
long l = i;   // ✅ automatic widening

// Narrowing: explicit cast required (possible data loss)
double d = 100.5;
int i = (int) d;   // ✅ → 100  (decimal part truncated)
System.out.println(i); // 100

// Overflow on narrowing
long l = 1000000000000L;
int i = (int) l;   // compiles but loses data

// Casting to incompatible type
String s = "hello";
int i = (int) s;   // ❌ CE: incompatible types
```

---

### 11.10 Assignment Operator

```java
// Compound assignment: +=, -=, *=, /=, %=
// Always includes implicit cast — safe for narrowing
byte b = 10;
b += 5;    // ✅ equivalent to b = (byte)(b + 5)
b = b + 5; // ❌ CE: possible loss of precision (int result, can't assign to byte)

// Chained assignment (right to left)
int a, b, c;
a = b = c = 10;   // c=10 first, then b=10, then a=10
```

---

### 11.11 Conditional (Ternary) Operator

```java
// condition ? value_if_true : value_if_false
int a = 10, b = 20;
int min = (a < b) ? a : b;   // 10

// Nesting
int x = 10;
String result = (x > 0) ? "positive" : (x < 0) ? "negative" : "zero";
// result = "positive"
```

---

### 11.12 new Operator

```java
new ClassName();       // creates object on heap
new int[5];            // creates array on heap
new int[][]{{1,2},{3,4}}; // creates 2D array on heap
```

---

### 11.13 Operator Precedence (High → Low)

```
Unary:          ++  --  ~  !  (cast)
Arithmetic:     *  /  %  →  +  -
Shift:          <<  >>  >>>
Relational:     <  <=  >  >=  instanceof
Equality:       ==  !=
Bitwise:        &  →  ^  →  |
Logical:        &&  →  ||
Ternary:        ?:
Assignment:     =  +=  -=  *=  /=  %=  etc.
```

---

### 11.14 Evaluation Order of Operands

> Java evaluates operands **left to right**.

```java
int a = 5;
int b = 6;
System.out.println(a + ++a); // 5 + 6 = 11  (left 'a'=5 evaluated first)

// Method calls — evaluated left to right
int m1() { System.out.print("m1 "); return 1; }
int m2() { System.out.print("m2 "); return 2; }
System.out.println(m1() + m2()); // "m1 m2 3"
```

---

### 11.15 new vs newInstance()

| Feature | `new` | `newInstance()` |
|---------|-------|----------------|
| Type | Operator | Method (`Class.newInstance()`) |
| Compile-time check | ✅ Yes | ❌ No (all errors at runtime) |
| Use case | Known class at compile time | Class name known only at runtime |
| Exception handling | Optional | Mandatory (checked exceptions) |

```java
// new
Dog d = new Dog();  // ✅ compile-time check

// newInstance()
Class c = Class.forName("Dog");
Object d = c.newInstance();  // runtime — must handle exceptions
```

---

### 11.16 instanceof vs isInstance()

| Feature | `instanceof` | `isInstance()` |
|---------|-------------|----------------|
| Type | Operator | Method (`Class.isInstance()`) |
| Class known | At compile time | At runtime |
| Usage | `obj instanceof ClassName` | `ClassName.class.isInstance(obj)` |

```java
Thread t = new Thread();
System.out.println(t instanceof Runnable);         // true
System.out.println(Runnable.class.isInstance(t));  // true
```

---

### 11.17 ClassNotFoundException vs NoClassDefFoundError

| | `ClassNotFoundException` | `NoClassDefFoundError` |
|-|--------------------------|----------------------|
| Type | Checked Exception | Error |
| Cause | Class not in classpath when loaded via `Class.forName()` | Class was present at compile time, missing at runtime |
| When | Runtime — explicit class loading | Runtime — JVM can't find .class |

---

## 12. Flow Control

### 12.1 if-else

```java
// Argument must be boolean
int x = 10;
if (x) { }       // ❌ CE: incompatible types (int not boolean)
if (x == 20) { } // ✅
if (b = true) { } // ✅ boolean assignment is valid
if (b == true) { } // ✅

// else and {} are both optional
if (true);  // ✅ valid empty statement
```

```java
// Without {}: only ONE statement — cannot be declarative
if (true)
    int x = 10;   // ❌ CE: variable declaration not allowed here
if (true)
    System.out.println("ok"); // ✅

// Only immediately next statement depends on if
if (false)
    System.out.println("A");   // skipped
    System.out.println("B");   // ✅ always executes (outside if)
```

> 💡 Compiler does **NOT** check unreachability in if-else (only in loops).

---

### 12.2 switch

```java
// Allowed types for switch argument:
// byte, short, char, int (Java 1.4)
// + Byte, Short, Character, Integer, enum (Java 1.5)
// + String (Java 1.7)

// Curly braces are MANDATORY (unlike other statements)
// Both case and default are optional

switch (x) {
    case 1: System.out.println("one"); break;
    case 2: System.out.println("two"); break;
    default: System.out.println("other");
}
```

**Case label rules:**
```java
// ✅ final variable — compiler replaces with value
final int x = 10;
switch (y) { case x: ... }  // ✅

// ❌ non-final variable
int x = 10;
switch (y) { case x: ... }  // ❌ CE: constant expression required

// ❌ range violation
byte b = 10;
switch (b) { case 1000: ... }  // ❌ CE: possible loss of precision

// ❌ duplicate case
switch (x) { case 97: ... case 'a': ... }  // ❌ CE: duplicate case label

// Independent statements (not under case/default) → CE
switch (x) {
    System.out.println("hello");  // ❌ CE
    case 1: ...
}
```

### Fall-Through

```java
int x = 1;
switch (x) {
    case 0: System.out.println("zero");
    case 1: System.out.println("one");    // ← matches here
    case 2: System.out.println("two");    // falls through
    case 3: System.out.println("three");  // falls through
    default: System.out.println("default"); // falls through
}
// Output: one two three default
```

### default Placement

```java
// default can be placed anywhere — executes if no case matches
switch (x) {
    default: System.out.println("default");
    case 0:  System.out.println("zero");
    case 1:  System.out.println("one");
}
// x=2: default → zero → one (falls through from default's position)
// x=0: zero → one
```

---

### 12.3 while

```java
// Argument must be boolean
while (true) { }   // ✅ infinite loop
while (1) { }      // ❌ CE: incompatible types

// Without {}: one statement, not declarative
while (true) int x = 10;  // ❌ CE

// Empty while
while (true);  // ✅ valid — infinite empty loop
```

**Unreachability:**
```java
while (true) { ... }
System.out.println("hi");  // ❌ CE: unreachable statement

while (false) { ... }      // ❌ CE: unreachable statement (loop body unreachable)

final boolean b = true;
while (b) { ... }
System.out.println("hi");  // ❌ CE: unreachable (compiler knows b is always true)

boolean b = true;
while (b) { ... }
System.out.println("hi");  // ✅ no CE (JVM evaluates non-final at runtime)
```

---

### 12.4 do-while

```java
// Executes at least ONCE regardless of condition
do {
    System.out.println("executed");
} while (false);    // ✅ semicolon after while is MANDATORY

// do-while with continue
int x = 0;
do {
    ++x; System.out.print(x + " ");
    if (++x < 5) continue;   // continue jumps to while condition (NOT top of loop)
    ++x; System.out.print(x + " ");
} while (++x < 10);
// Output: 1  4  6  8  10
```

---

### 12.5 for Loop

```java
// Execution order: ①init → ②check → ③body → ④update → ②check → ...
for (int i = 0; i < 5; i++) { }

// All 3 parts optional — for(;;) = infinite loop
for (;;) { System.out.println("infinite"); }

// Multiple initialization variables — SAME type only
for (int i=0, j=0; i<5; i++) { }           // ✅
for (int i=0, boolean b=true; i<5; i++) { } // ❌ CE: different types
for (int i=0, int j=0; i<5; i++) { }        // ❌ CE: type written twice

// Increment can be any Java statement
for (int i=0; i<5; System.out.println("hi")) { }  // ✅
```

---

### 12.6 for-each (Enhanced for — 1.5v)

```java
// Syntax: for (each_item : target)
int[] a = {10, 20, 30};
for (int x : a) {
    System.out.println(x);  // 10, 20, 30
}

// 2D array
int[][] a = {{1,2,3},{4,5}};
for (int[] row : a) {
    for (int x : row) System.out.print(x + " ");
}
// Output: 1 2 3 4 5

// Limitations:
// - Always left to right (can't reverse)
// - Can't access index
// - Can't replicate index-based operations
```

**Iterable vs Iterator:**

| | `Iterable` | `Iterator` |
|-|-----------|-----------|
| Package | `java.lang` | `java.util` |
| Methods | `iterator()` | `hasNext()`, `next()`, `remove()` |
| Introduced | 1.5v | 1.2v |

> Every array class and Collection interface implements `Iterable`.

---

### 12.7 break

> Valid in **3 places ONLY**: switch · loops · labeled blocks

```java
// In switch: stops fall-through
// In loops: exits loop based on condition
// In labeled block: exits block

l1: {
    System.out.println("A");
    if (x == 10) break l1;   // exits the labeled block
    System.out.println("B");  // skipped if x==10
}

break;  // ❌ CE outside switch/loop: break outside switch or loop
```

---

### 12.8 continue

> Valid **ONLY inside loops** — skips current iteration, continues next.

```java
for (int i = 0; i <= 10; i++) {
    if (i % 2 == 0) continue;
    System.out.println(i);   // prints 1 3 5 7 9
}

continue; // ❌ CE outside loop: continue outside of loop
```

---

### 12.9 Labeled break and continue

```java
// Break/continue a specific outer loop in nested loops
l1: for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (i == j) {
            // break     → skips rest of inner loop iteration
            // break l1  → exits both loops
            // continue  → next iteration of inner loop
            // continue l1 → next iteration of outer loop
        }
        System.out.println(i + "........." + j);
    }
}
```

**Output comparison (nested for i<3, j<3, if i==j):**

| Statement | Output |
|-----------|--------|
| `break` | 0..1, 0..2, 1..0, 1..2, 2..0, 2..1 |
| `break l1` | (no output — exits both loops immediately) |
| `continue` | 0..1, 0..2, 1..0, 1..2, 2..0, 2..1 |
| `continue l1` | 1..0, 2..0, 2..1 |

---

### 12.10 Unreachable Statements — if-else vs loops

```java
// Loops: compiler CHECKS unreachability
while (true) { ... }
stmt;            // ❌ CE: unreachable statement

// if-else: compiler does NOT check
if (true) { ... }
else { ... }     // ✅ no CE (else block unreachable but not flagged)
```

---

## 13. Java Source File Structure

### Key Rules

```java
// At most ONE public class per file; filename must match public class name
public class B { }   // File must be B.java
public class C { }   // ❌ CE: class C is public, should be declared in C.java

// No public class → any filename is valid
class A { } class B { } // Filename can be A.java, B.java, Ashok.java — all valid
```

### Compile vs Run

```
javac Bhaskar.java  →  generates A.class, B.class, C.class, D.class

java A  → A class main method is executed
java D  → ❌ RE: NoSuchMethodError: main     (D has no main())
java Ashok → ❌ RE: NoClassDefFoundError: Ashok
```

> 💡 Compile = source file · Run = class file · Each class gets its own `.class` file.

---

## 14. Import Statement

### Why Import?

```java
// Without import — verbose:
java.util.ArrayList l = new java.util.ArrayList();

// With import — clean:
import java.util.ArrayList;
ArrayList l = new ArrayList();  // ✅
```

### Types of Import

| Type | Example | Recommended? |
|------|---------|:------------:|
| Explicit | `import java.util.ArrayList;` | ✅ Yes — improves readability |
| Implicit (wildcard) | `import java.util.*;` | ❌ No — reduces readability |

### Import Rules

```java
import java.util;           // ❌ cannot import package itself
import java.util.ArrayList.*; // ❌ ArrayList is a class, not a package
import java.util.*;           // ✅
import java.util.ArrayList;   // ✅

// Ambiguity with wildcards (Date in both java.util and java.sql)
import java.util.*;
import java.sql.*;
Date d = new Date();  // ❌ CE: reference to Date is ambiguous

// Resolution: explicit import wins
import java.util.Date;
import java.sql.*;
Date d = new Date();  // ✅ uses java.util.Date (explicit wins)
```

### Compiler Priority (import resolution)

```
1. Explicit class import              (highest)
2. Classes in current working directory
3. Implicit (wildcard) import         (lowest)
```

### Sub-packages NOT included in wildcard

```
java
└── util
    └── regex
        └── Pattern
```

| Import | Works for Pattern? |
|--------|:-----------------:|
| `import java.*;` | ❌ |
| `import java.util.*;` | ❌ |
| `import java.util.regex.*;` | ✅ |
| `import java.util.regex.Pattern;` | ✅ |

### Auto-available packages (no import needed)

1. `java.lang` — `String`, `System`, `Math`, `Integer`, etc.
2. **Default package** (current working directory)

### `#include` (C) vs `import` (Java)

| Feature | `#include` (C/C++) | `import` (Java) |
|---------|-------------------|-----------------|
| Loading | **Static** — all headers loaded at compile | **Dynamic** — `.class` loaded when actually used |
| Memory | Wastage | No wastage |
| Effect | Compile time only | Compile time only |

> 💡 Import is **compile-time only** — more imports = more compile time, **no change in execution time**.

---

## 15. Static Import (1.5v)

> Allows accessing **static members** directly without class name.

```java
// Without static import:
System.out.println(Math.sqrt(4));    // 2.0
System.out.println(Math.max(10,20)); // 20

// With static import:
import static java.lang.Math.*;
System.out.println(sqrt(4));   // 2.0
System.out.println(max(10,20)); // 20

// Import System.out:
import static java.lang.System.out;
out.println("hello");  // ✅
```

> ⚠️ Sun says static import improves readability. Industry experts say it creates confusion.  
> **Recommendation: Avoid unless there is a specific requirement.**

### Ambiguity in Static Import

```java
import static java.lang.Integer.*;
import static java.lang.Byte.*;
System.out.println(MAX_VALUE);  // ❌ CE: ambiguous (Integer.MAX_VALUE vs Byte.MAX_VALUE)
```

**Compiler priority for static members:**
```
1. Current class static members         (highest)
2. Explicit static import
3. Implicit (wildcard) static import    (lowest)
```

### Valid vs Invalid Static Import Syntax

| Statement | Valid? |
|-----------|:------:|
| `import java.lang.Math.*;` | ❌ Math is a class, can't use `.*` on class in normal import |
| `import static java.lang.Math.*;` | ✅ |
| `import java.lang.Math;` | ✅ |
| `import static java.lang.Math;` | ❌ cannot statically import a class |
| `import static java.lang.Math.sqrt;` | ✅ |
| `import static java.lang.Math.sqrt();` | ❌ parentheses not allowed |
| `import static java.lang.Math.sqrt.*;` | ❌ sqrt is a method, not a package |

```
Normal import  starts with → class name (Math) or wildcard (util.*)
Static import  starts with → member name (Math.sqrt) or class wildcard (Math.*)
```

### General Import vs Static Import

| Feature | General Import | Static Import |
|---------|---------------|---------------|
| Imports | Classes/interfaces | Static members of a class |
| Benefit | Short class name — no fully qualified name | Access static member — no class name prefix |

---

## 16. Package Statement

> An **encapsulation mechanism** to group related classes and interfaces.

**Objectives:** Resolve naming conflicts · Improve modularity · Provide security

**Naming convention:** Internet domain name in reverse.

```
com.icicibank.loan.housingloan.Account
 │              │      │          │
domain       module submodule   class
(reversed)
```

### Compile a Package Program

```java
package com.durgajobs.itjobs;
class HydJobs {
    public static void main(String[] args) { System.out.println("package demo"); }
}
```

```bash
# Method 1 — simple compile (class in cwd):
javac HydJobs.java
# cwd/ └── HydJobs.class

# Method 2 — with -d flag (creates package structure):
javac -d . HydJobs.java
# cwd/ └── com/ └── durgajobs/ └── itjobs/ └── HydJobs.class

javac -d c: HydJobs.java
# c:/ └── com/ └── durgajobs/ └── itjobs/ └── HydJobs.class
```

- If destination doesn't exist → CE
- Package structure is **auto-created** by `-d` if not present

### Execute a Package Program

```bash
java com.durgajobs.itjobs.HydJobs   # fully qualified name MANDATORY
```

### Package Statement Rules

```java
// At most ONE package statement per file
package pack1;
package pack2;  // ❌ CE: class, interface, or enum expected

// Package must be FIRST non-comment statement
import java.util.*;
package pack1;   // ❌ CE: package comes before import
class A { }

// Correct order:
package pack1;       // ✅ first
import java.util.*;  // then imports
class A { }          // then class declarations
```

---

## 17. Modifiers

### 17.1 What are Modifiers?

Modifiers tell the JVM about properties of a class, method, or variable:
- Who can **access** it?
- Can it be **extended/overridden**?
- Is it **shared (static)** or per-object?

### 17.2 Class Modifiers

Only **5 modifiers** for top-level (outer) classes:

```
public | default | final | abstract | strictfp
```

Any other modifier on top-level class → ❌ CE: modifier not allowed here

| Modifier | Purpose |
|----------|---------|
| `public` | Accessible from anywhere |
| `default` | Accessible within same package only |
| `final` | Cannot be extended (no child class) |
| `abstract` | Cannot be instantiated; must be subclassed |
| `strictfp` | All floating-point follows IEEE 754 |

```java
public class A { }      // ✅ accessible anywhere
class B { }             // ✅ default — same package only
final class C { }       // ✅ cannot extend
abstract class D { }    // ✅ cannot instantiate
strictfp class E { }    // ✅ strict float

private class F { }    // ❌ CE: not allowed for top-level
protected class G { }  // ❌ CE: not allowed for top-level
static class H { }     // ❌ CE: not allowed for top-level
```

> 💡 Inner/nested classes can additionally use: `private`, `protected`, `static`

---

### 17.3 public Class

```java
// pack1/Animal.java
package pack1;
public class Animal {
    public void speak() { System.out.println("Animal speaks"); }
}

// pack2/Main.java
package pack2;
import pack1.Animal;
class Main {
    public static void main(String[] args) {
        Animal a = new Animal();  // ✅
        a.speak();  // Output: Animal speaks
    }
}
```

---

### 17.4 final Class

```java
final class Vehicle { }
class Car extends Vehicle { }  // ❌ CE: cannot inherit from final Vehicle

// Every method inside final class is implicitly final
// Variables inside final class are NOT automatically final
// Real-world example: java.lang.String is final
```

---

### 17.5 abstract Class

```java
abstract class Shape {
    abstract double area();  // must be implemented by child
}

class Circle extends Shape {
    double radius;
    Circle(double r) { this.radius = r; }

    @Override
    double area() { return Math.PI * radius * radius; }
}

// Shape s = new Shape(); // ❌ CE: cannot instantiate abstract class
Shape s = new Circle(5);  // ✅ parent reference, child object
System.out.println(s.area()); // 78.53...
```

**Abstract class rules:**
- Can have **zero abstract methods** (still valid)
- If even **one** abstract method exists → class **must** be abstract
- Child that doesn't implement all abstract methods → must also be `abstract`

---

### 17.6 strictfp Class

```java
strictfp class Calculator {
    double divide(double a, double b) {
        return a / b;  // IEEE 754 on all platforms
    }
}
```

> 📌 Introduced Java 1.2. From **Java 17** (JEP 306) — IEEE 754 is always used; `strictfp` still valid but has no effect.

---

### 17.7 Member (Access) Modifiers

**4 access modifiers for members:** `public`, `default`, `protected`, `private`

### Access Modifier Comparison Table

| Visibility | `private` | `default` | `protected` | `public` |
|---|:---:|:---:|:---:|:---:|
| Same class | ✅ | ✅ | ✅ | ✅ |
| Child class (same package) | ❌ | ✅ | ✅ | ✅ |
| Non-child (same package) | ❌ | ✅ | ✅ | ✅ |
| Child class (outside package) | ❌ | ❌ | ✅ (child ref only) | ✅ |
| Non-child (outside package) | ❌ | ❌ | ❌ | ✅ |

```
Least → Most accessible:
  private  <  default  <  protected  <  public
```

---

### 17.8 private Member

```java
class BankAccount {
    private double balance = 1000;   // hidden from outside

    public double getBalance() { return balance; }  // controlled access
}

BankAccount acc = new BankAccount();
// System.out.println(acc.balance); // ❌ CE: private
System.out.println(acc.getBalance()); // ✅ 1000.0
```

---

### 17.9 protected Member

```
protected = default + child classes in outside packages
```

```java
// pack1/Animal.java
package pack1;
public class Animal {
    protected void eat() { System.out.println("Eating..."); }
}

// pack2/Dog.java
package pack2;
import pack1.Animal;
class Dog extends Animal {
    public static void main(String[] args) {
        Dog d = new Dog();
        d.eat();        // ✅ via child reference — works

        Animal a = new Animal();
        a.eat();        // ❌ via parent reference from outside package — error!
    }
}
```

---

### 17.10 Variable Modifiers — final

#### final Instance Variables

> JVM won't provide default value — must initialize before constructor completes.

```java
// ✅ Option 1: At declaration
class Student { final int id = 101; }

// ✅ Option 2: Instance initializer block
class Student {
    final int id;
    { id = 101; }
}

// ✅ Option 3: Constructor
class Student {
    final int id;
    Student() { id = 101; }
}

// ❌ Inside a method — too late
class Student {
    final int id;
    void setId() { id = 101; }  // ❌ CE: cannot assign to final variable
}
```

#### final Static Variables

> Must be initialized before class loading completes.

```java
// ✅ Option 1: At declaration
class Config { final static int MAX = 100; }

// ✅ Option 2: Static block
class Config {
    final static int MAX;
    static { MAX = 100; }
}

// ❌ Inside constructor or method — CE
```

#### final Local Variables

```java
public static void main(String[] args) {
    final int x;
    x = 50;                // ✅ initialize before use
    System.out.println(x); // 50
    // x = 60;             // ❌ CE: cannot reassign final
}
```

> 💡 `final` is the **only** modifier allowed for local variables.

---

### 17.11 Method Modifiers

#### static

```java
class MathUtil {
    static int square(int n) { return n * n; }
}
System.out.println(MathUtil.square(5)); // 25 — no object needed

// Method hiding (NOT overriding) with static:
class Parent { static void show() { System.out.println("Parent"); } }
class Child extends Parent { static void show() { System.out.println("Child"); } }
```

#### abstract

```java
abstract class Animal {
    public abstract void sound();   // ✅ no body, ends with ;
    // public abstract void sound(){} // ❌ CE: has empty body
}

class Dog extends Animal {
    public void sound() { System.out.println("Woof!"); }  // ✅
}
```

**Illegal combinations with abstract:**

| Combination | Reason |
|-------------|--------|
| `abstract + final` | final prevents override; abstract requires it |
| `abstract + static` | static needs body; abstract has none |
| `abstract + private` | private hides from child; abstract needs child |
| `abstract + native` | native has external body; abstract has none |
| `abstract + synchronized` | synchronized needs body |
| `abstract + strictfp` | strictfp needs body |

#### final methods

```java
class Parent {
    final void login() { System.out.println("Secure login"); }
}
class Child extends Parent {
    // void login() { } // ❌ CE: cannot override final method
}
```

#### native

```java
class NativeDemo {
    static { System.loadLibrary("mylib"); }  // load native library
    public native void fastCompute();        // declaration only (no body)
}
```

**Objectives:** Improve performance · Reuse legacy C/C++ code · Machine-level memory access  
**Disadvantage:** Breaks Java's platform independence

#### synchronized

```java
class Counter {
    private int count = 0;
    synchronized void increment() { count++; }  // only one thread at a time
}
// ✅ Solves data inconsistency in multithreading
// ❌ Increases wait time — use only when needed
```

#### strictfp method

```java
class Demo {
    strictfp double calculate(double a, double b) { return a / b; }
}
// As of Java 17: IEEE 754 is default — strictfp has no effect but valid syntax
```

---

### 17.12 transient Modifier

> Variable only · Serialization: value is **not saved** — JVM saves default value instead.

```java
class User implements Serializable {
    String username = "alice";
    transient String password = "secret123";  // not serialized
}
// After deserialization: username = "alice", password = null
```

- `transient static` → useless (static not part of object state anyway)
- `transient final` → no impact (final variable's value always written directly)

---

### 17.13 volatile Modifier

> Variable only · Multithreading: ensures **master copy** is always used (no thread-local caching).

```java
class SharedFlag {
    volatile boolean running = true;  // all threads see latest value
    void stop() { running = false; }
}
```

> ⚠️ `final volatile` is **illegal** — final = never changes, volatile = keeps changing.  
> 📌 Largely outdated — prefer `java.util.concurrent` (Java 5+).

---

### 17.14 Modifier Summary Table

| Modifier | Classes | Methods | Variables | Blocks |
|----------|:-------:|:-------:|:---------:|:------:|
| `public` | ✅ | ✅ | ✅ | ❌ |
| `default` | ✅ | ✅ | ✅ | ❌ |
| `private` | ❌ top-level | ✅ | ✅ | ❌ |
| `protected` | ❌ top-level | ✅ | ✅ | ❌ |
| `final` | ✅ | ✅ | ✅ | ❌ |
| `abstract` | ✅ | ✅ | ❌ | ❌ |
| `static` | ❌ top-level | ✅ | ✅ | ✅ |
| `strictfp` | ✅ | ✅ | ❌ | ❌ |
| `synchronized` | ❌ | ✅ | ❌ | ✅ |
| `native` | ❌ | ✅ | ❌ | ❌ |
| `transient` | ❌ | ❌ | ✅ | ❌ |
| `volatile` | ❌ | ❌ | ✅ | ❌ |

---

## 18. Interfaces

### 18.1 What is an Interface?

An interface can be understood from **3 angles**:

**1 — Service Requirement Specification (SRS):** Client defines what services are needed; provider implements.  
**2 — Contract between client and service provider:** ATM screen = contract between bank and customer.  
**3 — 100% Pure Abstract Class:** Every method is implicitly `public abstract`.

```
Sun (defines JDBC API interface)
         │
      JDBC API ◄── interface
      /   |   \
Oracle  DB2  MySQL ◄── implementations (drivers)
```

---

### 18.2 Declaring & Implementing an Interface

```java
// Declaring
interface Drawable {
    void draw();     // implicitly public abstract
    void resize();   // implicitly public abstract
}

// Implementing — ALL methods must be public
class Circle implements Drawable {
    public void draw()   { System.out.println("Drawing circle"); }
    public void resize() { System.out.println("Resizing circle"); }
}

// If not implementing all methods → declare class abstract
abstract class PartialImpl implements Drawable {
    public void draw() { System.out.println("Drawing"); }
    // resize() not implemented → next concrete child must do it
}
```

---

### 18.3 extends vs implements

| Scenario | Keyword | Limit |
|----------|---------|-------|
| Class inherits class | `extends` | ONE only |
| Class implements interface | `implements` | Any NUMBER |
| Class extends + implements | both — `extends` first | ONE class + many interfaces |
| Interface extends interface | `extends` | Any NUMBER |

```java
// Multiple interfaces
interface Flyable { void fly(); }
interface Swimmable { void swim(); }

class Duck implements Flyable, Swimmable {
    public void fly()  { System.out.println("Duck flying"); }
    public void swim() { System.out.println("Duck swimming"); }
}

// Extends + Implements
class Animal { }
interface Talkable { void talk(); }
class Human extends Animal implements Talkable {
    public void talk() { System.out.println("Hello!"); }
}

// Interface extends multiple interfaces
interface A { void m1(); }
interface B { void m2(); }
interface C extends A, B { }  // C inherits both m1() and m2()

// ❌ Wrong order
class Three implements One extends Two { }  // ❌ CE
// ✅ Correct
class Three extends Two implements One { }  // ✅
```

**Quick rules:**

| Expression | Rule |
|-----------|------|
| `X extends Y, Z` | X, Y, Z must all be interfaces |
| `X extends Y` | both classes OR both interfaces |
| `X implements Y, Z` | X is class; Y, Z are interfaces |
| `X extends Y implements Z` | X, Y are classes; Z is interface |

---

### 18.4 Interface Methods

```java
interface Payment {
    void pay(double amount);                  // same as ↓
    public void pay(double amount);           // same as ↓
    abstract void pay(double amount);         // same as ↓
    public abstract void pay(double amount);  // all EQUAL
}
```

> Why `public`? So every implementing class can access it.  
> Why `abstract`? Implementing class provides the body.

**Modifiers NOT allowed on interface methods (pre-Java 8):**  
`private`, `protected`, `final`, `static`, `synchronized`, `native`, `strictfp`

---

### 18.5 Interface Variables

```java
interface Config {
    int MAX_USERS = 100;                      // same as ↓
    public static final int MAX_USERS = 100;  // EQUAL
}
```

> Why `public`? Available to all implementing classes.  
> Why `static`? Accessible without an object.  
> Why `final`? Implementing class can read but NOT modify.

```java
class App implements Config {
    public static void main(String[] args) {
        System.out.println(MAX_USERS);  // ✅ 100
        // MAX_USERS = 200;            // ❌ CE: cannot assign to final
    }
}

interface Bad  { int x; }      // ❌ CE: must initialize
interface Good { int x = 10; } // ✅

// NOT allowed on interface variables:
// private, protected, transient, volatile
```

---

### 18.6 Naming Conflicts

**Method Naming Conflicts:**

```java
// Case 1: Same signature + same return → ONE implementation enough ✅
interface Left  { void show(); }
interface Right { void show(); }
class Test implements Left, Right {
    public void show() { System.out.println("show"); }  // handles both
}

// Case 2: Same name, different args → overloading, implement both ✅
interface Left  { void show(); }
interface Right { void show(int x); }
class Test implements Left, Right {
    public void show()       { }
    public void show(int x)  { }
}

// Case 3: Same signature, different return types → IMPOSSIBLE ❌
interface Left  { void show(); }
interface Right { int  show(); }
// class Test implements Left, Right { }  // ❌ CE: impossible
```

**Variable Naming Conflicts:**

```java
interface Left  { int x = 888; }
interface Right { int x = 999; }

class Test implements Left, Right {
    public static void main(String[] args) {
        // System.out.println(x);     // ❌ CE: ambiguous
        System.out.println(Left.x);   // ✅ 888
        System.out.println(Right.x);  // ✅ 999
    }
}
```

---

### 18.7 Marker Interface

> Interface with **no methods and no variables** — grants special ability via JVM.

```java
// Built-in marker interfaces:
Serializable   // save/send objects over network
Cloneable      // clone objects
RandomAccess   // fast random access in lists

class Student implements Serializable {  // gains serialization ability
    String name; int rollno;
}
```

---

### 18.8 Adapter Class

> Implements an interface with **empty bodies for all methods** — so you override only what you need.

```java
// Problem without adapter:
interface WindowListener {
    void windowOpened();
    void windowClosed();
    void windowMinimized();
    void windowMaximized();  // 7 methods total
}
// Must implement all 7 even if you care only about windowClosed()

// ✅ Solution — Adapter Class:
abstract class WindowAdapter implements WindowListener {
    public void windowOpened()    {}
    public void windowClosed()    {}
    public void windowMinimized() {}
    public void windowMaximized() {}
}

class MyListener extends WindowAdapter {
    public void windowClosed() {
        System.out.println("Closed!");  // override only what matters
    }
}
```

**Real Java example:**

```
Servlet (Interface)
      ↓
GenericServlet (Abstract Adapter)
      ↓
HttpServlet (Abstract)
      ↓
YourServlet (Concrete — extends HttpServlet)
```

---

### 18.9 Interface vs Abstract Class vs Concrete Class

| Situation | Use |
|-----------|-----|
| Only requirement spec, no implementation | **Interface** |
| Partial implementation | **Abstract Class** |
| Full implementation, ready to use | **Concrete Class** |

### Detailed Comparison

| Feature | Interface | Abstract Class |
|---------|-----------|----------------|
| Methods | Always `public abstract` (pre-Java 8) | Any modifier |
| Variables | Always `public static final` | No restrictions |
| Constructor | ❌ Not allowed | ✅ Allowed |
| Static block | ❌ Not allowed | ✅ Allowed |
| Instance block | ❌ Not allowed | ✅ Allowed |
| Variable init | Must init at declaration | Can init later |
| Extending class | ❌ Cannot | ✅ Can extend one class |
| Performance | Faster | Slower |

**Why not always use Abstract Class?**

```java
abstract class X { ... }
class Test extends X { }  // Test can't extend anything else now!

interface X { ... }
class Test implements X { }  // Test is still free to extend another class
```

> 💡 If everything is abstract → prefer **interface** (better performance + keeps inheritance open).

**Why does Abstract Class have a constructor when you can't instantiate it?**

```java
// Constructor initializes instance variables for child objects
abstract class Animal {
    String name;
    Animal(String name) { this.name = name; }
}

class Dog extends Animal {
    Dog(String name) { super(name); }
    void bark() { System.out.println(name + " says Woof!"); }
}

Dog d = new Dog("Rex");
d.bark();  // Output: Rex says Woof!
```

---

### 18.10 Modern Java Interface Features (Java 8+)

| Feature | Version | Description |
|---------|---------|-------------|
| `default` methods | Java 8 | Methods with body; inherited by implementing classes |
| `static` methods | Java 8 | Called on interface name; not inherited |
| Functional Interface | Java 8 | Exactly one abstract method; used with lambdas |
| `private` methods | Java 9 | Helper methods inside interface |
| `private static` methods | Java 9 | Static helper methods |

```java
interface Greeting {
    // default method
    default void greet() { System.out.println("Hello!"); }

    // static method
    static void info() { System.out.println("Greeting interface"); }
}

class FormalGreeting implements Greeting { /* greet() inherited */ }

FormalGreeting f = new FormalGreeting();
f.greet();         // Hello!
Greeting.info();   // Greeting interface
```

```java
// Java 9 — private methods
interface Helper {
    private void log(String msg) { System.out.println("Log: " + msg); }
    default void doWork() { log("Working..."); }
}
```

```java
// Functional Interface (Java 8) + Lambda
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);  // exactly ONE abstract method
}

Calculator add = (a, b) -> a + b;
Calculator mul = (a, b) -> a * b;

System.out.println(add.calculate(3, 4));  // 7
System.out.println(mul.calculate(3, 4));  // 12
```

> 📌 Built-in functional interfaces: `Runnable`, `Callable`, `Comparator`, `Predicate`, `Function`, `Consumer`, `Supplier` — all in `java.util.function`

---

## 20. Quick Recap

### Identifiers & Keywords
```
Allowed chars: a-z A-Z 0-9 _ $   |   Cannot start with digit   |   Case sensitive
53 reserved words: 50 keywords + 3 literals (true, false, null)
goto, const → reserved but banned
strictfp(1.2) · assert(1.4) · enum(1.5)
```

### Data Types & Literals
```
Primitive (8): byte short int long float double char boolean
Default int literal → int    Default float literal → double (needs f suffix for float)
Java 7: binary literals (0b) + underscore in numbers
Widening: byte→short→int→long→float→double  |  char→int
```

### Variables
```
Instance → Heap → JVM default ✅ → cannot access from static context
Static   → Method Area → JVM default ✅ → one copy per class
Local    → Stack → NO default ❌ → must initialize → only final modifier allowed
```

### Arrays
```
int[] a = new int[3]   |   size at declaration ❌   |   negative size → NegativeArraySizeException
length (array) vs length() (String)
Anonymous: new int[]{10,20,30}  no size allowed
Element widening ≠ array widening   |   Type + Dimensions must match; size irrelevant
```

### Var-Args
```
type... varName   |   Must be LAST parameter   |   Only ONE per method
Internally: 1D array   |   Priority: LOWEST (general method wins)
Array → var-arg ✅   |   Var-arg → array ❌
```

### main() Method
```
public static void main(String[] args)
Invalid main → compiles fine → RE: NoSuchMethodError at runtime
Java 1.7+: main() mandatory even for static blocks
All command-line args are Strings; + = concatenation
```

### Operators
```
i++ → use then increment   |   ++i → increment then use
i = i++ → stays at original value (JVM saves original, assigns back)
10.0/0 → Infinity   |   0.0/0 → NaN   |   10/0 → ArithmeticException
instanceof: null always returns false
Compound assignment (+=) includes implicit cast
```

### Flow Control
```
if: argument must be boolean (not int like C)
switch: byte/short/char/int(1.4) + wrapper+enum(1.5) + String(1.7) | {} mandatory
while(true)→code after = CE | while(false)→body = CE | non-final var → no CE
do-while: continue jumps to while condition, NOT top of loop
for(;;) = infinite loop | all 3 parts optional
break: switch/loop/labeled block only | continue: loop only
Compiler checks unreachability in LOOPS only — NOT in if-else
```

### Source File / Import / Package
```
At most 1 public class; filename must match public class name
import = compile-time only; sub-packages NOT included in .*
java.lang + default package are auto-available
Compiler priority: explicit → current dir → wildcard
Package: at most 1; must be FIRST; javac -d . to create structure; fully qualified name to run
```

### Modifiers
```
Top-level class: public | default | final | abstract | strictfp
abstract+final ❌ | abstract+static ❌ | abstract+private ❌ | native+strictfp ❌ | final+volatile ❌
transient: skip in serialization | volatile: always use main memory
final instance var: init at declaration / instance block / constructor
final static var: init at declaration / static block
final local var: only final modifier allowed on local variables
```

### Interfaces
```
Methods: implicitly public abstract   |   Variables: implicitly public static final
Variables must be initialized at declaration
implements: multiple allowed | extends (interface): multiple allowed | extends comes BEFORE implements
Same sig + same return → one impl | Same sig + diff return → impossible ❌
Marker interface: no methods/variables (Serializable, Cloneable)
Adapter class: all empty methods → extend and override only needed
Interface preferred over abstract class if everything abstract (better perf + keeps inheritance open)
Java 8: default methods, static methods, functional interfaces
Java 9: private methods, private static methods
```

---

> 📘 **References:** Java SE Documentation — [docs.oracle.com/en/java](https://docs.oracle.com/en/java/index.html)  
> Covers Java 1.0 through Java 21+
