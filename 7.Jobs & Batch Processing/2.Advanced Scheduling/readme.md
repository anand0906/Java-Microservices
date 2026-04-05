# ⚙️ Spring Boot — Advanced Scheduling (Thread Pool & Dynamic Tasks)

---

## 📌 Table of Contents

1. [Default Single-Thread Problem](#1-default-single-thread-problem)
2. [ThreadPoolTaskScheduler — Configuring Thread Pool](#2-threadpooltaskscheduler--configuring-thread-pool)
3. [SchedulingConfigurer — Programmatic Configuration](#3-schedulingconfigurer--programmatic-configuration)
4. [TaskScheduler and TaskExecutor Interfaces](#4-taskscheduler-and-taskexecutor-interfaces)
5. [Scheduling Tasks at Runtime (Dynamic Scheduling)](#5-scheduling-tasks-at-runtime-dynamic-scheduling)
6. [CronTrigger and PeriodicTrigger](#6-crontrigger-and-periodictrigger)
7. [Cancelling a Task via ScheduledFuture](#7-cancelling-a-task-via-scheduledfuture)
8. [Storing Cron in DB and Reloading Dynamically](#8-storing-cron-in-db-and-reloading-dynamically)
9. [Common Mistakes & Fixes](#9-common-mistakes--fixes)
10. [Quick Reference Cheat Sheet](#10-quick-reference-cheat-sheet)

---

## 1. Default Single-Thread Problem

### What Happens By Default?

Spring uses **one single thread** for ALL `@Scheduled` tasks.

```
Thread-1 → Task A, Task B, Task C   ← all share ONE thread
```

### Why Is This Dangerous?

If one task is slow, it **blocks** all others.

```java
@Scheduled(fixedRate = 3000)
public void fastTask() {
    System.out.println("Fast task");   // should run every 3s
}

@Scheduled(fixedRate = 3000)
public void slowTask() throws Exception {
    Thread.sleep(10000);               // takes 10 seconds!
}
```

**What actually happens:**

```
00:00  fastTask  ✅
00:00  slowTask starts...
00:10  slowTask ends         ← blocked for 10s
00:10  fastTask runs         ← was supposed to run at 00:03, 00:06, 00:09 !
```

> 🔴 `fastTask` missed 3 executions because it was waiting for `slowTask` to finish.

### The Fix

Use a **thread pool** — multiple threads so tasks run in parallel.

```
Thread-1 → fastTask
Thread-2 → slowTask    ← runs independently, no blocking
```

---

## 2. `ThreadPoolTaskScheduler` — Configuring Thread Pool

### What Is It?

A Spring bean that replaces the default single thread with a **pool of threads**.

Just declare this bean and Spring automatically uses it for all `@Scheduled` tasks.

### Setup

```java
@Configuration
@EnableScheduling
public class SchedulerConfig {

    @Bean
    public ThreadPoolTaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(5);                        // 5 threads
        scheduler.setThreadNamePrefix("my-scheduler-"); // thread names in logs
        scheduler.initialize();
        return scheduler;
    }
}
```

That's all! Now your `@Scheduled` tasks run on 5 threads instead of 1.

### Thread Names in Logs

```
// Before (single thread):
[scheduling-1] Fast task
[scheduling-1] Slow task   ← same thread = they block each other

// After (thread pool):
[my-scheduler-1] Fast task
[my-scheduler-2] Slow task  ← different threads = run in parallel ✅
```

### How Many Threads?

Simple rule: **at least as many threads as your number of scheduled tasks**.

```
3 scheduled tasks → poolSize = 3 (minimum), 5 (safe)
```

### Useful Extra Options

```java
scheduler.setWaitForTasksToCompleteOnShutdown(true); // finish running tasks on app stop
scheduler.setAwaitTerminationSeconds(30);            // wait max 30s during shutdown
```

---

## 3. `SchedulingConfigurer` — Programmatic Configuration

### What Is It?

An interface you implement to configure scheduling **in code** — alternative to declaring a `@Bean`.

```java
@Configuration
@EnableScheduling
public class SchedulerConfig implements SchedulingConfigurer {

    @Override
    public void configureTasks(ScheduledTaskRegistrar registrar) {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(5);
        scheduler.setThreadNamePrefix("app-");
        scheduler.initialize();

        registrar.setTaskScheduler(scheduler);  // plug in the thread pool
    }
}
```

### Bonus — Register Tasks in Code (No `@Scheduled` Needed)

```java
@Override
public void configureTasks(ScheduledTaskRegistrar registrar) {
    // ... set scheduler ...

    // add a cron task in code
    registrar.addCronTask(
        () -> System.out.println("Hello from code!"),
        "0 0 9 * * *"
    );

    // add a fixed-rate task in code
    registrar.addFixedRateTask(
        () -> System.out.println("Every 5s"),
        5000
    );
}
```

### `@Bean` vs `SchedulingConfigurer` — Which to Use?

| Goal | Approach |
|---|---|
| Just need a thread pool | `@Bean ThreadPoolTaskScheduler` |
| Also want to register tasks in code | `SchedulingConfigurer` |

---

## 4. `TaskScheduler` and `TaskExecutor` Interfaces

### Two Interfaces, One Purpose: Background Execution

| Interface | What It Does |
|---|---|
| `TaskScheduler` | Run a task **at a specific time or interval** |
| `TaskExecutor` | Run a task **immediately in background** (no scheduling) |

`ThreadPoolTaskScheduler` implements **both** — one bean, two roles.

### `TaskScheduler` — Key Methods

```java
// run once, 10 seconds from now
taskScheduler.schedule(task, Instant.now().plusSeconds(10));

// run every 5 seconds (fixedRate style)
taskScheduler.scheduleAtFixedRate(task, Duration.ofSeconds(5));

// run every 5 seconds (fixedDelay style)
taskScheduler.scheduleWithFixedDelay(task, Duration.ofSeconds(5));

// run on a cron or periodic trigger
taskScheduler.schedule(task, trigger);
```

### How to Inject It

```java
@Component
public class MyService {

    private final TaskScheduler taskScheduler;

    public MyService(TaskScheduler taskScheduler) {  // Spring injects the bean
        this.taskScheduler = taskScheduler;
    }
}
```

> ✅ Inject `TaskScheduler` (interface), NOT `ThreadPoolTaskScheduler` (class).
> This makes the code easier to test and swap.

---

## 5. Scheduling Tasks at Runtime (Dynamic Scheduling)

### Static vs Dynamic

```
Static  → @Scheduled(fixedRate = 5000)  — fixed in code, needs redeploy to change
Dynamic → taskScheduler.schedule(...)   — created at runtime, can change anytime
```

### Simple Dynamic Scheduler

```java
@Component
public class DynamicScheduler {

    private final TaskScheduler taskScheduler;
    private ScheduledFuture<?> future;  // handle to the running task

    public DynamicScheduler(TaskScheduler taskScheduler) {
        this.taskScheduler = taskScheduler;
    }

    public void start(long intervalSeconds) {
        future = taskScheduler.scheduleAtFixedRate(
            () -> System.out.println("Running: " + LocalTime.now()),
            Duration.ofSeconds(intervalSeconds)
        );
    }

    public void stop() {
        if (future != null) future.cancel(false);
    }

    public void reschedule(long newIntervalSeconds) {
        stop();                    // cancel old
        start(newIntervalSeconds); // start new
    }
}
```

### Triggering via REST

```java
@RestController
public class SchedulerController {

    private final DynamicScheduler scheduler;

    public SchedulerController(DynamicScheduler scheduler) {
        this.scheduler = scheduler;
    }

    @PostMapping("/start/{seconds}")
    public String start(@PathVariable long seconds) {
        scheduler.start(seconds);
        return "Started every " + seconds + "s";
    }

    @PostMapping("/stop")
    public String stop() {
        scheduler.stop();
        return "Stopped";
    }
}
```

```
POST /start/5  → task runs every 5 seconds
POST /stop     → task stops
```

---

## 6. `CronTrigger` and `PeriodicTrigger`

When scheduling dynamically, you pass a **Trigger** object to define timing.

### `CronTrigger` — Cron-Based Timing

```java
// same as: @Scheduled(cron = "0 0 9 * * MON-FRI")
CronTrigger trigger = new CronTrigger("0 0 9 * * MON-FRI");

taskScheduler.schedule(
    () -> System.out.println("9 AM weekday task"),
    trigger
);
```

With timezone:

```java
CronTrigger trigger = new CronTrigger(
    "0 0 9 * * *",
    TimeZone.getTimeZone("Asia/Kolkata")
);
```

### `PeriodicTrigger` — Fixed Interval Timing

```java
// same as: @Scheduled(fixedRate = 5000)
PeriodicTrigger trigger = new PeriodicTrigger(Duration.ofSeconds(5));
trigger.setFixedRate(true);    // true = fixedRate, false = fixedDelay
trigger.setInitialDelay(Duration.ofSeconds(10));  // optional: wait before first run

taskScheduler.schedule(() -> System.out.println("Periodic"), trigger);
```

### Quick Comparison

| | `CronTrigger` | `PeriodicTrigger` |
|---|---|---|
| Use for | Any cron pattern | Simple intervals |
| Equivalent annotation | `@Scheduled(cron=...)` | `@Scheduled(fixedRate=...)` |

---

## 7. Cancelling a Task via `ScheduledFuture`

### What Is `ScheduledFuture`?

Every `taskScheduler.schedule(...)` returns a `ScheduledFuture<?>` — a **handle** to control the running task.

```java
ScheduledFuture<?> future = taskScheduler.scheduleAtFixedRate(task, Duration.ofSeconds(5));

future.cancel(false);   // stop the task
future.isCancelled();   // is it cancelled?
future.isDone();        // is it finished or cancelled?
```

### `cancel(false)` vs `cancel(true)`

```java
future.cancel(false);  // wait for current run to finish, then stop  ← use this
future.cancel(true);   // interrupt thread immediately
```

### Managing Multiple Tasks by Name

```java
@Component
public class TaskRegistry {

    private final TaskScheduler taskScheduler;
    private final Map<String, ScheduledFuture<?>> tasks = new ConcurrentHashMap<>();

    public TaskRegistry(TaskScheduler taskScheduler) {
        this.taskScheduler = taskScheduler;
    }

    public void add(String name, Runnable task, String cron) {
        cancel(name);  // cancel existing first to avoid duplicates
        ScheduledFuture<?> future = taskScheduler.schedule(task, new CronTrigger(cron));
        tasks.put(name, future);
    }

    public void cancel(String name) {
        ScheduledFuture<?> f = tasks.remove(name);  // remove + cancel
        if (f != null) f.cancel(false);
    }

    public String status(String name) {
        ScheduledFuture<?> f = tasks.get(name);
        if (f == null)        return "NOT_FOUND";
        if (f.isCancelled())  return "CANCELLED";
        return "RUNNING";
    }
}
```

> ⚠️ Use `ConcurrentHashMap` — multiple threads may add/cancel tasks simultaneously.

> ⚠️ Call `tasks.remove(name)` when cancelling — otherwise the map grows forever (memory leak).

---

## 8. Storing Cron in DB and Reloading Dynamically

### Why Store Cron in DB?

| Hardcoded cron | DB-stored cron |
|---|---|
| Change = redeploy | Change = update DB row |
| Can't enable/disable at runtime | Just flip `enabled` column |

### DB Table

```sql
CREATE TABLE task_config (
    task_name  VARCHAR(100) PRIMARY KEY,
    cron_expr  VARCHAR(100) NOT NULL,
    enabled    BOOLEAN DEFAULT TRUE
);

INSERT INTO task_config VALUES ('daily-report', '0 0 9 * * *',   true);
INSERT INTO task_config VALUES ('cleanup',      '0 0 0 * * SUN', true);
```

### Entity & Repository

```java
@Entity
public class TaskConfig {
    @Id
    private String taskName;
    private String cronExpr;
    private boolean enabled;
    // getters / setters
}

public interface TaskConfigRepo extends JpaRepository<TaskConfig, String> {
    List<TaskConfig> findByEnabledTrue();
}
```

### Load All Tasks on Startup

```java
@Component
public class DynamicTaskLoader {

    private final TaskScheduler taskScheduler;
    private final TaskConfigRepo repo;
    private final Map<String, ScheduledFuture<?>> tasks = new ConcurrentHashMap<>();

    public DynamicTaskLoader(TaskScheduler taskScheduler, TaskConfigRepo repo) {
        this.taskScheduler = taskScheduler;
        this.repo = repo;
    }

    // Use ApplicationReadyEvent — DB is fully ready at this point
    @EventListener(ApplicationReadyEvent.class)
    public void loadAll() {
        repo.findByEnabledTrue().forEach(this::schedule);
    }

    public void schedule(TaskConfig config) {
        // cancel old if running, then start fresh
        ScheduledFuture<?> old = tasks.remove(config.getTaskName());
        if (old != null) old.cancel(false);

        ScheduledFuture<?> future = taskScheduler.schedule(
            () -> System.out.println("[" + config.getTaskName() + "] ran"),
            new CronTrigger(config.getCronExpr())
        );
        tasks.put(config.getTaskName(), future);
    }
}
```

### Admin Endpoint — Change Cron Without Restart

```java
@RestController
@RequestMapping("/admin/tasks")
public class TaskAdminController {

    private final TaskConfigRepo repo;
    private final DynamicTaskLoader loader;

    public TaskAdminController(TaskConfigRepo repo, DynamicTaskLoader loader) {
        this.repo = repo;
        this.loader = loader;
    }

    @PutMapping("/{name}/cron")
    public String updateCron(@PathVariable String name, @RequestParam String cron) {
        repo.findById(name).ifPresent(config -> {
            config.setCronExpr(cron);
            repo.save(config);        // 1. save to DB
            loader.schedule(config);  // 2. cancel old + start new
        });
        return "Updated: " + name + " → " + cron;
    }
}
```

```
PUT /admin/tasks/daily-report/cron?cron=0 0 8 * * *
→ daily-report now runs at 8 AM — no restart needed!
```

### Full Flow

```
App starts
  └── ApplicationReadyEvent → reads DB → schedules all enabled tasks

Admin calls PUT /admin/tasks/{name}/cron
  └── saves new cron to DB
        └── cancels old ScheduledFuture
              └── creates new ScheduledFuture with new cron
```

---

## 9. Common Mistakes & Fixes

### ❌ No thread pool — tasks block each other
```java
// No ThreadPoolTaskScheduler bean → default single thread
```
✅ Always define `ThreadPoolTaskScheduler` with `poolSize > 1`.

---

### ❌ Losing the `ScheduledFuture` — can never cancel
```java
taskScheduler.scheduleAtFixedRate(task, Duration.ofSeconds(5));  // future lost!
```
✅ Store it:
```java
ScheduledFuture<?> future = taskScheduler.scheduleAtFixedRate(task, Duration.ofSeconds(5));
```

---

### ❌ Rescheduling without cancelling — two tasks run at once
```java
future = taskScheduler.schedule(task, new CronTrigger(cron));  // old still running!
```
✅ Cancel first:
```java
if (future != null) future.cancel(false);
future = taskScheduler.schedule(task, new CronTrigger(cron));
```

---

### ❌ Using `HashMap` — not thread-safe
```java
Map<String, ScheduledFuture<?>> tasks = new HashMap<>();  // ❌
```
✅ Use:
```java
Map<String, ScheduledFuture<?>> tasks = new ConcurrentHashMap<>(); // ✅
```

---

### ❌ Loading DB tasks in `@PostConstruct` — DB may not be ready
```java
@PostConstruct
public void load() { repo.findAll(); }  // ❌ DB may not be ready yet
```
✅ Use `ApplicationReadyEvent`:
```java
@EventListener(ApplicationReadyEvent.class)
public void load() { repo.findAll(); }  // ✅ DB is fully ready
```

---

### ❌ Injecting the class instead of the interface
```java
@Autowired ThreadPoolTaskScheduler scheduler;  // ❌ tightly coupled
```
✅ Use the interface:
```java
@Autowired TaskScheduler scheduler;  // ✅ easy to test and swap
```

---

## 10. Quick Reference Cheat Sheet

```
THREAD POOL SETUP
─────────────────────────────────────────────────────
@Bean ThreadPoolTaskScheduler taskScheduler() {
    setPoolSize(5)
    setThreadNamePrefix("app-")
    setWaitForTasksToCompleteOnShutdown(true)
    initialize()
}

SCHEDULING CONFIGURER
─────────────────────────────────────────────────────
implements SchedulingConfigurer
  configureTasks(ScheduledTaskRegistrar registrar)
    registrar.setTaskScheduler(scheduler)
    registrar.addCronTask(runnable, "0 0 9 * * *")
    registrar.addFixedRateTask(runnable, 5000)

TASK SCHEDULER METHODS
─────────────────────────────────────────────────────
schedule(task, Instant)              → run once at time
schedule(task, trigger)              → cron or periodic
scheduleAtFixedRate(task, Duration)  → fixedRate
scheduleWithFixedDelay(task, Duration) → fixedDelay

TRIGGERS
─────────────────────────────────────────────────────
new CronTrigger("0 0 9 * * *")
new CronTrigger("0 0 9 * * *", TimeZone.getTimeZone("Asia/Kolkata"))
new PeriodicTrigger(Duration.ofSeconds(5))
  .setFixedRate(true)         → fixedRate style
  .setFixedRate(false)        → fixedDelay style
  .setInitialDelay(Duration)  → wait before first run

SCHEDULED FUTURE
─────────────────────────────────────────────────────
ScheduledFuture<?> f = taskScheduler.schedule(...)
f.cancel(false)    → graceful stop (finish current run)
f.cancel(true)     → interrupt immediately
f.isCancelled()    → boolean
f.isDone()         → boolean

GOLDEN RULES
─────────────────────────────────────────────────────
✅ ConcurrentHashMap for task storage (thread-safe)
✅ Cancel before rescheduling (avoid duplicates)
✅ Always store ScheduledFuture (or you can't cancel)
✅ Inject TaskScheduler interface, not the class
✅ ApplicationReadyEvent (not @PostConstruct) for DB load
✅ poolSize ≥ number of scheduled tasks
```

---

> 💡 **Rule of thumb:** Pool size = number of scheduled tasks + 2 buffer threads.

> 💡 **Remember:** No `ScheduledFuture` stored = no way to cancel the task ever.

---

*End of Advanced Scheduling Notes 🚀*
