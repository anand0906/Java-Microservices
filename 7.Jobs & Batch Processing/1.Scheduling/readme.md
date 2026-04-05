# 🕐 Spring Boot Scheduling 

---

## 📌 Table of Contents

1. [What is Scheduling? Why Use It?](#1-what-is-scheduling-why-use-it)
2. [Job vs Task vs Cron Job — Key Differences](#2-job-vs-task-vs-cron-job--key-differences)
3. [@EnableScheduling — How and Where](#3-enablescheduling--how-and-where)
4. [@Scheduled Method Rules](#4-scheduled-method-rules)
5. [fixedRate vs fixedDelay](#5-fixedrate-vs-fixeddelay)
6. [initialDelay — Delaying First Execution](#6-initialdelay--delaying-first-execution)
7. [@Schedules — Multiple Triggers on One Method](#7-schedules--multiple-triggers-on-one-method)
8. [Externalizing Values to application.properties](#8-externalizing-values-to-applicationproperties)
9. [Cron Expression — Complete Deep Dive](#9-cron-expression--complete-deep-dive)
10. [Full Working Example](#10-full-working-example)
11. [Common Mistakes & Fixes](#11-common-mistakes--fixes)
12. [Quick Reference Cheat Sheet](#12-quick-reference-cheat-sheet)

---

## 1. What is Scheduling? Why Use It?

### 🔍 What is Scheduling?

Scheduling means **running a piece of code automatically at a specific time or interval** — without any user clicking a button.

Think of it like a **TV timer** — you set it once, and it turns on/off by itself.

### 🤔 Why Do We Need It?

In real applications, many tasks need to run **automatically in the background**:

| Real-World Example | What Runs Automatically |
|---|---|
| Email service | Send newsletter every Sunday at 9 AM |
| Banking app | Calculate interest every midnight |
| E-commerce | Clear expired carts every 30 minutes |
| Monitoring app | Check server health every 10 seconds |
| Report system | Generate PDF report every Monday morning |

### ✅ Benefits of Scheduling

- **No human needed** — runs automatically
- **Consistent timing** — never forgets
- **Non-blocking** — runs in background, users don't wait
- **Centralized** — one place to manage all background tasks

### ❌ Without Scheduling

You'd have to manually trigger every task. Imagine calling your bank at midnight to calculate interest — not realistic!

---

## 2. Job vs Task vs Cron Job — Key Differences

These 3 words are often used interchangeably but have subtle differences.

### 📋 Simple Table

| Term | Meaning | Example |
|---|---|---|
| **Task** | A small unit of work | "Send one email" |
| **Job** | A bigger unit of work, may contain multiple tasks | "Monthly report job" (fetch data + generate PDF + send email) |
| **Cron Job** | A job **scheduled using a cron expression** (time pattern) | "Run every Monday at 8 AM" |

### 🧠 Think of it like this:

```
Job = A whole project
Task = One step in that project
Cron Job = A job that runs on a cron schedule (like "every day at midnight")
```

### 📝 Example in Simple Terms

```
Job: "Send Monthly Invoice"
  Task 1: Fetch all users
  Task 2: Calculate their bill
  Task 3: Generate PDF
  Task 4: Send email

Cron Job: Run this "Send Monthly Invoice" job on the 1st of every month at 9 AM
```

### 🔑 In Spring Context

- `@Scheduled` — used to schedule a **method (task/job)**
- Cron expression inside `@Scheduled` — makes it a **cron job**
- Spring calls the whole scheduled method a **"scheduled task"**

---

## 3. `@EnableScheduling` — How and Where

### 🔍 What does it do?

`@EnableScheduling` **turns on** Spring's scheduling engine.

Without it, even if you write `@Scheduled` on methods, **nothing will run**. It's like a master switch.

### 📌 Where to Put It

**Option 1 — On the main application class (Most Common)**

```java
@SpringBootApplication
@EnableScheduling                  // ← Add here
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

**Option 2 — On a separate config class (Clean approach)**

```java
@Configuration
@EnableScheduling                  // ← Add here
public class SchedulingConfig {
    // nothing else needed!
}
```

> ✅ Both are correct. Option 2 is cleaner for large projects.

### 🚫 What happens without it?

```java
// @EnableScheduling is missing!

@Component
public class MyScheduler {

    @Scheduled(fixedRate = 5000)
    public void myTask() {
        System.out.println("This will NEVER print!");  // Never runs!
    }
}
```

### ✅ Correct Setup

```java
// Step 1: Enable scheduling
@SpringBootApplication
@EnableScheduling
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}

// Step 2: Create scheduler
@Component
public class MyScheduler {

    @Scheduled(fixedRate = 5000)
    public void myTask() {
        System.out.println("Now this runs every 5 seconds!");  ✅
    }
}
```

### ⚠️ Important Notes

- Add `@EnableScheduling` **only once** in your app
- The class with `@Scheduled` must be a **Spring Bean** — annotate it with `@Component`, `@Service`, etc.
- No extra Maven dependency needed — it's part of `spring-boot-starter`

---

## 4. `@Scheduled` Method Rules

### 📜 The Two Golden Rules

Before diving in, memorize these:

```
Rule 1: Return type MUST be void
Rule 2: Parameters MUST be zero (empty)
```

### ✅ Correct Method Signatures

```java
@Scheduled(fixedRate = 1000)
public void correctMethod() {           // ✅ void, no params
    System.out.println("Correct!");
}
```

### ❌ Wrong Method Signatures

```java
// ❌ Wrong — has a return type
@Scheduled(fixedRate = 1000)
public String wrongReturn() {
    return "hello";  // Spring ignores this return value anyway!
}

// ❌ Wrong — has parameters
@Scheduled(fixedRate = 1000)
public void wrongParams(String name) {  // Who would pass this? Nobody!
    System.out.println("Hello " + name);
}
```

### 🤔 Why These Rules?

- **void** — Spring calls this method on its own. Nobody is waiting for a result. So return type must be `void`.
- **no parameters** — Spring calls this automatically. It has no way to know what values to pass. So no parameters allowed.

### 📝 Complete Valid Example

```java
@Component
public class ReminderScheduler {

    // ✅ Correct
    @Scheduled(fixedRate = 3000)
    public void sendReminder() {
        System.out.println("Reminder sent at: " + LocalTime.now());
    }
}
```

### 🔒 Access Modifier

- Can be `public`, `protected`, or even `private`
- Spring can call `private` methods too (via reflection)

```java
@Scheduled(fixedRate = 5000)
private void privateTaskAlsoWorks() {   // ✅ This works!
    System.out.println("Running...");
}
```

---

## 5. `fixedRate` vs `fixedDelay`

This is the **most important concept** in Spring Scheduling. Many developers get confused here.

### 📖 Simple Definitions

| Attribute | Meaning |
|---|---|
| `fixedRate` | Run every X ms, **measured from the START of the previous execution** |
| `fixedDelay` | Run every X ms, **measured from the END of the previous execution** |

### 🎯 Visual Diagram

**fixedRate = 5000 (5 seconds)**

```
|--- Task runs (2s) ---|--- wait 3s ---|--- Task runs (2s) ---|--- wait 3s ---|
 <-------- 5 seconds -------->         <-------- 5 seconds -------->
```

- Task starts at 0s → finishes at 2s
- Next task starts at 5s (regardless of when previous finished)
- Gap between END of task and START of next = 3s

---

**fixedDelay = 5000 (5 seconds)**

```
|--- Task runs (2s) ---|--- wait 5s ---|--- Task runs (2s) ---|--- wait 5s ---|
```

- Task starts at 0s → finishes at 2s
- Wait 5 seconds AFTER it finishes
- Next task starts at 7s

---

### 🧠 Real-Life Analogy

Imagine you run laps around a track:

- **fixedRate** = "Start a new lap every 10 minutes" — if your lap takes 8 mins, you rest 2 mins. If it takes 11 mins, the next one starts immediately (overlap possible!)
- **fixedDelay** = "Rest 10 minutes AFTER you finish each lap" — no rush, no overlap ever.

### 💻 Code Example

```java
@Component
public class ScheduleDemo {

    // Fires every 5 seconds FROM THE START of last execution
    @Scheduled(fixedRate = 5000)
    public void fixedRateTask() {
        System.out.println("fixedRate: " + LocalTime.now());
        // If this takes 3 seconds, next fires 2 seconds after it ends
    }

    // Waits 5 seconds AFTER the last execution ENDS
    @Scheduled(fixedDelay = 5000)
    public void fixedDelayTask() {
        System.out.println("fixedDelay: " + LocalTime.now());
        // No matter how long this takes, next fires 5 seconds after
    }
}
```

### 📊 Side-by-Side Comparison

| Feature | `fixedRate` | `fixedDelay` |
|---|---|---|
| Measured from | **Start** of last run | **End** of last run |
| Overlap possible? | Yes (if task takes too long) | No (always waits) |
| Best for | Regular heartbeats, polling | Tasks that should not overlap |
| Example use | "Check server health every 10s" | "Process queue, then wait 10s" |

### ⚠️ Overlap Warning for fixedRate

```java
@Scheduled(fixedRate = 3000)
public void riskyTask() {
    // This takes 5 seconds!
    Thread.sleep(5000);
    // Next execution tries to start at 3s but previous isn't done yet!
    // Spring queues it — they won't actually overlap by default
    // But tasks pile up in queue!
}
```

> 💡 **Tip:** If your task might run longer than the rate, use `fixedDelay` to be safe.

---

## 6. `initialDelay` — Delaying First Execution

### 🔍 What is initialDelay?

By default, a `@Scheduled` task runs **immediately when the app starts**.

`initialDelay` lets you say: **"Wait X milliseconds before running for the first time."**

### 🤔 Why Would You Need It?

- Wait for the database to be fully ready before querying
- Wait for other services/beans to be initialized
- Stagger multiple tasks so they don't all fire at once on startup
- Avoid running heavy tasks during application boot

### 💻 Code Example

```java
@Component
public class DelayedTask {

    // Starts immediately, then every 5s
    @Scheduled(fixedRate = 5000)
    public void noDelay() {
        System.out.println("Runs right away, then every 5s");
    }

    // Waits 10 seconds first, then every 5s
    @Scheduled(fixedRate = 5000, initialDelay = 10000)
    public void withDelay() {
        System.out.println("First run after 10s, then every 5s");
    }
}
```

### 📊 Timeline Comparison

**Without initialDelay:**
```
App Starts → 0s: Run → 5s: Run → 10s: Run → 15s: Run ...
```

**With initialDelay = 10000:**
```
App Starts → (wait 10s) → 10s: Run → 15s: Run → 20s: Run ...
```

### ✅ Works With All Types

```java
// With fixedRate
@Scheduled(fixedRate = 5000, initialDelay = 3000)
public void task1() { }

// With fixedDelay
@Scheduled(fixedDelay = 5000, initialDelay = 3000)
public void task2() { }

// With cron — initialDelay is NOT supported with cron!
// @Scheduled(cron = "0 * * * * *", initialDelay = 3000) // ❌ Won't work
```

> ⚠️ `initialDelay` does **not** work with `cron` expressions!

---

## 7. `@Schedules` — Multiple Triggers on One Method

### 🔍 What is @Schedules?

Sometimes you want **one method to run at multiple different schedules**.

`@Schedules` is a **container annotation** that holds multiple `@Scheduled` annotations.

### 💡 Real-World Example

"Send report every day at 9 AM **AND** every Sunday at 6 PM"

### 💻 Code Example

```java
@Component
public class MultiScheduleTask {

    @Schedules({
        @Scheduled(fixedRate = 5000),           // Every 5 seconds
        @Scheduled(cron = "0 0 9 * * MON")      // Every Monday 9 AM
    })
    public void runAtMultipleTimes() {
        System.out.println("Task triggered at: " + LocalTime.now());
    }
}
```

### 🔄 Alternative: Java 8+ Repeated Annotations

Since Java 8, you can use `@Scheduled` directly multiple times (without `@Schedules`):

```java
@Scheduled(fixedRate = 5000)
@Scheduled(cron = "0 0 9 * * MON")
public void runAtMultipleTimes() {
    System.out.println("Task triggered!");
}
```

> ✅ Both ways work. The Java 8 way is cleaner.

### ⚠️ Important Rules

- All the normal rules still apply (void, no params)
- Each `@Scheduled` inside runs **independently**
- If fixedRate fires AND cron fires at same time → method runs **twice**

---

## 8. Externalizing Values to `application.properties`

### 🔍 Why Externalize?

Hardcoding delay values in code is a bad practice:

```java
@Scheduled(fixedRate = 5000)  // What if you want to change this?
```

To change it, you'd have to **recompile and redeploy** your app. Bad!

Instead, put values in `application.properties` and reference them — change without redeployment.

### 🔧 How to Do It

**Step 1: Add values in `application.properties`**

```properties
# application.properties
app.scheduler.fixedRate=5000
app.scheduler.fixedDelay=3000
app.scheduler.initialDelay=10000
app.scheduler.cron=0 * * * * *
```

**Step 2: Use `${}` placeholder in `@Scheduled`**

```java
@Component
public class ExternalizedScheduler {

    @Scheduled(fixedRateString = "${app.scheduler.fixedRate}")
    public void fixedRateTask() {
        System.out.println("Fixed rate from properties!");
    }

    @Scheduled(fixedDelayString = "${app.scheduler.fixedDelay}")
    public void fixedDelayTask() {
        System.out.println("Fixed delay from properties!");
    }

    @Scheduled(
        fixedRateString = "${app.scheduler.fixedRate}",
        initialDelayString = "${app.scheduler.initialDelay}"
    )
    public void withInitialDelay() {
        System.out.println("Rate and delay from properties!");
    }

    @Scheduled(cron = "${app.scheduler.cron}")
    public void cronTask() {
        System.out.println("Cron from properties!");
    }
}
```

### 🔑 Notice the Naming Difference

| Hardcoded | Externalized (String version) |
|---|---|
| `fixedRate = 5000` | `fixedRateString = "${...}"` |
| `fixedDelay = 3000` | `fixedDelayString = "${...}"` |
| `initialDelay = 1000` | `initialDelayString = "${...}"` |
| `cron = "0 * * * *"` | `cron = "${...}"` (same attribute!) |

> ✅ `cron` attribute already accepts a String, so no "String" suffix needed.

### 💡 With Default Values

What if the property is missing? Provide a default:

```java
@Scheduled(fixedRateString = "${app.scheduler.fixedRate:5000}")
//                                                         ^^^^^ default = 5000
public void taskWithDefault() {
    System.out.println("Runs even if property is missing!");
}
```

### 🌍 Using application.yml (Alternative)

```yaml
# application.yml
app:
  scheduler:
    fixedRate: 5000
    cron: "0 * * * * *"
```

---

## 9. Cron Expression — Complete Deep Dive

---

### 9.1 What is a Cron Expression?

A cron expression is a **time pattern string** that tells Spring exactly **when** to run a task.

Think of it like writing a rule: "Run this every Monday at 9 AM" — but in a compact code form.

```java
@Scheduled(cron = "0 0 9 * * MON")
public void weeklyTask() { ... }
//           ↑ ↑ ↑ ↑  ↑   ↑
//           s m h d  mo  weekday
```

---

### 9.2 Spring Cron — The 6 Fields

Spring uses **6 fields** (not 5 like Linux). This is a very common source of bugs!

```
┌─────────── second       (0 - 59)
│ ┌───────── minute       (0 - 59)
│ │ ┌─────── hour         (0 - 23)
│ │ │ ┌───── day-of-month (1 - 31)
│ │ │ │ ┌─── month        (1 - 12  OR  JAN-DEC)
│ │ │ │ │ ┌─ day-of-week  (0 - 7   OR  SUN-SAT)
│ │ │ │ │ │                (0 and 7 both mean Sunday)
* * * * * *
```

#### 📋 Field Summary Table

| Position | Field | Allowed Values | Allowed Names |
|---|---|---|---|
| 1st | Second | 0–59 | — |
| 2nd | Minute | 0–59 | — |
| 3rd | Hour | 0–23 | — |
| 4th | Day of Month | 1–31 | — |
| 5th | Month | 1–12 | JAN, FEB, MAR, APR, MAY, JUN, JUL, AUG, SEP, OCT, NOV, DEC |
| 6th | Day of Week | 0–7 | SUN, MON, TUE, WED, THU, FRI, SAT |

> ⚠️ Both `0` and `7` mean **Sunday** in day-of-week field.

---

### 9.3 Spring Cron vs Unix/Linux Cron

This is the **#1 mistake** developers make when switching between Linux cron and Spring cron.

#### Side-by-Side Comparison

| Feature | Unix/Linux Cron | Spring Cron |
|---|---|---|
| Number of fields | **5** | **6** |
| Fields | `min hour day month weekday` | `sec min hour day month weekday` |
| Seconds support | ❌ No | ✅ Yes |
| Year field | ❌ No | ❌ No |
| Cron macros (`@daily`) | ✅ Yes (in some systems) | ✅ Yes (Spring 5.3+) |

#### Visual Difference

```
# Unix/Linux (5 fields) — NO seconds!
#  ┌── minute (0-59)
#  │  ┌── hour (0-23)
#  │  │  ┌── day (1-31)
#  │  │  │  ┌── month (1-12)
#  │  │  │  │  ┌── weekday (0-7)
   0  9  *  *  MON        ← "Every Monday at 9:00 AM"

# Spring (6 fields) — HAS seconds!
#  ┌── second (0-59)
#  │  ┌── minute (0-59)
#  │  │  ┌── hour (0-23)
#  │  │  │  ┌── day (1-31)
#  │  │  │  │  ┌── month (1-12)
#  │  │  │  │  │  ┌── weekday (0-7)
   0  0  9  *  *  MON     ← "Every Monday at 9:00:00 AM"
```

#### ❌ Classic Bug

```java
// Copy-pasted from Linux crontab — WRONG in Spring!
@Scheduled(cron = "0 9 * * MON")   // ❌ Only 5 fields — Spring will THROW an error

// Correct Spring version
@Scheduled(cron = "0 0 9 * * MON") // ✅ 6 fields
```

> 🧠 **Memory Tip:** Spring adds `seconds` at the **beginning**. Just prepend `0` to a Linux cron expression.

---

### 9.4 Wildcards, Ranges, Lists, Step Values

These are the building blocks of every cron expression. Master these 5 symbols and you can write any cron.

---

#### `*` — Wildcard (Every possible value)

Means: "every value in this field"

```
*  in seconds field  = every second (0,1,2,...,59)
*  in minutes field  = every minute
*  in hours field    = every hour
*  in day field      = every day
```

```java
// Every second of every minute of every hour... = runs every second
@Scheduled(cron = "* * * * * *")

// At second 0, every minute, every hour = runs every minute
@Scheduled(cron = "0 * * * * *")

// At 00:00, every day = runs every day at midnight
@Scheduled(cron = "0 0 0 * * *")
```

---

#### `-` — Range (From X to Y, inclusive)

Means: "every value between X and Y"

```
1-5   = 1, 2, 3, 4, 5
MON-FRI = Monday, Tuesday, Wednesday, Thursday, Friday
9-17  = 9, 10, 11, 12, 13, 14, 15, 16, 17
```

```java
// Every second from second 10 to second 20
@Scheduled(cron = "10-20 * * * * *")
// fires at: :10, :11, :12 ... :20 of every minute

// Every weekday (Mon to Fri) at 8 AM
@Scheduled(cron = "0 0 8 * * MON-FRI")

// Every hour from 9 AM to 5 PM, at minute 0
@Scheduled(cron = "0 0 9-17 * * *")
// fires at: 9:00, 10:00, 11:00 ... 17:00
```

---

#### `,` — List (Specific values)

Means: "run at exactly these values"

```
1,15,30    = at 1st, 15th, and 30th
MON,WED,FRI = Monday, Wednesday, Friday
JAN,JUL     = January and July
```

```java
// At seconds 0, 15, 30, 45 of every minute (every 15 seconds)
@Scheduled(cron = "0,15,30,45 * * * * *")

// Every Monday, Wednesday, Friday at 9 AM
@Scheduled(cron = "0 0 9 * * MON,WED,FRI")

// On 1st and 15th of every month at midnight
@Scheduled(cron = "0 0 0 1,15 * *")

// In January and July only, every day at midnight
@Scheduled(cron = "0 0 0 * JAN,JUL *")
```

---

#### `/` — Step (Every N values)

Means: "every N steps starting from X"

Format: `start/step`

```
0/15   = starting at 0, every 15 → 0, 15, 30, 45
5/10   = starting at 5, every 10 → 5, 15, 25, 35, 45, 55
*/5    = every 5 (same as 0/5)  → 0, 5, 10, 15 ...
```

```java
// Every 5 seconds
@Scheduled(cron = "0/5 * * * * *")
// fires at: :00, :05, :10, :15 ... :55

// Every 15 minutes
@Scheduled(cron = "0 0/15 * * * *")
// fires at: :00, :15, :30, :45 of every hour

// Every 2 hours starting at midnight
@Scheduled(cron = "0 0 0/2 * * *")
// fires at: 00:00, 02:00, 04:00 ... 22:00

// Every 10 seconds starting from second 5
@Scheduled(cron = "5/10 * * * * *")
// fires at: :05, :15, :25, :35, :45, :55
```

---

#### `?` — No specific value (Only in day-of-month or day-of-week)

Use `?` when you want to say "I don't care about this field" — needed when you specify the other day field.

```
You CANNOT set both day-of-month AND day-of-week at the same time.
Use ? in the one you want to ignore.
```

```java
// Every Monday — don't care which day of month
@Scheduled(cron = "0 0 9 ? * MON")
//                        ↑ "? = don't specify day-of-month"

// On the 15th of every month — don't care which weekday it is
@Scheduled(cron = "0 0 9 15 * ?")
//                              ↑ "? = don't specify day-of-week"
```

> 💡 **Simple rule:** If you use `day-of-week`, put `?` in `day-of-month`, and vice versa.

---

#### Combining Everything

```java
// Every 30 seconds on weekdays (Mon-Fri) during business hours (9-17)
@Scheduled(cron = "0/30 * 9-17 * * MON-FRI")

// Every 15 minutes on Mon, Wed, Fri
@Scheduled(cron = "0 0/15 * * * MON,WED,FRI")

// On 1st and 15th of Jan and Jul at 8:30 AM
@Scheduled(cron = "0 30 8 1,15 JAN,JUL ?")
```

---

### 9.5 Timezone Handling — `zone` Attribute

#### 🔍 Why Timezone Matters

By default, Spring uses the **JVM's default timezone** (usually the server's OS timezone).

Problem: If your server is in UTC but your users are in India, "9 AM" means different things!

```java
// Without zone — uses JVM default timezone (could be anything)
@Scheduled(cron = "0 0 9 * * *")
public void ambiguousTask() { ... }

// With zone — always runs at 9 AM INDIA time, no matter where server is
@Scheduled(cron = "0 0 9 * * *", zone = "Asia/Kolkata")
public void clearTask() { ... }
```

#### 📋 How to Use the `zone` Attribute

```java
@Component
public class TimezoneScheduler {

    // India Standard Time (IST = UTC+5:30)
    @Scheduled(cron = "0 0 9 * * *", zone = "Asia/Kolkata")
    public void indiaTask() {
        System.out.println("9 AM IST");
    }

    // US Eastern Time
    @Scheduled(cron = "0 0 9 * * *", zone = "America/New_York")
    public void usEastTask() {
        System.out.println("9 AM ET");
    }

    // UTC explicitly
    @Scheduled(cron = "0 0 0 * * *", zone = "UTC")
    public void utcMidnight() {
        System.out.println("Midnight UTC");
    }

    // London time (handles BST/GMT automatically)
    @Scheduled(cron = "0 0 8 * * MON-FRI", zone = "Europe/London")
    public void londonBusiness() {
        System.out.println("8 AM London");
    }
}
```

#### 🌍 Common Timezone IDs

| Region | Timezone ID |
|---|---|
| India | `Asia/Kolkata` |
| UTC | `UTC` or `GMT` |
| New York | `America/New_York` |
| Los Angeles | `America/Los_Angeles` |
| London | `Europe/London` |
| Tokyo | `Asia/Tokyo` |
| Sydney | `Australia/Sydney` |
| Dubai | `Asia/Dubai` |
| Singapore | `Asia/Singapore` |

> ✅ Always use **IANA timezone IDs** (like `Asia/Kolkata`), not short codes like `IST` — short codes are ambiguous (IST = India, Israel, and Ireland Standard Time!).

#### Externalizing Timezone Too

```properties
# application.properties
app.scheduler.timezone=Asia/Kolkata
```

```java
@Scheduled(cron = "${app.scheduler.cron}", zone = "${app.scheduler.timezone}")
public void task() { ... }
```

---

### 9.6 Cron Macros — Spring 5.3+

#### 🔍 What are Cron Macros?

Spring 5.3+ supports **shortcut names** instead of writing full 6-field cron expressions.

They are called **macros** — predefined nicknames for common schedules.

#### 📋 All Available Macros

| Macro | Equivalent Cron | Meaning |
|---|---|---|
| `@yearly` (or `@annually`) | `0 0 0 1 1 *` | Once a year — Jan 1st at midnight |
| `@monthly` | `0 0 0 1 * *` | Once a month — 1st at midnight |
| `@weekly` | `0 0 0 * * 0` | Once a week — Sunday at midnight |
| `@daily` (or `@midnight`) | `0 0 0 * * *` | Once a day — midnight |
| `@hourly` | `0 0 * * * *` | Once an hour — at minute 0 |

#### 💻 Code Examples

```java
@Component
public class MacroScheduler {

    // Once every year — January 1st at midnight
    @Scheduled(cron = "@yearly")
    public void yearlyTask() {
        System.out.println("Happy New Year task!");
    }

    // Once every month — 1st of the month at midnight
    @Scheduled(cron = "@monthly")
    public void monthlyTask() {
        System.out.println("Monthly cleanup!");
    }

    // Once every week — Sunday at midnight
    @Scheduled(cron = "@weekly")
    public void weeklyTask() {
        System.out.println("Weekly report!");
    }

    // Once every day — at midnight
    @Scheduled(cron = "@daily")
    public void dailyTask() {
        System.out.println("Daily backup!");
    }

    // Alias for @daily
    @Scheduled(cron = "@midnight")
    public void midnightTask() {
        System.out.println("Same as @daily!");
    }

    // Once every hour — at the top of the hour (:00)
    @Scheduled(cron = "@hourly")
    public void hourlyTask() {
        System.out.println("Hourly health check!");
    }
}
```

#### ✅ Macro vs Full Expression — Same Result

```java
// These two do EXACTLY the same thing:
@Scheduled(cron = "@daily")
@Scheduled(cron = "0 0 0 * * *")

// These two do EXACTLY the same thing:
@Scheduled(cron = "@hourly")
@Scheduled(cron = "0 0 * * * *")
```

#### ⚠️ Macros Don't Support Custom Timing

Macros are shortcuts — you can't customize them:

```java
// ❌ You can't say "@daily at 9 AM" — macros are fixed
// If you need 9 AM daily, write the full cron:
@Scheduled(cron = "0 0 9 * * *")   // ✅ Every day at 9 AM
```

> 💡 Use macros for **simple, standard schedules**. Use full expressions for **any custom timing**.

---

### 9.7 Externalizing Cron to `application.properties`

(See also Section 8 for general externalization.)

#### The Right Way

```properties
# application.properties
app.cron.daily-report   = 0 0 9 * * MON-FRI
app.cron.weekly-cleanup = 0 0 0 * * SUN
app.cron.hourly-ping    = 0 0 * * * *
app.cron.timezone       = Asia/Kolkata
```

```java
@Component
public class ExternalCronScheduler {

    @Scheduled(cron = "${app.cron.daily-report}", zone = "${app.cron.timezone}")
    public void dailyReport() {
        System.out.println("Report at 9 AM IST, Mon-Fri");
    }

    @Scheduled(cron = "${app.cron.weekly-cleanup}")
    public void weeklyCleanup() {
        System.out.println("Cleanup every Sunday midnight");
    }

    // With default fallback — if property missing, use @hourly
    @Scheduled(cron = "${app.cron.hourly-ping:@hourly}")
    public void ping() {
        System.out.println("Ping!");
    }
}
```

#### 🔴 Disabling a Scheduled Task via Properties

A very useful trick — set cron to the special value `"-"` to **disable** a task:

```properties
# Disable the task in production
app.cron.debug-task = -
```

```java
@Scheduled(cron = "${app.cron.debug-task:-}")
public void debugTask() {
    // Won't run if property is "-" or missing (default is also "-")
    System.out.println("Debug task");
}
```

> ✅ `"-"` is Spring's magic value to disable a `@Scheduled` task via properties. Very handy for environment-specific control!

---

### 9.8 Conditional Scheduling

Sometimes you want a task to run **only in certain environments** (dev, prod) or **only when a feature flag is enabled**.

---

#### Method 1: `@ConditionalOnProperty`

Run the entire scheduler bean **only if a property is set to a specific value**.

```properties
# application.properties
app.scheduling.enabled=true
```

```java
@Component
@ConditionalOnProperty(
    name = "app.scheduling.enabled",
    havingValue = "true",
    matchIfMissing = false   // don't run if property is missing
)
public class ConditionalScheduler {

    @Scheduled(fixedRate = 5000)
    public void task() {
        System.out.println("Only runs when app.scheduling.enabled=true");
    }
}
```

Now in `application-prod.properties`:

```properties
app.scheduling.enabled=true
```

And in `application-dev.properties`:

```properties
app.scheduling.enabled=false   # ← scheduler bean not even created in dev
```

> ✅ The whole `@Component` bean is **not created** if the condition fails — cleanest approach.

---

#### Method 2: Spring Profiles — `@Profile`

Run scheduler **only in specific Spring profiles** (like `prod`, `staging`).

```java
@Component
@Profile("prod")    // ← Only active when spring.profiles.active=prod
public class ProdOnlyScheduler {

    @Scheduled(cron = "0 0 2 * * *")
    public void nightlyBackup() {
        System.out.println("Nightly backup — PROD only!");
    }
}
```

```java
@Component
@Profile({"dev", "staging"})   // ← Active in dev OR staging
public class DevScheduler {

    @Scheduled(fixedRate = 10000)
    public void devTask() {
        System.out.println("Dev/Staging test task");
    }
}
```

**Activate a profile:**

```properties
# application.properties
spring.profiles.active=prod
```

Or via command line:

```bash
java -jar myapp.jar --spring.profiles.active=prod
```

---

#### Method 3: Conditional Check Inside the Method

The simplest approach — check a flag inside the method body.

```properties
app.feature.email-scheduler=false
```

```java
@Component
public class FeatureFlagScheduler {

    @Value("${app.feature.email-scheduler:true}")
    private boolean emailSchedulerEnabled;

    @Scheduled(fixedRate = 60000)
    public void sendEmails() {
        if (!emailSchedulerEnabled) {
            return;   // Skip — feature is disabled
        }
        System.out.println("Sending emails...");
    }
}
```

> ⚠️ The method still **gets called** (just returns early). If you want zero overhead, use `@ConditionalOnProperty` instead.

---

#### Comparison of All 3 Approaches

| Approach | Bean Created? | Overhead | Best For |
|---|---|---|---|
| `@ConditionalOnProperty` | ❌ Not if disabled | Zero | Disabling whole scheduler |
| `@Profile` | ❌ Not if wrong profile | Zero | Environment-specific tasks |
| Flag check inside method | ✅ Always | Tiny | Per-method feature flags |

---

### 9.9 Common Cron Examples Reference

```java
// ── Every N seconds / minutes ──────────────────────────────
@Scheduled(cron = "*/5  * * * * *")   // every 5 seconds
@Scheduled(cron = "0 */5 * * * *")    // every 5 minutes
@Scheduled(cron = "0 */15 * * * *")   // every 15 minutes
@Scheduled(cron = "0 */30 * * * *")   // every 30 minutes

// ── Every N hours ───────────────────────────────────────────
@Scheduled(cron = "0 0 */2 * * *")    // every 2 hours
@Scheduled(cron = "0 0 */6 * * *")    // every 6 hours
@Scheduled(cron = "0 0 */12 * * *")   // every 12 hours

// ── Specific times ──────────────────────────────────────────
@Scheduled(cron = "0 0 0 * * *")      // midnight every day
@Scheduled(cron = "0 0 6 * * *")      // 6 AM every day
@Scheduled(cron = "0 30 8 * * *")     // 8:30 AM every day
@Scheduled(cron = "0 0 9 * * MON-FRI")// 9 AM weekdays only

// ── Weekly ──────────────────────────────────────────────────
@Scheduled(cron = "0 0 0 * * SUN")    // Sunday midnight
@Scheduled(cron = "0 0 9 * * MON")    // Monday 9 AM

// ── Monthly ─────────────────────────────────────────────────
@Scheduled(cron = "0 0 0 1 * *")      // 1st of every month
@Scheduled(cron = "0 0 0 L * *")      // last day of every month (Quartz only)
@Scheduled(cron = "0 0 0 1,15 * *")   // 1st and 15th of every month

// ── Specific months ─────────────────────────────────────────
@Scheduled(cron = "0 0 0 1 1 *")      // Jan 1st (new year)
@Scheduled(cron = "0 0 9 * 1,7 *")    // 9 AM in January and July
```

---

### 9.10 Quick Cron Cheat Sheet

```
FIELD       │ VALUES           │ SPECIAL CHARS
────────────┼──────────────────┼───────────────────────
second      │ 0–59             │  * , - /
minute      │ 0–59             │  * , - /
hour        │ 0–23             │  * , - /
day-month   │ 1–31             │  * , - / ?
month       │ 1–12 / JAN-DEC   │  * , - /
day-week    │ 0–7  / SUN-SAT   │  * , - / ?

SPECIAL CHARS:
  *    = every value
  ,    = list: 1,3,5
  -    = range: 1-5
  /    = step: 0/15 (every 15 from 0)
  ?    = don't care (day fields only)

MACROS (Spring 5.3+):
  @yearly   = 0 0 0 1 1 *     (Jan 1st midnight)
  @monthly  = 0 0 0 1 * *     (1st of month)
  @weekly   = 0 0 0 * * 0     (Sunday midnight)
  @daily    = 0 0 0 * * *     (midnight)
  @midnight = 0 0 0 * * *     (same as @daily)
  @hourly   = 0 0 * * * *     (top of every hour)
  -         = disabled         (special disable value)

SPRING vs LINUX:
  Linux: min  hour day month weekday     (5 fields)
  Spring: sec min  hour day month weekday (6 fields)
```

---

## 10. Full Working Example

Here's a complete, realistic Spring Boot project putting it all together:

### 📁 Project Structure

```
src/
└── main/
    ├── java/
    │   └── com/example/
    │       ├── MyApp.java
    │       └── scheduler/
    │           └── AppScheduler.java
    └── resources/
        └── application.properties
```

### MyApp.java

```java
@SpringBootApplication
@EnableScheduling
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

### application.properties

```properties
app.scheduler.heartbeat.rate=5000
app.scheduler.report.cron=0 0 9 * * MON
app.scheduler.cleanup.delay=10000
app.scheduler.startup.delay=3000
```

### AppScheduler.java

```java
@Component
public class AppScheduler {

    // 1. Simple fixedRate — check server health every 5 seconds
    @Scheduled(fixedRateString = "${app.scheduler.heartbeat.rate}")
    public void heartbeatCheck() {
        System.out.println("[HEALTH] Server OK at " + LocalTime.now());
    }

    // 2. fixedDelay — process queue, wait 10s after each run
    @Scheduled(fixedDelayString = "${app.scheduler.cleanup.delay}")
    public void cleanupExpiredSessions() {
        System.out.println("[CLEANUP] Sessions cleaned at " + LocalTime.now());
    }

    // 3. Cron — weekly report every Monday at 9 AM
    @Scheduled(cron = "${app.scheduler.report.cron}")
    public void weeklyReport() {
        System.out.println("[REPORT] Weekly report generated!");
    }

    // 4. initialDelay — wait 3s after startup, then every 5s
    @Scheduled(
        fixedRate = 5000,
        initialDelayString = "${app.scheduler.startup.delay}"
    )
    public void delayedTask() {
        System.out.println("[DELAYED] Task ran at " + LocalTime.now());
    }

    // 5. Multiple schedules — runs every 5s AND every minute
    @Scheduled(fixedRate = 5000)
    @Scheduled(cron = "0 * * * * *")
    public void multiTriggerTask() {
        System.out.println("[MULTI] Triggered at " + LocalTime.now());
    }
}
```

### 📤 Sample Output

```
[HEALTH]  Server OK at 10:00:00
[HEALTH]  Server OK at 10:00:05
[CLEANUP] Sessions cleaned at 10:00:05
[DELAYED] Task ran at 10:00:03
[MULTI]   Triggered at 10:00:00 (from cron)
[MULTI]   Triggered at 10:00:05 (from fixedRate)
```

---

## 11. Common Mistakes & Fixes

### ❌ Mistake 1: Forgot `@EnableScheduling`

```java
// Missing @EnableScheduling!
@SpringBootApplication
public class MyApp { ... }

// Nothing runs!
@Scheduled(fixedRate = 5000)
public void task() { ... }
```

✅ **Fix:** Add `@EnableScheduling` to main class or config class.

---

### ❌ Mistake 2: Class Not a Spring Bean

```java
// No @Component, @Service, etc.!
public class MyScheduler {
    @Scheduled(fixedRate = 5000)
    public void task() { ... }  // Spring doesn't manage this class!
}
```

✅ **Fix:** Add `@Component` (or `@Service`) to the class.

---

### ❌ Mistake 3: Method Has Return Type

```java
@Scheduled(fixedRate = 5000)
public String task() {     // ❌ Must be void!
    return "done";
}
```

✅ **Fix:** Change to `public void task()`.

---

### ❌ Mistake 4: Using `fixedRate` with String

```java
@Scheduled(fixedRate = "${app.rate}")  // ❌ fixedRate takes long, not String!
public void task() { ... }
```

✅ **Fix:** Use `fixedRateString`:

```java
@Scheduled(fixedRateString = "${app.rate}")  // ✅
```

---

### ❌ Mistake 5: `initialDelay` with cron

```java
@Scheduled(cron = "0 * * * * *", initialDelay = 5000)  // ❌ Ignored!
public void task() { ... }
```

✅ **Fix:** `initialDelay` only works with `fixedRate` and `fixedDelay`.

---

### ❌ Mistake 6: Wrong Cron Field Count (Linux vs Spring)

```java
// Linux cron has 5 fields, Spring cron has 6!
@Scheduled(cron = "0 9 * * MON")    // ❌ 5 fields — Spring throws an error!
@Scheduled(cron = "0 0 9 * * MON")  // ✅ 6 fields — second minute hour day month weekday
```

---

### ❌ Mistake 7: Using Short Timezone Codes

```java
// Short codes are ambiguous — IST means India, Israel AND Ireland!
@Scheduled(cron = "0 0 9 * * *", zone = "IST")  // ❌ Ambiguous!
@Scheduled(cron = "0 0 9 * * *", zone = "Asia/Kolkata")  // ✅ Unambiguous IANA ID
```

---

### ❌ Mistake 8: Using Macros on Spring < 5.3

```java
// @daily, @hourly etc. only work on Spring 5.3+ / Spring Boot 2.4+
@Scheduled(cron = "@daily")  // ❌ Throws error on older Spring versions!

// Check your Spring Boot version — if < 2.4, write the full cron:
@Scheduled(cron = "0 0 0 * * *")  // ✅ Works on all versions
```

---

### ❌ Mistake 9: Setting Both day-of-month AND day-of-week

```java
// Cannot specify both — use ? on the one you don't need
@Scheduled(cron = "0 0 9 15 * MON")   // ❌ Conflict! 15th AND Monday?
@Scheduled(cron = "0 0 9 15 * ?")     // ✅ 15th of every month
@Scheduled(cron = "0 0 9 ? * MON")    // ✅ Every Monday
```

---

## 12. Quick Reference Cheat Sheet

```
┌──────────────────────────────────────────────────────────────────────┐
│                    SPRING SCHEDULING CHEAT SHEET                     │
├─────────────────────────┬────────────────────────────────────────────┤
│ Enable                  │ @EnableScheduling on @Configuration class  │
├─────────────────────────┼────────────────────────────────────────────┤
│ Method rules            │ public void methodName() — no params       │
├─────────────────────────┼────────────────────────────────────────────┤
│ fixedRate               │ Every N ms from START of last run          │
│ fixedDelay              │ Every N ms from END of last run            │
│ initialDelay            │ Wait N ms before FIRST run                 │
├─────────────────────────┼────────────────────────────────────────────┤
│ Externalized            │ fixedRateString    = "${property}"         │
│                         │ fixedDelayString   = "${property}"         │
│                         │ initialDelayString = "${property}"         │
│                         │ cron               = "${property}"         │
│ Disable via property    │ cron = "${prop:-}"  (set prop="-")         │
├─────────────────────────┼────────────────────────────────────────────┤
│ Multiple triggers       │ @Scheduled(...)                            │
│                         │ @Scheduled(...)                            │
│                         │ public void method() {}                    │
├─────────────────────────┼────────────────────────────────────────────┤
│ Cron format (6 fields)  │ second minute hour day month weekday       │
│                         │ "0 0 9 * * MON"    = Mon 9AM               │
│                         │ "0 0 9 * * ?"      = use ? for day fields  │
├─────────────────────────┼────────────────────────────────────────────┤
│ Cron special chars      │ *  = every value                           │
│                         │ ,  = list:   MON,WED,FRI                   │
│                         │ -  = range:  MON-FRI                       │
│                         │ /  = step:   0/15 = every 15 from 0        │
│                         │ ?  = any (day-month or day-week only)      │
├─────────────────────────┼────────────────────────────────────────────┤
│ Cron macros (5.3+)      │ @yearly  = 0 0 0 1 1 *  (Jan 1st)         │
│                         │ @monthly = 0 0 0 1 * *  (1st of month)    │
│                         │ @weekly  = 0 0 0 * * 0  (Sunday)          │
│                         │ @daily   = 0 0 0 * * *  (midnight)        │
│                         │ @hourly  = 0 0 * * * *  (top of hour)     │
├─────────────────────────┼────────────────────────────────────────────┤
│ Timezone                │ zone = "Asia/Kolkata"  (IANA ID)           │
│                         │ zone = "${app.timezone}"                   │
├─────────────────────────┼────────────────────────────────────────────┤
│ Conditional scheduling  │ @ConditionalOnProperty(name="x",           │
│                         │   havingValue="true")    ← best            │
│                         │ @Profile("prod")         ← by env          │
│                         │ if (!enabled) return;    ← inside method   │
├─────────────────────────┼────────────────────────────────────────────┤
│ Spring vs Linux cron    │ Linux: 5 fields (no seconds)               │
│                         │ Spring: 6 fields (seconds first!)          │
└─────────────────────────┴────────────────────────────────────────────┘
```

### ⏱️ Time Reference

| Value | Milliseconds |
|---|---|
| 1 second | 1000 |
| 30 seconds | 30000 |
| 1 minute | 60000 |
| 5 minutes | 300000 |
| 1 hour | 3600000 |

---

> 💡 **Pro Tip:** Always prefer `fixedDelay` over `fixedRate` when tasks might take variable time. Use `fixedRate` only when you need strict timing like polling or heartbeat checks.

> 💡 **Pro Tip:** Always externalize cron/delay values to `application.properties`. This allows changing schedules without redeployment.

---
