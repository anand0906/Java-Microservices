# ☕ Java — Operators & Assignments

> **Java Version:** Java 1.0+  
> **Topics:** Increment/Decrement · Arithmetic · String Concat · Relational · Equality · instanceof · Bitwise · Short Circuit · Type Cast · Assignment · Conditional · new · Precedence · Evaluation Order · new vs newInstance() · instanceof vs isInstance() · ClassNotFoundException vs NoClassDefFoundError

---

## 📑 Index

| # | Topic |
|---|-------|
| 1 | [Increment & Decrement Operators](#1-increment--decrement-operators) |
| 2 | [Arithmetic Operators](#2-arithmetic-operators) |
| 3 | [String Concatenation Operator](#3-string-concatenation-operator) |
| 4 | [Relational Operators](#4-relational-operators) |
| 5 | [Equality Operators](#5-equality-operators) |
| 6 | [instanceof Operator](#6-instanceof-operator) |
| 7 | [Bitwise Operators](#7-bitwise-operators) |
| 8 | [Short Circuit Operators](#8-short-circuit-operators) |
| 9 | [Type Cast Operator](#9-type-cast-operator) |
| 10 | [Assignment Operator](#10-assignment-operator) |
| 11 | [Conditional Operator](#11-conditional-operator) |
| 12 | [new Operator](#12-new-operator) |
| 13 | [Operator Precedence](#13-operator-precedence) |
| 14 | [Evaluation Order of Operands](#14-evaluation-order-of-operands) |
| 15 | [new vs newInstance()](#15-new-vs-newinstance) |
| 16 | [instanceof vs isInstance()](#16-instanceof-vs-isinstance) |
| 17 | [ClassNotFoundException vs NoClassDefFoundError](#17-classnotfoundexception-vs-noclassdeffounderror) |

---

## 1. Increment & Decrement Operators

### 📌 Definition
- **Increment (`++`)** → increases variable value by 1
- **Decrement (`--`)** → decreases variable value by 1
- Each comes in two forms: **pre** (change first, then use) and **post** (use first, then change)

### 📋 Forms

| Form | Syntax | Meaning |
|------|--------|---------|
| Pre-increment | `y = ++x` | Increment x first, then assign to y |
| Post-increment | `y = x++` | Assign x to y first, then increment x |
| Pre-decrement | `y = --x` | Decrement x first, then assign to y |
| Post-decrement | `y = x--` | Assign x to y first, then decrement x |

### 📊 Behavior Table (initial x = 10)

| Expression | Value of y | Final value of x |
|-----------|:----------:|:----------------:|
| `y = ++x` | 11 | 11 |
| `y = x++` | 10 | 11 |
| `y = --x` | 9 | 9 |
| `y = x--` | 10 | 9 |

---

### ⚠️ Rules

**Rule 1 — Only on variables, NOT on constants**  
`++` needs a variable to store the updated value. Constants have no storage to update.

```java
int x = 4;
int y = ++x;   // ✅  y = 5

int y = ++4;   // ❌ CE: unexpected type
               //       required: variable, found: value
```

**Rule 2 — Nesting NOT allowed**  
`++x` returns a value, not a variable. You cannot apply `++` on a value again.

```java
int x = 4;
int y = ++(++x);  // ❌ CE: unexpected type
                  //       required: variable, found: value
```

**Rule 3 — NOT applicable on `final` variables**  
`final` makes a variable constant — its value cannot be changed. `++` tries to change it.

```java
final int x = 4;
x++;   // ❌ CE: cannot assign a value to final variable x
```

**Rule 4 — Applicable for all primitives EXCEPT `boolean`**  
`boolean` has no numeric meaning, so `++`/`--` make no sense on it.

```java
int    x  = 10;   x++;   // ✅  → 11
char   ch = 'a';  ch++;  // ✅  → 'b'  (Unicode value incremented)
double d  = 10.5; d++;   // ✅  → 11.5

boolean b = true;
b++;   // ❌ CE: operator ++ cannot be applied to boolean
```

---

### 🔍 `b++` vs `b = b+1` — Key Difference

**Rule:** When applying any arithmetic operator between `a` and `b`, result type = **`max(int, type of a, type of b)`**  
So `byte + 1` → result is `int`. Assigning back to `byte` without cast → CE.

```java
byte b = 10;
b = b + 1;   // ❌ CE: possible loss of precision
             //       found: int, required: byte
```

With `++`, the **compiler automatically performs internal type casting**:

> `b++`  →  internally  →  `b = (type of b)(b + 1)`  →  `b = (byte)(b + 1)`

```java
byte b = 10;
b++;         // ✅  auto-cast: b = (byte)(10 + 1) = 11
b += 1;      // ✅  compound operators also auto-cast
```

> 💡 `b++` and `b+=1` work but `b = b+1` does NOT — because auto type casting only happens in `++`/`--` and compound operators.

---

## 2. Arithmetic Operators

### 📌 Result Type Rule

> **Result type = max(int, type of a, type of b)**

The minimum result type is always `int` — even `byte + byte` gives `int`.

### 📊 Type Promotion Table

| Expression | Result Type |
|-----------|:-----------:|
| `byte + byte` | `int` |
| `byte + short` | `int` |
| `short + short` | `int` |
| `char + char` | `int` |
| `char + int` | `int` |
| `byte + char` | `int` |
| `int + long` | `long` |
| `long + long` | `long` |
| `long + float` | `float` |
| `float + double` | `double` |
| `int + double` | `double` |

```java
System.out.println('a' + 'b');  // 195   → 97 + 98 = int
System.out.println('a' + 1);    // 98    → char promoted to int
System.out.println('a' + 1.2);  // 98.2  → result is double

byte a = 10, b = 20;
byte c = a + b;          // ❌ CE: possible loss of precision (result is int)
byte c = (byte)(a + b);  // ✅  explicit cast needed
```

---

### 📌 Infinity in Java

**Integral arithmetic** (`byte`, `short`, `int`, `long`) — **NO concept of Infinity**.  
Dividing by zero in integral arithmetic → `RuntimeException`.

**Floating point arithmetic** (`float`, `double`) — **HAS a concept of Infinity**.  
`Float` and `Double` classes contain constants `POSITIVE_INFINITY` and `NEGATIVE_INFINITY`.  
So dividing float/double by zero gives `Infinity` — no exception thrown.

```java
System.out.println(10 / 0);      // ❌ RE: ArithmeticException: / by zero
System.out.println(10 / 0.0);    // ✅  Infinity
System.out.println(-10 / 0.0);   // ✅  -Infinity
```

---

### 📌 NaN (Not a Number)

**Integral arithmetic** — NO way to represent undefined results.  
`0/0` in integral → `ArithmeticException`.

**Floating point arithmetic** — `Float` and `Double` have a constant `NaN` for undefined results.  
`0.0/0.0` → `NaN`, no exception.

```java
System.out.println(0 / 0);      // ❌ RE: ArithmeticException: / by zero
System.out.println(0.0 / 0.0);  // ✅  NaN
System.out.println(-0.0 / 0.0); // ✅  NaN
```

**For any value `x` including `NaN` — ALL comparisons return `false` except `!=` which returns `true`.**

```java
System.out.println(10 < Float.NaN);          // false
System.out.println(10 <= Float.NaN);         // false
System.out.println(10 > Float.NaN);          // false
System.out.println(10 >= Float.NaN);         // false
System.out.println(10 == Float.NaN);         // false
System.out.println(Float.NaN == Float.NaN);  // false  ⚠️ NaN ≠ itself!

System.out.println(10 != Float.NaN);         // true
System.out.println(Float.NaN != Float.NaN);  // true
```

---

### 📌 ArithmeticException — 3 Key Points

1. It is a **RuntimeException** — not a compile-time error
2. Occurs **only in integral arithmetic** — never in floating point
3. Only operations that cause it: **`/`** and **`%`**

---

## 3. String Concatenation Operator

### 📌 Definition
The `+` operator is the **only overloaded operator in Java**.
- If **at least one operand is a String** → `+` acts as **String concatenation**
- If **both operands are numbers** → `+` acts as **arithmetic addition**

### 📌 Evaluation: Left to Right

```java
String a = "java";
int b = 10, c = 20, d = 30;

// Each step evaluated left → right:
System.out.println(a + b + c + d); // "java"+10 → "java10"+20 → "java1020"+30 → "java102030"
System.out.println(b + c + d + a); // 10+20=30, 30+30=60, 60+"java" → "60java"
System.out.println(b + c + a + d); // 10+20=30, 30+"java"→"30java", "30java"+30 → "30java30"
System.out.println(b + a + c + d); // 10+"java"→"10java", +"20"→"10java20", +"30"→"10java2030"
```

### ⚠️ Compile Errors

```java
String a = "java";
int b = 10, c = 20, d = 30;

a = b + c + d;  // ❌ CE: incompatible types — int cannot be assigned to String
b = a + c + d;  // ❌ CE: incompatible types — String cannot be assigned to int

a = a + b + c;  // ✅  "java" + 10 + 20 = "java1020"
b = b + c + d;  // ✅  10 + 20 + 30 = 60  (no String involved)
```

---

## 4. Relational Operators (`<`, `<=`, `>`, `>=`)

### 📌 Key Points
- Applicable for **every primitive type EXCEPT `boolean`**
- **Cannot be applied to object types** (String, Thread, etc.)
- **Nesting is NOT allowed** — result of comparison is `boolean`, and `>` cannot be applied on `boolean`

```java
System.out.println(10 < 10.5);    // true   (int promoted to double)
System.out.println('a' > 100.5);  // false  ('a' = 97)
System.out.println('b' > 'a');    // true   (98 > 97)

// ❌ boolean not allowed
System.out.println(true > false);
// CE: operator > cannot be applied to boolean, boolean

// ❌ Object types not allowed
System.out.println("java" > "hello");
// CE: operator > cannot be applied to String, String

// ❌ Nesting not allowed
System.out.println(10 > 20 > 30);
// (10>20) = false → false > 30
// CE: operator > cannot be applied to boolean, int
```

---

## 5. Equality Operators (`==`, `!=`)

### 📌 Key Points
- Applicable for **every primitive type including `boolean`**
- Also applicable for **object types**
- For object references: `r1 == r2` returns `true` **only if both point to the same object** (same memory address)
- `==` is for **reference comparison / address comparison** — NOT content comparison

### Primitives

```java
System.out.println(10 == 20);        // false
System.out.println('a' == 97.0);     // true   ('a' = 97, promoted to double)
System.out.println(false == false);  // true
```

### Object References

```java
Thread t1 = new Thread();
Thread t2 = new Thread();
Thread t3 = t1;

System.out.println(t1 == t2);  // false  (different objects in memory)
System.out.println(t1 == t3);  // true   (t3 points to same object as t1)
```

### ⚠️ Type Relationship Required
To use `==` between objects, there **must be a relationship** (parent-child or same type). Otherwise → CE.

```java
Thread t = new Thread();
Object o = new Object();
String s = new String("java");

System.out.println(t == o);  // false ✅  (Thread extends Object — related)
System.out.println(o == s);  // false ✅  (String extends Object — related)
System.out.println(s == t);  // ❌ CE: incompatible types — String and Thread (no relation)
```

### null Comparisons

```java
String s = new String("java");
System.out.println(s == null);     // false  (s points to an object)

String s = null;
System.out.println(s == null);     // true
System.out.println(null == null);  // true   (null == null is always true)
```

### 📌 `==` vs `.equals()`

| | `==` | `.equals()` |
|---|:----:|:-----------:|
| Purpose | Reference comparison (same object?) | Content comparison (same value?) |
| Works on | Primitives + Objects | Objects only |

```java
String s1 = new String("java");
String s2 = new String("java");

System.out.println(s1 == s2);       // false  (different objects in heap)
System.out.println(s1.equals(s2));  // true   (same content)
```

---

## 6. instanceof Operator

### 📌 Definition
Used to **check whether a given object is of a particular type or not**.  
Syntax: `objectReference instanceof ClassName/InterfaceName`  
Very useful when dealing with collections of mixed types — check before casting.

```java
Object o = list.get(0);
if (o instanceof Student) {
    Student s = (Student) o;   // safe cast
} else if (o instanceof Customer) {
    Customer c = (Customer) o;
}
```

### Basic Examples

```java
Thread t = new Thread();
// Thread extends Object and implements Runnable

System.out.println(t instanceof Thread);    // true
System.out.println(t instanceof Object);    // true   (parent)
System.out.println(t instanceof Runnable);  // true   (implemented interface)
```

### ⚠️ Type Relationship Required
There **must be a relationship** between the object type and class being checked. Otherwise → CE: inconvertible types.

```java
Thread t = new Thread();
System.out.println(t instanceof String);
// ❌ CE: inconvertible types — found: Thread, required: String
```

### ⚠️ Checking Parent as Child → `false`

```java
Object o = new Object();
System.out.println(o instanceof String);  // false  (o is a pure Object)

Object o = new String("java");
System.out.println(o instanceof String);  // true   (o actually holds a String)
```

### ⚠️ `null instanceof X` → always `false`

```java
System.out.println(null instanceof String);  // false
System.out.println(null instanceof Object);  // false
```

---

## 7. Bitwise Operators

### 📌 `&`, `|`, `^` — applicable for both boolean AND integral types

| Operator | Name | Rule |
|:--------:|------|------|
| `&` | AND | `true` only if **both** are `true` |
| `\|` | OR | `true` if **at least one** is `true` |
| `^` | XOR | `true` only if **both are different** |

```java
// Boolean
System.out.println(true & false);  // false
System.out.println(true | false);  // true
System.out.println(true ^ false);  // true   (different)
System.out.println(true ^ true);   // false  (same)

// Integral (operates on binary bits)
// 4 = 100,  5 = 101
System.out.println(4 & 5);  // 4  →  100 & 101 = 100 = 4
System.out.println(4 | 5);  // 5  →  100 | 101 = 101 = 5
System.out.println(4 ^ 5);  // 1  →  100 ^ 101 = 001 = 1
```

---

### 📌 `~` Bitwise Complement — integral types ONLY

- Flips all bits of the number (0 → 1, 1 → 0)
- The **Most Significant Bit (MSB)** acts as sign bit: `0` = positive, `1` = negative
- Positive numbers stored directly; negative numbers stored in **2's complement form**
- **Cannot be applied to `boolean`**

```java
System.out.println(~4);     // -5
// 4  =  0000...0100
// ~4 =  1111...1011  (MSB = 1 → negative)
// 2's complement: flip bits of 4 → 1011, add 1 → 0101 = 5 → so ~4 = -5

System.out.println(~true);  // ❌ CE: operator ~ cannot be applied to boolean
```

---

### 📌 `!` Boolean Complement — boolean types ONLY

- Reverses the boolean value
- **Cannot be applied to integral types**

```java
System.out.println(!true);   // false
System.out.println(!false);  // true
System.out.println(!4);      // ❌ CE: operator ! cannot be applied to int
```

---

### 📊 Applicability Summary

| Operator | Boolean | Integral |
|:--------:|:-------:|:--------:|
| `&`, `\|`, `^` | ✅ | ✅ |
| `~` (bitwise complement) | ❌ | ✅ |
| `!` (boolean complement) | ✅ | ❌ |

---

## 8. Short Circuit Operators (`&&`, `||`)

### 📌 Definition
Short circuit operators work exactly like `&` and `|` but with **one key difference — the second argument is evaluated only when necessary**:

- `x && y` → `y` is evaluated **only if `x` is `true`**  
  (if `x` is false, result is already false — no need to check `y`)
- `x || y` → `y` is evaluated **only if `x` is `false`**  
  (if `x` is true, result is already true — no need to check `y`)

### 📊 `&` / `|` vs `&&` / `||` Comparison

| Feature | `&`, `\|` | `&&`, `\|\|` |
|---------|:---------:|:------------:|
| Both arguments always evaluated? | ✅ Always | ❌ Second is optional |
| Performance | Lower | Higher |
| Applicable for | Boolean + Integral | **Boolean only** |

### 📊 Practical Difference

```java
int x = 10, y = 15;
if (++x < 10 OP ++y > 15) { x++; } else { y++; }
```

| Operator | x | y | Why |
|:--------:|:-:|:-:|-----|
| `&` | 11 | 17 | Both sides always evaluated → condition true (`false & true = false`) → else → `y++` |
| `\|` | 12 | 16 | Both sides evaluated → condition true (`false | true = true`) → `x++` |
| `&&` | 11 | 16 | `++x<10` is false → second operand **NOT evaluated** → y stays 16 → else → `y++` |
| `\|\|` | 12 | 16 | `++x<10` is false → second IS evaluated → `++y>15` true → `x++` |

### Short Circuit Prevents Exceptions

```java
int x = 10;
if (++x < 10 && (x / 0) > 10) {  // ++x<10 is false → x/0 is NEVER evaluated
    System.out.println("Hello");
} else {
    System.out.println("Hi");     // Output: Hi  (no ArithmeticException!)
}
```

> 💡 Use `&&` and `||` for better performance and to avoid unnecessary operations or exceptions.

---

## 9. Type Cast Operator

There are **2 types** of type casting in Java:

### 📌 Implicit Type Casting (Widening / Upcasting)
- Performed **automatically by the compiler**
- Done when assigning a **lower datatype value → higher datatype variable**
- Also called **Widening** or **Upcasting**
- **No loss of information**

```
byte → short → int → long → float → double
               ↑
             char
```

```java
int    x = 'a';  System.out.println(x);  // 97    (char → int, auto)
double d = 10;   System.out.println(d);  // 10.0  (int → double, auto)
```

---

### 📌 Explicit Type Casting (Narrowing / Downcasting)
- Performed **by the programmer** using cast syntax `(type)`
- Done when assigning a **higher datatype value → smaller datatype variable**
- Also called **Narrowing** or **Downcasting**
- **May cause loss of information** — MSBs (Most Significant Bits) are lost; only LSBs kept

```java
int  x = 130;
byte b = x;          // ❌ CE: possible loss of precision (found: int, required: byte)
byte b = (byte) x;   // ✅  → -126  (130 doesn't fit in byte; MSBs dropped)

int   x = 150;
short s = (short) x; // ✅  → 150  (fits in short range)
byte  b = (byte)  x; // ✅  → -106 (doesn't fit; MSBs dropped)
```

### ⚠️ Floating Point → Integral (decimal digits lost)
When casting floating point to integral, the **digits after the decimal point are always lost**.

```java
double d = 130.456;
int  x = (int)  d;   System.out.println(x);  // 130   (decimal dropped)
byte b = (byte) d;   System.out.println(b);  // -126  (decimal dropped + MSBs lost)

float f = 150.1234f;
int   i = (int) f;   System.out.println(i);  // 150
```

---

## 10. Assignment Operator

There are **3 types** of assignment operators:

### 📌 1. Simple Assignment
```java
int x = 10;  // ✅
```

### 📌 2. Chained Assignment
Assigns the **same value to multiple variables** at once. Evaluated **right to left**.

```java
int a, b, c, d;
a = b = c = d = 20;  // ✅  all variables get 20
System.out.println(a + "---" + b + "---" + c + "---" + d); // 20---20---20---20
```

> ⚠️ Chained assignment **at the time of declaration** is NOT allowed — the other variables don't exist yet.

```java
int a = b = c = d = 20;
// ❌ CE: cannot find symbol — b, c, d not yet declared
```

### 📌 3. Compound Assignment
Mixes an operator with `=`. Available: `+=` `-=` `*=` `/=` `%=` `&=` `|=` `^=` `>>=` `>>>=` `<<=`

**Key feature:** Internal type casting is performed **automatically by the compiler** — same as `++`/`--`.

```java
int a = 10;
a += 20;   // same as a = a + 20 → a = 30

byte b = 10;
b = b + 1;   // ❌ CE: possible loss of precision (b+1 is int)
b += 1;      // ✅  auto-casts: b = (byte)(b + 1) = 11
b++;         // ✅  same behavior

byte b = 127;
b += 3;
System.out.println(b);  // -126  (overflow → wraps around due to auto-cast)
```

### Tricky Compound Chain Example

```java
int a, b, c, d;
a = b = c = d = 20;
a += b -= c *= d /= 2;
// d /= 2  →  d = 20/2  = 10
// c *= 10 →  c = 20*10 = 200
// b -= 200 → b = 20-200 = -180
// a += -180 → a = 20+(-180) = -160
System.out.println(a + "---" + b + "---" + c + "---" + d);
// Output: -160---(-180)---200---10
```

---

## 11. Conditional Operator (`? :`)

### 📌 Definition
- The **only ternary operator** in Java
- Syntax: `condition ? valueIfTrue : valueIfFalse`
- **Nesting is possible** — another `?:` can be used as one of the values

```java
int x = (10 > 20) ? 30 : 40;
System.out.println(x);  // 40  (condition false → takes second value)

// Nested
int x = (10 > 20) ? 30 : ((40 > 50) ? 60 : 70);
System.out.println(x);  // 70
```

### ⚠️ Type Compatibility
The result of `?:` follows normal type rules. If result is `int` and assigned to `byte` → CE.

```java
int a = 10, b = 20;
byte c1 = (a > b) ? 30 : 40;  // ❌ CE: possible loss of precision (result is int)
int  c2 = (a > b) ? 30 : 40;  // ✅
```

---

## 12. new Operator

### 📌 Key Points
- Used to **create objects** in Java — allocates memory in the **heap**
- Every time `new` is called, a **fresh new object** is created
- There is **no `delete` operator** in Java — destruction of useless objects is the responsibility of the **Garbage Collector (GC)**
- The `[]` operator is used to **declare and create arrays**

```java
Test t   = new Test();    // ✅  creates object
int[] a  = new int[5];    // ✅  creates array using []
```

---

## 13. Operator Precedence

> Operators with **higher priority** are evaluated first.

| Priority | Category | Operators |
|:--------:|----------|-----------|
| 1 ← highest | Unary | `[]`, `x++`, `x--`, `++x`, `--x`, `~`, `!`, `new`, `(type)` |
| 2 | Arithmetic (×÷) | `*`, `/`, `%` |
| 3 | Arithmetic (+−) | `+`, `-` |
| 4 | Shift | `>>`, `>>>`, `<<` |
| 5 | Comparison | `<`, `<=`, `>`, `>=`, `instanceof` |
| 6 | Equality | `==`, `!=` |
| 7 | Bitwise | `&`, `^`, `\|` |
| 8 | Short circuit | `&&`, `\|\|` |
| 9 | Conditional | `?:` |
| 10 ← lowest | Assignment | `=`, `+=`, `-=`, `*=`, `/=`, `%=` ... |

---

## 14. Evaluation Order of Operands

### 📌 Key Rule
Java has **no precedence for operands** — only for operators.  
**All operands are evaluated from left to right** before any operator is applied.

```java
// m1() prints and returns its argument
// Even though * has higher precedence than +,
// ALL operands (1,2,3,4,5,6) are printed first (left to right)
System.out.println(m1(1) + m1(2) * m1(3) / m1(4) * m1(5) + m1(6));
// Output lines: 1  2  3  4  5  6
// Then result:  1 + 2*3/4*5 + 6 = 1 + 6/4*5 + 6 = 1+1*5+6 = 12
```

### ⚠️ Tricky: `x = x++`

```java
int x = 10;
x = x++;
System.out.println(x);  // 10  ← NOT 11!
```

Steps:
1. Old value of `x` (10) is **saved** for the assignment
2. `x` is **incremented** to 11 (post-increment side effect)
3. The **saved old value (10)** is assigned back to `x`
4. Final result: `x = 10`

### Compound Increment Expression

```java
int i = 1;
i += ++i + i++ + ++i + i++;
// i = i  +  ++i  +  i++  +  ++i  +  i++
// i = 1  +   2   +   2   +   4   +   4   = 13
System.out.println(i);  // 13
```

---

## 15. new vs newInstance()

### 📌 Definition
- **`new`** — operator to create objects when the **class name is known at compile time**
- **`newInstance()`** — method in `Class` to create objects when the **class name is available dynamically at runtime**

```java
// new — class name hardcoded at compile time
Test t = new Test();

// newInstance() — class name passed dynamically at runtime
Object o = Class.forName(args[0]).newInstance();
System.out.println(o.getClass().getName());
```

### 📊 Comparison Table

| Feature | `new` | `newInstance()` |
|---------|:-----:|:---------------:|
| Type | Operator | Method in `Class` |
| Class name available | Compile time | Runtime (dynamically) |
| No-arg constructor required? | ❌ Not required | ✅ Must exist → else `InstantiationException` |
| If `.class` file missing | `NoClassDefFoundError` (unchecked) | `ClassNotFoundException` (checked) |

---

## 16. instanceof vs isInstance()

### 📌 Definition
- **`instanceof`** — operator, used when **type is known at compile time**
- **`isInstance()`** — method in `Class`, used when **type is available dynamically at runtime**

```java
// instanceof — type known at compile time
String s = new String("java");
System.out.println(s instanceof Object);  // true

// isInstance() — type provided dynamically
// java Test Test   → true
// java Test String → false
// java Test Object → true
Class.forName(args[0]).isInstance(t);
```

### 📊 Comparison Table

| Feature | `instanceof` | `isInstance()` |
|---------|:------------:|:--------------:|
| Type | Operator | Method in `Class` |
| When type is known | Compile time | Runtime (dynamic) |
| Syntax | `obj instanceof ClassName` | `Class.forName(name).isInstance(obj)` |

---

## 17. ClassNotFoundException vs NoClassDefFoundError

### 📌 Definition
Both occur when a `.class` file is **missing at runtime** — but triggered in different ways:

- **`NoClassDefFoundError`** → class name is **hard-coded** (using `new`) → **Unchecked** error
- **`ClassNotFoundException`** → class name is **provided dynamically** (`Class.forName()`) → **Checked** exception — must handle

```java
// Hard-coded → NoClassDefFoundError (unchecked)
Test t = new Test();
// If Test.class absent at runtime → NoClassDefFoundError

// Dynamic → ClassNotFoundException (checked)
Object o = Class.forName("Test").newInstance();
// If Test.class absent at runtime → ClassNotFoundException
```

### 📊 Comparison Table

| Feature | `NoClassDefFoundError` | `ClassNotFoundException` |
|---------|:----------------------:|:------------------------:|
| Triggered by | Hard-coded `new ClassName()` | Dynamic `Class.forName("name")` |
| Exception type | **Unchecked** Error | **Checked** Exception |
| Must handle? | ❌ Not required | ✅ Must catch or declare `throws` |

---

## 📝 Quick Recap

| Topic | Key Rule |
|-------|----------|
| `b++` vs `b=b+1` | `b++` auto-casts; `b=b+1` does NOT |
| Arithmetic result | `max(int, type of a, type of b)` — minimum is always `int` |
| Integral `/0` | `ArithmeticException` |
| Float `/0.0` | `Infinity` or `NaN` — no exception |
| `NaN != NaN` | `true` — NaN is never equal to anything |
| `==` on objects | Reference/address check — NOT content |
| `.equals()` | Content comparison |
| `null instanceof X` | Always `false` |
| `~` operator | Integral only |
| `!` operator | Boolean only |
| `&`, `\|`, `^` | Both boolean and integral |
| `&&`, `\|\|` | Boolean only — skip second operand when possible |
| Operands evaluation | Always **left to right** — regardless of operator precedence |
| `x = x++` | Result is original value — post-increment save/restore trap |
| `new` vs `newInstance()` | `new` = compile time; `newInstance()` = runtime (needs no-arg constructor) |
| `instanceof` vs `isInstance()` | `instanceof` = compile time; `isInstance()` = runtime |
