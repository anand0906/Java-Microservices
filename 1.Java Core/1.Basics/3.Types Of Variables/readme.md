# ☕ Java — Types of Variables

> **Java Version Coverage:** Java 1.0+  
> Covers: Primitive vs Reference · Instance · Static · Local · Uninitialized Arrays

---

## 📑 Index

1. [Division 1 — By Value Type](#1-division-1--by-value-type)
2. [Division 2 — By Behavior & Position](#2-division-2--by-behavior--position)
3. [Instance Variables](#3-instance-variables)
4. [Static Variables](#4-static-variables)
5. [Local Variables](#5-local-variables)
6. [Uninitialized Arrays](#6-uninitialized-arrays)
7. [Summary](#7-summary)

---

## 1. Division 1 — By Value Type

| Type | Description | Example |
|------|-------------|---------|
| **Primitive variable** | Holds a primitive value directly | `int x = 10;` |
| **Reference variable** | Holds a reference (address) to an object | `Student s = new Student();` |

---

## 2. Division 2 — By Behavior & Position

```
Variables
├── Instance Variables   → one copy per object
├── Static Variables     → one copy per class
└── Local Variables      → one copy per thread / method call
```

---

## 3. Instance Variables

> Value **varies from object to object** → separate copy created for each object.

- Declared **inside class**, but **outside** any method, block, or constructor
- Stored in the **Heap** (as part of the object)
- Created when object is created, destroyed when object is destroyed
- JVM provides **default values** — no explicit initialization needed

### Access Rules

```java
class Test {
    int i = 10;

    public static void main(String[] args) {
        // System.out.println(i);   // ❌ CE: non-static variable i cannot be
                                    //        referenced from a static context
        Test t = new Test();
        System.out.println(t.i);    // ✅ 10  — access via object reference in static area
        t.methodOne();
    }

    public void methodOne() {
        System.out.println(i);      // ✅ 10  — direct access in instance area
    }
}
```

### Default Value Example

```java
class Test {
    boolean b;   // no initialization

    public static void main(String[] args) {
        Test t = new Test();
        System.out.println(t.b);   // false  (JVM default)
    }
}
```

> 💡 Also called: **object-level variables** or **attributes**.

---

## 4. Static Variables

> Value is **same across all objects** → only **one copy** per class, shared by all objects.

- Declared **inside class** with `static` keyword, outside any method/block/constructor
- Stored in the **Method Area**
- Created at **class loading**, destroyed at **class unloading**
- JVM provides **default values** — no explicit initialization needed
- Can be accessed from both **instance and static areas**

### Class Loading Lifecycle

```
java Test
  1. Start JVM
  2. Create & start Main Thread
  3. Locate Test.class
  4. Load Test.class        ← static variable CREATED here
  5. Execute main()
  6. Unload Test.class      ← static variable DESTROYED here
  7. Terminate Main Thread
  8. Shutdown JVM
```

### Access — 3 ways (class name recommended)

```java
class Test {
    static int i = 10;

    public static void main(String[] args) {
        Test t = new Test();
        System.out.println(t.i);     // ✅ 10  — via object reference (not recommended)
        System.out.println(Test.i);  // ✅ 10  — via class name (recommended)
        System.out.println(i);       // ✅ 10  — direct (within same class)
    }
}
```

### Default Value Example

```java
class Test {
    static String s;   // no initialization

    public static void main(String[] args) {
        System.out.println(s);   // null  (JVM default)
    }
}
```

### Instance vs Static — Key Difference

```java
class Test {
    int x = 10;       // instance — one copy per object
    static int y = 20; // static  — one copy for entire class

    public static void main(String[] args) {
        Test t1 = new Test();
        t1.x = 888;   // changes only t1's copy of x
        t1.y = 999;   // changes the single shared copy of y

        Test t2 = new Test();
        System.out.println(t2.x + "----" + t2.y);
        // Output: 10----999
        // t2.x is fresh (10), but t2.y shares the change (999)
    }
}
```

---

## 5. Local Variables

> Declared **inside** a method, block, or constructor to meet **temporary** needs.

- Stored in the **Stack**
- Created when the block starts, destroyed when the block ends
- **JVM does NOT provide default values** → must initialize before use
- Only modifier allowed: **`final`**

### Scope Example — variable out of scope

```java
public static void main(String[] args) {
    int i = 0;
    for (int j = 0; j < 3; j++) {
        i = i + j;
    }
    System.out.println(i);   // ✅ 3
    System.out.println(j);   // ❌ CE: cannot find symbol  (j is out of scope)
}
```

### Scope Example — try/catch

```java
try {
    int i = Integer.parseInt("ten");
} catch (NumberFormatException e) {
    System.out.println(i);   // ❌ CE: cannot find symbol  (i declared inside try)
}
```

### Must initialize before use

```java
// ❌ Risky — x may not be initialized
public static void main(String[] args) {
    int x;
    if (args.length > 0) {
        x = 10;
    }
    System.out.println(x);   // ❌ CE: variable x might not have been initialized
}

// ✅ Safe — both branches initialize x
public static void main(String[] args) {
    int x;
    if (args.length > 0) {
        x = 10;
    } else {
        x = 20;
    }
    System.out.println(x);   // ✅
}
// java Test a    → 10
// java Test a b  → 10
// java Test      → 20
```

> ⚠️ Do NOT initialize local variables inside logical blocks (`if`, `for`, etc.) —  
> there's no guarantee they'll execute. **Initialize at declaration instead.**

### Only `final` modifier is allowed

```java
public static void main(String[] args) {
    public    int x = 10;   // ❌ CE: illegal start of expression
    private   int x = 10;   // ❌ CE: illegal start of expression
    protected int x = 10;   // ❌ CE: illegal start of expression
    static    int x = 10;   // ❌ CE: illegal start of expression
    volatile  int x = 10;   // ❌ CE: illegal start of expression
    transient int x = 10;   // ❌ CE: illegal start of expression

    final int x = 10;       // ✅ only valid modifier for local variables
}
```

---

## 6. Uninitialized Arrays

> Once an array object is **created**, all its elements are always auto-initialized.  
> But the **array reference itself** follows variable rules.

### Instance Level

```java
class Test {
    int[] a;   // instance array — reference gets default null

    public static void main(String[] args) {
        Test t = new Test();
        System.out.println(t.a);     // null    (reference not yet pointing to array)
        System.out.println(t.a[0]);  // ❌ RE: NullPointerException
    }
}

// After construction:
int[] a = new int[3];
System.out.println(obj.a);     // [I@3e25a5
System.out.println(obj.a[0]);  // 0  (default int)
```

### Static Level

```java
static int[] a;
System.out.println(a);     // null
System.out.println(a[0]);  // ❌ RE: NullPointerException

static int[] a = new int[3];
System.out.println(a);     // [I@3e25a5
System.out.println(a[0]);  // 0
```

### Local Level

```java
int[] a;
System.out.println(a);     // ❌ CE: variable a might not have been initialized

int[] a = new int[3];
System.out.println(a);     // [I@3e25a5
System.out.println(a[0]);  // 0
```

### Quick Comparison

| Level | Uninitialized reference | After `new int[3]` → element |
|-------|------------------------|------------------------------|
| Instance | `null` (JVM default) | `0` |
| Static | `null` (JVM default) | `0` |
| Local | ❌ CE: not initialized | `0` |

> 💡 **Array elements are always auto-initialized** (0, false, null, etc.) regardless of where the array is declared — once the array object is created with `new`.

---

## 7. Summary

### Variable Comparison Table

| Feature | Instance | Static | Local |
|---------|----------|--------|-------|
| Where declared | Inside class, outside method | Inside class with `static` | Inside method/block/constructor |
| Stored in | Heap | Method Area | Stack |
| Created | At object creation | At class loading | At block execution |
| Destroyed | At object destruction | At class unloading | At block completion |
| Default value (JVM) | ✅ Yes | ✅ Yes | ❌ No — must initialize manually |
| One copy per | Object | Class | Thread |
| Thread safe? | ❌ No | ❌ No | ✅ Yes |
| Allowed modifiers | All | All | `final` only |

### All Possible Variable Combinations

```
Every variable in Java is:
  ├── Primitive  ──┬── Instance
  │                ├── Static
  │                └── Local
  │
  └── Reference  ──┬── Instance
                   ├── Static
                   └── Local
```

> 💡 **Key Rules to Remember:**
> - Instance/Static → JVM gives default values ✅
> - Local → You MUST initialize before use ❌
> - Local variables → only `final` modifier allowed
> - Static variables → class name access is recommended
> - Instance variables → can't access directly from `static` context
