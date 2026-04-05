# 🗂️ Spring Batch
---

## Table of Contents

1. [What is Spring Batch and When to Use It](#1-what-is-spring-batch-and-when-to-use-it)
2. [Core Concepts — Job, Step, ItemReader, ItemProcessor, ItemWriter](#2-core-concepts)
3. [JobLauncher — Triggering Batch Jobs](#3-joblauncher)
4. [Integrating `@Scheduled` with `JobLauncher`](#4-scheduled--joblauncher)
5. [Chunk-Oriented Processing vs Tasklet](#5-chunk-oriented-vs-tasklet)

---

## 1. What is Spring Batch and When to Use It

### What is Spring Batch?

Spring Batch is a **framework for processing large volumes of data in bulk**, in a structured, reliable, and restartable way.

Think of it as the answer to:

> "I need to read 1 million records from a CSV, transform them, and write them to a database — reliably, with error handling, and with the ability to resume if it crashes."

`@Scheduled` can *trigger* code periodically, but it has **no idea** how to:
- Resume from where it stopped after a crash
- Track which records were processed
- Retry failed records
- Process in chunks (without loading everything into memory)
- Report job progress and status

Spring Batch handles all of that.

---

### The Simple Mental Model

```
Without Spring Batch (@Scheduled):
─────────────────────────────────
Scheduler fires → your method runs → reads ALL data → processes → writes
                  If it crashes → starts from zero again 😢
                  No tracking, no retry, no chunking

With Spring Batch:
──────────────────
Scheduler fires → JobLauncher → Job → Step → Read 100 → Process → Write 100
                                           → Read 100 → Process → Write 100
                                           → Read 100 → Process → Write 100
                  If it crashes → resumes from chunk 7 😊
                  Tracks status in DB, retries failed items
```

---

### When to Use Spring Batch vs `@Scheduled`

| Situation | Use |
|---|---|
| Run a quick DB cleanup every night | `@Scheduled` |
| Send a reminder email every hour | `@Scheduled` |
| Process 500K CSV rows and insert to DB | **Spring Batch** |
| Generate a monthly billing report from millions of records | **Spring Batch** |
| ETL pipeline (extract → transform → load) | **Spring Batch** |
| Job must resume from failure (not restart from scratch) | **Spring Batch** |
| Need to track job history / execution status | **Spring Batch** |
| Retry individual failed records without failing the whole job | **Spring Batch** |
| Process items in parallel (partitioning) | **Spring Batch** |

### One-Line Rule

> If you're processing **many records** and care about **reliability, restartability, or progress tracking** — use Spring Batch. For simple periodic tasks — use `@Scheduled`.

---

### Spring Batch Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
```

Spring Batch needs a **database** to store job metadata (execution history, step status, etc.).

```properties
# Spring Batch auto-creates its schema tables
spring.batch.jdbc.initialize-schema=always
```

**Tables Spring Batch creates:**

```
BATCH_JOB_INSTANCE       → one row per unique job + parameters combo
BATCH_JOB_EXECUTION      → one row per actual run of a job
BATCH_JOB_EXECUTION_PARAMS → parameters passed to the job
BATCH_STEP_EXECUTION     → one row per step per run (read/write/skip counts)
BATCH_JOB_EXECUTION_CONTEXT → serialized state for restart
BATCH_STEP_EXECUTION_CONTEXT → serialized step state for restart
```

---

## 2. Core Concepts

### The Big Picture

```
┌──────────────────────────────────────────────────────────┐
│                         JOB                              │
│  (the entire batch operation, e.g. "Process Orders")     │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │                    STEP 1                          │  │
│  │  (one phase of work, e.g. "Read and save orders")  │  │
│  │                                                    │  │
│  │   ItemReader → ItemProcessor → ItemWriter          │  │
│  │   (read)        (transform)     (write)            │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │                    STEP 2                          │  │
│  │  (e.g. "Send confirmation emails")                 │  │
│  │   Tasklet (simple single operation)                │  │
│  └────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

---

### `Job`

A `Job` is the **entire batch process**. It has a name and is made up of one or more `Step`s.

```java
@Bean
public Job processOrdersJob(Step readAndSaveStep, Step sendEmailStep) {
    return new JobBuilder("processOrdersJob", jobRepository)
        .start(readAndSaveStep)
        .next(sendEmailStep)
        .build();
}
```

- A `Job` has a **name** (used to identify it in the metadata tables)
- Steps run **in sequence** by default (`start → next → next`)
- You can add conditional flow (run step B only if step A succeeded)

---

### `Step`

A `Step` is **one unit of work** inside a Job. A Job has at least one Step.

Each Step is either:
- **Chunk-oriented** — Reader → Processor → Writer (for large data)
- **Tasklet** — a simple block of code (for one-off operations)

```java
@Bean
public Step readAndSaveStep(ItemReader<Order> reader,
                             ItemProcessor<Order, Order> processor,
                             ItemWriter<Order> writer) {
    return new StepBuilder("readAndSaveStep", jobRepository)
        .<Order, Order>chunk(100, transactionManager)  // process 100 at a time
        .reader(reader)
        .processor(processor)
        .writer(writer)
        .build();
}
```

---

### `ItemReader`

`ItemReader` reads **one item at a time**. Spring Batch calls it in a loop until it returns `null` (which means "no more data").

```java
// Simple custom reader
public class OrderReader implements ItemReader<Order> {
    private final List<Order> orders;
    private int index = 0;

    public OrderReader(List<Order> orders) {
        this.orders = orders;
    }

    @Override
    public Order read() {
        if (index < orders.size()) {
            return orders.get(index++);  // return one item
        }
        return null;  // signals "done"
    }
}
```

**Built-in readers Spring Batch provides:**

| Reader | Use Case |
|---|---|
| `FlatFileItemReader` | Read CSV / flat files |
| `JdbcCursorItemReader` | Read from DB row by row (cursor) |
| `JpaPagingItemReader` | Read from DB in pages via JPA |
| `JsonItemReader` | Read JSON files |
| `KafkaItemReader` | Read from Kafka topic |

**Example — CSV Reader:**

```java
@Bean
public FlatFileItemReader<Order> csvReader() {
    return new FlatFileItemReaderBuilder<Order>()
        .name("orderCsvReader")
        .resource(new ClassPathResource("orders.csv"))
        .delimited()
        .names("id", "customer", "amount")
        .targetType(Order.class)
        .build();
}
```

---

### `ItemProcessor`

`ItemProcessor` takes one item, **transforms or filters** it, and returns the result.

- Return a transformed object → it moves to the writer
- Return `null` → the item is **skipped** (filtered out)

```java
public class OrderProcessor implements ItemProcessor<Order, Order> {

    @Override
    public Order process(Order order) {
        if (order.getAmount() <= 0) {
            return null;  // skip invalid orders
        }
        order.setStatus("PROCESSED");
        return order;
    }
}
```

`ItemProcessor` is **optional**. If you don't need transformation, skip it.

---

### `ItemWriter`

`ItemWriter` receives a **chunk** (a list of items) and writes them all at once.

```java
public class OrderWriter implements ItemWriter<Order> {

    private final OrderRepository repo;

    public OrderWriter(OrderRepository repo) {
        this.repo = repo;
    }

    @Override
    public void write(Chunk<? extends Order> chunk) {
        repo.saveAll(chunk.getItems());   // write the whole chunk in one DB call
    }
}
```

**Built-in writers:**

| Writer | Use Case |
|---|---|
| `JdbcBatchItemWriter` | Write to DB using JDBC (batched SQL) |
| `JpaItemWriter` | Write via JPA |
| `FlatFileItemWriter` | Write to CSV / text file |
| `JsonFileItemWriter` | Write to JSON file |

---

### How They Work Together (Chunk Processing)

```
chunk size = 3

ItemReader reads:   [Order1] [Order2] [Order3]   → chunk full
ItemProcessor:      process(Order1) → Order1'
                    process(Order2) → null (skipped)
                    process(Order3) → Order3'
ItemWriter:         write([Order1', Order3'])      → one DB call

ItemReader reads:   [Order4] [Order5] [Order6]   → chunk full
...repeat...

ItemReader reads:   [null]   → done!
```

One **transaction** wraps each chunk. If writing chunk 5 fails, chunks 1–4 are already committed. Restart begins from chunk 5.

---

## 3. `JobLauncher`

### What is `JobLauncher`?

`JobLauncher` is the **entry point to run a Job**. You pass it a `Job` and `JobParameters`, and it executes the job.

```java
@Autowired
private JobLauncher jobLauncher;

@Autowired
private Job processOrdersJob;

public void runJob() throws Exception {
    JobParameters params = new JobParametersBuilder()
        .addLong("startTime", System.currentTimeMillis())  // makes each run unique
        .toJobParameters();

    JobExecution execution = jobLauncher.run(processOrdersJob, params);
    System.out.println("Job status: " + execution.getStatus());
}
```

### Why `JobParameters`?

Spring Batch uses `Job name + JobParameters` to identify a **unique job instance**.

If you try to run the same job with the **exact same parameters**, Spring Batch will **refuse** (it considers it already done).

That's why we add a timestamp or unique ID to parameters — to make each run a new instance.

```java
// ❌ Same params every time → Spring Batch says "already completed, won't re-run"
new JobParametersBuilder().toJobParameters();

// ✅ Unique params each time → treated as a fresh run
new JobParametersBuilder()
    .addLong("run.id", System.currentTimeMillis())
    .toJobParameters();
```

### `JobExecution` — What You Get Back

```java
JobExecution execution = jobLauncher.run(job, params);

execution.getStatus();          // COMPLETED, FAILED, STARTED, STOPPED
execution.getStartTime();       // when it started
execution.getEndTime();         // when it ended
execution.getStepExecutions();  // details of each step
```

### Async `JobLauncher`

By default, `JobLauncher` runs the job **synchronously** (blocks until done). For long jobs, use async:

```java
@Bean
public JobLauncher asyncJobLauncher(JobRepository jobRepository) {
    TaskExecutorJobLauncher launcher = new TaskExecutorJobLauncher();
    launcher.setJobRepository(jobRepository);
    launcher.setTaskExecutor(new SimpleAsyncTaskExecutor());  // runs in background thread
    launcher.afterPropertiesSet();
    return launcher;
}
```

With async launcher, `jobLauncher.run()` returns immediately with status `STARTED`, and the job runs in the background.

---

## 4. Integrating `@Scheduled` with `JobLauncher`

### Why Combine Them?

Spring Batch doesn't schedule itself — it only knows *how* to run a job. `@Scheduled` provides the *trigger* (when to run).

This is the most common pattern: **`@Scheduled` tells `JobLauncher` to fire a batch job at a set time**.

```
@Scheduled (the clock)
     │
     ▼
JobLauncher (the starter)
     │
     ▼
Job → Step → Reader → Processor → Writer
```

### Example

```java
@Component
public class BatchScheduler {

    @Autowired
    private JobLauncher jobLauncher;

    @Autowired
    private Job processOrdersJob;

    @Scheduled(cron = "0 0 2 * * ?")   // 2 AM every day
    public void runBatchJob() {
        try {
            JobParameters params = new JobParametersBuilder()
                .addLong("run.id", System.currentTimeMillis())
                .toJobParameters();

            JobExecution execution = jobLauncher.run(processOrdersJob, params);
            System.out.println("Batch finished: " + execution.getStatus());

        } catch (Exception e) {
            System.err.println("Batch failed: " + e.getMessage());
        }
    }
}
```

### Disable Spring Batch Auto-Run on Startup

By default, Spring Boot runs all batch jobs on application startup! Disable this:

```properties
spring.batch.job.enabled=false
```

Now jobs only run when you explicitly call `jobLauncher.run(...)`.

### Async Variant (Fire and Forget)

For very long jobs, use the async launcher so `@Scheduled` doesn't block:

```java
@Scheduled(cron = "0 0 2 * * ?")
public void runBatchJob() throws Exception {
    JobParameters params = new JobParametersBuilder()
        .addLong("run.id", System.currentTimeMillis())
        .toJobParameters();

    asyncJobLauncher.run(processOrdersJob, params);  // returns immediately
    System.out.println("Batch job submitted to background");
}
```

### Combining with ShedLock (For Clusters)

If you're in a cluster, `@Scheduled` fires on all nodes. Wrap it with ShedLock so only one node launches the batch job:

```java
@Scheduled(cron = "0 0 2 * * ?")
@SchedulerLock(name = "processOrdersBatch", lockAtMostFor = "2h")
public void runBatchJob() throws Exception {
    jobLauncher.run(processOrdersJob, ...);
}
```

---

## 5. Chunk-Oriented Processing vs Tasklet

### Two Ways to Define a Step

```
Step
 ├── Chunk-Oriented  → Reader → Processor → Writer (for bulk data)
 └── Tasklet         → single execute() method (for one-off tasks)
```

---

### Chunk-Oriented Processing

**What it is:** Process data in fixed-size chunks. Each chunk is one transaction.

**When to use:** When you're reading many records and writing them to somewhere.

```
Chunk size = 100

Read 100 items  → Process each → Write 100 → COMMIT
Read 100 items  → Process each → Write 100 → COMMIT
Read 100 items  → Process each → Write 100 → COMMIT
...
```

**Why chunks?**
- You don't load the entire dataset into memory (only 100 items at a time)
- If item 450 fails, items 1–400 are already committed (not lost)
- Restart begins from item 401, not item 1

```java
@Bean
public Step chunkStep() {
    return new StepBuilder("chunkStep", jobRepository)
        .<Order, Order>chunk(100, transactionManager)
        .reader(orderReader())
        .processor(orderProcessor())
        .writer(orderWriter())
        .faultTolerant()
            .skip(InvalidOrderException.class)   // skip bad records
            .skipLimit(10)                        // but only up to 10 skips
            .retry(TransientDataAccessException.class)  // retry on DB timeout
            .retryLimit(3)
        .build();
}
```

**Skip & Retry:** Chunk processing supports item-level skip and retry — if one record fails, you can skip it (log it) and continue with the rest of the chunk.

---

### Tasklet

**What it is:** A simple interface with one method — `execute()`. Do anything inside it. Spring Batch calls it once per step (or until you signal `FINISHED`).

**When to use:** For setup/teardown tasks, file moves, sending a single notification, clearing a table, calling an API once.

```java
public class CleanupTasklet implements Tasklet {

    @Override
    public RepeatStatus execute(StepContribution contribution,
                                ChunkContext chunkContext) {
        System.out.println("Deleting temp files...");
        // delete files, truncate table, send email, etc.
        return RepeatStatus.FINISHED;   // tells Spring Batch: step is done
    }
}
```

```java
@Bean
public Step cleanupStep() {
    return new StepBuilder("cleanupStep", jobRepository)
        .tasklet(new CleanupTasklet(), transactionManager)
        .build();
}
```

`RepeatStatus.FINISHED` → step completes.
`RepeatStatus.CONTINUABLE` → Spring Batch calls `execute()` again (use for polling loops).

---

### Side-by-Side Comparison

| Feature | Chunk-Oriented | Tasklet |
|---|---|---|
| Best for | Large data (thousands of records) | One-off operations |
| Memory | Loads only one chunk at a time | You control memory |
| Transaction | One per chunk | One per execute() call |
| Built-in retry/skip | ✅ yes | ❌ manual |
| Restart | From last committed chunk | From beginning of tasklet |
| Example | Read CSV → write to DB | Delete temp files, send 1 email |

---

### Real-World Job: Combining Both

```java
@Bean
public Job orderProcessingJob() {
    return new JobBuilder("orderProcessingJob", jobRepository)
        .start(validateFileStep())     // Tasklet: check if file exists
        .next(processOrdersStep())     // Chunk: read CSV → process → write DB
        .next(archiveFileStep())       // Tasklet: move file to archive folder
        .next(sendSummaryEmailStep())  // Tasklet: send completion email
        .build();
}
```

```
Step 1 (Tasklet)  → Does the file exist? If not, fail early.
Step 2 (Chunk)    → Read 500K rows from CSV, process, write to DB (in chunks of 1000)
Step 3 (Tasklet)  → Move processed file to /archive
Step 4 (Tasklet)  → Email ops team: "Job done, 498,234 records processed, 1,766 skipped"
```

---

## 🗺️ Full Architecture — How Everything Connects

```
┌───────────────────────────────────────────────────────────────────┐
│  @Scheduled (trigger — every night at 2 AM)                       │
│       │                                                           │
│       ▼                                                           │
│  JobLauncher.run(job, params)                                     │
│       │                                                           │
│       ▼                                                           │
│  Job: "processOrdersJob"                                          │
│  │                                                                │
│  ├── Step 1 (Tasklet)   → check file exists                       │
│  │                                                                │
│  ├── Step 2 (Chunk-100) → ItemReader  → reads CSV row by row      │
│  │                      → ItemProcessor → validates, transforms   │
│  │                      → ItemWriter   → saves to DB in batches   │
│  │                        [chunk 1 committed] [chunk 2 committed] │
│  │                        [chunk N committed]                     │
│  │                                                                │
│  └── Step 3 (Tasklet)   → send summary email                      │
│                                                                   │
│  Job status written to: BATCH_JOB_EXECUTION table                 │
│  Step status written to: BATCH_STEP_EXECUTION table               │
│  (read count, write count, skip count, start/end time)            │
└───────────────────────────────────────────────────────────────────┘
```

---

## 🔑 Key Takeaways

| Topic | Remember This |
|---|---|
| Spring Batch vs `@Scheduled` | `@Scheduled` = trigger logic. Batch = process large data reliably |
| `Job` | The entire batch operation; made of one or more Steps |
| `Step` | One unit of work; either Chunk-oriented or Tasklet |
| `ItemReader` | Reads one item at a time; returns `null` when done |
| `ItemProcessor` | Transforms one item; return `null` to skip it |
| `ItemWriter` | Writes a whole chunk at once (list of items) |
| `JobLauncher` | The API to start a Job; needs `Job` + `JobParameters` |
| `JobParameters` | Must be unique per run (use timestamp) or Batch rejects re-run |
| `@Scheduled` + `JobLauncher` | `@Scheduled` is the clock; `JobLauncher` is the starter |
| `spring.batch.job.enabled=false` | Prevents jobs from auto-running on startup |
| Chunk-oriented | One transaction per chunk; supports skip/retry; memory efficient |
| Tasklet | Simple `execute()` method; for one-off steps (file move, email, cleanup) |
| Chunk restart | On crash, restarts from last committed chunk — not from beginning |
| Skip/Retry | `faultTolerant().skip(Ex.class).retry(Ex.class)` on chunk step |
