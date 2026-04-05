# 🏭 Spring Batch & Scheduling

---

## Table of Contents

1. [Idempotency — Jobs That Are Safe to Re-Run](#1-idempotency)
2. [Retry Logic on Failure — `spring-retry`](#2-retry-logic)
3. [Job Chaining — Running Jobs in Sequence](#3-job-chaining)
4. [Externalize Cron Expressions to `application.properties`](#4-externalize-cron-expressions)
5. [Graceful Shutdown — Letting Running Jobs Finish](#5-graceful-shutdown)
6. [Logging Job Start / End / Duration](#6-logging-job-startend-duration)
7. [Spring Actuator — Exposing Scheduler Metrics](#7-spring-actuator)
8. [Alerting on Job Failures](#8-alerting-on-job-failures)
9. [Dead Job Detection](#9-dead-job-detection)

---

## 1. Idempotency

### What is Idempotency?

An **idempotent job** is one you can run **multiple times** and always get the **same result** — no duplicates, no corruption, no side effects from re-running.

```
Non-idempotent:
  Run 1 → inserts 1000 rows ✅
  Run 2 → inserts 1000 MORE rows ❌ (duplicates!)

Idempotent:
  Run 1 → inserts 1000 rows ✅
  Run 2 → sees rows exist → skips / updates → still 1000 rows ✅
```

### Why Does It Matter?

In production, jobs re-run all the time — due to:
- Manual retries after a failure
- Cron firing twice (clock skew, server restart)
- Someone running the job to "fix" bad data
- Quartz misfire catch-up

If your job isn't idempotent, every re-run causes damage.

---

### Pattern 1 — Upsert Instead of Insert

Don't blindly insert. Use `INSERT ... ON DUPLICATE KEY UPDATE` (MySQL) or `MERGE` / `ON CONFLICT` (PostgreSQL).

```java
// ❌ Not idempotent — inserts a duplicate every time
orderRepo.save(new Order(id, "PROCESSED"));

// ✅ Idempotent — update if exists, insert if not
@Query("""
    INSERT INTO orders (id, status)
    VALUES (:id, :status)
    ON DUPLICATE KEY UPDATE status = :status
""")
void upsertOrder(long id, String status);
```

---

### Pattern 2 — Status Check Before Processing

Before processing a record, check if it's already been processed.

```java
public class OrderProcessor implements ItemProcessor<Order, Order> {

    @Override
    public Order process(Order order) {
        if ("PROCESSED".equals(order.getStatus())) {
            return null;  // already done — skip it
        }
        order.setStatus("PROCESSED");
        return order;
    }
}
```

---

### Pattern 3 — Idempotency Key / Run Token

Give each job run a unique token. Before doing any work, check if work for that token is already done.

```java
@Scheduled(cron = "0 0 2 * * ?")
public void runNightlyJob() {
    String runDate = LocalDate.now().toString();  // "2025-04-01"

    if (jobRunRepo.existsByRunDate(runDate)) {
        log.info("Job already ran for {}. Skipping.", runDate);
        return;
    }

    doTheWork();

    jobRunRepo.save(new JobRun(runDate, Instant.now()));
}
```

The `job_runs` table acts as a **run ledger**. Even if the job fires 5 times, it runs once.

---

### Pattern 4 — Processed Timestamps

Add a `processed_at` column. The reader query filters to unprocessed records only.

```java
// Reader only picks up unprocessed records
@Bean
public JdbcCursorItemReader<Order> orderReader(DataSource ds) {
    return new JdbcCursorItemReaderBuilder<Order>()
        .name("orderReader")
        .dataSource(ds)
        .sql("SELECT * FROM orders WHERE processed_at IS NULL")
        .rowMapper(new BeanPropertyRowMapper<>(Order.class))
        .build();
}

// Writer sets processed_at after writing
public void write(Chunk<? extends Order> chunk) {
    for (Order o : chunk.getItems()) {
        o.setProcessedAt(Instant.now());
    }
    orderRepo.saveAll(chunk.getItems());
}
```

Re-running the job picks up zero records because all rows now have `processed_at` set.

---

### Idempotency Checklist

```
✅ Using upsert instead of plain insert?
✅ Checking status before processing?
✅ Readers filter by "unprocessed" status?
✅ No side effects that can't be safely repeated (e.g., sending emails)?
✅ External calls (APIs, emails) guarded with idempotency keys?
```

> **Special case — emails/notifications:** These are the hardest to make idempotent. Track in a DB table: `INSERT INTO notifications_sent (user_id, type, date) ... ON DUPLICATE KEY IGNORE`. Before sending, check the table.

---

## 2. Retry Logic on Failure

### Why Retry?

Many failures in production are **transient** — temporary network blips, DB timeouts, third-party API hiccups. Retrying after a short wait often succeeds.

```
Job calls payment API → timeout (network blip)
  → Wait 1 second → retry
  → Wait 2 seconds → retry
  → Wait 4 seconds → retry
  → Still failing → give up → alert ops team
```

Without retry, one transient error fails the entire job. With retry, these errors are invisible to the user.

---

### Option A — Spring Retry (Method-Level)

`spring-retry` lets you annotate any method to be retried automatically.

**Dependency:**

```xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
```

**Enable it:**

```java
@SpringBootApplication
@EnableRetry
public class App { }
```

**Annotate the method:**

```java
@Service
public class PaymentService {

    @Retryable(
        retryFor  = { PaymentTimeoutException.class },  // only retry on this exception
        maxAttempts = 3,                                 // try 3 times total
        backoff = @Backoff(delay = 1000, multiplier = 2) // 1s, 2s, 4s waits
    )
    public void processPayment(Order order) {
        externalPaymentApi.charge(order);
    }

    @Recover  // called if all retries fail
    public void recoverPayment(PaymentTimeoutException e, Order order) {
        log.error("Payment failed after retries for order {}", order.getId());
        order.setStatus("PAYMENT_FAILED");
        orderRepo.save(order);
    }
}
```

**How it flows:**

```
Call processPayment()
  → PaymentTimeoutException thrown
  → Wait 1000ms → retry (attempt 2)
  → PaymentTimeoutException thrown
  → Wait 2000ms → retry (attempt 3)
  → PaymentTimeoutException thrown
  → All retries exhausted → @Recover called
```

---

### Option B — Spring Batch Built-in Retry (Chunk-Level)

For Spring Batch chunk steps, retry is built in — no extra dependency needed.

```java
@Bean
public Step processStep() {
    return new StepBuilder("processStep", jobRepository)
        .<Order, Order>chunk(100, transactionManager)
        .reader(reader())
        .processor(processor())
        .writer(writer())
        .faultTolerant()
            .retry(TransientDataAccessException.class)  // retry on DB timeout
            .retryLimit(3)                               // max 3 retries per item
            .skip(InvalidOrderException.class)           // skip bad data
            .skipLimit(50)                               // skip at most 50 items
        .build();
}
```

**Retry vs Skip — when to use each:**

| Scenario | Use |
|---|---|
| Transient error (DB timeout, API blip) | `retry` — it may succeed next time |
| Bad data (validation fails, null field) | `skip` — retrying won't help |
| Persistent error (always fails) | `skip` after `retryLimit` exceeded |

---

### Option C — `RetryTemplate` (Programmatic)

For non-Batch code where you need fine-grained control:

```java
RetryTemplate retryTemplate = RetryTemplate.builder()
    .maxAttempts(3)
    .exponentialBackoff(1000, 2, 10000)  // start=1s, multiplier=2, max=10s
    .retryOn(IOException.class)
    .build();

retryTemplate.execute(ctx -> {
    externalApi.call();   // retried automatically
    return null;
});
```

---

### Backoff Strategies

```
Fixed backoff:       1s → 1s → 1s
Linear backoff:      1s → 2s → 3s
Exponential backoff: 1s → 2s → 4s → 8s  ← most common (avoids thundering herd)
Jitter:              1s ± random → 2s ± random  ← prevents all nodes retrying at same time
```

```java
// Exponential with jitter
@Backoff(delay = 1000, multiplier = 2, random = true)
```

---

## 3. Job Chaining — Running Jobs in Sequence

### What is Job Chaining?

Running multiple Jobs **one after another**, where the next Job starts only after the previous one succeeds (or based on some condition).

**Important distinction:**
- **Steps inside one Job** — use `.start().next().next()` inside `JobBuilder` (same job, sequential steps)
- **Job chaining** — multiple independent Jobs triggered in sequence

---

### Pattern 1 — Steps Inside One Job (Preferred)

If the operations are related, put them as Steps inside one Job. Spring Batch handles sequencing natively.

```java
@Bean
public Job orderProcessingJob() {
    return new JobBuilder("orderProcessingJob", jobRepository)
        .start(validateStep())        // Step 1
        .next(processOrdersStep())    // Step 2 — runs only if Step 1 succeeded
        .next(generateReportStep())   // Step 3
        .next(sendEmailStep())        // Step 4
        .build();
}
```

---

### Pattern 2 — `JobExecutionDecider` (Conditional Flow)

Run different Steps based on the result of a previous Step.

```java
@Bean
public Job conditionalJob() {
    return new JobBuilder("conditionalJob", jobRepository)
        .start(processStep())
        .next(decider())                        // evaluate condition
            .on("HAS_ERRORS").to(errorStep())   // if errors found
            .on("NO_ERRORS").to(successStep())  // if clean run
        .end()
        .build();
}

@Bean
public JobExecutionDecider decider() {
    return (jobExecution, stepExecution) -> {
        int skipCount = (int) stepExecution.getSkipCount();
        return skipCount > 0
            ? new FlowExecutionStatus("HAS_ERRORS")
            : new FlowExecutionStatus("NO_ERRORS");
    };
}
```

---

### Pattern 3 — `JobOperator` Chain (Multiple Separate Jobs)

When jobs are independent but need to run in sequence:

```java
@Component
public class JobChainLauncher {

    @Autowired private JobLauncher jobLauncher;
    @Autowired private Job extractJob;
    @Autowired private Job transformJob;
    @Autowired private Job loadJob;

    @Scheduled(cron = "0 0 1 * * ?")
    public void runEtlChain() throws Exception {
        long runId = System.currentTimeMillis();

        // Step 1 — Extract
        JobExecution e1 = jobLauncher.run(extractJob, params(runId, "extract"));
        if (e1.getStatus() != BatchStatus.COMPLETED) {
            log.error("Extract failed. Aborting chain.");
            return;
        }

        // Step 2 — Transform (only if extract succeeded)
        JobExecution e2 = jobLauncher.run(transformJob, params(runId, "transform"));
        if (e2.getStatus() != BatchStatus.COMPLETED) {
            log.error("Transform failed. Aborting chain.");
            return;
        }

        // Step 3 — Load
        jobLauncher.run(loadJob, params(runId, "load"));
    }

    private JobParameters params(long runId, String phase) {
        return new JobParametersBuilder()
            .addLong("runId", runId)
            .addString("phase", phase)
            .toJobParameters();
    }
}
```

---

### Pattern 4 — `JobExecutionListener` Triggers Next Job

After a Job completes, its listener fires the next Job automatically.

```java
public class ChainedJobListener implements JobExecutionListener {

    @Autowired private JobLauncher jobLauncher;
    @Autowired private Job nextJob;

    @Override
    public void afterJob(JobExecution jobExecution) {
        if (jobExecution.getStatus() == BatchStatus.COMPLETED) {
            try {
                jobLauncher.run(nextJob, new JobParametersBuilder()
                    .addLong("run.id", System.currentTimeMillis())
                    .toJobParameters());
            } catch (Exception e) {
                log.error("Failed to launch next job", e);
            }
        }
    }
}
```

```java
@Bean
public Job firstJob() {
    return new JobBuilder("firstJob", jobRepository)
        .listener(chainedJobListener())
        .start(firstStep())
        .build();
}
```

---

### Choosing the Right Pattern

| Situation | Pattern |
|---|---|
| Related operations (same domain) | Steps inside one Job |
| Different logic, run conditionally | `JobExecutionDecider` |
| Independent ETL phases | `JobOperator` chain in `@Scheduled` |
| Event-driven (job B after job A) | `JobExecutionListener` |

---

## 4. Externalize Cron Expressions to `application.properties`

### Why Externalize?

Hardcoded cron expressions:
- Require a **code change + redeploy** to adjust the schedule
- Can't be changed per environment (dev runs hourly, prod runs nightly)
- Are invisible to ops teams without reading source code

Externalized cron expressions:
- Change schedule via **config file or environment variable** — no redeploy
- Different values per environment (`dev`, `staging`, `prod`)
- Visible and auditable in config management tools

---

### How to Do It

**Step 1 — Define in `application.properties`:**

```properties
# application.properties
jobs.order-processing.cron=0 0 2 * * ?
jobs.report-generation.cron=0 30 6 * * MON-FRI
jobs.cleanup.cron=0 0 0 * * SUN
```

**Step 2 — Reference in `@Scheduled`:**

```java
@Scheduled(cron = "${jobs.order-processing.cron}")
public void runOrderProcessing() {
    jobLauncher.run(orderJob, params());
}

@Scheduled(cron = "${jobs.report-generation.cron}")
public void runReportGeneration() {
    jobLauncher.run(reportJob, params());
}
```

Spring resolves `${...}` at startup from `application.properties`.

---

### Per-Environment Values

```properties
# application-dev.properties
jobs.order-processing.cron=0 * * * * ?       # every minute in dev

# application-prod.properties
jobs.order-processing.cron=0 0 2 * * ?       # 2 AM in prod
```

Spring Boot loads the right file based on the active profile (`spring.profiles.active=prod`).

---

### Override with Environment Variables

In Kubernetes / Docker, override without changing files:

```bash
# Shell / Docker env
JOBS_ORDER_PROCESSING_CRON="0 0 3 * * ?"   # override to 3 AM
```

Spring Boot maps `JOBS_ORDER_PROCESSING_CRON` → `jobs.order-processing.cron` automatically (relaxed binding).

---

### Disabling a Job Without Deploying

Use a special Spring cron value to disable a schedule entirely:

```properties
# Disables the job — "-" is a Spring special value meaning "never run"
jobs.cleanup.cron=-
```

```java
@Scheduled(cron = "${jobs.cleanup.cron}")
public void cleanup() { ... }
```

When cron is `-`, Spring skips scheduling this method entirely. No code change, no redeploy.

---

### Validate Cron on Startup

To catch typos early (not at runtime):

```java
@Component
public class CronValidator implements ApplicationListener<ApplicationReadyEvent> {

    @Value("${jobs.order-processing.cron}")
    private String cron;

    @Override
    public void onApplicationEvent(ApplicationReadyEvent event) {
        try {
            CronExpression.parse(cron);
            log.info("Cron expression valid: {}", cron);
        } catch (IllegalArgumentException e) {
            throw new RuntimeException("Invalid cron: " + cron, e);
        }
    }
}
```

---

## 5. Graceful Shutdown — Letting Running Jobs Finish

### The Problem

When a Spring Boot app shuts down (SIGTERM from Kubernetes, rolling deployment, manual stop), running jobs are killed mid-execution.

```
Chunk 1 → committed ✅
Chunk 2 → committed ✅
Chunk 3 → processing... ← SIGTERM → killed 💀
  → Chunk 3 is lost
  → Half the orders are processed, half are not
  → Data is now inconsistent
```

### What Graceful Shutdown Means

Give running jobs a **window of time** to finish their current unit of work before the JVM exits.

```
SIGTERM received
  → Stop accepting new jobs ✅
  → Let current chunk finish → commit → stop ✅
  → Let current tasklet finish → stop ✅
  → JVM exits cleanly
```

---

### Step 1 — Enable Spring Boot Graceful Shutdown

```properties
# Allow up to 2 minutes for in-flight requests/jobs to finish
server.shutdown=graceful
spring.lifecycle.timeout-per-shutdown-phase=120s
```

This tells Spring Boot: on shutdown, wait up to 120 seconds for active threads to finish.

---

### Step 2 — Configure Scheduler Thread Pool Shutdown

```java
@Configuration
public class SchedulerConfig implements SchedulingConfigurer {

    @Override
    public void configureTasks(ScheduledTaskRegistrar registrar) {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(5);
        scheduler.setThreadNamePrefix("scheduler-");
        scheduler.setWaitForTasksToCompleteOnShutdown(true);  // ← key setting
        scheduler.setAwaitTerminationSeconds(120);             // ← wait up to 2 min
        scheduler.initialize();
        registrar.setTaskScheduler(scheduler);
    }
}
```

`setWaitForTasksToCompleteOnShutdown(true)` — the scheduler won't shut down until all running tasks complete.

---

### Step 3 — Handle `SIGTERM` in Long-Running Jobs

For jobs that run for many minutes, check for a shutdown signal periodically:

```java
@Component
public class LongRunningJob {

    @Autowired
    private ApplicationContext ctx;

    @Scheduled(fixedDelay = 60000)
    public void run() {
        List<Order> orders = fetchOrders();

        for (Order order : orders) {
            if (!ctx.isActive()) {
                log.warn("Shutdown detected. Stopping after current item.");
                break;   // stop the loop cleanly
            }
            processOrder(order);
        }
    }
}
```

`ctx.isActive()` returns `false` once the Spring context begins shutting down.

---

### Step 4 — Spring Batch Graceful Stop

Spring Batch has a built-in stop mechanism. You can signal a job to stop after its current chunk:

```java
@Autowired
private JobOperator jobOperator;

// Call this on shutdown or via an admin endpoint
public void stopJob(long executionId) throws Exception {
    jobOperator.stop(executionId);
    // Batch finishes current chunk → marks job as STOPPED
    // On restart → resumes from next chunk
}
```

The job status becomes `STOPPED` (not `FAILED`). On the next run, Spring Batch resumes from the last committed chunk automatically.

---

### Kubernetes — Termination Grace Period

In Kubernetes, configure `terminationGracePeriodSeconds` to give the pod enough time:

```yaml
spec:
  containers:
    - name: batch-app
  terminationGracePeriodSeconds: 180   # 3 minutes
```

This must be **greater than** your `spring.lifecycle.timeout-per-shutdown-phase`.

---

## 6. Logging Job Start / End / Duration

### Why Log This?

- Know exactly when a job ran and how long it took
- Detect slow jobs before they become problems
- Correlate job runs with errors in logs
- Build dashboards / SLAs ("nightly job must finish before 3 AM")

---

### Option A — `JobExecutionListener`

Spring Batch's listener fires `beforeJob` and `afterJob` callbacks.

```java
@Component
public class JobLoggingListener implements JobExecutionListener {

    private static final Logger log = LoggerFactory.getLogger(JobLoggingListener.class);

    @Override
    public void beforeJob(JobExecution jobExecution) {
        log.info("[JOB START] name={} runId={} params={}",
            jobExecution.getJobInstance().getJobName(),
            jobExecution.getId(),
            jobExecution.getJobParameters());
    }

    @Override
    public void afterJob(JobExecution jobExecution) {
        long durationMs = Duration.between(
            jobExecution.getStartTime(),
            jobExecution.getEndTime()
        ).toMillis();

        log.info("[JOB END] name={} status={} duration={}ms readCount={} writeCount={} skipCount={}",
            jobExecution.getJobInstance().getJobName(),
            jobExecution.getStatus(),
            durationMs,
            jobExecution.getStepExecutions().stream().mapToLong(StepExecution::getReadCount).sum(),
            jobExecution.getStepExecutions().stream().mapToLong(StepExecution::getWriteCount).sum(),
            jobExecution.getStepExecutions().stream().mapToLong(StepExecution::getSkipCount).sum()
        );
    }
}
```

Register with the Job:

```java
@Bean
public Job myJob(JobLoggingListener loggingListener) {
    return new JobBuilder("myJob", jobRepository)
        .listener(loggingListener)
        .start(myStep())
        .build();
}
```

**Sample log output:**

```
[JOB START] name=orderProcessingJob runId=1234 params={run.id=1711929600000}
[JOB END]   name=orderProcessingJob status=COMPLETED duration=47823ms readCount=95000 writeCount=94200 skipCount=800
```

---

### Option B — `StepExecutionListener` (Per-Step Logging)

```java
@Component
public class StepLoggingListener implements StepExecutionListener {

    @Override
    public void beforeStep(StepExecution stepExecution) {
        log.info("[STEP START] step={}", stepExecution.getStepName());
    }

    @Override
    public ExitStatus afterStep(StepExecution stepExecution) {
        log.info("[STEP END] step={} status={} read={} write={} skip={} duration={}ms",
            stepExecution.getStepName(),
            stepExecution.getStatus(),
            stepExecution.getReadCount(),
            stepExecution.getWriteCount(),
            stepExecution.getSkipCount(),
            Duration.between(stepExecution.getStartTime(), stepExecution.getEndTime()).toMillis()
        );
        return stepExecution.getExitStatus();
    }
}
```

---

### Option C — Logging for Plain `@Scheduled` Jobs

No listeners here — use AOP or manual logging.

```java
@Scheduled(cron = "${jobs.cleanup.cron}")
public void cleanupJob() {
    long start = System.currentTimeMillis();
    log.info("[JOB START] cleanupJob at {}", Instant.now());

    try {
        doCleanup();
        long duration = System.currentTimeMillis() - start;
        log.info("[JOB END] cleanupJob COMPLETED in {}ms", duration);

    } catch (Exception e) {
        long duration = System.currentTimeMillis() - start;
        log.error("[JOB END] cleanupJob FAILED in {}ms. Error: {}", duration, e.getMessage(), e);
    }
}
```

---

### Structured Logging (JSON) — Production Best Practice

In production, use structured logs (JSON) so they're searchable in tools like ELK / Loki / Datadog:

```java
log.info("job_event=JOB_END job_name={} status={} duration_ms={} read={} write={} skip={}",
    jobName, status, durationMs, readCount, writeCount, skipCount);
```

Or with MDC (Mapped Diagnostic Context) to tag all logs in a job run:

```java
@Override
public void beforeJob(JobExecution je) {
    MDC.put("jobName", je.getJobInstance().getJobName());
    MDC.put("jobRunId", String.valueOf(je.getId()));
}

@Override
public void afterJob(JobExecution je) {
    MDC.clear();
}
```

Every log line inside the job now automatically carries `jobName` and `jobRunId`.

---

## 7. Spring Actuator — Exposing Scheduler Metrics

### What Actuator Gives You

Spring Actuator exposes **endpoints** over HTTP to inspect the health and internals of your app — including scheduled tasks and batch jobs.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```properties
# Expose all actuator endpoints (restrict in prod!)
management.endpoints.web.exposure.include=health,info,metrics,scheduledtasks,batches
management.endpoint.health.show-details=always
```

---

### `/actuator/scheduledtasks` — See All Scheduled Tasks

```
GET http://localhost:8080/actuator/scheduledtasks
```

```json
{
  "cron": [
    {
      "runnable": { "target": "com.example.BatchScheduler.runOrderProcessing" },
      "expression": "0 0 2 * * ?"
    }
  ],
  "fixedRate": [
    {
      "runnable": { "target": "com.example.HealthCheckJob.run" },
      "initialDelay": 0,
      "interval": 60000
    }
  ]
}
```

You can see every scheduled task, its cron expression, and interval — without reading source code.

---

### `/actuator/metrics` — Job Timing Metrics

Spring Boot auto-publishes metrics for `@Scheduled` methods (via Micrometer):

```
GET http://localhost:8080/actuator/metrics/spring.batch.job
GET http://localhost:8080/actuator/metrics/spring.batch.step
```

For `@Scheduled`, Micrometer records:
- `scheduled.tasks.running` — how many scheduled tasks are currently running
- Timer metrics per task (name, duration)

---

### Custom Metrics with Micrometer

Publish custom metrics from your jobs:

```java
@Component
public class BatchScheduler {

    private final MeterRegistry meterRegistry;

    public BatchScheduler(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }

    @Scheduled(cron = "${jobs.order-processing.cron}")
    public void runOrderProcessing() {
        Timer.Sample sample = Timer.start(meterRegistry);

        try {
            jobLauncher.run(orderJob, params());
            meterRegistry.counter("batch.job.success", "job", "orderProcessing").increment();

        } catch (Exception e) {
            meterRegistry.counter("batch.job.failure", "job", "orderProcessing").increment();
            throw e;

        } finally {
            sample.stop(meterRegistry.timer("batch.job.duration", "job", "orderProcessing"));
        }
    }
}
```

These metrics flow to **Prometheus → Grafana** (or any Micrometer-compatible backend):

```
batch_job_success_total{job="orderProcessing"} 42
batch_job_failure_total{job="orderProcessing"} 2
batch_job_duration_seconds{job="orderProcessing"} 47.8
```

---

### `/actuator/health` — Batch Job Health

```java
@Component
public class BatchJobHealthIndicator implements HealthIndicator {

    @Autowired
    private JobExplorer jobExplorer;

    @Override
    public Health health() {
        List<JobInstance> instances = jobExplorer.getLastJobInstances("orderProcessingJob", 1);

        if (instances.isEmpty()) {
            return Health.unknown().withDetail("message", "No runs yet").build();
        }

        JobExecution last = jobExplorer.getLastJobExecution(instances.get(0));
        BatchStatus status = last.getStatus();

        if (status == BatchStatus.COMPLETED) {
            return Health.up()
                .withDetail("lastRun", last.getEndTime())
                .withDetail("status", status)
                .build();
        }

        return Health.down()
            .withDetail("lastRun", last.getEndTime())
            .withDetail("status", status)
            .build();
    }
}
```

```
GET /actuator/health

{
  "status": "DOWN",
  "components": {
    "batchJobHealthIndicator": {
      "status": "DOWN",
      "details": { "lastRun": "2025-04-01T02:14:00Z", "status": "FAILED" }
    }
  }
}
```

---

## 8. Alerting on Job Failures

### Why Alerting Matters

A failed job is silent by default. No one knows unless they:
- Check logs manually
- Notice data is wrong hours later
- A customer complains

Good alerting tells you the moment a job fails — with enough context to debug.

---

### Pattern 1 — `JobExecutionListener` → Email / Slack Alert

```java
@Component
public class AlertingJobListener implements JobExecutionListener {

    @Autowired
    private AlertService alertService;

    @Override
    public void afterJob(JobExecution jobExecution) {
        if (jobExecution.getStatus() == BatchStatus.FAILED) {

            String jobName = jobExecution.getJobInstance().getJobName();
            String error   = jobExecution.getAllFailureExceptions()
                                .stream()
                                .map(Throwable::getMessage)
                                .collect(Collectors.joining(", "));

            alertService.sendAlert(
                "🚨 Job FAILED: " + jobName,
                "Job: "     + jobName     + "\n" +
                "RunId: "   + jobExecution.getId() + "\n" +
                "Time: "    + jobExecution.getEndTime() + "\n" +
                "Error: "   + error
            );
        }
    }
}
```

```java
@Service
public class AlertService {

    @Autowired
    private SlackClient slackClient;   // or JavaMailSender, PagerDuty, etc.

    public void sendAlert(String title, String body) {
        slackClient.postMessage("#ops-alerts", title + "\n" + body);
    }
}
```

---

### Pattern 2 — `@Scheduled` Job — Try/Catch Alert

```java
@Scheduled(cron = "${jobs.cleanup.cron}")
public void cleanupJob() {
    try {
        doCleanup();
    } catch (Exception e) {
        log.error("cleanupJob FAILED", e);
        alertService.sendAlert(
            "🚨 Scheduled Job FAILED: cleanupJob",
            "Error: " + e.getMessage()
        );
    }
}
```

---

### Pattern 3 — Prometheus Alertmanager Rule

If you use Prometheus + Grafana, define an alert rule:

```yaml
# prometheus alert rule
groups:
  - name: batch_jobs
    rules:
      - alert: BatchJobFailed
        expr: batch_job_failure_total > 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Batch job failure detected"
          description: "Job {{ $labels.job }} has failed {{ $value }} times"
```

Alertmanager routes this to Slack / PagerDuty / email.

---

### Pattern 4 — Dead Man's Switch (Healthchecks.io / Cronitor)

For **"job should have run but didn't"** alerts (missed executions):

1. Register your job at [healthchecks.io](https://healthchecks.io) or Cronitor
2. At the end of a successful run, ping a URL
3. If the URL isn't pinged within the expected window → you get an alert

```java
@Scheduled(cron = "0 0 2 * * ?")
public void nightlyJob() {
    try {
        doWork();
        // Ping the healthcheck URL on success
        restTemplate.getForObject("https://hc-ping.com/your-uuid", String.class);
    } catch (Exception e) {
        log.error("Nightly job failed", e);
        alertService.sendAlert("Job failed", e.getMessage());
    }
}
```

If the job doesn't run at all (app was down, CRON was misconfigured), the ping never happens → Healthchecks.io alerts you.

---

## 9. Dead Job Detection

### What is a Dead Job?

A **dead job** is one that:
- Is stuck in `STARTED` or `UNKNOWN` status in the DB (node crashed mid-run)
- Has been running for far longer than expected (infinite loop, deadlock)
- Was supposed to run but never did (missed scheduled window)

Spring Batch stores job status in `BATCH_JOB_EXECUTION`. A dead job leaves a stale `STARTED` row.

---

### Detecting Stuck Jobs

```java
@Component
public class DeadJobDetector {

    @Autowired
    private JobExplorer jobExplorer;

    @Autowired
    private AlertService alertService;

    @Scheduled(fixedRate = 300000)   // check every 5 minutes
    public void detectDeadJobs() {
        List<String> jobNames = jobExplorer.getJobNames();

        for (String jobName : jobNames) {
            List<JobInstance> instances = jobExplorer.getLastJobInstances(jobName, 5);

            for (JobInstance instance : instances) {
                JobExecution exec = jobExplorer.getLastJobExecution(instance);

                if (exec == null) continue;

                boolean isStuck = exec.getStatus() == BatchStatus.STARTED
                    && exec.getStartTime().isBefore(Instant.now().minus(Duration.ofHours(2)));

                if (isStuck) {
                    alertService.sendAlert(
                        "💀 Dead Job Detected: " + jobName,
                        "ExecutionId: " + exec.getId() + "\n" +
                        "Started at: " + exec.getStartTime() + "\n" +
                        "Still STARTED after 2 hours — possible crash/deadlock"
                    );
                }
            }
        }
    }
}
```

---

### Restarting / Abandoning Stuck Jobs

Spring Batch won't restart a job that's in `STARTED` status (it thinks it's already running).

You must **abandon** the stale execution first:

```java
@Autowired
private JobOperator jobOperator;

public void abandonStaleJob(long executionId) throws Exception {
    jobOperator.abandon(executionId);
    // Status changes from STARTED → ABANDONED
    // Now the job can be relaunched fresh
}
```

Or mark it as `FAILED` and then restart:

```java
@Autowired
private JobRepository jobRepository;

public void markFailedAndRestart(long executionId) throws Exception {
    JobExecution exec = jobExplorer.getJobExecution(executionId);
    exec.upgradeStatus(BatchStatus.FAILED);
    exec.setEndTime(Instant.now());
    jobRepository.update(exec);
    // Now restart it
    jobLauncher.run(exec.getJobInstance().getJob(), exec.getJobParameters());
}
```

---

### Detecting Jobs That Never Ran

Check for missed scheduled windows:

```java
@Scheduled(cron = "0 30 2 * * ?")   // check at 2:30 AM — nightly job should be done by now
public void checkNightlyJobRan() {
    List<JobInstance> instances = jobExplorer.getLastJobInstances("nightlyReportJob", 1);

    if (instances.isEmpty()) {
        alertService.sendAlert("⚠️ nightlyReportJob has never run!", "Check configuration.");
        return;
    }

    JobExecution last = jobExplorer.getLastJobExecution(instances.get(0));
    LocalDate lastRunDate = last.getStartTime().atZone(ZoneId.systemDefault()).toLocalDate();

    if (!lastRunDate.equals(LocalDate.now())) {
        alertService.sendAlert(
            "⚠️ nightlyReportJob did not run today!",
            "Last run was: " + lastRunDate
        );
    }
}
```

---

### Summary — Dead Job Detection Strategies

| Strategy | Detects |
|---|---|
| Poll `BATCH_JOB_EXECUTION` for STARTED > N hours | Stuck / crashed jobs |
| `JobOperator.abandon()` | Clears zombie STARTED status |
| Check last run date at expected completion time | Jobs that never started |
| Healthchecks.io ping | Missed runs (external watchdog) |
| Prometheus alert on `spring.batch.job` metrics | Long-running jobs via Grafana |

---

## 🗺️ Full Production Architecture

```
                         ┌─────────────────────────────┐
                         │   application.properties     │
                         │   jobs.nightly.cron=0 0 2 * *│
                         └──────────────┬──────────────┘
                                        │ ${jobs.nightly.cron}
                                        ▼
                              ┌──────────────────┐
                              │   @Scheduled      │
                              │  + @SchedulerLock │  ← ShedLock (one node wins)
                              └────────┬─────────┘
                                       │
                                       ▼
                              ┌──────────────────┐
                              │   JobLauncher     │
                              │  .run(job, params)│
                              └────────┬─────────┘
                                       │
              ┌────────────────────────┼────────────────────────┐
              ▼                        ▼                        ▼
     ┌────────────────┐    ┌───────────────────────┐  ┌────────────────────┐
     │ JobLogging     │    │        JOB             │  │  AlertingListener  │
     │ Listener       │    │  Step1 (Tasklet)       │  │  on FAILED →       │
     │ beforeJob/     │    │  Step2 (Chunk-100)     │  │  Slack / Email     │
     │ afterJob       │    │    Reader→Proc→Writer  │  └────────────────────┘
     └────────────────┘    │    @Retryable (3x)     │
                           │    skip bad records     │
                           └───────────┬────────────┘
                                       │
                      ┌────────────────┼────────────────┐
                      ▼                ▼                 ▼
             ┌──────────────┐  ┌────────────┐  ┌──────────────────┐
             │ BATCH_JOB_   │  │ Micrometer │  │  Dead Job        │
             │ EXECUTION    │  │ Metrics    │  │  Detector        │
             │ (history DB) │  │ → Grafana  │  │  (every 5 min)   │
             └──────────────┘  └────────────┘  └──────────────────┘
```

---

## 🔑 Key Takeaways

| Topic | Remember This |
|---|---|
| Idempotency | Use upsert, status checks, or a run-ledger table to prevent duplicate effects |
| Hardest idempotency case | Emails/notifications — track in a `notifications_sent` table |
| `@Retryable` | Retries on transient errors; `@Recover` handles final failure |
| Batch chunk retry | `.faultTolerant().retry(Ex.class).retryLimit(3)` — per item, not per chunk |
| Backoff | Exponential + jitter is the gold standard for distributed systems |
| Job chaining | Prefer Steps inside one Job; use `JobOperator` chain for independent jobs |
| Externalize cron | `@Scheduled(cron = "${jobs.x.cron}")` — change schedule without redeploy |
| Disable a job | Set cron to `-` in properties — Spring skips scheduling it |
| Graceful shutdown | `server.shutdown=graceful` + `setWaitForTasksToCompleteOnShutdown(true)` |
| Spring Batch stop | `jobOperator.stop(id)` → finishes current chunk → status = STOPPED → restartable |
| Logging | `JobExecutionListener` + MDC for correlated logs across all steps |
| Actuator | `/actuator/scheduledtasks` shows all tasks; custom Micrometer metrics for dashboards |
| Alerting | Listener on `FAILED` → Slack/email; Prometheus rules for metric-based alerts |
| Dead Man's Switch | Healthchecks.io ping on success — alerts if job never runs |
| Dead job | `STARTED` status stuck > N hours → alert → `jobOperator.abandon()` → relaunch |
