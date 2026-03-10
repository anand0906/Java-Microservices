# ☕ Java — Var-Arg Methods (Variable Arguments)

> **Introduced in:** Java 1.5  
> Covers: Syntax · Rules · General vs Var-arg · Array vs Var-arg

---

## 📑 Index

1. [Introduction](#1-introduction)
2. [Syntax & Basic Example](#2-syntax--basic-example)
3. [Internally — How It Works](#3-internally--how-it-works)
4. [Rules & Cases](#4-rules--cases)
5. [General Method vs Var-Arg — Priority](#5-general-method-vs-var-arg--priority)
6. [Single Dimensional Array vs Var-Arg](#6-single-dimensional-array-vs-var-arg)

---

## 1. Introduction

| Before Java 1.5 | From Java 1.5 |
|----------------|--------------|
| Can't declare method with variable no. of arguments | Can declare **var-arg methods** (`int... x`) |
| Change in args → must write a new method | One method handles **any number of arguments** |
| More code, less readable | Cleaner and more readable |

---

## 2. Syntax & Basic Example

```java
// Declaration
public static void methodOne(int... x) { }

// Can be called with 0, 1, or many arguments
methodOne();
methodOne(10);
methodOne(10, 20, 30);
```

```java
class Test {
    public static void methodOne(int... x) {
        System.out.println("var-arg method");
    }

    public static void main(String[] args) {
        methodOne();           // var-arg method
        methodOne(10);         // var-arg method
        methodOne(10, 20, 30); // var-arg method
    }
}
```

---

## 3. Internally — How It Works

> Var-arg parameter is implemented internally as a **single dimensional array**.  
> So you can use index or enhanced for-loop inside the method.

### Using index (`x[i]`)

```java
public static void sum(int... x) {
    int total = 0;
    for (int i = 0; i < x.length; i++) {
        total = total + x[i];
    }
    System.out.println("The sum: " + total);
}
```

### Using enhanced for-loop (cleaner ✅)

```java
public static void sum(int... x) {
    int total = 0;
    for (int x1 : x) {
        total = total + x1;
    }
    System.out.println("The sum: " + total);
}
```

```java
sum();             // The sum: 0
sum(10);           // The sum: 10
sum(10, 20);       // The sum: 30
sum(10, 20, 30, 40); // The sum: 100
```

---

## 4. Rules & Cases

### ✅ Valid Var-Arg Declarations

```java
methodOne(int... x)    // ✅
methodOne(int ...x)    // ✅
methodOne(int...x)     // ✅

methodOne(int x...)    // ❌ CE: dots must come after type
methodOne(int. ..x)    // ❌ CE: invalid syntax
methodOne(int .x..)    // ❌ CE: invalid syntax
```

---

### Mixing Var-Arg with General Parameters

```java
methodOne(int a, int... b)      // ✅ general first, var-arg last
methodOne(String s, int... x)   // ✅

methodOne(int... a, int b)      // ❌ CE: var-arg must be last parameter
```

> ⚠️ **Var-arg parameter must always be the last parameter.**

---

### Only One Var-Arg Per Method

```java
methodOne(int... a, int... b)   // ❌ CE: cannot have more than one var-arg
```

---

### Passing Array as Var-Arg Argument

```java
class Test {
    public static void methodOne(int... x) {
        System.out.println("var-arg method");
    }

    public static void main(String[] args) {
        methodOne(new int[]{10, 20, 30});  // ✅ array treated as var-arg
    }
}
// Output: var-arg method
```

---

### Can't Declare Both `int[]` and `int...` — Ambiguous

```java
class Test {
    public void methodOne(int[] i) { }
    public void methodOne(int... i) { }
    // ❌ CE: cannot declare both methodOne(int[]) and methodOne(int...) in Test
}
```

> They are internally the same — compiler sees it as a duplicate method.

---

## 5. General Method vs Var-Arg — Priority

> **Var-arg method always gets the LEAST priority.**  
> If any other method matches, that gets called first.  
> (Same concept as `default` in a `switch`.)

```java
class Test {
    public static void methodOne(int i) {
        System.out.println("general method");
    }

    public static void methodOne(int... i) {
        System.out.println("var-arg method");
    }

    public static void main(String[] args) {
        methodOne();        // var-arg method  (no general match)
        methodOne(10, 20);  // var-arg method  (no general match)
        methodOne(10);      // general method  (exact match → general wins)
    }
}
```

---

## 6. Single Dimensional Array vs Var-Arg

### Rule 1 — Array CAN be replaced with var-arg ✅

```java
// Normal main
public static void main(String[] args) { }

// Can be replaced with var-arg
public static void main(String... args) {
    System.out.println("var-arg main");   // ✅ works perfectly
}
```

### Rule 2 — Var-arg CANNOT be replaced with array ❌

```java
methodOne(int... x)   // can NOT replace this with int[] in all cases
```

---

### Var-Arg of Arrays — `int[]... x`

```java
// methodOne(int... x)   → x becomes int[]   (1D array)
// methodOne(int[]... x) → x becomes int[][] (2D array)
```

```java
class Test {
    public static void methodOne(int[]... x) {
        for (int[] a : x) {
            System.out.println(a[0]);  // first element of each array
        }
    }

    public static void main(String[] args) {
        int[] l = {10, 20, 30};
        int[] m = {40, 50};
        methodOne(l, m);
    }
}
// Output:
// 10
// 40
```

---

## Summary

| Rule | Details |
|------|---------|
| Syntax | `type... varName` — dots after type |
| Position | Var-arg must be **last** parameter |
| Count | Only **one** var-arg per method |
| Internally | Treated as a **1D array** |
| Priority | **Lowest** — general method wins on exact match |
| With array | Can pass array as var-arg ✅, but can't declare both `int[]` and `int...` ❌ |
| Replace array | Array → var-arg ✅ &nbsp;&nbsp; Var-arg → array ❌ |

> 💡 Think of `int... x` as syntactic sugar for `int[] x` with the added benefit of calling without manually creating an array.
