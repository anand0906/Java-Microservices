# ☕ Java — Arrays

> **Java Version Coverage:** Java 1.0+  
> Covers: Declaration · Construction · Initialization · length · Anonymous Arrays · Element & Variable Assignments

---

## 📑 Index

1. [Introduction](#1-introduction)
2. [Array Declaration](#2-array-declaration)
3. [Array Construction](#3-array-construction)
4. [Array Initialization](#4-array-initialization)
5. [Declaration + Construction + Initialization in One Line](#5-declaration--construction--initialization-in-one-line)
6. [length vs length()](#6-length-vs-length)
7. [Anonymous Arrays](#7-anonymous-arrays)
8. [Array Element Assignments](#8-array-element-assignments)
9. [Array Variable Assignments](#9-array-variable-assignments)

---

## 1. Introduction

> An array is an **indexed collection** of a **fixed number** of **homogeneous** data elements.

| ✅ Advantage | ❌ Disadvantage |
|-------------|----------------|
| Represent multiple values with one name → better readability | **Fixed size** — can't grow or shrink after creation |
| — | Must know size in advance (not always possible) |

> 💡 To overcome the fixed-size problem → use **Collections** (e.g., `ArrayList`).

---

## 2. Array Declaration

### Single Dimensional

```java
int[] a;    // ✅ recommended — type and name are clearly separated
int []a;    // ✅ valid
int a[];    // ✅ valid
```

> ⚠️ **Never specify size at declaration** — that causes a compile error.

```java
int[] a;      // ✅
int[5] a;     // ❌ CE: illegal start of expression
```

---

### Two Dimensional (6 valid ways)

```java
int[][] a;
int [][]a;
int a[][];
int[] []a;
int[] a[];
int []a[];
// All 6 are valid ✅
```

---

### Three Dimensional (10 valid ways)

```java
int[][][] a;   int [][][]a;   int a[][][];
int[] [][]a;   int[] a[][];   int[] []a[];
int[][] []a;   int[][] a[];   int []a[][];   int [][]a[];
// All 10 are valid ✅
```

---

### ⚠️ Mixed Declaration Tricky Cases

```java
int[] a1, b1;      // ✅  a1 = 1D,  b1 = 1D
int[] a2[], b2;    // ✅  a2 = 2D,  b2 = 1D
int[] []a3, b3;    // ✅  a3 = 2D,  b3 = 2D
int[] a, []b;      // ❌ CE: illegal start of expression
```

> 💡 Dimension before the variable name applies **only to the first variable** in the declaration.

---

## 3. Array Construction

> Every array in Java is an **object** → created using `new`.

```java
int[] a = new int[3];   // ✅
```

### Rules

**Rule 1 — Size is mandatory:**

```java
int[] a = new int[3];   // ✅
int[] a = new int[];    // ❌ CE: array dimension missing
```

**Rule 2 — Size zero is valid:**

```java
int[] a = new int[0];
System.out.println(a.length); // 0  ✅
```

**Rule 3 — Negative size → Runtime Exception:**

```java
int[] a = new int[-3];  // ❌ RE: NegativeArraySizeException
```

**Rule 4 — Allowed size types: `byte`, `short`, `char`, `int` only:**

```java
int[] a = new int['a'];  // ✅ char → 97
byte  b = 10;
int[] a = new int[b];    // ✅

int[] a = new int[10l];   // ❌ CE: possible loss of precision (long not allowed)
int[] a = new int[10.5];  // ❌ CE: possible loss of precision (float not allowed)
```

**Rule 5 — Max size = `Integer.MAX_VALUE` (2,147,483,647):**

```java
int[] a = new int[2147483647];   // ✅ (may throw OutOfMemoryError at runtime)
int[] a = new int[2147483648];   // ❌ CE: integer number too large
```

---

### Multi-Dimensional Array Construction

> Java implements multidimensional arrays as **arrays of arrays** (not a matrix) → better memory utilization.

```java
// 2D — jagged array (rows of different sizes)
int[][] a = new int[2][];
a[0] = new int[3];   // row 0 has 3 elements
a[1] = new int[2];   // row 1 has 2 elements
```

**Valid / Invalid 2D & 3D constructions:**

```java
int[]   a = new int[];         // ❌ CE: array dimension missing
int[][] a = new int[3][4];     // ✅ 3 rows, 4 cols
int[][] a = new int[3][];      // ✅ 3 rows, sizes defined later
int[][] a = new int[][4];      // ❌ CE: ']' expected (must specify from left)
int[][][] a = new int[3][4][5]; // ✅
int[][][] a = new int[3][4][];  // ✅
int[][][] a = new int[3][][5];  // ❌ CE: ']' expected
```

---

## 4. Array Initialization

> All array elements are **automatically initialized** to their default values when created.

| Type | Default Value |
|------|--------------|
| `int`, `byte`, `short`, `long` | `0` |
| `float`, `double` | `0.0` |
| `boolean` | `false` |
| `char` | `0` (blank space) |
| Object reference | `null` |

---

### Example 1 — 1D array

```java
int[] a = new int[3];
System.out.println(a);    // [I@3e25a5  (classname@hashcode)
System.out.println(a[0]); // 0  (default)
```

### Example 2 — 2D array (fully constructed)

```java
int[][] a = new int[2][3];
System.out.println(a);       // [[I@3e25a5
System.out.println(a[0]);    // [I@19821f
System.out.println(a[0][0]); // 0
```

### Example 3 — 2D array (partially constructed) ⚠️

```java
int[][] a = new int[2][];
System.out.println(a);       // [[I@3e25a5
System.out.println(a[0]);    // null  (not yet constructed)
System.out.println(a[0][0]); // ❌ RE: NullPointerException
```

### Example 4 — Index out of bounds ⚠️

```java
int[] a = new int[4];
a[0] = 10;
a[3] = 40;  // ✅ last valid index

a[4]  = 50; // ❌ RE: ArrayIndexOutOfBoundsException: 4
a[-4] = 60; // ❌ RE: ArrayIndexOutOfBoundsException: -4
```

> 💡 Valid index range: **0 to length-1**. Any other index → `ArrayIndexOutOfBoundsException`.

---

## 5. Declaration + Construction + Initialization in One Line

```java
// 1D
char[]   ch = {'a', 'e', 'i', 'o', 'u'};               // ✅
String[] s  = {"Java", "Python", "C++"};                // ✅

// 2D
int[][] a = {{10, 20, 30}, {40, 50}};                   // ✅ jagged

// 3D
int[][][] a = {{{10,20,30},{40,50}},{{60},{70,80},{90,100,110}}};
```

**Accessing 3D elements:**

```java
int[][][] a = {{{10,20,30},{40,50}},{{60},{70,80},{90,100,110}}};

System.out.println(a[0][1][1]); // 50   ✅
System.out.println(a[1][2][1]); // 100  ✅
System.out.println(a[1][2][2]); // 110  ✅
System.out.println(a[1][1][1]); // 80   ✅

System.out.println(a[1][0][2]); // ❌ RE: ArrayIndexOutOfBoundsException: 2
System.out.println(a[2][1][0]); // ❌ RE: ArrayIndexOutOfBoundsException: 2
```

> ⚠️ This shortcut requires **declaration + construction + initialization all in one line**.  
> Splitting into multiple lines → Compile Error.

---

## 6. length vs length()

| | `length` | `length()` |
|---|---------|-----------|
| Type | `final` variable | `final` method |
| Applies to | **Arrays** only | **String** objects only |
| Returns | Size of the array | Number of characters |

```java
// Array → use .length (variable)
int[] x = new int[3];
System.out.println(x.length);   // 3  ✅
System.out.println(x.length()); // ❌ CE: cannot find symbol

// String → use .length() (method)
String s = "Java";
System.out.println(s.length());  // 4  ✅
System.out.println(s.length);    // ❌ CE: cannot find symbol
```

### `length` in Multi-Dimensional Arrays

> `length` gives **only the base (row) size**, not the total element count.

```java
int[][] a = new int[6][3];
System.out.println(a.length);    // 6  (rows)
System.out.println(a[0].length); // 3  (cols in row 0)
```

> 💡 No direct way to get total size of a multi-dimensional array.  
> Indirect: `a[0].length + a[1].length + a[2].length + ...`

---

## 7. Anonymous Arrays

> An array **without a name** — created for **one-time / instant use**.

```java
new int[]{10, 20, 30, 40};        // ✅ 1D anonymous
new int[][]{{10,20},{30,40}};     // ✅ 2D anonymous

new int[3]{10, 20, 30};           // ❌ CE: ';' expected (no size in anonymous)
```

**Giving a name makes it a regular array:**

```java
int[] a = new int[]{10, 20, 30, 40};  // ✅ no longer anonymous
```

**Best use case — pass directly into a method without declaring a variable:**

```java
class Test {
    public static void main(String[] args) {
        System.out.println(sum(new int[]{10, 20, 30, 40})); // 100
    }

    public static int sum(int[] x) {
        int total = 0;
        for (int x1 : x) total += x1;
        return total;
    }
}
```

> 💡 The array is used only once inside `sum()` — anonymous array is perfect here.

---

## 8. Array Element Assignments

### Case 1 — Primitive Arrays

> Any type that can be **widened (promoted)** to the declared type is allowed.

```java
int[] a = new int[5];
a[0] = 97;        // ✅ int
a[1] = 'a';       // ✅ char → promoted to int
byte b = 10;
a[2] = b;         // ✅ byte → promoted to int
short s = 20;
a[3] = s;         // ✅ short → promoted to int

a[4] = 10L;       // ❌ CE: possible loss of precision (long → int not allowed)
```

---

### Case 2 — Object Type Arrays

> Declared type **or its child class objects** allowed.

```java
Object[] a = new Object[3];
a[0] = new Integer(10);      // ✅ child of Object
a[1] = new Object();         // ✅ same type
a[2] = new String("Java");   // ✅ child of Object

Number[] n = new Number[3];
n[0] = new Integer(10);      // ✅ Integer extends Number
n[1] = new Double(10.5);     // ✅ Double extends Number
n[2] = new String("Java");   // ❌ CE: incompatible types (String is not a Number)
```

---

### Case 3 — Interface Type Arrays

> Only **implemented class objects** allowed.

```java
Runnable[] r = new Runnable[2];
r[0] = new Thread();            // ✅ Thread implements Runnable
r[1] = new String("Java");      // ❌ CE: incompatible types
```

---

### Summary Table

| Array Type | Allowed Element Types |
|-----------|----------------------|
| Primitive array | Any type promotable to declared type |
| Object type array | Declared type or its child class objects |
| Interface type array | Implemented class objects |
| Abstract class type array | Child class objects |

---

## 9. Array Variable Assignments

### Case 1 — Element promotion ≠ Array promotion

```java
int[]  a  = {10, 20, 30};
char[] ch = {'a', 'b', 'c'};

int[] b = a;    // ✅ same type
int[] c = ch;   // ❌ CE: incompatible types
                // char can be promoted to int, but char[] CANNOT be assigned to int[]
```

> ⚠️ **Element-level widening does NOT apply at the array level.**

---

### Case 2 — Reassigning arrays (only reference is copied, not elements)

```java
int[] a = {10, 20, 30, 40, 50};
int[] b = {80, 90};

a = b;   // ✅ 'a' now points to b's array (size doesn't matter, only type)
b = a;   // ✅
```

> 💡 Only the **reference** changes. Internal elements are not copied. Sizes don't matter.

---

### Case 3 — Dimensions must match

```java
int[][] a = new int[3][];

a[0] = new int[4];      // ✅ 1D assigned to 1D slot
a[0] = new int[4][5];   // ❌ CE: incompatible types (2D into 1D slot)
a[0] = 10;              // ❌ CE: incompatible types (int into array slot)
```

---

### Object Array Assignment (child → parent allowed)

```java
String[] s = {"A", "B"};
Object[] o = s;   // ✅ String[] can be assigned to Object[] (child → parent)
```

---

### Practical Example — args reassignment

```java
class Test {
    public static void main(String[] args) {
        String[] argh = {"A", "B"};
        args = argh;
        System.out.println(args.length); // 2

        // ❌ Wrong — i <= args.length causes AIOOBE on last iteration
        for (int i = 0; i <= args.length; i++) {
            System.out.println(args[i]); // RE: ArrayIndexOutOfBoundsException: 2
        }

        // ✅ Correct — use i < args.length
        for (int i = 0; i < args.length; i++) {
            System.out.println(args[i]); // A, B
        }

        // ✅ Best — use enhanced for loop
        for (String s : args) {
            System.out.println(s); // A, B
        }
    }
}
```

---

### Assignment Rules — Summary

| What must match | Important? |
|----------------|-----------|
| Type | ✅ Must match |
| Dimensions | ✅ Must match |
| Size | ❌ Doesn't matter |

---

> 💡 **Key Takeaways:**
> - Arrays are **fixed-size objects** — use Collections for dynamic sizing.
> - Always use `i < arr.length` (not `<=`) in loops.
> - `length` for arrays, `length()` for Strings — never mix them up!
