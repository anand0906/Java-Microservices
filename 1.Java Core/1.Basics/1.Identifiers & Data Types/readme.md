# ☕ Core Java Basics

> **Java Version Coverage:** Java 1.2 → Java 17+  
> Covers: Identifiers · Reserved Words · Data Types · Literals

---

## 📑 Index

1. [Identifiers](#1-identifiers)
2. [Reserved Words](#2-reserved-words)
3. [Data Types](#3-data-types)
4. [Literals](#4-literals)

---

## 1. Identifiers

> Any **name** in a Java program — class name, method name, variable name, or label — is called an **Identifier**.

### ✅ Rules

| Rule | Description | Valid | Invalid |
|------|-------------|-------|---------|
| Allowed chars | `a-z`, `A-Z`, `0-9`, `_`, `$` | `total_number` | `Total#` |
| No digit start | Can't start with a digit | `ABC123` | `123ABC` |
| Case sensitive | `number` ≠ `Number` ≠ `NUMBER` | — | — |
| No reserved words | Can't use keywords as names | `count` | `int if = 10` |
| No length limit | Recommended ≤ 15 chars | — | — |

### Case Sensitivity Example

```java
class Test {
    int number  = 10;
    int Number  = 20;
    int NUMBER  = 30;
    int NuMbEr  = 40;
    // All 4 are different variables ✅
}
```

### Reserved Word as Identifier — Compile Error

```java
int if  = 10;   // ❌ CE: illegal start of expression
int for = 5;    // ❌ CE: illegal start of expression
```

### Class/Interface Name as Identifier — Legal but Bad Practice

```java
int String   = 10;
int Runnable = 20;
System.out.println(String);    // Output: 10  ✅ (compiles, but avoid!)
System.out.println(Runnable);  // Output: 20  ✅ (compiles, but avoid!)
```

---

## 2. Reserved Words

> Words with **pre-defined meaning** in Java. Cannot be used as identifiers.  
> Total: **53 reserved words**

### 🗂️ Categories

```
Reserved Words (53)
├── Keywords (50)
│   ├── Data Types      → byte, short, int, long, float, double, char, boolean
│   ├── Flow Control    → if, else, switch, case, default, for, do, while,
│   │                     break, continue, return
│   ├── Modifiers       → public, private, protected, static, final, abstract,
│   │                     synchronized, native, strictfp, transient, volatile
│   ├── Exception       → try, catch, finally, throw, throws, assert
│   ├── Class related   → class, package, import, extends, implements, interface
│   ├── Object related  → new, instanceof, super, this
│   ├── Return type     → void
│   └── Unused          → goto, const  ⚠️ (reserved but banned)
│
└── Reserved Literals (3)
    ├── true
    ├── false
    └── null
```

### 📌 Version-wise New Keywords

| Keyword | Introduced In | Purpose |
|---------|--------------|---------|
| `strictfp` | Java 1.2 | Strict floating-point calculation |
| `assert` | Java 1.4 | Testing/debugging assertions |
| `enum` | Java 1.5 | Group of named constants |

```java
// enum example (Java 1.5+)
enum Beer { KF, RC, KO, FO }
```

### ⚠️ Common Traps — Valid or Not?

| List | Valid? | Reason |
|------|--------|--------|
| `final, finally, finalize` | ❌ | `finalize` is a method, not a keyword |
| `throw, throws, thrown` | ❌ | `thrown` doesn't exist in Java |
| `break, continue, return, exit` | ❌ | `exit` is not a keyword |
| `byte, short, Integer, long` | ❌ | `Integer` is a wrapper class |
| `extends, implements, imports` | ❌ | `imports` doesn't exist |
| `instanceof, sizeOf` | ❌ | `sizeOf` doesn't exist |
| `new, delete` | ❌ | `delete` doesn't exist (GC handles cleanup) |
| `public, static, void` | ✅ | All valid keywords |
| `main, String, args` | ❌ | Not keywords |

> 💡 All Java keywords are **lowercase only**.  
> `instanceof` not `instanceOf` · `strictfp` not `strictFp` · `synchronized` not `syncronize`

---

## 3. Data Types

> Java is a **strongly typed** language — every variable and expression must have a declared type, checked at compile time.

### 🗂️ Type Tree

```
Data Types
├── Primitive (8)
│   ├── Numeric
│   │   ├── Integral   → byte · short · int · long
│   │   └── Decimal    → float · double
│   ├── Character      → char
│   └── Boolean        → boolean
│
└── Non-Primitive
    └── Object References  (default value = null)
```

> ⚠️ Java is **not pure OOP** — it uses primitive types and lacks operator overloading & multiple inheritance.

---

### 📊 Primitive Data Types — Quick Reference

| Type | Size | Range | Wrapper | Default |
|------|------|-------|---------|---------|
| `byte` | 1 byte | -2⁷ to 2⁷-1 &nbsp;(-128 to 127) | `Byte` | `0` |
| `short` | 2 bytes | -2¹⁵ to 2¹⁵-1 &nbsp;(-32,768 to 32,767) | `Short` | `0` |
| `int` | 4 bytes | -2³¹ to 2³¹-1 &nbsp;(-2,147,483,648 to 2,147,483,647) | `Integer` | `0` |
| `long` | 8 bytes | -2⁶³ to 2⁶³-1 | `Long` | `0` |
| `float` | 4 bytes | -3.4e38 to 3.4e38 | `Float` | `0.0` |
| `double` | 8 bytes | -1.7e308 to 1.7e308 | `Double` | `0.0` |
| `char` | 2 bytes | 0 to 2¹⁶-1 &nbsp;(0 to 65,535, unsigned) | `Character` | `0` (blank) |
| `boolean` | JVM-dependent | `true` / `false` only | `Boolean` | `false` |

> `boolean` and `char` are the only **unsigned** types. All others support `+ve` and `-ve` values.

---

### 📈 Widening Order (automatic, no data loss)

```
byte → short → int → long → float → double
                ↑
              char
```

---

### 3.1 byte

Size: **1 byte** &nbsp;|&nbsp; Range: **-2⁷ to 2⁷-1** (-128 to 127)

```java
byte b = 10;       // ✅
byte b = 127;      // ✅ max value

byte b = 130;      // ❌ CE: possible loss of precision
                   //       found: int, required: byte
byte b = 10.5;     // ❌ CE: possible loss of precision (decimal not allowed)
byte b = true;     // ❌ CE: incompatible types
byte b = "hello";  // ❌ CE: incompatible types
                   //       found: java.lang.String, required: byte
```

> 💡 Best use: handling data **streams** from files or networks.

---

### 3.2 short

Size: **2 bytes** &nbsp;|&nbsp; Range: **-2¹⁵ to 2¹⁵-1** (-32,768 to 32,767)

```java
short s = 130;      // ✅
short s = 32767;    // ✅ max value

short s = 32768;    // ❌ CE: possible loss of precision
short s = true;     // ❌ CE: incompatible types
```

> 💡 Rarely used — designed for 16-bit processors (e.g., 8086) which are now outdated.

---

### 3.3 int

Size: **4 bytes** &nbsp;|&nbsp; Range: **-2³¹ to 2³¹-1** (-2,147,483,648 to 2,147,483,647)

```java
int i = 130;         // ✅
int i = 2147483647;  // ✅ max value

int i = 10.5;        // ❌ CE: possible loss of precision
int i = true;        // ❌ CE: incompatible types
```

> 💡 Most commonly used integral type.

---

### 3.4 long

Size: **8 bytes** &nbsp;|&nbsp; Range: **-2⁶³ to 2⁶³-1**

```java
long l = 10;              // ✅ int literal auto-promoted to long
long l = 10L;             // ✅ explicit long literal
long fileSize = f.length(); // ✅ file length can exceed int range

int x = 10L;              // ❌ CE: possible loss of precision
                          //       found: long, required: int
```

> 💡 Use when `int` is not enough — e.g., file sizes, timestamps, large counters.

---

### 3.5 float vs double

| | `float` | `double` |
|---|---------|---------|
| Size | 4 bytes | 8 bytes |
| Precision | 5–6 decimal places | 14–15 decimal places |
| Range | -3.4e38 to 3.4e38 | -1.7e308 to 1.7e308 |
| Suffix | `f` or `F` required | `d`/`D` optional |

```java
double d = 123.456;    // ✅ default is double
double d = 123.456D;   // ✅ explicit double
float  f = 123.456f;   // ✅ explicit float
float  f = 10e2F;      // ✅ = 1000.0

float  f = 123.456;    // ❌ CE: possible loss of precision (missing f)
float  f = 10e2;       // ❌ CE: possible loss of precision

double d = 0123.456;   // ✅ treated as decimal (not octal)
double d = 0x123.456;  // ❌ CE: malformed floating point literal

double d = 10e2;
System.out.println(d); // 1000.0  (10 × 10²)

double d = 0xBeef;
System.out.println(d); // 48879.0  ✅ hex integral auto-promoted to double

int x = 10.0;          // ❌ CE: possible loss of precision
                       //    floating point literal cannot go to integral type
```

---

### 3.6 boolean

Size: **JVM-dependent** &nbsp;|&nbsp; Values: **`true` or `false` only**

```java
boolean b = true;    // ✅
boolean b = false;   // ✅

boolean b = True;    // ❌ CE: cannot find symbol
boolean b = TRUE;    // ❌ CE: cannot find symbol
boolean b = "true";  // ❌ CE: incompatible types
boolean b = 0;       // ❌ CE: incompatible types  (unlike C/C++)
boolean b = 1;       // ❌ CE: incompatible types
```

---

### 3.7 char

Size: **2 bytes** &nbsp;|&nbsp; Range: **0 to 2¹⁶-1** (0 to 65,535, Unicode)

> Java uses **Unicode** (2 bytes), unlike C/C++ which uses ASCII (1 byte, 0–255).

```java
// Single character in single quotes
char c = 'A';        // ✅
char c = a;          // ❌ CE: cannot find symbol (missing quotes)
char c = "A";        // ❌ CE: incompatible types (double quotes = String)
char c = 'AB';       // ❌ CE: unclosed character literal

// Integer (Unicode decimal value)
char c = 97;         // ✅ → 'a'
char c = 65535;      // ✅ max
char c = 65536;      // ❌ CE: possible loss of precision (out of range)

// Unicode escape \uXXXX
char c = '\u0061';   // ✅ → 'a'
char c = \u0062;     // ❌ CE: cannot find symbol (no quotes)
char c = '\iface';   // ❌ CE: illegal escape character
```

---

## 4. Literals

> A **literal** is any constant value assigned to a variable.  
> `int x = 10;` → `10` is a literal.

---

### 4.1 Integral Literals

| Form | Prefix | Allowed Digits | Example | Output |
|------|--------|---------------|---------|--------|
| Decimal | none | 0–9 | `int x = 10` | 10 |
| Octal | `0` | 0–7 | `int x = 010` | 8 |
| Hexadecimal | `0x` / `0X` | 0–9, A–F | `int x = 0x10` | 16 |
| Binary *(Java 7+)* | `0b` / `0B` | 0–1 | `int x = 0b111` | 7 |

```java
int x = 10;    System.out.println(x); // 10
int y = 010;   System.out.println(y); // 8
int z = 0x10;  System.out.println(z); // 16
int b = 0b111; System.out.println(b); // 7  ← Java 7+
```

**Valid / Invalid declarations:**

```java
int x = 0777;     // ✅ valid octal
int x = 0786;     // ❌ CE: integer number too large (8,9 not valid in octal)
int x = 0xFACE;   // ✅ valid hex
int x = 0xbeef;   // ✅ valid hex (A–F case-insensitive)
int x = 0xBeer;   // ❌ CE: ';' expected  ('r' is not a hex digit)
int x = 0xabb2cd; // ✅ valid hex
```

**long suffix:**

```java
long l = 10L;   // ✅ explicit long
long l = 10;    // ✅ int auto-widened to long
int  x = 10L;   // ❌ CE: possible loss of precision
                //       found: long, required: int
```

**byte / short — no direct literal, compiler checks range:**

```java
byte b  = 127;    // ✅ within byte range (-2⁷ to 2⁷-1)
byte b  = 128;    // ❌ CE: possible loss of precision
short s = 32767;  // ✅ within short range (-2¹⁵ to 2¹⁵-1)
short s = 32768;  // ❌ CE: possible loss of precision
```

---

### 4.2 Floating-Point Literals

> Default type is **`double`**. Use `f`/`F` suffix for `float`.

```java
double d = 123.456;    // ✅ double (default)
double d = 123.456D;   // ✅ explicit double
float  f = 123.456f;   // ✅ explicit float
float  f = 10e2F;      // ✅ exponential float = 1000.0

float  f = 123.456;    // ❌ CE: possible loss of precision
float  f = 10e2;       // ❌ CE: possible loss of precision

double d = 0123.456;   // ✅ treated as decimal (not octal for floats)
double d = 0x123.456;  // ❌ CE: malformed floating point literal

double d = 10e2;
System.out.println(d); // 1000.0  (10 × 10²)

int x = 10.0;          // ❌ CE: possible loss of precision
```

---

### 4.3 Boolean Literals

> Only `true` or `false` (lowercase). `0`/`1` are NOT valid (unlike C/C++).

```java
boolean b = true;    // ✅
boolean b = false;   // ✅

boolean b = True;    // ❌ CE: cannot find symbol
boolean b = TRUE;    // ❌ CE: cannot find symbol
boolean b = "true";  // ❌ CE: incompatible types
boolean b = 0;       // ❌ CE: incompatible types
boolean b = 1;       // ❌ CE: incompatible types
```

---

### 4.4 Char Literals

**3 ways to specify a char:**

```java
// 1. Single character in single quotes
char c = 'a';        // ✅
char c = a;          // ❌ CE: cannot find symbol
char c = "a";        // ❌ CE: incompatible types
char c = 'ab';       // ❌ CE: unclosed character literal

// 2. Integer (Unicode decimal 0–65535)
char c = 97;         // ✅ → 'a'
char c = 65535;      // ✅ max (2¹⁶-1)
char c = 65536;      // ❌ CE: possible loss of precision

// 3. Unicode escape \uXXXX (4-digit hex)
char c = '\u0061';   // ✅ → 'a'
char c = '\u0000';   // ✅ → null/blank (default char value)
char c = \u0062;     // ❌ CE: cannot find symbol (no quotes)
char c = '\iface';   // ❌ CE: illegal escape character
```

**Escape Characters:**

| Escape | Meaning |
|--------|---------|
| `\n` | New line |
| `\t` | Horizontal tab |
| `\r` | Carriage return |
| `\f` | Form feed |
| `\b` | Backspace |
| `\'` | Single quote |
| `\"` | Double quote |
| `\\` | Backslash |

```java
char c = '\n';   // ✅ newline
char c = '\t';   // ✅ tab
char c = '\l';   // ❌ CE: illegal escape character ('\l' doesn't exist)
char c = '/n';   // ❌ CE: unclosed character literal (wrong slash direction)
```

---

### 4.5 String Literals

```java
String s = "Hello Java";   // ✅
String s = "";             // ✅ empty string is valid
```

---

### 4.6 Java 7+ Literal Enhancements

#### Binary Literals *(Java 7+)*

```java
int x = 0b1010;    // ✅ = 10
int y = 0B111;     // ✅ = 7
System.out.println(x); // 10
System.out.println(y); // 7
```

#### Underscore in Numeric Literals *(Java 7+)*

> Removed at **compile time** — only improves readability.

```java
double d = 1_23_456.7_8_9;   // ✅ → 123456.789
int    n = 1_000_000;         // ✅ → 1000000

double d = _123.456;   // ❌ can't start with underscore
double d = 123.456_;   // ❌ can't end with underscore
double d = 123_.456;   // ❌ can't be adjacent to decimal point
double d = 123._456;   // ❌ can't be adjacent to decimal point
```

---

### 📋 Literals — Full Summary

| Literal | Default Type | Example | Change type with |
|---------|-------------|---------|-----------------|
| `10` | `int` | `int x = 10` | `10L` → `long` |
| `10.5` | `double` | `double d = 10.5` | `10.5f` → `float` |
| `'a'` | `char` | `char c = 'a'` | — |
| `true` / `false` | `boolean` | `boolean b = true` | — |
| `"text"` | `String` | `String s = "hi"` | — |
| `0b101` *(Java 7+)* | `int` | `int x = 0b101` | `0b101L` → `long` |
| `1_000` *(Java 7+)* | `int` | `int x = 1_000` | — |

---

> 💡 **Key Rule:** Java catches **type mismatches at compile time** — not at runtime.  
> When in doubt, check: size, sign, and suffix!
