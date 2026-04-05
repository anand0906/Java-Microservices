# 🕐 Spring Scheduling — Complete Revision Notes

---

## Table of Contents

**Part A — `@Scheduled` Basics**
1. [What is Scheduling & Why Use It](#1-what-is-scheduling--why-use-it)
2. [Job vs Task vs Cron Job](#2-job-vs-task-vs-cron-job)
3. [`@EnableScheduling`](#3-enablescheduling)
4. [`@Scheduled` Method Rules](#4-scheduled-method-rules)
5. [`fixedRate` vs `fixedDelay`](#5-fixedrate-vs-fixeddelay)
6. [`initialDelay`](#6-initialdelay)
7. [Multiple Triggers on One Method](#7-multiple-triggers-on-one-method)
8. [Externalizing Values to Properties](#8-externalizing-values-to-properties)

**Part B — Cron Expressions**

9. [Cron Expression — 6 Fields](#9-cron-expression--6-fields)
10. [Spring Cron vs Linux Cron](#10-spring-cron-vs-linux-cron)
11. [Wildcards, Ranges, Lists, Steps](#11-wildcards-ranges-lists-steps)
12. [Timezone — `zone` Attribute](#12-timezone--zone-attribute)
13. [Cron Macros (Spring 5.3+)](#13-cron-macros-spring-53)
14. [Conditional Scheduling](#14-conditional-scheduling)
15. [Common Cron Examples](#15-common-cron-examples)

**Part C — Thread Pool & Dynamic Scheduling**

16. [Default Single-Thread Problem](#16-default-single-thread-problem)
17. [`ThreadPoolTaskScheduler`](#17-threadpooltaskscheduler)
18. [`SchedulingConfigurer`](#18-schedulingconfigurer)
19. [`TaskScheduler` and `TaskExecutor`](#19-taskscheduler-and-taskexecutor)
20. [Dynamic Scheduling at Runtime](#20-dynamic-scheduling-at-runtime)
21. [`CronTrigger` and `PeriodicTrigger`](#21-crontrigger-and-periodictrigger)
22. [Cancelling via `ScheduledFuture`](#22-cancelling-via-scheduledfuture)
23. [Storing Cron in DB — Reload Dynamically](#23-storing-cron-in-db--reload-dynamically)

**Part D — Preventing Overlap & Concurrency**

24. [Preventing Overlapping Executions](#24-preventing-overlapping-executions)
25. [`@Async` + `@Scheduled`](#25-async--scheduled)
26. [Thread Safety in Shared State](#26-thread-safety-in-shared-state)

**Part E — Distributed & Cluster Scheduling**

27. [The Cluster Problem](#27-the-cluster-problem)
28. [ShedLock — Distributed Lock](#28-shedlock--distributed-lock)
29. [ShedLock Backends (DB / Redis)](#29-shedlock-backends-db--redis)
30. [Quartz Clustered Setup](#30-quartz-clustered-setup)
31. [Quartz vs `@Scheduled`](#31-quartz-vs-scheduled)

**Part F — Quartz Deep Dive**

32. [Core Quartz Concepts](#32-core-quartz-concepts)
33. [`CronTrigger` vs `SimpleTrigger`](#33-crontrigger-vs-simpletrigger-quartz)
34. [Persisting Jobs — `JDBCJobStore`](#34-jdbcjobstore)
35. [Quartz Misfire Instructions](#35-quartz-misfire-instructions)
36. [Spring Boot + Quartz Auto-Config](#36-spring-boot--quartz-auto-config)
37. [`JobRunr` — Modern Alternative](#37-jobrunr--modern-alternative)

**Part G — Spring Batch**

38. [What is Spring Batch & When to Use It](#38-what-is-spring-batch--when-to-use-it)
39. [Core Batch Concepts](#39-core-batch-concepts)
40. [`JobLauncher` — Triggering Batch Jobs](#40-joblauncher)
41. [`@Scheduled` + `JobLauncher`](#41-scheduled--joblauncher)
42. [Chunk-Oriented vs Tasklet](#42-chunk-oriented-vs-tasklet)

**Part H — Production Patterns**

43. [Idempotency — Safe to Re-Run](#43-idempotency)
44. [Retry Logic — `spring-retry`](#44-retry-logic)
45. [Job Chaining](#45-job-chaining)
46. [Misfire Handling](#46-misfire-handling)
47. [Graceful Shutdown](#47-graceful-shutdown)
48. [Logging Job Start / End / Duration](#48-logging)
49. [Spring Actuator — Scheduler Metrics](#49-actuator)
50. [Alerting on Job Failures](#50-alerting)
51. [Dead Job Detection](#51-dead-job-detection)

---

## Part A — `@Scheduled` Basics

---

### 1. What is Scheduling & Why Use It

Scheduling = **running code automatically at a set time or interval**, without a user triggering it.

| Use Case | Runs Automatically |
|---|---|
| Banking | Interest calculation at midnight |
| E-commerce | Clear expired carts every 30 min |
| Monitoring | Check server health every 10s |
| Reports | Generate PDF every Monday 9 AM |

---

### 2. Job vs Task vs Cron Job

| Term | Meaning |
|---|---|
| **Task** | A small unit of work — "send one email" |
| **Job** | A bigger operation, may have multiple tasks |
| **Cron Job** | A job triggered by a cron time expression |

In Spring: `@Scheduled` schedules a method (task/job). Adding a cron expression makes it a cron job.

---

### 3. `@EnableScheduling`

The **master switch** — without it, `@Scheduled` methods never run.

```java
@SpringBootApplication
@EnableScheduling          // ← turn on the engine
public class App { }
```

Or in a config class (cleaner for large projects):

```java
@Configuration
@EnableScheduling
public class SchedulingConfig { }
```

> Add `@EnableScheduling` **only once**. The scheduled class must be a Spring bean (`@Component`, `@Service`, etc.).

---

### 4. `@Scheduled` Method Rules

Two golden rules:

```
1. Return type → void
2. Parameters  → none
```

```java
@Scheduled(fixedRate = 5000)
public void myTask() { }     // ✅ correct

@Scheduled(fixedRate = 5000)
public String wrong() { }    // ❌ return type not void

@Scheduled(fixedRate = 5000)
public void wrong(String x) { } // ❌ has parameters
```

Access modifier can be `public`, `protected`, or `private` — Spring uses reflection.

---

### 5. `fixedRate` vs `fixedDelay`

| | `fixedRate` | `fixedDelay` |
|---|---|---|
| Measured from | **Start** of last run | **End** of last run |
| Overlap possible? | Yes (if task is slow) | No (always waits) |
| Best for | Heartbeats, polling | Tasks that must not overlap |

```java
@Scheduled(fixedRate  = 5000)  // next fires 5s from START of previous
@Scheduled(fixedDelay = 5000)  // next fires 5s from END   of previous
```

**Timeline analogy:**
```
fixedRate=5s, task takes 2s:
  START──2s──END───3s wait───START──2s──END...
  |←────── 5s ──────────────|

fixedDelay=5s, task takes 2s:
  START──2s──END──────5s wait──────START...
```

> If task duration is unpredictable, prefer `fixedDelay` to avoid pileup.

---

### 6. `initialDelay`

Wait N ms before the **very first execution**.

```java
@Scheduled(fixedRate = 5000, initialDelay = 10000)
public void task() { }
// App starts → waits 10s → runs → every 5s after
```

> ⚠️ `initialDelay` does **not** work with `cron`.

---

### 7. Multiple Triggers on One Method

```java
@Scheduled(fixedRate = 5000)
@Scheduled(cron = "0 0 9 * * MON")
public void task() { }   // fires on BOTH schedules independently
```

Each trigger runs independently — if both fire at the same time, the method runs twice.

---

### 8. Externalizing Values to Properties

Never hardcode intervals. Put them in `application.properties`:

```properties
app.scheduler.rate=5000
app.scheduler.cron=0 0 9 * * MON-FRI
```

```java
@Scheduled(fixedRateString    = "${app.scheduler.rate}")
@Scheduled(fixedDelayString   = "${app.scheduler.rate}")
@Scheduled(initialDelayString = "${app.scheduler.rate}")
@Scheduled(cron               = "${app.scheduler.cron}")  // cron uses same attr name
```

Key difference: `fixedRate` (long) vs `fixedRateString` (String with `${...}`).

**Default fallback:**
```java
@Scheduled(fixedRateString = "${app.rate:5000}")  // 5000 if property missing
```

**Disable a task via properties:**
```properties
app.scheduler.cron=-   # Spring's magic value to disable
```

---

## Part B — Cron Expressions

---

### 9. Cron Expression — 6 Fields

Spring cron has **6 fields** (not 5 like Linux):

```
┌────── second       (0–59)
│ ┌──── minute       (0–59)
│ │ ┌── hour         (0–23)
│ │ │ ┌ day-of-month (1–31)
│ │ │ │ ┌ month      (1–12 or JAN-DEC)
│ │ │ │ │ ┌ day-of-week (0–7 or SUN-SAT, 0&7=Sunday)
│ │ │ │ │ │
0 0 9 * * MON        ← "Every Monday at 9:00:00 AM"
```

---

### 10. Spring Cron vs Linux Cron

| | Linux Cron | Spring Cron |
|---|---|---|
| Fields | 5 (no seconds) | 6 (seconds first) |
| Format | `min hour day month weekday` | `sec min hour day month weekday` |

```
Linux:  0 9 * * MON      ← 5 fields
Spring: 0 0 9 * * MON    ← 6 fields (prepend seconds)
```

> Most common mistake: copying Linux cron into Spring — always add the seconds field.

---

### 11. Wildcards, Ranges, Lists, Steps

| Symbol | Meaning | Example |
|---|---|---|
| `*` | Every value | `* * * * * *` = every second |
| `-` | Range | `MON-FRI`, `9-17` |
| `,` | List | `MON,WED,FRI`, `1,15` |
| `/` | Step (every N) | `0/15` = 0,15,30,45 |
| `?` | Don't care (day fields only) | Use in one of the two day fields |

```java
@Scheduled(cron = "0 0/15 9-17 * * MON-FRI")
// Every 15 min, between 9AM-5PM, on weekdays
```

**Rule for `?`:** You cannot set both day-of-month AND day-of-week. Use `?` for the one you don't need.

```java
@Scheduled(cron = "0 0 9 ? * MON")   // every Monday (? in day-of-month)
@Scheduled(cron = "0 0 9 15 * ?")    // 15th of every month (? in day-of-week)
```

---

### 12. Timezone — `zone` Attribute

```java
@Scheduled(cron = "0 0 9 * * *", zone = "Asia/Kolkata")
```

Without `zone`, Spring uses the JVM's default timezone (server OS timezone) — unpredictable in cloud.

Always use **IANA timezone IDs** — never short codes like `IST` (ambiguous: India, Israel, Ireland).

| Region | ID |
|---|---|
| India | `Asia/Kolkata` |
| UTC | `UTC` |
| New York | `America/New_York` |
| London | `Europe/London` |

---

### 13. Cron Macros (Spring 5.3+)

| Macro | Equivalent | Meaning |
|---|---|---|
| `@yearly` | `0 0 0 1 1 *` | Jan 1st midnight |
| `@monthly` | `0 0 0 1 * *` | 1st of month midnight |
| `@weekly` | `0 0 0 * * 0` | Sunday midnight |
| `@daily` | `0 0 0 * * *` | Every midnight |
| `@midnight` | `0 0 0 * * *` | Same as `@daily` |
| `@hourly` | `0 0 * * * *` | Top of every hour |

```java
@Scheduled(cron = "@daily")   // same as "0 0 0 * * *"
```

Macros are fixed — for custom times like "9 AM daily", write the full expression.

---

### 14. Conditional Scheduling

**Option 1 — `@ConditionalOnProperty` (best — bean not even created):**

```java
@Component
@ConditionalOnProperty(name = "app.scheduling.enabled", havingValue = "true")
public class MyScheduler {
    @Scheduled(fixedRate = 5000)
    public void task() { }
}
```

**Option 2 — `@Profile` (by environment):**

```java
@Component
@Profile("prod")
public class ProdScheduler {
    @Scheduled(cron = "0 0 2 * * *")
    public void nightlyJob() { }
}
```

**Option 3 — flag check inside method (simplest):**

```java
@Value("${app.feature.enabled:true}")
private boolean enabled;

@Scheduled(fixedRate = 5000)
public void task() {
    if (!enabled) return;
    doWork();
}
```

| Approach | Bean Created | Best For |
|---|---|---|
| `@ConditionalOnProperty` | ❌ No | Disabling whole scheduler |
| `@Profile` | ❌ No | Per-environment tasks |
| Flag inside method | ✅ Yes | Per-feature flags |

---

### 15. Common Cron Examples

```
Every 5 seconds:     */5 * * * * *
Every 5 minutes:     0 */5 * * * *
Every 30 minutes:    0 */30 * * * *
Every 2 hours:       0 0 */2 * * *
Midnight daily:      0 0 0 * * *
9 AM weekdays:       0 0 9 * * MON-FRI
Monday 9 AM:         0 0 9 ? * MON
1st of month:        0 0 0 1 * *
1st and 15th:        0 0 0 1,15 * *
Sunday midnight:     0 0 0 * * SUN
```

---

## Part C — Thread Pool & Dynamic Scheduling

---

### 16. Default Single-Thread Problem

Spring uses **one thread** for all `@Scheduled` tasks by default.

```
Thread-1 handles: TaskA, TaskB, TaskC   ← sequential!
```

If TaskB is slow, TaskA and TaskC are delayed.

```java
@Scheduled(fixedRate = 3000)
public void fast() { }         // should run every 3s

@Scheduled(fixedRate = 3000)
public void slow() {
    Thread.sleep(10000);       // blocks the single thread for 10s!
}
// Result: fast() misses 3 executions
```

---

### 17. `ThreadPoolTaskScheduler`

Replace the single thread with a pool — declare this bean and Spring uses it automatically:

```java
@Bean
public ThreadPoolTaskScheduler taskScheduler() {
    ThreadPoolTaskScheduler s = new ThreadPoolTaskScheduler();
    s.setPoolSize(5);
    s.setThreadNamePrefix("sched-");
    s.setWaitForTasksToCompleteOnShutdown(true);
    s.setAwaitTerminationSeconds(30);
    s.initialize();
    return s;
}
```

> Rule of thumb: `poolSize` ≥ number of scheduled tasks (add 2 buffer).

---

### 18. `SchedulingConfigurer`

Programmatic configuration — also lets you register tasks in code:

```java
@Configuration
@EnableScheduling
public class SchedulerConfig implements SchedulingConfigurer {

    @Override
    public void configureTasks(ScheduledTaskRegistrar registrar) {
        ThreadPoolTaskScheduler s = new ThreadPoolTaskScheduler();
        s.setPoolSize(5);
        s.initialize();
        registrar.setTaskScheduler(s);

        // Register tasks in code (no @Scheduled needed)
        registrar.addCronTask(() -> System.out.println("cron task"), "0 0 9 * * *");
        registrar.addFixedRateTask(() -> System.out.println("rate task"), 5000);
    }
}
```

| Goal | Approach |
|---|---|
| Just need a thread pool | `@Bean ThreadPoolTaskScheduler` |
| Also register tasks in code | `SchedulingConfigurer` |

---

### 19. `TaskScheduler` and `TaskExecutor`

| Interface | Purpose |
|---|---|
| `TaskScheduler` | Run at a specific time or interval |
| `TaskExecutor` | Run immediately in a background thread |

`ThreadPoolTaskScheduler` implements **both**. Inject the interface, not the class:

```java
@Autowired TaskScheduler taskScheduler;   // ✅ interface
// NOT:
@Autowired ThreadPoolTaskScheduler ...;   // ❌ tightly coupled
```

Key `TaskScheduler` methods:

```java
taskScheduler.schedule(task, Instant.now().plusSeconds(10));  // once, at a time
taskScheduler.scheduleAtFixedRate(task, Duration.ofSeconds(5));
taskScheduler.scheduleWithFixedDelay(task, Duration.ofSeconds(5));
taskScheduler.schedule(task, trigger);  // with CronTrigger/PeriodicTrigger
```

---

### 20. Dynamic Scheduling at Runtime

Schedule, stop, and reschedule tasks while the app is running:

```java
@Component
public class DynamicScheduler {

    @Autowired private TaskScheduler taskScheduler;
    private ScheduledFuture<?> future;

    public void start(long seconds) {
        future = taskScheduler.scheduleAtFixedRate(
            () -> System.out.println("running"),
            Duration.ofSeconds(seconds)
        );
    }

    public void stop() {
        if (future != null) future.cancel(false);
    }

    public void reschedule(long newSeconds) {
        stop();
        start(newSeconds);
    }
}
```

Expose via REST → change schedule without restarting the app.

---

### 21. `CronTrigger` and `PeriodicTrigger`

Used when scheduling dynamically (via `taskScheduler.schedule(...)`):

```java
// Cron-based
CronTrigger cron = new CronTrigger("0 0 9 * * *");
CronTrigger cronTz = new CronTrigger("0 0 9 * * *",
                         TimeZone.getTimeZone("Asia/Kolkata"));

// Interval-based
PeriodicTrigger periodic = new PeriodicTrigger(Duration.ofSeconds(5));
periodic.setFixedRate(true);                      // true=fixedRate, false=fixedDelay
periodic.setInitialDelay(Duration.ofSeconds(10)); // optional

taskScheduler.schedule(myTask, cron);
taskScheduler.schedule(myTask, periodic);
```

---

### 22. Cancelling via `ScheduledFuture`

`taskScheduler.schedule(...)` returns a `ScheduledFuture<?>` — your handle to control the task:

```java
ScheduledFuture<?> future = taskScheduler.scheduleAtFixedRate(task, Duration.ofSeconds(5));

future.cancel(false);   // finish current run, then stop  ← prefer this
future.cancel(true);    // interrupt immediately
future.isCancelled();
future.isDone();
```

**Managing multiple tasks by name:**

```java
Map<String, ScheduledFuture<?>> tasks = new ConcurrentHashMap<>();

public void add(String name, Runnable task, String cron) {
    cancel(name);   // cancel old first!
    tasks.put(name, taskScheduler.schedule(task, new CronTrigger(cron)));
}

public void cancel(String name) {
    ScheduledFuture<?> f = tasks.remove(name);
    if (f != null) f.cancel(false);
}
```

> Always use `ConcurrentHashMap` — task operations happen from multiple threads.
> Always `remove` from map when cancelling — prevent memory leak.

---

### 23. Storing Cron in DB — Reload Dynamically

Change schedule at runtime by updating a DB row, not redeploying code.

**DB table:**
```sql
CREATE TABLE task_config (
    task_name VARCHAR(100) PRIMARY KEY,
    cron_expr VARCHAR(100),
    enabled   BOOLEAN DEFAULT TRUE
);
```

**Load on startup + reschedule on change:**

```java
@EventListener(ApplicationReadyEvent.class)  // not @PostConstruct — DB not ready yet
public void loadAll() {
    repo.findByEnabledTrue().forEach(this::schedule);
}

public void schedule(TaskConfig config) {
    ScheduledFuture<?> old = tasks.remove(config.getTaskName());
    if (old != null) old.cancel(false);
    tasks.put(config.getTaskName(),
        taskScheduler.schedule(() -> run(config), new CronTrigger(config.getCronExpr()))
    );
}
```

**Admin endpoint:**
```java
@PutMapping("/{name}/cron")
public void updateCron(@PathVariable String name, @RequestParam String cron) {
    // 1. save to DB, 2. cancel old, 3. start new
}
```

> Use `ApplicationReadyEvent` not `@PostConstruct` — DB is guaranteed ready at that point.

---

## Part D — Preventing Overlap & Concurrency

---

### 24. Preventing Overlapping Executions

`@Scheduled` has **no built-in lock**. If a task takes longer than its interval, executions pile up.

**Solution 1 — `fixedDelay` instead of `fixedRate`:** next run starts only after previous ends.

**Solution 2 — `AtomicBoolean` guard:**

```java
private final AtomicBoolean running = new AtomicBoolean(false);

@Scheduled(fixedRate = 5000)
public void task() {
    if (!running.compareAndSet(false, true)) return; // skip if already running
    try { doWork(); }
    finally { running.set(false); }
}
```

**Solution 3 — ShedLock** (for distributed — see Section 28).

---

### 25. `@Async` + `@Scheduled`

Combines to make each scheduled invocation run in its **own thread** from a pool:

```java
@EnableScheduling
@EnableAsync        // ← required
public class App { }
```

```java
@Scheduled(fixedRate = 5000)
@Async              // each trigger → new thread from pool
public void task() { }
```

> ⚠️ `@Async` can **re-introduce overlap** — each trigger fires a new async task even if previous hasn't finished. Use only when parallel execution is intentional.

---

### 26. Thread Safety in Shared State

If a job runs in multiple threads and touches shared state, you have a race condition.

```java
private int count = 0;

@Scheduled(fixedRate = 1000)
@Async
public void task() {
    count++;  // ❌ not atomic — two threads can read same value simultaneously
}
```

**Fix 1 — `AtomicInteger`:**
```java
private final AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet();  // ✅ atomic
```

**Fix 2 — Stateless jobs (best practice):**
Read from DB → process → write to DB. No shared in-memory state. The DB handles concurrency via transactions.

---

## Part E — Distributed & Cluster Scheduling

---

### 27. The Cluster Problem

If you deploy to 3 servers, `@Scheduled` fires on **all 3 nodes** simultaneously:

```
Server-1: fires job at 8AM  ──┐
Server-2: fires job at 8AM  ──┼── same job runs 3 times! 💥
Server-3: fires job at 8AM  ──┘
```

`@Scheduled` is in-memory only. Each JVM runs its own scheduler independently.

---

### 28. ShedLock — Distributed Lock

ShedLock stores a lock row in a shared DB. Only the node that acquires the lock runs the job.

```java
@EnableSchedulerLock(defaultLockAtMostFor = "10m")
public class App { }
```

```java
@Bean
public LockProvider lockProvider(DataSource ds) {
    return new JdbcTemplateLockProvider(ds);
}
```

```java
@Scheduled(cron = "0 0 8 * * ?")
@SchedulerLock(name = "morningJob", lockAtMostFor = "5m", lockAtLeastFor = "1m")
public void morningJob() { }
```

**Lock timing:**

| Setting | Effect |
|---|---|
| `lockAtMostFor` | Max lock duration — releases even if node crashes (prevents zombie lock) |
| `lockAtLeastFor` | Min lock duration — prevents another node from immediately re-locking |

**DB table needed:**
```sql
CREATE TABLE shedlock (
    name VARCHAR(64) PRIMARY KEY,
    lock_until TIMESTAMP NOT NULL,
    locked_at  TIMESTAMP NOT NULL,
    locked_by  VARCHAR(255) NOT NULL
);
```

---

### 29. ShedLock Backends (DB / Redis)

**JDBC:**
```java
return new JdbcTemplateLockProvider(dataSource);
```

**Redis:**
```java
return new RedisLockProvider(connectionFactory, "my-app");
// Redis uses TTL — lock auto-expires, no cleanup needed
```

| | JDBC | Redis |
|---|---|---|
| Extra infra | Uses existing DB | Needs Redis |
| Expiry | By timestamp | By TTL (auto) |
| Best for | Apps already on RDBMS | Apps already on Redis |

---

### 30. Quartz Clustered Setup

Quartz stores job state in a **shared DB** — all nodes share one job store. Quartz uses DB row-level locking so only one node picks up each trigger.

```properties
spring.quartz.job-store-type=jdbc
spring.quartz.properties.org.quartz.jobStore.isClustered=true
spring.quartz.properties.org.quartz.scheduler.instanceId=AUTO
```

Each node sends heartbeats. If a node's heartbeat is stale, Quartz detects it as failed and **recovers its jobs** on another node.

---

### 31. Quartz vs `@Scheduled`

| Need | Use |
|---|---|
| Simple periodic task, single node | `@Scheduled` |
| Multi-node, no duplicate runs | `@Scheduled` + ShedLock |
| Jobs must survive restarts | Quartz + JDBCJobStore |
| Complex scheduling + misfire recovery | Quartz |
| Full distributed + dashboard | JobRunr |

```
Single node, simple      → @Scheduled
Multi-node, lightweight  → @Scheduled + ShedLock
Multi-node, full power   → Quartz clustered + JDBCJobStore
Modern distributed       → JobRunr
```

---

## Part F — Quartz Deep Dive

---

### 32. Core Quartz Concepts

```
Scheduler  — the engine (starts, stops, manages everything)
  │
  ├── JobDetail  — WHAT to run (class + config + data)
  │       │
  │       └── Job  — the actual work (implements Job interface)
  │
  └── Trigger   — WHEN to run it (cron or simple interval)
```

```java
// Job — the work
public class ReportJob implements Job {
    @Override
    public void execute(JobExecutionContext ctx) {
        String type = ctx.getMergedJobDataMap().getString("type");
        System.out.println("Generating " + type + " report");
    }
}

// JobDetail — the config
JobDetail detail = JobBuilder.newJob(ReportJob.class)
    .withIdentity("reportJob", "group1")
    .usingJobData("type", "daily")
    .storeDurably()
    .build();

// Trigger — the schedule
Trigger trigger = TriggerBuilder.newTrigger()
    .forJob(detail)
    .withSchedule(CronScheduleBuilder.cronSchedule("0 0 8 * * ?"))
    .build();
```

Quartz creates a **new instance** of your Job class per execution — jobs are stateless by design.

One `JobDetail` can have **multiple Triggers** (e.g., run daily AND on-demand).

---

### 33. `CronTrigger` vs `SimpleTrigger` (Quartz)

**`CronTrigger`** — calendar-based (time of day, day of week):
```java
CronScheduleBuilder.cronSchedule("0 0 8 * * MON-FRI")
```

**`SimpleTrigger`** — fixed interval or limited count:
```java
SimpleScheduleBuilder.simpleSchedule()
    .withIntervalInSeconds(30)
    .repeatForever()
```

| | CronTrigger | SimpleTrigger |
|---|---|---|
| Fixed interval | ❌ | ✅ |
| Time of day | ✅ | ❌ |
| Limited runs | ❌ | ✅ |
| Best for | Business schedules | Polling, retries |

---

### 34. `JDBCJobStore`

By default Quartz uses `RAMJobStore` — jobs lost on restart. `JDBCJobStore` persists to DB.

```properties
spring.quartz.job-store-type=jdbc
spring.quartz.jdbc.initialize-schema=always   # auto-create tables (dev)
```

Quartz creates these tables: `QRTZ_JOB_DETAILS`, `QRTZ_TRIGGERS`, `QRTZ_FIRED_TRIGGERS`, `QRTZ_LOCKS`, etc.

After restart → Quartz reads from DB → resumes all jobs from where they left off.

---

### 35. Quartz Misfire Instructions

A **misfire** = trigger's scheduled time passed but couldn't fire (app was down, thread pool full).

**For `CronTrigger`:**

| Instruction | Behavior |
|---|---|
| `withMisfireHandlingInstructionDoNothing()` | Skip missed fires, resume normal |
| `withMisfireHandlingInstructionFireAndProceed()` | Fire once now, resume normal |
| `withMisfireHandlingInstructionIgnoreMisfires()` | Fire all missed times immediately |

```java
CronScheduleBuilder.cronSchedule("0 0 8 * * ?")
    .withMisfireHandlingInstructionFireAndProceed()
```

**For `SimpleTrigger`:** `FIRE_NOW`, `RESCHEDULE_NEXT_WITH_REMAINING_COUNT`, `IGNORE_MISFIRE_POLICY`.

---

### 36. Spring Boot + Quartz Auto-Config

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

Spring Boot auto-configures the `Scheduler` bean and wires Spring's `ApplicationContext` into Quartz — so `@Autowired` works inside Job classes.

Register via beans (auto-detected by Spring Boot):

```java
@Bean
public JobDetail myJobDetail() {
    return JobBuilder.newJob(MyJob.class).withIdentity("myJob").storeDurably().build();
}

@Bean
public Trigger myTrigger(JobDetail myJobDetail) {
    return TriggerBuilder.newTrigger().forJob(myJobDetail)
        .withSchedule(CronScheduleBuilder.cronSchedule("0 0 8 * * ?")).build();
}
```

```properties
spring.quartz.overwrite-existing-jobs=true    # dev: update jobs on restart
spring.batch.job.enabled=false                # prevent auto-run on startup
```

---

### 37. JobRunr — Modern Alternative

A modern Java background job scheduler with a built-in dashboard UI.

```java
// Enqueue now
BackgroundJob.enqueue(() -> emailService.send("user@x.com"));

// Schedule for later
BackgroundJob.schedule(LocalDateTime.now().plusHours(1), () -> reportService.run());

// Recurring
BackgroundJob.scheduleRecurrently("daily-report", Cron.daily(), () -> reportService.run());
```

| Feature | `@Scheduled` | Quartz | JobRunr |
|---|---|---|---|
| Setup | Very easy | Medium | Easy |
| Clustering | ❌ (need ShedLock) | ✅ | ✅ |
| Dashboard | ❌ | ❌ | ✅ |
| Auto-retry | ❌ | Manual | ✅ |
| Lambda API | ❌ | ❌ | ✅ |

---

## Part G — Spring Batch

---

### 38. What is Spring Batch & When to Use It

`@Scheduled` triggers code. Spring Batch **processes large volumes of data reliably** — with restartability, chunking, and progress tracking.

| Use `@Scheduled` | Use Spring Batch |
|---|---|
| Simple periodic cleanup | Process 500K CSV rows |
| Send one reminder email | Monthly billing for millions |
| Short tasks | ETL pipelines |
| No restart needed | Must resume from crash point |

Spring Batch needs a DB for metadata:

```properties
spring.batch.jdbc.initialize-schema=always
spring.batch.job.enabled=false   # don't auto-run on startup
```

---

### 39. Core Batch Concepts

```
Job
 └── Step 1 (Chunk: Reader → Processor → Writer)
 └── Step 2 (Tasklet: one-off operation)
```

**`ItemReader`** — reads one item at a time; returns `null` when done.

**`ItemProcessor`** — transforms one item; return `null` to skip it.

**`ItemWriter`** — receives a full chunk (list) and writes all at once.

```java
// Reader — read one at a time
public Order read() {
    return index < list.size() ? list.get(index++) : null;
}

// Processor — transform or filter
public Order process(Order o) {
    if (o.getAmount() <= 0) return null;  // skip
    o.setStatus("PROCESSED");
    return o;
}

// Writer — write the whole chunk
public void write(Chunk<? extends Order> chunk) {
    repo.saveAll(chunk.getItems());   // one DB call for the whole chunk
}
```

**Built-in readers:** `FlatFileItemReader` (CSV), `JdbcCursorItemReader`, `JpaPagingItemReader`
**Built-in writers:** `JdbcBatchItemWriter`, `JpaItemWriter`, `FlatFileItemWriter`

---

### 40. `JobLauncher`

The entry point to run a Job:

```java
JobParameters params = new JobParametersBuilder()
    .addLong("run.id", System.currentTimeMillis())  // must be unique per run!
    .toJobParameters();

JobExecution exec = jobLauncher.run(myJob, params);
exec.getStatus();  // COMPLETED, FAILED, STARTED...
```

> Same `Job name + JobParameters` = Spring Batch refuses to re-run (considers it done).
> Always add a timestamp or unique ID to params.

For long jobs, use async launcher — `run()` returns immediately with status `STARTED`.

---

### 41. `@Scheduled` + `JobLauncher`

`@Scheduled` = the clock. `JobLauncher` = the starter.

```java
@Scheduled(cron = "0 0 2 * * ?")
public void runBatch() throws Exception {
    JobParameters params = new JobParametersBuilder()
        .addLong("run.id", System.currentTimeMillis())
        .toJobParameters();
    jobLauncher.run(myJob, params);
}
```

In a cluster, wrap with ShedLock so only one node launches the job:

```java
@Scheduled(cron = "0 0 2 * * ?")
@SchedulerLock(name = "myBatch", lockAtMostFor = "2h")
public void runBatch() throws Exception { ... }
```

---

### 42. Chunk-Oriented vs Tasklet

**Chunk-oriented** — for bulk data processing:

```java
new StepBuilder("step", jobRepository)
    .<Order, Order>chunk(100, transactionManager)  // 100 items per transaction
    .reader(reader()).processor(processor()).writer(writer())
    .faultTolerant()
        .skip(InvalidOrderException.class).skipLimit(10)
        .retry(TransientDataAccessException.class).retryLimit(3)
    .build();
```

One transaction per chunk. If chunk 5 fails, chunks 1–4 are committed. Restart resumes from chunk 5.

**Tasklet** — for one-off operations (file move, send email, truncate table):

```java
public class CleanupTasklet implements Tasklet {
    @Override
    public RepeatStatus execute(StepContribution c, ChunkContext ctx) {
        deleteTempFiles();
        return RepeatStatus.FINISHED;
    }
}
```

| | Chunk | Tasklet |
|---|---|---|
| Best for | Bulk data (thousands of records) | One-off steps |
| Transaction | One per chunk | One per execute() |
| Skip/Retry | ✅ built-in | ❌ manual |
| Restart | From last committed chunk | From start of tasklet |

**Real-world job combining both:**

```java
return new JobBuilder("orderJob", jobRepository)
    .start(validateFileStep())    // Tasklet: check file exists
    .next(processOrdersStep())    // Chunk: read CSV → DB
    .next(archiveFileStep())      // Tasklet: move file
    .next(sendEmailStep())        // Tasklet: notify team
    .build();
```

---

## Part H — Production Patterns

---

### 43. Idempotency

An idempotent job produces the **same result no matter how many times it runs** — no duplicates.

**Pattern 1 — Upsert instead of insert:**
```java
// INSERT ... ON DUPLICATE KEY UPDATE  (MySQL)
// INSERT ... ON CONFLICT DO UPDATE    (PostgreSQL)
```

**Pattern 2 — Status check before processing:**
```java
public Order process(Order o) {
    if ("PROCESSED".equals(o.getStatus())) return null; // skip
    o.setStatus("PROCESSED");
    return o;
}
```

**Pattern 3 — Run-ledger table:**
```java
if (jobRunRepo.existsByRunDate(LocalDate.now())) return; // already ran today
```

**Pattern 4 — `processed_at` column — reader filters it out:**
```sql
SELECT * FROM orders WHERE processed_at IS NULL
```

> Hardest case: **emails/notifications** — track in a `notifications_sent` table with a unique constraint on `(user_id, type, date)`.

---

### 44. Retry Logic

**`@Retryable` (spring-retry):**
```java
@Retryable(retryFor = TimeoutException.class, maxAttempts = 3,
           backoff = @Backoff(delay = 1000, multiplier = 2))
public void callApi() { ... }

@Recover
public void recover(TimeoutException e) { /* all retries failed */ }
```

**Spring Batch chunk retry (built-in):**
```java
.faultTolerant()
    .retry(TransientDataAccessException.class).retryLimit(3)
    .skip(InvalidOrderException.class).skipLimit(50)
```

| Use | When |
|---|---|
| `retry` | Transient error (timeout, blip) — may succeed next time |
| `skip` | Bad data — retrying won't help |

**Backoff strategies:**
```
Fixed:       1s → 1s → 1s
Exponential: 1s → 2s → 4s  ← most common
With jitter: 1s±r → 2s±r   ← prevents thundering herd in clusters
```

---

### 45. Job Chaining

**Option 1 — Steps inside one Job (preferred for related operations):**
```java
return new JobBuilder("etlJob", jobRepository)
    .start(extractStep())
    .next(transformStep())
    .next(loadStep())
    .build();
```

**Option 2 — Conditional flow with `JobExecutionDecider`:**
```java
.start(processStep())
.next(decider())
    .on("HAS_ERRORS").to(errorStep())
    .on("NO_ERRORS").to(successStep())
```

**Option 3 — Sequential jobs in `@Scheduled`:**
```java
JobExecution e1 = jobLauncher.run(extractJob, params);
if (e1.getStatus() != BatchStatus.COMPLETED) return;  // abort chain
JobExecution e2 = jobLauncher.run(transformJob, params);
```

---

### 46. Misfire Handling

**`@Scheduled`:** simply **skips** missed executions — no catch-up, no notification.

```
App down 10:00–10:03, job runs every minute
→ 10:01, 10:02 missed → not recovered
→ App restarts → resumes at 10:04
```

**Quartz:** configurable via misfire instructions (Section 35) — can fire missed executions on recovery.

---

### 47. Graceful Shutdown

Prevent jobs from being killed mid-execution during deployments or restarts.

```properties
server.shutdown=graceful
spring.lifecycle.timeout-per-shutdown-phase=120s
```

```java
scheduler.setWaitForTasksToCompleteOnShutdown(true);
scheduler.setAwaitTerminationSeconds(120);
```

For long-running loops, check shutdown signal:
```java
while (hasMore() && ctx.isActive()) {  // ctx = ApplicationContext
    processNext();
}
```

For Spring Batch:
```java
jobOperator.stop(executionId);
// Batch finishes current chunk → status = STOPPED → restartable
```

Kubernetes: set `terminationGracePeriodSeconds` greater than your shutdown timeout.

---

### 48. Logging

**`JobExecutionListener` (Spring Batch):**
```java
public void beforeJob(JobExecution je) {
    log.info("[JOB START] name={} id={}", je.getJobInstance().getJobName(), je.getId());
}

public void afterJob(JobExecution je) {
    long ms = Duration.between(je.getStartTime(), je.getEndTime()).toMillis();
    log.info("[JOB END] status={} duration={}ms read={} write={} skip={}",
        je.getStatus(), ms, readCount, writeCount, skipCount);
}
```

**MDC tagging** — all logs in a job run carry the same `jobRunId`:
```java
public void beforeJob(JobExecution je) {
    MDC.put("jobRunId", String.valueOf(je.getId()));
}
public void afterJob(JobExecution je) { MDC.clear(); }
```

**For plain `@Scheduled`:**
```java
long start = System.currentTimeMillis();
try {
    doWork();
    log.info("[DONE] {}ms", System.currentTimeMillis() - start);
} catch (Exception e) {
    log.error("[FAILED] {}ms - {}", System.currentTimeMillis() - start, e.getMessage());
}
```

---

### 49. Actuator

```properties
management.endpoints.web.exposure.include=health,scheduledtasks,metrics
```

**`/actuator/scheduledtasks`** — lists every `@Scheduled` method with its cron/rate.

**Custom Micrometer metrics:**
```java
meterRegistry.counter("batch.job.success", "job", "orderJob").increment();
meterRegistry.timer("batch.job.duration", "job", "orderJob");
```

Flows to Prometheus → Grafana for dashboards and alerts.

**Custom health indicator:**
```java
@Component
public class JobHealthIndicator implements HealthIndicator {
    public Health health() {
        return lastJobFailed ? Health.down().build() : Health.up().build();
    }
}
```

---

### 50. Alerting

**`JobExecutionListener` → Slack/email:**
```java
public void afterJob(JobExecution je) {
    if (je.getStatus() == BatchStatus.FAILED) {
        alertService.send("🚨 Job FAILED: " + je.getJobInstance().getJobName());
    }
}
```

**`@Scheduled` try/catch:**
```java
try { doWork(); }
catch (Exception e) { alertService.send("Job failed: " + e.getMessage()); }
```

**Dead Man's Switch (Healthchecks.io)** — ping a URL on success; if no ping within window → alert:
```java
// At end of successful run:
restTemplate.getForObject("https://hc-ping.com/your-uuid", String.class);
```

Catches missed runs (app was down, cron misconfigured) — something a Batch listener cannot detect.

---

### 51. Dead Job Detection

A **dead job** = stuck in `STARTED` status in `BATCH_JOB_EXECUTION` because the node crashed.

**Detect stuck jobs:**
```java
@Scheduled(fixedRate = 300000)
public void detectDeadJobs() {
    jobExplorer.getJobNames().forEach(name -> {
        JobExecution exec = jobExplorer.getLastJobExecution(...);
        boolean stuck = exec.getStatus() == BatchStatus.STARTED
            && exec.getStartTime().isBefore(Instant.now().minus(Duration.ofHours(2)));
        if (stuck) alertService.send("💀 Dead job: " + name);
    });
}
```

**Clear the zombie lock:**
```java
jobOperator.abandon(executionId);   // STARTED → ABANDONED → can now relaunch
```

**Detect jobs that never ran today:**
```java
@Scheduled(cron = "0 30 2 * * ?")  // check at 2:30 AM
public void checkJobRan() {
    JobExecution last = jobExplorer.getLastJobExecution(...);
    if (!last.getStartTime().toLocalDate().equals(LocalDate.now())) {
        alertService.send("⚠️ Nightly job didn't run today!");
    }
}
```

| Strategy | Detects |
|---|---|
| Poll STARTED > N hours | Stuck/crashed jobs |
| `jobOperator.abandon()` | Clears zombie STARTED status |
| Check last run date at expected time | Jobs that never started |
| Healthchecks.io ping | Missed runs (external watchdog) |

---

## 🗺️ Architecture Summary

```
SINGLE NODE
───────────
@Scheduled → runs job (single thread by default)
@Scheduled + ThreadPoolTaskScheduler → parallel jobs
@Scheduled + @Async → each trigger in separate pool thread

MULTI-NODE (cluster)
─────────────────────
@Scheduled + ShedLock  → DB/Redis lock, one node runs
Quartz clustered       → shared DB job store, one node picks trigger
JobRunr               → shared DB, built-in dashboard + retry

LARGE DATA PROCESSING
──────────────────────
@Scheduled + JobLauncher → triggers Spring Batch
Spring Batch Job
  → Tasklet steps    (validate input, send email, move files)
  → Chunk steps      (read 500K rows, process, write to DB in chunks of 100)
     Reader → Processor → Writer
     One transaction per chunk
     Skip bad records, retry transient errors
     Resume from last committed chunk on restart
```

---

## 🔑 Master Cheat Sheet

```
ENABLE          @EnableScheduling on @Configuration class

METHOD RULES    public void method()  — void, no params

TIMING
  fixedRate     every N ms from START of last run
  fixedDelay    every N ms from END of last run
  initialDelay  wait N ms before FIRST run (not with cron)
  cron          "sec min hour day month weekday"

EXTERNALIZE     fixedRateString = "${prop}"  (note: String suffix)
                cron = "${prop}"  (no suffix needed)
                cron = "-"        disables the task

CRON FIELDS     sec  min  hour  day  month  weekday
                0    0    9     *    *      MON-FRI
SYMBOLS         *=every  ,=list  -=range  /=step  ?=don't care (day fields)
MACROS          @daily @hourly @weekly @monthly @yearly
LINUX vs SPRING Linux=5 fields,  Spring=6 fields (add sec at start)

THREAD POOL     @Bean ThreadPoolTaskScheduler — poolSize ≥ task count
                inject TaskScheduler interface, not the class

DYNAMIC         taskScheduler.schedule(task, trigger) → ScheduledFuture<?>
                future.cancel(false) — always store the future
                ConcurrentHashMap for multi-task registry

CLUSTER FIX
  Lightweight   @SchedulerLock — lockAtMostFor prevents zombie locks
  Full power    Quartz + JDBCJobStore + isClustered=true

QUARTZ          Job (work) + JobDetail (config) + Trigger (when) + Scheduler (engine)
                JDBCJobStore → persist to DB → survive restarts
                Misfire: FireAndProceed=fire once now, DoNothing=skip

SPRING BATCH    Job → Steps → (ItemReader → ItemProcessor → ItemWriter)
                Chunk = one transaction per N items, restarts from last chunk
                Tasklet = one execute() call, for one-off steps
                JobLauncher.run(job, params) — params must be unique per run
                spring.batch.job.enabled=false — prevent auto-run on startup

PRODUCTION
  Idempotency   upsert, status-check, run-ledger, processed_at column
  Retry         @Retryable(retryFor, maxAttempts, backoff) + @Recover
  Shutdown      server.shutdown=graceful + setWaitForTasksToCompleteOnShutdown
  Logging       JobExecutionListener + MDC tags
  Alerting      afterJob FAILED → Slack; Healthchecks.io for missed runs
  Dead job      STARTED > 2h → alert → jobOperator.abandon() → relaunch
```
