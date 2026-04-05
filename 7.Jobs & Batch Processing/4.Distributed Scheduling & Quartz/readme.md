# 🕐 Spring Scheduling & Quartz

---

## Table of Contents

1. [Quick Recap — What is `@Scheduled`?](#1-quick-recap)
2. [Preventing Overlapping Executions](#2-preventing-overlapping-executions)
3. [`@Async` + `@Scheduled` Combination](#3-async--scheduled)
4. [Thread Safety in Shared State Inside Jobs](#4-thread-safety-in-shared-state)
5. [Misfire Handling — What Happens When a Job is Missed](#5-misfire-handling)
6. [The Cluster Problem](#6-the-cluster-problem)
7. [ShedLock — Annotation-Based Distributed Lock](#7-shedlock)
8. [ShedLock with DB / Redis Backend](#8-shedlock-backends)
9. [Quartz Clustered Setup](#9-quartz-clustered-setup)
10. [JobRunr — Modern Alternative](#10-jobrunr)
11. [Quartz vs `@Scheduled` — When to Use Which](#11-quartz-vs-scheduled)
12. [Core Quartz Concepts](#12-core-quartz-concepts)
13. [CronTrigger vs SimpleTrigger](#13-crontrigger-vs-simpletrigger)
14. [Persisting Jobs to DB (JDBCJobStore)](#14-jdbcjobstore)
15. [Quartz Misfire Instructions](#15-quartz-misfire-instructions)
16. [Spring Boot + Quartz Auto-Configuration](#16-spring-boot--quartz-auto-configuration)

---

## 1. Quick Recap

`@Scheduled` is Spring's built-in annotation to run a method on a fixed schedule.

```java
@Scheduled(fixedRate = 5000)   // runs every 5 seconds
public void myJob() {
    System.out.println("Running...");
}
```

**Key requirement:** You need `@EnableScheduling` on a config class.

```java
@SpringBootApplication
@EnableScheduling
public class App { }
```

**Default behavior:**
- All `@Scheduled` jobs share a **single thread** (the scheduler thread pool has size 1 by default).
- Jobs run **sequentially**, one at a time.

---

## 2. Preventing Overlapping Executions

### The Problem

`@Scheduled` has **no built-in lock**. If a job takes longer than its interval, a second instance starts before the first one finishes.

```
fixedRate = 5 seconds, but job takes 8 seconds

Timeline:
t=0  → Job-1 starts
t=5  → Job-2 starts  ← OVERLAPS with Job-1 still running!
t=8  → Job-1 finishes
t=10 → Job-3 starts
```

This causes **double processing**, data corruption, or DB conflicts.

### Solution 1 — Use `fixedDelay` instead of `fixedRate`

```java
// fixedRate  → next run starts 5s after PREVIOUS START
// fixedDelay → next run starts 5s after PREVIOUS END
@Scheduled(fixedDelay = 5000)
public void myJob() { ... }
```

`fixedDelay` **prevents overlap** because the next run only begins after the current one finishes.

### Solution 2 — Manual Lock with `AtomicBoolean`

```java
private final AtomicBoolean running = new AtomicBoolean(false);

@Scheduled(fixedRate = 5000)
public void myJob() {
    if (!running.compareAndSet(false, true)) {
        System.out.println("Still running, skip this cycle");
        return;
    }
    try {
        doWork();
    } finally {
        running.set(false);
    }
}
```

**`compareAndSet(false, true)`** = "if currently false, set to true and return true" — atomic, no race condition.

### Solution 3 — `ShedLock` (for distributed systems — covered in Section 7)

---

## 3. `@Async` + `@Scheduled` Combination

### Why Combine Them?

By default, `@Scheduled` uses a **single thread**. If Job A takes 10 seconds, Job B waits even if they're unrelated.

Combining `@Async` with `@Scheduled` makes each scheduled invocation run in a **separate thread from a thread pool**.

### Setup

```java
// Step 1: Enable both
@SpringBootApplication
@EnableScheduling
@EnableAsync
public class App { }
```

```java
// Step 2: Configure a thread pool for @Async
@Configuration
public class AsyncConfig {
    @Bean
    public TaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor exec = new ThreadPoolTaskExecutor();
        exec.setCorePoolSize(5);
        exec.setMaxPoolSize(10);
        exec.setQueueCapacity(25);
        exec.initialize();
        return exec;
    }
}
```

```java
// Step 3: Annotate the job
@Scheduled(fixedRate = 5000)
@Async
public void myJob() {
    System.out.println("Running in thread: " + Thread.currentThread().getName());
}
```

### What Happens

```
t=0  → @Scheduled fires → @Async picks it up → pool-thread-1 runs it
t=5  → @Scheduled fires → @Async picks it up → pool-thread-2 runs it (in parallel!)
t=10 → @Scheduled fires → @Async picks it up → pool-thread-3 runs it
```

### ⚠️ Warning

`@Async` + `@Scheduled` can cause **overlapping executions** again! Each trigger fires a new async task even if the previous one hasn't finished.

> Use `@Async` only when parallel execution is intentional (e.g., sending emails to multiple users). For a job that must not overlap, prefer `fixedDelay` or a lock.

---

## 4. Thread Safety in Shared State Inside Jobs

### The Problem

If a scheduled job reads/writes a shared variable and runs in multiple threads (via `@Async` or multiple scheduler instances), you have a **race condition**.

```java
// DANGEROUS — not thread safe
private int counter = 0;

@Scheduled(fixedRate = 1000)
@Async
public void count() {
    counter++;  // read-modify-write is NOT atomic!
    System.out.println(counter);
}
```

Two threads can read `counter = 5` at the same time, both write `6`, and you lose an increment.

### Solution 1 — `AtomicInteger`

```java
private final AtomicInteger counter = new AtomicInteger(0);

@Scheduled(fixedRate = 1000)
@Async
public void count() {
    int val = counter.incrementAndGet();  // atomic
    System.out.println(val);
}
```

### Solution 2 — `synchronized` block

```java
private int counter = 0;

@Scheduled(fixedRate = 1000)
@Async
public synchronized void count() {
    counter++;
    System.out.println(counter);
}
```

> `synchronized` on the method means only one thread runs it at a time — which defeats the purpose of `@Async`. Use it only when truly needed.

### Solution 3 — Stateless Jobs (Best Practice)

The cleanest solution: **don't store state in the job bean**. Read from DB, process, write to DB — all within the job execution. The DB is your shared state, and transactions handle concurrency.

```java
@Scheduled(fixedRate = 5000)
public void processOrders() {
    List<Order> pending = orderRepo.findByStatus("PENDING");
    for (Order o : pending) {
        orderService.process(o);   // logic lives in service
    }
}
```

---

## 5. Misfire Handling — What Happens When a Job is Missed

### What is a Misfire?

A **misfire** occurs when a scheduled job **cannot fire at its expected time** because:
- The application was down
- The thread pool was busy
- System clock skew

```
Job scheduled for 10:00 AM
App was restarting from 9:55 to 10:05

→ 10:00 AM trigger was MISSED → misfire!
```

### `@Scheduled` — No Built-in Misfire Handling

`@Scheduled` simply **skips** missed executions. When the app comes back up, it resumes normally from that point forward. No catch-up.

```
Scheduled every 1 min
App down: 10:00 – 10:03

→ 10:01 missed, 10:02 missed, 10:03 missed
→ App restarts at 10:03 → next run at 10:04 (no catch-up)
```

### When is This a Problem?

- **Report generation** — a daily report at midnight was skipped. No catch-up means the report is simply missing.
- **Payment processing** — a batch at 9 AM missed. Transactions not processed.

For such cases, you need **Quartz** with proper misfire instructions (covered in Section 15).

---

## 6. The Cluster Problem

### The Scenario

You deploy your Spring Boot app on **3 servers** for high availability.

```
Server-1: @Scheduled(cron = "0 0 8 * * ?")  → fires at 8 AM
Server-2: @Scheduled(cron = "0 0 8 * * ?")  → fires at 8 AM
Server-3: @Scheduled(cron = "0 0 8 * * ?")  → fires at 8 AM
```

At 8 AM, **all three servers fire the job**. The same report gets generated 3 times. The same emails go out 3 times. Payments are processed 3 times. 💥

### Why Does This Happen?

`@Scheduled` is **in-memory only**. Each JVM runs its own scheduler independently. There's no coordination between nodes.

### Solutions

| Approach | How it Works |
|---|---|
| **ShedLock** | DB/Redis lock — only one node acquires the lock and runs the job |
| **Quartz Clustered** | Quartz stores job state in DB — only one node claims and executes |
| **Single-node scheduling** | Only one instance has scheduling enabled (fragile, not HA) |

---

## 7. ShedLock — Annotation-Based Distributed Lock

### What is ShedLock?

ShedLock is a **lightweight library** that ensures a scheduled task runs **on only one node at a time** in a cluster, using a lock stored in a shared database (or Redis, etc.).

### How it Works

```
At 8 AM:
Server-1 → tries to acquire lock → SUCCESS → runs the job
Server-2 → tries to acquire lock → FAILS (already held) → skips
Server-3 → tries to acquire lock → FAILS (already held) → skips

Job finishes on Server-1 → lock released
```

The lock is a **row in a DB table** (or a key in Redis).

### Step 1 — Add Dependency

```xml
<!-- pom.xml -->
<dependency>
    <groupId>net.javacrumbs.shedlock</groupId>
    <artifactId>shedlock-spring</artifactId>
    <version>5.10.0</version>
</dependency>

<!-- For JDBC (DB) backend -->
<dependency>
    <groupId>net.javacrumbs.shedlock</groupId>
    <artifactId>shedlock-provider-jdbc-template</artifactId>
    <version>5.10.0</version>
</dependency>
```

### Step 2 — Create Lock Table in DB

```sql
CREATE TABLE shedlock (
    name       VARCHAR(64)  NOT NULL,
    lock_until TIMESTAMP    NOT NULL,
    locked_at  TIMESTAMP    NOT NULL,
    locked_by  VARCHAR(255) NOT NULL,
    PRIMARY KEY (name)
);
```

### Step 3 — Configure ShedLock

```java
@Configuration
@EnableScheduling
@EnableSchedulerLock(defaultLockAtMostFor = "10m")
public class SchedulerConfig {

    @Bean
    public LockProvider lockProvider(DataSource dataSource) {
        return new JdbcTemplateLockProvider(dataSource);
    }
}
```

**`defaultLockAtMostFor`** = maximum time a lock can be held, even if the node crashes. This prevents a **zombie lock** (lock held forever because the node died while holding it).

### Step 4 — Annotate the Job

```java
@Scheduled(cron = "0 0 8 * * ?")
@SchedulerLock(
    name = "morningReportJob",
    lockAtMostFor  = "5m",   // max time to hold the lock
    lockAtLeastFor = "1m"    // min time to hold (prevents immediate re-lock by another node)
)
public void morningReport() {
    System.out.println("Running on: " + InetAddress.getLocalHost().getHostName());
}
```

### Lock Timing Explained

```
lockAtMostFor  = 5m
→ Even if the node crashes mid-job, the lock is released after 5 minutes
→ Prevents zombie lock

lockAtLeastFor = 1m
→ Even if the job finishes in 2 seconds, the lock is held for at least 1 minute
→ Prevents rapid-fire re-execution if the scheduler has clock skew between nodes
```

---

## 8. ShedLock Backends

### Backend 1 — JDBC (Relational DB)

```java
@Bean
public LockProvider lockProvider(DataSource dataSource) {
    return new JdbcTemplateLockProvider(
        JdbcTemplateLockProvider.Configuration.builder()
            .withJdbcTemplate(new JdbcTemplate(dataSource))
            .usingDbTime()  // use DB time, not app server time (avoids clock skew)
            .build()
    );
}
```

**Works with:** MySQL, PostgreSQL, Oracle, H2, etc.

### Backend 2 — Redis

```xml
<dependency>
    <groupId>net.javacrumbs.shedlock</groupId>
    <artifactId>shedlock-provider-redis-spring</artifactId>
    <version>5.10.0</version>
</dependency>
```

```java
@Bean
public LockProvider lockProvider(RedisConnectionFactory connectionFactory) {
    return new RedisLockProvider(connectionFactory, "my-app");
}
```

Redis stores the lock as a key with a TTL (expiry). When the lock expires, it's automatically released — no cleanup needed.

### Comparison

| Feature | JDBC | Redis |
|---|---|---|
| Extra infrastructure | Uses existing DB | Needs Redis |
| Lock expiry | By timestamp | By TTL (automatic) |
| Performance | Slightly slower | Very fast |
| Best for | Apps already using RDBMS | Apps already using Redis |

---

## 9. Quartz Clustered Setup

### Why Quartz for Clustering?

Unlike `@Scheduled` + ShedLock (external lock + in-memory scheduler), Quartz is a **full job scheduling engine** that natively supports clustering via a **shared DB**.

All nodes share one job store (database). Quartz uses row-level DB locking to ensure only **one node** picks up and executes a trigger.

```
DB (Quartz tables):
  QRTZ_TRIGGERS → trigger with next_fire_time = 08:00
  QRTZ_LOCKS    → row-level lock

At 8:00 AM:
  Server-1 → acquires QRTZ_LOCKS row → fires the job → marks trigger done
  Server-2 → tries QRTZ_LOCKS → BLOCKED → waits → sees job already done → skips
  Server-3 → same as Server-2
```

### Configuration

```properties
# application.properties

spring.quartz.job-store-type=jdbc
spring.quartz.properties.org.quartz.jobStore.isClustered=true
spring.quartz.properties.org.quartz.jobStore.clusterCheckinInterval=20000
spring.quartz.properties.org.quartz.scheduler.instanceId=AUTO
spring.quartz.properties.org.quartz.scheduler.instanceName=MyClusteredScheduler
```

| Property | Meaning |
|---|---|
| `isClustered=true` | Enable clustering mode |
| `clusterCheckinInterval` | How often (ms) nodes check in to mark themselves alive |
| `instanceId=AUTO` | Each node gets a unique ID (hostname + timestamp) |

### How Node Failure is Detected

Each node writes a heartbeat (`clusterCheckinInterval = 20s`). If a node's heartbeat is stale (missed by `clusterCheckinInterval * 2`), Quartz assumes it failed and **recovers its jobs** on another node.

---

## 10. JobRunr — Modern Alternative Overview

### What is JobRunr?

JobRunr is a **modern Java job scheduling library** (inspired by Hangfire from .NET). It lets you schedule background jobs with minimal boilerplate, with a nice **dashboard UI**.

### Key Features

| Feature | Detail |
|---|---|
| Dashboard | Built-in web UI to see jobs, retries, failures |
| Distributed | Works in cluster out of the box |
| Retry logic | Automatic retries with backoff |
| Lambda-based API | Schedule jobs as Java lambdas |
| Persistent | Jobs stored in DB or Redis |

### Quick Example

```java
// Fire a job in the background right now
BackgroundJob.enqueue(() -> emailService.send("hello@example.com"));

// Schedule for later
BackgroundJob.schedule(
    LocalDateTime.now().plusHours(1),
    () -> reportService.generate("daily")
);

// Recurring job (like cron)
BackgroundJob.scheduleRecurrently(
    "daily-report",
    Cron.daily(),
    () -> reportService.generate("daily")
);
```

### JobRunr vs Quartz vs @Scheduled

| Feature | `@Scheduled` | Quartz | JobRunr |
|---|---|---|---|
| Setup complexity | Very easy | Medium | Easy |
| Clustering | ❌ (need ShedLock) | ✅ native | ✅ native |
| Dashboard | ❌ | ❌ (3rd party) | ✅ built-in |
| Retry on failure | ❌ | Manual | ✅ automatic |
| Job as Lambda | ❌ | ❌ | ✅ |
| Best for | Simple single-node | Complex enterprise | Modern distributed |

---

## 11. Quartz vs `@Scheduled` — When to Use Which

### Use `@Scheduled` When:

- Single-node application (or don't care about duplicate runs)
- Simple, periodic tasks (health checks, cache warmup, cleanup)
- Quick setup needed, no extra dependencies
- In-memory is fine (jobs don't need to survive restarts)

```java
// Perfect use case for @Scheduled
@Scheduled(fixedRate = 60000)
public void evictExpiredCache() {
    cacheManager.evictExpired();
}
```

### Use Quartz When:

- **Clustered** environment and job must run exactly once
- Jobs must **survive application restart** (persisted to DB)
- Complex scheduling (cron, calendar-based, one-off, chains)
- Need **misfire recovery** (catch up on missed jobs)
- Jobs have **data/parameters** that need to be stored

```
Example: Nightly billing job
  - Must run exactly once at 11 PM, even in a 5-node cluster
  - If server crashes mid-job, another node should recover it
  → Use Quartz with JDBCJobStore + clustering
```

### Decision Tree

```
Is it a simple, periodic task?
  └── Yes → Is the app single-node or are duplicates OK?
               └── Yes → @Scheduled  ✅
               └── No  → @Scheduled + ShedLock  ✅
  └── No  → Does the job need to survive restarts / recover from crashes?
               └── No  → @Scheduled + ShedLock  ✅
               └── Yes → Quartz with JDBCJobStore  ✅
```

---

## 12. Core Quartz Concepts

### The Four Core Building Blocks

```
┌─────────────────────────────────────────────────────┐
│                    SCHEDULER                        │
│  (The engine — starts, stops, manages everything)   │
│                                                     │
│   JobDetail ──────────── Trigger                    │
│   (WHAT to run)          (WHEN to run it)           │
│        │                                            │
│      Job                                            │
│   (The actual work)                                 │
└─────────────────────────────────────────────────────┘
```

### `Job` Interface

The actual work your job does. Implement the `execute` method.

```java
public class SendEmailJob implements Job {
    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        String email = context.getMergedJobDataMap().getString("email");
        System.out.println("Sending email to: " + email);
    }
}
```

Quartz creates a **new instance** of your Job class for every execution. So Jobs are **stateless by design**.

### `JobDetail`

`JobDetail` is the **configuration** of a job — it defines:
- Which class to instantiate (`SendEmailJob`)
- A unique name + group
- `JobDataMap` — parameters to pass to the job

```java
JobDetail jobDetail = JobBuilder.newJob(SendEmailJob.class)
    .withIdentity("emailJob", "group1")
    .usingJobData("email", "user@example.com")  // passed to execute()
    .storeDurably()  // keep the job even if no trigger is associated
    .build();
```

### `Trigger`

`Trigger` defines **when** a job fires — the schedule.

```java
Trigger trigger = TriggerBuilder.newTrigger()
    .withIdentity("emailTrigger", "group1")
    .forJob(jobDetail)
    .withSchedule(CronScheduleBuilder.cronSchedule("0 0 8 * * ?"))  // 8 AM daily
    .build();
```

### `Scheduler`

The `Scheduler` is the **engine** that manages everything. You register jobs and triggers with it.

```java
Scheduler scheduler = new StdSchedulerFactory().getScheduler();
scheduler.start();
scheduler.scheduleJob(jobDetail, trigger);
```

In Spring Boot, Quartz auto-creates the Scheduler as a bean — you just inject it.

### Relationship Summary

```
Scheduler holds many → JobDetails
Scheduler holds many → Triggers
Each Trigger fires   → one Job
One JobDetail can    → have multiple Triggers (e.g., daily AND weekly)
```

---

## 13. `CronTrigger` vs `SimpleTrigger`

### `SimpleTrigger` — Fixed Intervals

Use when you want to fire at fixed intervals or a fixed number of times.

```java
// Fire every 10 seconds, forever
SimpleScheduleBuilder schedule = SimpleScheduleBuilder
    .simpleSchedule()
    .withIntervalInSeconds(10)
    .repeatForever();

Trigger trigger = TriggerBuilder.newTrigger()
    .withSchedule(schedule)
    .build();
```

```java
// Fire exactly 5 times, every 30 seconds
SimpleScheduleBuilder schedule = SimpleScheduleBuilder
    .simpleSchedule()
    .withIntervalInSeconds(30)
    .withRepeatCount(5);       // 0 means once, 5 means 6 total fires (1 initial + 5 repeats)
```

### `CronTrigger` — Calendar-Based Scheduling

Use when you need time-of-day, day-of-week, or complex scheduling logic.

```java
CronTrigger trigger = TriggerBuilder.newTrigger()
    .withSchedule(CronScheduleBuilder.cronSchedule("0 0 8 * * MON-FRI"))
    .build();
```

### Cron Expression Format

```
┌───────────── second       (0-59)
│ ┌─────────── minute       (0-59)
│ │ ┌───────── hour         (0-23)
│ │ │ ┌─────── day-of-month (1-31)
│ │ │ │ ┌───── month        (1-12 or JAN-DEC)
│ │ │ │ │ ┌─── day-of-week  (1-7 or SUN-SAT)
│ │ │ │ │ │
0 0 8 * * MON-FRI
```

| Expression | Meaning |
|---|---|
| `0 0 8 * * ?` | 8 AM every day |
| `0 0/30 * * * ?` | Every 30 minutes |
| `0 0 0 1 * ?` | Midnight on 1st of every month |
| `0 0 8 * * MON-FRI` | 8 AM on weekdays only |
| `0 0 8,20 * * ?` | 8 AM and 8 PM every day |

### Comparison

| Feature | SimpleTrigger | CronTrigger |
|---|---|---|
| Fixed interval | ✅ | ❌ |
| Limited fire count | ✅ | ❌ |
| Time of day | ❌ | ✅ |
| Day of week | ❌ | ✅ |
| Best for | Polling, retries | Business schedules |

---

## 14. Persisting Jobs to DB (`JDBCJobStore`)

### Why Persist?

By default, Quartz uses `RAMJobStore` — all job/trigger data lives in memory. When the app restarts, **all jobs are lost**.

`JDBCJobStore` persists all job/trigger data to a **relational database**. After a restart, Quartz reads them back and resumes.

### What Gets Stored?

```
QRTZ_JOB_DETAILS      → your JobDetail (class name, data map)
QRTZ_TRIGGERS         → your Triggers (cron/simple, next fire time)
QRTZ_CRON_TRIGGERS    → cron expression details
QRTZ_SIMPLE_TRIGGERS  → interval/repeat count details
QRTZ_FIRED_TRIGGERS   → currently running triggers (for crash recovery)
QRTZ_SCHEDULER_STATE  → heartbeats per node (for clustering)
QRTZ_LOCKS            → row-level locks (for clustering)
```

### Setup

**Step 1 — Create Quartz tables in DB**

Quartz provides SQL scripts for each DB. For MySQL:

```sql
-- Run the Quartz-provided MySQL schema (tables_mysql.sql from quartz download)
-- Key tables created:
CREATE TABLE QRTZ_JOB_DETAILS (...);
CREATE TABLE QRTZ_TRIGGERS (...);
CREATE TABLE QRTZ_CRON_TRIGGERS (...);
-- ...etc
```

**Step 2 — Configure Spring Boot**

```properties
spring.quartz.job-store-type=jdbc
spring.quartz.jdbc.initialize-schema=never   # or 'always' for auto-create (dev only)
spring.quartz.properties.org.quartz.jobStore.class=org.quartz.impl.jdbcjobstore.JobStoreTX
spring.quartz.properties.org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.StdJDBCDelegate
spring.quartz.properties.org.quartz.jobStore.dataSource=myDS
spring.quartz.properties.org.quartz.jobStore.tablePrefix=QRTZ_
```

**Step 3 — Make Job `Serializable` (or use `JobDataMap` carefully)**

```java
// Option A: Job class itself doesn't need to be serializable,
// but JobDataMap values must be primitives or Strings

JobDetail job = JobBuilder.newJob(ReportJob.class)
    .usingJobData("reportType", "daily")   // String — fine
    .usingJobData("limit", 100)             // int — fine
    .build();
```

---

## 15. Quartz Misfire Instructions

### What is a Misfire (in Quartz)?

A **misfire** in Quartz happens when a trigger's scheduled fire time passes, but the trigger **couldn't fire** because:

1. The application was down
2. The thread pool had no available threads
3. The job was paused

Quartz detects misfires using a `misfireThreshold` (default: 60 seconds). If a trigger is late by more than this, it's a misfire.

### Misfire Instructions for `CronTrigger`

```java
CronScheduleBuilder schedule = CronScheduleBuilder
    .cronSchedule("0 0 8 * * ?")
    .withMisfireHandlingInstructionDoNothing();   // ← misfire instruction
```

| Instruction | Behavior |
|---|---|
| `withMisfireHandlingInstructionDoNothing()` | Ignore missed fires, wait for next scheduled time |
| `withMisfireHandlingInstructionFireAndProceed()` | Fire once now (catch up), then resume normal schedule |
| `withMisfireHandlingInstructionIgnoreMisfires()` | Fire all missed occurrences immediately (catch-up mode) |

### Misfire Instructions for `SimpleTrigger`

| Instruction | Behavior |
|---|---|
| `MISFIRE_INSTRUCTION_FIRE_NOW` | Fire immediately, then resume normal interval |
| `MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_REMAINING_COUNT` | Skip missed, reschedule with original repeat count intact |
| `MISFIRE_INSTRUCTION_RESCHEDULE_NEXT_WITH_EXISTING_COUNT` | Skip missed, fire at next interval (most common) |
| `MISFIRE_INSTRUCTION_IGNORE_MISFIRE_POLICY` | Fire all missed occurrences in rapid succession |

### Practical Example

```java
// Daily report job — if missed, fire once and continue normally
CronTrigger trigger = TriggerBuilder.newTrigger()
    .withSchedule(
        CronScheduleBuilder.cronSchedule("0 0 8 * * ?")
            .withMisfireHandlingInstructionFireAndProceed()
    )
    .build();
```

```java
// Heartbeat job — if missed, just skip. Don't catch up.
CronTrigger trigger = TriggerBuilder.newTrigger()
    .withSchedule(
        CronScheduleBuilder.cronSchedule("0 * * * * ?")
            .withMisfireHandlingInstructionDoNothing()
    )
    .build();
```

---

## 16. Spring Boot + Quartz Auto-Configuration

### Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

Spring Boot auto-configures:
- A `Scheduler` bean
- `JobStore` (RAM or JDBC based on config)
- Wires Spring's `ApplicationContext` into Quartz (so you can `@Autowired` inside Jobs!)

### Making `@Autowired` Work Inside Quartz Jobs

Quartz creates job instances with `new`. Spring's `@Autowired` won't work by default. Spring Boot solves this with `SpringBeanJobFactory`:

```java
// Spring Boot does this automatically via QuartzAutoConfiguration
// You can @Autowired inside your Job like this:

public class ReportJob implements Job {

    @Autowired
    private ReportService reportService;  // ✅ works in Spring Boot!

    @Override
    public void execute(JobExecutionContext ctx) {
        reportService.generateDailyReport();
    }
}
```

### Registering Jobs with Spring Boot

Instead of manually calling `scheduler.scheduleJob()`, Spring Boot lets you register via `@Bean`:

```java
@Configuration
public class QuartzConfig {

    @Bean
    public JobDetail reportJobDetail() {
        return JobBuilder.newJob(ReportJob.class)
            .withIdentity("reportJob")
            .storeDurably()
            .build();
    }

    @Bean
    public Trigger reportJobTrigger(JobDetail reportJobDetail) {
        return TriggerBuilder.newTrigger()
            .forJob(reportJobDetail)
            .withIdentity("reportTrigger")
            .withSchedule(CronScheduleBuilder.cronSchedule("0 0 8 * * ?"))
            .build();
    }
}
```

Spring Boot's `QuartzAutoConfiguration` detects all `JobDetail` and `Trigger` beans and registers them automatically.

### Full `application.properties` Reference

```properties
# Job store type: memory (default) or jdbc
spring.quartz.job-store-type=jdbc

# Auto-create Quartz tables (use 'always' for dev, 'never' for prod)
spring.quartz.jdbc.initialize-schema=always

# Clustering
spring.quartz.properties.org.quartz.jobStore.isClustered=true
spring.quartz.properties.org.quartz.scheduler.instanceId=AUTO
spring.quartz.properties.org.quartz.scheduler.instanceName=MyApp

# Thread pool size (how many jobs can run in parallel)
spring.quartz.properties.org.quartz.threadPool.threadCount=5

# Misfire threshold (ms) — triggers late by more than this = misfire
spring.quartz.properties.org.quartz.jobStore.misfireThreshold=60000

# Overwrite existing jobs on startup (useful in dev)
spring.quartz.overwrite-existing-jobs=true
```

---

## 🗺️ Big Picture — How Everything Fits Together

```
SINGLE NODE, SIMPLE
────────────────────
App → @Scheduled → runs job every N seconds
      (in-memory, single thread, no clustering)


SINGLE NODE, PARALLEL
──────────────────────
App → @Scheduled + @Async → each trigger runs in its own thread
      (still in-memory, but concurrent)


MULTI-NODE, LIGHTWEIGHT
────────────────────────
Nodes → @Scheduled + ShedLock → DB/Redis lock → only one node runs
         (in-memory scheduler, distributed lock, simple setup)


MULTI-NODE, FULL POWER
──────────────────────
Nodes → Quartz → JDBCJobStore (shared DB) → only one node picks trigger
         (persisted jobs, misfire recovery, cluster-aware)


MULTI-NODE, MODERN
───────────────────
Nodes → JobRunr → shared DB/Redis → only one node runs each job
         (lambda-based API, built-in dashboard, auto-retry)
```

---

## 🔑 Key Takeaways

| Topic | Remember This |
|---|---|
| `@Scheduled` overlap | No built-in lock — use `fixedDelay` or `AtomicBoolean` |
| `@Async` + `@Scheduled` | Each trigger fires in a new thread — can still overlap |
| Thread safety | Make jobs stateless; use `AtomicXxx` if you must share state |
| Misfire (`@Scheduled`) | Simply skips missed executions — no catch-up |
| Cluster problem | All nodes fire — need ShedLock or Quartz clustering |
| ShedLock | Lightweight DB/Redis lock — easiest cluster fix |
| ShedLock `lockAtMostFor` | Prevents zombie locks if a node crashes mid-job |
| Quartz core | Job (work) + JobDetail (config) + Trigger (when) + Scheduler (engine) |
| CronTrigger | Calendar-based: time-of-day, day-of-week |
| SimpleTrigger | Fixed interval or limited fire count |
| JDBCJobStore | Jobs survive restarts; enables Quartz clustering |
| Quartz misfire | `FireAndProceed` = fire once now; `DoNothing` = skip; `Ignore` = fire all |
| Spring Boot Quartz | Auto-config wires Spring context into Jobs — `@Autowired` works! |
| ShedLock vs Quartz | ShedLock = simpler lock; Quartz = full engine with persistence |
