# 🧠 Backend Developer Interview Questions

> Theory-first, deeply explained, with simple Java examples

---

## 📑 Index

| # | Topic |
|---|-------|
| [1](#1-url-shortening-service) | URL Shortening Service (Bitly) |
| [2](#2-monolithic-vs-microservices) | Monolithic vs Microservices Architecture |
| [3](#3-real-time-chat-application) | Real-Time Chat Application Backend |
| [4](#4-high-availability-in-distributed-systems) | High Availability in Distributed Systems |
| [5](#5-rate-limiting-system) | Rate Limiting System |
| [6](#6-zero-downtime-database-migrations) | Zero-Downtime Database Migrations |
| [7](#7-content-delivery-system-cdn) | Content Delivery System (CDN) |
| [8](#8-horizontal-vs-vertical-scaling) | Horizontal vs Vertical Scaling |
| [9](#9-notification-system) | Notification System with Retry |
| [10](#10-load-balancer) | Load Balancer & Session Persistence |

---

## 1. URL Shortening Service

### 🧩 What is it?

A URL shortener converts a long, hard-to-share URL into a compact alias. When someone visits the short URL, they are **redirected** to the original long URL.

```
Long:   https://www.example.com/blog/2024/how-to-design-scalable-systems-part-1
Short:  https://bit.ly/abc123
```

The service must handle **millions of redirects per day**, so performance, availability, and low latency are critical.

---

### 📐 Theory: How the System Works End-to-End

There are two main operations:

**1. Write (shorten a URL)**
- User submits a long URL
- System generates a unique short code
- Mapping is stored in the database
- Short URL is returned to the user

**2. Read (redirect)**
- User visits the short URL
- System looks up the short code → finds the long URL
- HTTP 302 redirect is sent back to the browser

```
WRITE FLOW:
Client ──POST /shorten──► API Server ──► ID Generator ──► DB (store mapping)
                                             ↓
                                         Base62 Encode
                                         (ID → short code)

READ FLOW:
Client ──GET /abc123──► API Server ──► Redis Cache ──► (miss) DB
                              │                              │
                              └──── 302 Redirect ◄──────────┘
```

---

### 📐 Theory: Why Base62?

We need to generate **short, unique codes** from numeric database IDs. Base62 uses 62 characters: `A-Z` (26) + `a-z` (26) + `0-9` (10).

- With **6 characters** of Base62: `62^6 = ~56 billion` unique codes
- This is enough for any realistic URL shortening service

We take the **auto-incremented DB ID** (e.g. 100000) and convert it to a Base62 string. This guarantees uniqueness because DB IDs are unique.

```
ID = 1      → "B"
ID = 61     → "9"
ID = 62     → "BA"
ID = 100000 → "q0U"
```

```java
public class Base62Encoder {

    private static final String CHARS =
        "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";

    public static String encode(long id) {
        StringBuilder result = new StringBuilder();
        while (id > 0) {
            result.append(CHARS.charAt((int)(id % 62)));
            id /= 62;
        }
        return result.reverse().toString();
    }
}
// ID = 100000 → "q0U"
```

---

### 📐 Theory: Why Cache Redirects?

Reads (redirects) vastly outnumber writes (URL creation). A URL like `bit.ly/abc123` might be clicked **millions of times**, but it was only created **once**.

- Without cache → every click hits the DB → DB becomes bottleneck
- With Redis cache → 99% of clicks served from memory in < 1ms

The cache stores the `shortCode → longUrl` mapping. On a cache miss, fetch from DB and populate cache for next time.

```java
@GetMapping("/{shortCode}")
public ResponseEntity<Void> redirect(@PathVariable String shortCode) {
    // 1. Check Redis first (fast — in-memory)
    String longUrl = redisCache.get(shortCode);

    if (longUrl == null) {
        // 2. Miss → go to DB (slow — disk)
        longUrl = urlRepository.findByShortCode(shortCode);
        if (longUrl == null) return ResponseEntity.notFound().build();

        // 3. Populate cache for future requests
        redisCache.set(shortCode, longUrl, Duration.ofDays(7));
    }

    // 4. Redirect user — 302 is temporary, browser does not cache it
    return ResponseEntity.status(302)
                         .location(URI.create(longUrl))
                         .build();
}
```

---

### 📐 Theory: 301 vs 302 Redirect

| Code | Type | Browser Caches? | Analytics? |
|------|------|-----------------|------------|
| **301** | Permanent | Yes — skips server next time | Lost (browser goes direct) |
| **302** | Temporary | No — always asks server | Tracked every click |

> Most URL shorteners use **302** so they can track every click for analytics.

---

### ⚖️ Scalability Concerns

| Problem | Cause | Solution |
|---------|-------|----------|
| DB becomes bottleneck | Too many reads | Redis cache in front of DB |
| Single DB write failure | No redundancy | Primary + replica DB |
| High global latency | Far origin server | Geo-distributed edge nodes |
| ID collisions | Concurrent writes | DB auto-increment guarantees uniqueness |

---

## 2. Monolithic vs Microservices

### 🧩 What are they?

**Monolith**: The entire application — user management, orders, payments, notifications — is built and deployed as **one single unit**. It is one codebase, one build artifact (e.g. a `.jar` file), deployed together.

**Microservices**: The application is split into many **small, independent services**. Each service owns its own codebase, its own database, and can be deployed independently.

---

### 📐 Theory: Monolithic Architecture

In a monolith, all modules call each other **in-process** (within the same JVM). There is no network overhead between modules.

```
┌─────────────────────────────────────────┐
│              MONOLITH                   │
│                                         │
│   ┌──────────┐    ┌──────────────────┐  │
│   │   Auth   │───►│  Order Service   │  │
│   └──────────┘    └──────────────────┘  │
│         │                  │            │
│   ┌──────────┐    ┌──────────────────┐  │
│   │  Users   │    │ Payment Service  │  │
│   └──────────┘    └──────────────────┘  │
│                                         │
└─────────────────────┬───────────────────┘
                      │
              One Shared Database
```

**Advantages of Monolith:**
- Simple to develop — one repo, one deployment
- Easy to debug — everything in one place, one log stream
- No network latency between modules
- Easy to test end-to-end
- Great for small teams (2–5 developers)

**Disadvantages of Monolith:**
- As it grows, the codebase becomes hard to understand
- One small bug (e.g. a memory leak in notifications) can crash the whole app
- You must deploy the entire app to change one module
- You cannot scale just the bottleneck — you scale everything or nothing
- Long build and test times as the codebase grows

---

### 📐 Theory: Microservices Architecture

Each service is an **independent process** that communicates over the network using HTTP REST, gRPC, or message queues.

```
                        ┌─────────────────┐
                        │   API Gateway   │  ← Single entry point for all clients
                        └────────┬────────┘
               ┌─────────────────┼─────────────────┐
               ▼                 ▼                 ▼
        ┌────────────┐   ┌────────────┐   ┌────────────┐
        │    Auth    │   │   Order    │   │  Payment   │
        │  Service   │   │  Service   │   │  Service   │
        └─────┬──────┘   └─────┬──────┘   └─────┬──────┘
              │                │                 │
           Auth DB          Order DB         Payment DB   ← Each service owns its own DB
                                │
                         ┌──────┴──────┐
                         │  Message    │   ← Async communication
                         │   Queue     │   (Kafka / RabbitMQ)
                         └─────────────┘
```

**Advantages of Microservices:**
- Independent deployment — update payments without touching orders
- Independent scaling — scale only the bottleneck service
- Technology freedom — auth in Java, recommendation engine in Python
- Fault isolation — payments crashing does not bring down orders
- Smaller codebases are easier for teams to understand and own

**Disadvantages of Microservices:**
- **Distributed systems complexity** — network failures, partial failures
- **Data consistency** — no shared DB means no simple transactions across services
- **Observability** — logs, traces, metrics spread across many services
- **Operational overhead** — need Docker, Kubernetes, CI/CD per service
- **Latency** — in-process calls (nanoseconds) become network calls (milliseconds)

---

### 📝 Code Comparison

**Monolith** — direct in-process calls:
```java
public class OrderService {
    // Direct call — same JVM, no network, no failure risk
    public void placeOrder(Order order) {
        inventoryService.reserve(order);      // Same JVM
        paymentService.charge(order);         // Same JVM
        notificationService.sendEmail(order); // Same JVM
    }
}
```

**Microservices** — calls over network, must handle failures:
```java
public class OrderService {
    public void placeOrder(Order order) {
        try {
            // Network call — can fail, can be slow, can time out
            restClient.post("http://inventory-service/reserve", order);
            restClient.post("http://payment-service/charge", order);
            // Async event — fire and forget (notification not on critical path)
            kafkaProducer.send("order.placed", order);
        } catch (HttpClientErrorException e) {
            // Must handle network errors explicitly
            compensate(order);
        }
    }
}
```

---

### 📐 Theory: When to Migrate Monolith → Microservices?

You should consider migrating when you experience these specific pain points:

- **Deployment bottleneck**: Teams block each other when deploying
- **Scaling bottleneck**: One module needs 10x resources but you must scale the whole app
- **Team size**: More than ~8–10 engineers working on one codebase
- **Technology needs**: One part of the app needs a different tech stack

> **Tip for interviews**: Do not say "microservices are always better." Interviewers appreciate knowing that many successful companies (Shopify, Stack Overflow) run at massive scale on a monolith.

---

### ✅ Decision Guide

| Situation | Choose |
|-----------|--------|
| Early startup, MVP | Monolith |
| Small team (< 8 engineers) | Monolith |
| Well-defined bounded domains | Microservices |
| Independent scaling needs | Microservices |
| Multiple teams needing autonomy | Microservices |

---

## 3. Real-Time Chat Application

### 🧩 What is it?

A backend that allows thousands of users to exchange messages **instantly** — with latency under a second — like WhatsApp or Slack.

---

### 📐 Theory: Why Not Regular HTTP?

Regular HTTP follows a **request-response** model: the client asks, the server responds, the connection closes. For chat, the server needs to **push** a message to a user without the user asking. You cannot do this with plain HTTP.

```
HTTP (Pull model) — bad for chat:
Client: "Any new messages?"  → Server: "No"
Client: "Any new messages?"  → Server: "No"     ← Must keep polling
Client: "Any new messages?"  → Server: "Yes! Here's one"

WebSocket (Push model) — good for chat:
Client connects once, keeps connection open
Server pushes message instantly when it arrives  ← No polling needed
```

Three technologies for real-time communication:

| Technology | How | Best For |
|------------|-----|----------|
| **WebSocket** | Persistent bidirectional TCP connection | Chat, games, live feeds |
| **Server-Sent Events (SSE)** | One-way server → client stream | Notifications, live scores |
| **Long Polling** | Client waits, server holds until data | Fallback for older browsers |

> **Chat apps use WebSockets** because both sides need to send messages at any time.

---

### 📐 Theory: Connection Management

Every connected user occupies a WebSocket connection (an open TCP socket) on the server. A single server can handle roughly **10,000–100,000** concurrent connections depending on hardware and message volume.

Each connection is stored in memory and associated with the user ID.

```java
// In-memory store: userId → WebSocket session
public class SessionStore {
    // ConcurrentHashMap because multiple threads handle connections simultaneously
    private static final Map<String, Session> sessions = new ConcurrentHashMap<>();

    public static void add(String userId, Session session) {
        sessions.put(userId, session);
    }

    public static Session get(String userId) {
        return sessions.get(userId);
    }

    public static void remove(String userId) {
        sessions.remove(userId);
    }

    public static boolean isOnline(String userId) {
        return sessions.containsKey(userId);
    }
}
```

```java
@ServerEndpoint("/chat/{userId}")
public class ChatEndpoint {

    @OnOpen
    public void onOpen(Session session, @PathParam("userId") String userId) {
        SessionStore.add(userId, session);
        presenceService.setOnline(userId);  // Tell others this user is online
    }

    @OnMessage
    public void onMessage(String rawMessage, @PathParam("userId") String senderId) {
        ChatMessage msg = objectMapper.readValue(rawMessage, ChatMessage.class);

        // Always persist the message first (so it can be retrieved later)
        messageRepo.save(msg);

        // Then try to deliver in real-time
        Session recipientSession = SessionStore.get(msg.getRecipientId());
        if (recipientSession != null && recipientSession.isOpen()) {
            recipientSession.getBasicRemote().sendText(rawMessage);  // Online: deliver now
        } else {
            offlineQueue.enqueue(msg.getRecipientId(), msg);  // Offline: deliver on reconnect
        }
    }

    @OnClose
    public void onClose(@PathParam("userId") String userId) {
        SessionStore.remove(userId);
        presenceService.setOffline(userId);
    }
}
```

---

### 📐 Theory: Scaling Across Multiple Servers

One server holds WebSocket connections in its local memory. When you add a second server, **Server 1 does not know about users connected to Server 2**.

```
PROBLEM:
User A (on Server 1) sends a message to User B (on Server 2)
Server 1 looks in its local SessionStore → User B not found!
Message is lost.

SOLUTION — Redis Pub/Sub:
Server 1 publishes the message to Redis channel "user:B"
Server 2 is subscribed to "user:B" → receives it → delivers to User B
```

```java
// Server 1: publish to Redis when recipient is not local
public void routeMessage(ChatMessage msg) {
    Session localSession = SessionStore.get(msg.getRecipientId());
    if (localSession != null) {
        localSession.getBasicRemote().sendText(serialize(msg));  // Local delivery
    } else {
        // Recipient is on another server — publish to Redis
        redisPublisher.publish("user:" + msg.getRecipientId(), serialize(msg));
    }
}

// Every server subscribes to its own channel at startup.
// When Redis delivers a message, forward it to the local WebSocket session.
```

---

### 📐 Theory: Message Storage — Why Cassandra?

Chat generates **enormous amounts of write-heavy data**. Messages are:
- Written once (when sent)
- Read in time order (load conversation history)
- Rarely updated or deleted

Cassandra is designed for exactly this: it handles millions of writes per second and stores data sorted by time per conversation, making message history queries extremely fast.

```
Cassandra partition key: (conversationId)
Clustering key:          (timestamp DESC)

→ "Give me last 50 messages of conversation X" = single fast partition scan
```

---

### 🏗️ Full Architecture

```
Client A ──WS──► Server 1 ──► Redis Pub/Sub ──► Server 2 ──WS──► Client B
                     │                                │
                     └──────────► Kafka ◄────────────┘
                                     │
                               Message Worker
                                     │
                               Cassandra DB
                               (persistent storage)
```

---

## 4. High Availability in Distributed Systems

### 🧩 What is it?

High Availability (HA) means the system remains **operational and accessible** even when individual components fail. The goal is to minimize both the frequency and duration of outages.

---

### 📐 Theory: What Causes Downtime?

Understanding failure sources is the foundation of HA design:

- **Hardware failure** — disk dies, server burns out (expected in large fleets)
- **Software failure** — memory leak, unhandled exception, deadlock
- **Network partition** — servers cannot communicate with each other
- **Deployment failure** — new code has a bug that crashes the service
- **Dependency failure** — a downstream service (payment API, database) goes down
- **Traffic spike** — sudden surge overwhelms the system

HA design assumes **failures will happen** and builds the system to survive them.

---

### 📐 Theory: The Nine Nines

Uptime is measured in "nines." Each additional nine reduces allowed downtime by 10x.

| Uptime SLA | Downtime per Year | Downtime per Month |
|------------|-------------------|--------------------|
| 99% ("two nines") | 3.65 days | 7.3 hours |
| 99.9% ("three nines") | 8.7 hours | 43 minutes |
| 99.99% ("four nines") | 52 minutes | 4.4 minutes |
| 99.999% ("five nines") | 5.2 minutes | 26 seconds |

> Most production systems target 99.9% to 99.99%. Five nines requires extreme engineering effort.

---

### 📐 Theory: Redundancy — No Single Point of Failure

**Redundancy** means having multiple copies of every component so that when one fails, another takes over. Any component with only one instance is a **Single Point of Failure (SPOF)**.

```
❌ Single Point of Failure:
Client → Load Balancer → App Server → Database
                                          ↑
                             DB goes down = total outage

✅ Redundant design:
Client → Load Balancer → App Server 1 ─┐
                       → App Server 2 ─┤─► Primary DB ──replicates──► Replica DB
                       → App Server 3 ─┘              ↑ failover if primary dies
```

Types of redundancy:
- **Active-Active**: All replicas serve traffic simultaneously (better throughput)
- **Active-Passive**: One replica is standby, takes over only on failure (simpler)

---

### 📐 Theory: Circuit Breaker Pattern

When a downstream service (e.g. payment gateway) starts failing, continued calls to it will:
1. Waste thread resources waiting for timeouts
2. Slow down the entire system
3. Cause cascade failures — your service slows, callers to your service slow, and so on

The Circuit Breaker pattern **stops calling a failing service** and returns a fallback immediately, giving the downstream service time to recover.

```
Circuit states:
CLOSED   → Calls pass through normally
OPEN     → All calls fail immediately (no network call made) ← trips after N failures
HALF-OPEN → Allows one test call through → if it succeeds, go back to CLOSED
```

```java
@CircuitBreaker(name = "paymentService",
                fallbackMethod = "paymentFallback")
public PaymentResult charge(Order order) {
    return paymentGateway.charge(order.getAmount());
    // If this fails 5 times in 10 seconds → circuit OPENS
    // All subsequent calls skip this and go directly to fallback
}

// Fallback — called when circuit is open
public PaymentResult paymentFallback(Order order, Exception e) {
    // Queue for retry rather than failing the order entirely
    retryQueue.enqueue(order);
    return PaymentResult.pending("Payment will be processed shortly");
}
```

---

### 📐 Theory: Graceful Degradation

When part of the system fails, the rest should still function — just with reduced features. This is better than a total outage.

```
Normal:                   User sees personalized recommendations + live inventory
Recommendations fail:     User sees popular products instead        ← degraded, not broken
Inventory service fails:  Show "check availability" button          ← degraded, not broken
Both fail:                User can still browse and add to cart      ← core function preserved
```

```java
public List<Product> getHomePage(String userId) {
    List<Product> recommended;
    try {
        recommended = recommendationService.getPersonalized(userId);  // may fail
    } catch (Exception e) {
        recommended = productService.getTopSellers();  // safe fallback
    }
    return recommended;
}
```

---

### 📐 Theory: Health Checks

Every service exposes a `/health` endpoint. Load balancers and orchestrators (Kubernetes) poll it every few seconds. If it fails, traffic is rerouted away.

```java
@GetMapping("/health")
public ResponseEntity<Map<String, String>> health() {
    Map<String, String> status = new LinkedHashMap<>();
    status.put("status", "UP");
    status.put("db", dbConnectionPool.isHealthy() ? "UP" : "DOWN");
    status.put("cache", redisClient.ping() ? "UP" : "DOWN");

    boolean allHealthy = status.values().stream().allMatch("UP"::equals);
    return ResponseEntity
        .status(allHealthy ? 200 : 503)
        .body(status);
}
// Returns: { "status": "UP", "db": "UP", "cache": "UP" }
```

---

## 5. Rate Limiting System

### 🧩 What is it?

Rate limiting controls **how many requests a client can make** to an API within a given time window. It protects the backend from abuse, DDoS attacks, and accidental overloading.

---

### 📐 Theory: Why Rate Limiting?

Without rate limiting, a single misbehaving client can:
- Exhaust server thread pools, causing all clients to slow down
- Hammer the database with millions of reads
- Run up cloud costs
- Create a denial-of-service for legitimate users

Rate limiting enforces a **fair usage policy** at the infrastructure level.

---

### 📐 Theory: The Four Algorithms

#### Algorithm 1: Fixed Window Counter

Divide time into fixed windows (e.g. 0–60s, 60–120s). Count requests per user per window.

```
Window: 0–60s
User A: 100 requests ← at limit

Window: 60–120s
User A: 2 requests ← counter resets

Problem: User can send 100 requests at second 59 and 100 more at second 61
         = 200 requests in 2 seconds (bursts at window boundary)
```

#### Algorithm 2: Sliding Window Log

Track the **exact timestamp** of each request. On each request, count how many timestamps are within the last 60 seconds.

```
Timestamps stored: [t-55s, t-40s, t-20s, t-10s, t-5s, t-now]
Window: last 60s → count = 6
```

Most accurate, but uses **more memory** (stores every timestamp).

#### Algorithm 3: Token Bucket (Most Common in Practice)

A bucket holds N tokens (max capacity). Each request consumes 1 token. Tokens are added back at a fixed rate.

```
Bucket capacity:  100 tokens
Refill rate:      100 tokens per minute

State at 12:00:00 → 100 tokens (full)
5 requests arrive → 95 tokens
10 more requests  → 85 tokens
85 more requests  → 0 tokens → next request: REJECTED (HTTP 429)
Tokens slowly refill over time
```

Allows **short bursts** above the average rate (as long as the bucket is not empty), which feels natural to users.

#### Algorithm 4: Leaky Bucket

Requests enter a queue (the "bucket"). They are processed at a **fixed constant rate**, no matter how fast they arrive.

```
Requests arrive: many at once (bursty)
Requests leave:  steady, fixed rate

Smooths out traffic spikes — good for protecting backends that cannot handle bursts.
```

---

### 📐 Theory: Where to Implement?

```
Client → [API Gateway / Nginx] → [App Server] → [Database]
              ↑
   Best place: API Gateway
   - Before any business logic runs
   - Single enforcement point for all services
   - No need to add it to every service
```

---

### 📝 Implementation: Sliding Window using Redis

```java
public class RateLimiter {

    private final RedisTemplate<String, String> redis;
    private static final int LIMIT = 100;
    private static final int WINDOW_SECONDS = 60;

    public boolean isAllowed(String userId) {
        String key = "ratelimit:" + userId;
        long now = System.currentTimeMillis();
        long windowStart = now - (WINDOW_SECONDS * 1000L);

        // Add this request with current timestamp as score
        redis.opsForZSet().add(key, UUID.randomUUID().toString(), now);

        // Remove all entries older than the window
        redis.opsForZSet().removeRangeByScore(key, 0, windowStart);

        // Set expiry so key cleans itself up
        redis.expire(key, Duration.ofSeconds(WINDOW_SECONDS + 10));

        // Count how many requests are in the current window
        long count = redis.opsForZSet().zCard(key);
        return count <= LIMIT;
    }
}
```

```java
// Filter applied to every incoming request
@Component
public class RateLimitFilter implements Filter {

    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) res;

        String userId = extractUserId(request);  // From JWT or API key

        if (!rateLimiter.isAllowed(userId)) {
            response.setStatus(429);  // Too Many Requests
            response.setHeader("Retry-After", "60");
            response.getWriter().write("Rate limit exceeded. Try again in 60 seconds.");
            return;
        }
        chain.doFilter(req, res);  // Proceed normally
    }
}
```

---

### 📊 Algorithm Comparison

| Algorithm | Memory | Burst Handling | Accuracy | Complexity |
|-----------|--------|----------------|----------|------------|
| Fixed Window | Low | Poor (boundary burst) | Low | Simple |
| Sliding Window Log | High | Good | High | Medium |
| Token Bucket | Low | Good (allows short bursts) | Medium | Medium |
| Leaky Bucket | Low | Smooths all bursts | Medium | Medium |

---

## 6. Zero-Downtime Database Migrations

### 🧩 What is it?

Changing the database schema — adding, renaming, or removing columns/tables — while the application continues to serve traffic, with **no downtime and no errors** during the transition.

---

### 📐 Theory: Why Is This Hard?

In a typical deployment, there is a moment when **old code and new code run simultaneously**. This happens because:

- During a rolling deploy, old instances are still running while new ones start
- If you change the DB schema before deploying new code → old code breaks
- If you deploy new code before changing the schema → new code breaks

```
TIMELINE:
T1: DB has columns "first_name", "last_name"
T2: Deploy new code that reads "full_name"
T3: Old instances still running → they only know "first_name", "last_name" → OK
T4: New instances starting → they expect "full_name" → CRASH (column does not exist)
```

---

### 📐 Theory: The Expand → Migrate → Contract Pattern

This is the **industry-standard** pattern for safe schema changes. It splits one risky change into three safe deployments.

```
Step 1 — EXPAND:    Add new column. Both old and new code work.
Step 2 — MIGRATE:   Backfill data. Both still work.
Step 3 — CONTRACT:  Remove old column. Only new code runs.

              [V1 code]       [V1 + V2 code]      [V2 code]
                  │                 │                  │
DB:   [first_name] ──► [first_name  ──► [first_name ──► [full_name]
      [last_name ]     [last_name ]     [last_name ]
                       [full_name ←NEW] [full_name ←FILLED]
```

---

### 📐 Theory: Step-by-Step Walkthrough

**Goal**: Rename `first_name` + `last_name` into a single `full_name` column.

#### Phase 1 — Expand (Add new column, keep old)

```sql
-- Safe: adding a nullable column never breaks existing code
ALTER TABLE users ADD COLUMN full_name VARCHAR(200) NULL;
```

Deploy new code that **writes to both** old and new columns:

```java
public void updateUser(String userId, String first, String last) {
    user.setFirstName(first);               // Write old column (for old instances)
    user.setLastName(last);                 // Write old column (for old instances)
    user.setFullName(first + " " + last);   // Write new column (for new instances)
    // During transition: BOTH are written, BOTH are readable
}
```

#### Phase 2 — Migrate (Backfill existing rows)

```sql
-- Fill full_name for all existing rows that do not have it yet
UPDATE users
SET full_name = CONCAT(first_name, ' ', last_name)
WHERE full_name IS NULL;
```

For large tables, do this in **batches** to avoid locking the table:

```sql
-- Batch migration: update 1000 rows at a time
UPDATE users SET full_name = CONCAT(first_name, ' ', last_name)
WHERE full_name IS NULL
LIMIT 1000;
-- Repeat until 0 rows are updated
```

#### Phase 3 — Contract (Remove old columns)

Only after **all instances** are running the new code:

```sql
-- Now safe to remove: no code reads these columns anymore
ALTER TABLE users DROP COLUMN first_name;
ALTER TABLE users DROP COLUMN last_name;
```

---

### 📐 Theory: Flyway — Automate Migrations

Flyway runs SQL migration files in order, automatically, at application startup. It tracks which migrations have already been applied in a `flyway_schema_history` table.

```
src/main/resources/db/migration/
├── V1__create_users_table.sql
├── V2__add_full_name_column.sql    ← Phase 1
├── V3__backfill_full_name.sql      ← Phase 2
└── V4__drop_old_name_columns.sql   ← Phase 3
```

```java
// Flyway runs automatically on startup in Spring Boot
// Just add the dependency — no extra configuration needed
// It runs V2, V3, V4 only if they have not been run before
```

---

### ⚠️ Common Dangerous Migrations (Avoid These)

| Dangerous Operation | Why It Breaks | Safe Alternative |
|---------------------|---------------|-----------------|
| `DROP COLUMN` before updating code | Old code still reads it | Use Expand-Migrate-Contract |
| `RENAME COLUMN` directly | Old code cannot find the renamed column | Add new → copy → drop old |
| `ALTER COLUMN` type change | May truncate data silently | New column → migrate → drop |
| Non-null column without default | Old INSERT statements miss it → error | Add as nullable first |

---

## 7. Content Delivery System (CDN)

### 🧩 What is it?

A CDN is a **globally distributed network of servers** (called edge nodes or Points of Presence — PoPs) that cache and serve content from locations physically close to the user, reducing latency dramatically.

---

### 📐 Theory: Why Does Distance Cause Latency?

Data travels through fiber-optic cables at roughly **200,000 km/second** (about 2/3 the speed of light). A request from India to a server in the USA (~13,000 km away) takes at minimum **65ms** just for the signal to travel there and back. Add TCP handshake, TLS negotiation, and server processing → easily **200–400ms**.

A CDN edge node in India serves the same content in **5–20ms**.

```
Without CDN:
User (Mumbai) ─────13,000 km────► Origin (Virginia, USA)
Round trip: ~200ms + processing

With CDN:
User (Mumbai) ─────50 km────► Edge Node (Mumbai)
Round trip: ~5ms (if cached)
                    │
             (cache miss only)
                    └──────────────────────────────► Origin (Virginia)
```

---

### 📐 Theory: How CDN Caching Works

1. User requests `https://example.com/logo.png`
2. Request goes to the nearest **edge node**
3. Edge node checks its local cache:
   - **Cache HIT**: Returns image immediately (fast path)
   - **Cache MISS**: Fetches from origin, caches it, returns to user (first time only)
4. All subsequent requests for the same file are served from the edge cache

```
Cache hit rate determines CDN effectiveness:
- 90% hit rate → only 10% of requests reach origin
- 99% hit rate → only 1% of requests reach origin → massive reduction in origin load
```

---

### 📐 Theory: Cache-Control Headers

The origin server uses `Cache-Control` HTTP headers to tell the CDN **how long** to cache a response.

```java
@GetMapping("/static/image/{id}")
public ResponseEntity<byte[]> getImage(@PathVariable String id) {
    byte[] imageData = imageService.load(id);
    String etag = DigestUtils.md5DigestAsHex(imageData);

    return ResponseEntity.ok()
        .header("Cache-Control", "public, max-age=86400, stale-while-revalidate=3600")
        //                        ↑        ↑              ↑
        //                     CDN can   Cache 1 day    Serve stale for 1 hr while refreshing
        .header("ETag", etag)  // Fingerprint — client sends back to check if content changed
        .header("Vary", "Accept-Encoding")  // Cache separate copies for gzip vs non-gzip
        .body(imageData);
}
```

**Cache-Control directives explained:**

| Directive | Meaning |
|-----------|---------|
| `public` | CDN may cache this response |
| `private` | Only the browser may cache this (not CDN) |
| `max-age=86400` | Cache for 86400 seconds (1 day) |
| `no-cache` | Always revalidate with origin before serving |
| `no-store` | Never cache anywhere (sensitive data) |
| `stale-while-revalidate` | Serve stale content while fetching fresh copy in background |

---

### 📐 Theory: Cache Invalidation

**The hardest problem in CDN caching**: when you update a file, the CDN may keep serving the old cached version for hours.

**Strategy 1 — Cache Busting (Best)**: Include a version hash in the filename. Changing the file changes the filename, which means a new URL, which means the CDN has no cached version for it.

```
Before: /static/app.js              (CDN caches for 1 day)
After:  /static/app.abc123def.js    (new URL = automatic cache miss = fresh version served)
```

**Strategy 2 — Explicit Purge**: Call the CDN API to delete cached copies from all edge nodes.

```java
public void deployNewVersion(String filePath, byte[] newContent) {
    storageService.save(filePath, newContent);
    // Tell CDN to purge old cached version from all edge nodes globally
    cdnClient.purge(List.of(filePath));
    // Note: purge propagation takes 30s–2min to reach all edges worldwide
}
```

---

### 📐 Theory: What to Cache vs. Not Cache

| Cache on CDN | Never Cache |
|--------------|-------------|
| Images, CSS, JS, fonts | User account data |
| Static HTML pages | Authentication tokens |
| Videos and downloads | Shopping cart contents |
| Public API responses | Real-time prices or inventory |
| Favicons and manifests | User-uploaded private files |

---

## 8. Horizontal vs Vertical Scaling

### 🧩 What is it?

Scaling means **increasing a system's capacity** to handle more load. There are two fundamental approaches: making one machine bigger (vertical) or adding more machines (horizontal).

---

### 📐 Theory: Vertical Scaling (Scale Up)

You upgrade the existing server to a more powerful machine — more CPU cores, more RAM, faster storage.

```
Before:                    After:
┌──────────────────┐       ┌────────────────────┐
│  4 CPU cores     │  ──►  │  32 CPU cores      │
│  16 GB RAM       │       │  256 GB RAM        │
│  SSD 500 GB      │       │  NVMe 10 TB        │
└──────────────────┘       └────────────────────┘
  ~1,000 req/s               ~10,000 req/s
```

**When vertical scaling works well:**
- Databases — many databases are hard to distribute (e.g. single-node PostgreSQL)
- Applications with lots of shared in-memory state
- Quick emergency fix when you suddenly need more capacity
- Simple applications where horizontal complexity is not worth it

**Fundamental limits of vertical scaling:**
- Hardware has an absolute ceiling — you cannot exceed the largest available machine
- More expensive per unit — doubling RAM costs more than double the price
- Requires downtime to resize (stop → resize → restart)
- Single point of failure — if this one big server crashes, everything goes down

---

### 📐 Theory: Horizontal Scaling (Scale Out)

You add more servers of the same size and distribute load across them with a load balancer.

```
Before:                   After:
                           ┌──────────────────┐
                           │  Server 1        │
                           │  4 CPU cores     │
                           └──────────────────┘
┌──────────────────┐              ↑
│  Server 1        │  ──►  Load Balancer
│  4 CPU cores     │              ↓
└──────────────────┘       ┌──────────────────┐
                           │  Server 2        │
                           │  4 CPU cores     │
                           └──────────────────┘
                                  ↓
                           ┌──────────────────┐
                           │  Server 3        │
                           │  4 CPU cores     │
                           └──────────────────┘
  ~1,000 req/s               ~3,000 req/s (easily add more)
```

For horizontal scaling to work, your application must be **stateless** — all server instances must be identical and interchangeable. Session data must live in a shared external store, not in server memory.

---

### 📐 Theory: Stateless Design — The Key Enabler

```java
// STATEFUL — breaks horizontal scaling
@RestController
public class BadController {
    // This HashMap lives only in the memory of one server!
    private Map<String, User> loggedInUsers = new HashMap<>();

    @PostMapping("/login")
    public void login(@RequestBody LoginRequest req) {
        loggedInUsers.put(req.getSessionId(), findUser(req));
        // Session stored on Server 1 only
        // If next request goes to Server 2 → user not found!
    }
}
```

```java
// STATELESS — works with horizontal scaling
@RestController
public class GoodController {

    @PostMapping("/login")
    public String login(@RequestBody LoginRequest req) {
        User user = authService.authenticate(req);
        // JWT is self-contained: user info + expiry + signature
        // Any server can validate it — no shared state needed
        return Jwts.builder()
            .setSubject(user.getId())
            .setExpiration(new Date(System.currentTimeMillis() + 86400000))
            .signWith(secretKey)
            .compact();
    }

    @GetMapping("/profile")
    public User getProfile(@RequestHeader("Authorization") String token) {
        // Any server can decode this JWT without calling any other service
        String userId = Jwts.parser().setSigningKey(secretKey)
            .parseClaimsJws(token.replace("Bearer ", ""))
            .getBody().getSubject();
        return userRepo.findById(userId);  // User data in shared DB, not server memory
    }
}
```

---

### 📐 Theory: Auto-Scaling

Horizontal scaling can be **fully automatic**. Cloud platforms monitor CPU and memory and add or remove servers dynamically based on rules you define.

```
Rule: If average CPU > 70% for 5 minutes → add 2 more servers
Rule: If average CPU < 30% for 10 minutes → remove 1 server

Example (lunch hour traffic spike):
11:50 → 2 servers, 1,000 req/s (CPU 30%)
12:01 → Traffic jumps to 3,000 req/s (CPU 95%) → auto-scale triggers
12:03 → 4 servers added → CPU drops to 50%
14:00 → Traffic returns to normal → extra servers removed
```

---

### 🔍 Comparison Summary

| Factor | Vertical (Scale Up) | Horizontal (Scale Out) |
|--------|---------------------|------------------------|
| **How** | Bigger single machine | More machines |
| **Limit** | Hardware ceiling | Nearly unlimited |
| **Cost** | Expensive, non-linear | Cheaper commodity hardware |
| **Downtime** | Requires restart | Zero downtime (rolling) |
| **Fault tolerance** | Single point of failure | Survives individual failures |
| **Complexity** | Simple — no code changes | Requires stateless design |
| **Best for** | Databases, legacy apps | Web servers, APIs, stateless services |

---

## 9. Notification System

### 🧩 What is it?

A backend system that sends **Email, SMS, and Push notifications** reliably, at scale, with automatic retries when delivery fails.

---

### 📐 Theory: Why Is This Harder Than It Looks?

Sending a notification seems simple, but at scale there are several serious challenges:

1. **Reliability** — External providers (Twilio, FCM, SES) have their own downtime. What happens when they are unavailable?
2. **Retries** — Failed sends must be retried, but not indefinitely. Must avoid infinite loops.
3. **Deduplication** — If a retry occurs, the user must NOT receive the same notification twice.
4. **Scale** — During a flash sale, millions of notifications may need to go out simultaneously.
5. **Ordering** — Some notifications must be sent in order (e.g. "order confirmed" before "order shipped").
6. **Provider rate limits** — External services enforce their own rate limits (e.g. Twilio: 100 SMS/second).

---

### 📐 Theory: Asynchronous Architecture

Notifications should be sent **asynchronously** — decoupled from the main request. When a user places an order, the API response should not be delayed because the notification service is slow.

```
Synchronous (BAD):
Client ──► Order API ──► Save order ──► Send email ──► Return 200
                                             ↑
                           This can take 2-3 seconds! User waits.

Asynchronous (GOOD):
Client ──► Order API ──► Save order ──► Publish event ──► Return 200 immediately
                                              │
                                              ▼
                                       Notification Worker
                                       (processes in background)
```

```java
// Order service publishes an event — does not send notification directly
public void placeOrder(Order order) {
    orderRepo.save(order);
    // Fire and forget — notification worker handles delivery asynchronously
    kafkaProducer.send(new ProducerRecord<>(
        "notifications",
        new NotificationEvent("ORDER_CONFIRMED", order.getUserId(), order.getId())
    ));
    // Returns immediately without waiting for notification delivery
}
```

---

### 📐 Theory: Strategy Pattern for Multiple Channels

Different notification channels (Email, SMS, Push) have different APIs but share the same purpose: deliver a message to a user. The Strategy pattern encapsulates each channel behind a common interface.

```java
// Common interface for all channels
public interface NotificationChannel {
    void send(Notification notification) throws NotificationException;
    ChannelType getType();
}

// Email via AWS SES
public class EmailChannel implements NotificationChannel {
    public void send(Notification n) throws NotificationException {
        try {
            sesClient.sendEmail(SendEmailRequest.builder()
                .destination(Destination.builder().toAddresses(n.getRecipientEmail()).build())
                .message(Message.builder()
                    .subject(Content.builder().data(n.getSubject()).build())
                    .body(Body.builder()
                        .html(Content.builder().data(n.getBody()).build()).build())
                    .build())
                .build());
        } catch (SesException e) {
            throw new NotificationException("Email send failed", e);
        }
    }
    public ChannelType getType() { return ChannelType.EMAIL; }
}

// SMS via Twilio
public class SmsChannel implements NotificationChannel {
    public void send(Notification n) throws NotificationException {
        Message.creator(
            new PhoneNumber(n.getRecipientPhone()),
            twilioPhoneNumber,
            n.getBody()
        ).create();
    }
    public ChannelType getType() { return ChannelType.SMS; }
}
```

---

### 📐 Theory: Exponential Backoff with Jitter

When a send fails, you must wait before retrying. Reasons:

- **Why wait at all?** The external service may be overloaded. Retrying immediately makes it worse.
- **Why exponential?** Each retry waits twice as long. This gives the service time to recover.
- **Why jitter?** If 1,000 jobs all fail at the same time and retry at exactly the same moment, they create a **thundering herd** that overwhelms the service again. Random jitter spreads them out.

```java
public void sendWithRetry(Notification notification, NotificationChannel channel) {
    int maxAttempts = 4;
    int baseDelayMs = 1000;  // 1 second

    for (int attempt = 1; attempt <= maxAttempts; attempt++) {
        try {
            channel.send(notification);
            markSent(notification);
            return;  // Success — stop retrying
        } catch (NotificationException e) {
            log.warn("Attempt {}/{} failed: {}", attempt, maxAttempts, e.getMessage());

            if (attempt == maxAttempts) {
                deadLetterQueue.send(notification, e.getMessage());
                return;  // All attempts exhausted
            }

            // Exponential delay: 1s, 2s, 4s, 8s...
            long exponentialDelay = (long)(baseDelayMs * Math.pow(2, attempt - 1));
            long jitter = (long)(Math.random() * 500);  // 0 to 500ms random
            Thread.sleep(exponentialDelay + jitter);
        }
    }
}
// Attempt 1 fails → wait ~1.2s
// Attempt 2 fails → wait ~2.3s
// Attempt 3 fails → wait ~4.1s
// Attempt 4 fails → Dead Letter Queue
```

---

### 📐 Theory: Idempotency — Never Send Twice

If Kafka redelivers a message (which it can do), the notification worker processes it again. Without deduplication, the user receives the same notification twice. We prevent this with a unique idempotency key stored in Redis.

```java
public void processNotification(NotificationEvent event) {
    // Build a key unique to this specific notification delivery
    String idempotencyKey = event.getUserId() + ":" + event.getEventId() + ":" + event.getChannel();

    // Check if already sent
    if (sentLog.exists(idempotencyKey)) {
        log.info("Duplicate — skipped: {}", idempotencyKey);
        return;
    }

    // Send the notification
    notificationChannel.send(buildNotification(event));

    // Record that it was sent — keep for 7 days (TTL)
    sentLog.set(idempotencyKey, "sent", Duration.ofDays(7));
}
```

---

### 🏗️ Full Architecture

```
Order Service ──► Kafka (topic: notifications)
                          │
              ┌───────────┼────────────┐
              ▼           ▼            ▼
         Email          SMS          Push
         Worker        Worker       Worker
           │              │            │
          SES           Twilio        FCM
           │              │            │
           └──────────────┴────────────┘
                          │
               (failure → Retry Queue → Dead Letter Queue)
```

---

## 10. Load Balancer

### 🧩 What is it?

A load balancer sits in front of a group of servers and **distributes incoming requests** across them. It ensures no single server is overwhelmed while others sit idle, and it routes traffic away from unhealthy servers automatically.

---

### 📐 Theory: Layer 4 vs Layer 7 Load Balancing

Load balancers operate at different levels of the network stack:

**Layer 4 (Transport Layer) — TCP/UDP:**
- Makes routing decisions based on **IP address and port** only
- Does NOT read the HTTP request content
- Very fast — minimal processing overhead
- Example: AWS Network Load Balancer

**Layer 7 (Application Layer) — HTTP:**
- Reads the actual **HTTP request** (URL path, headers, cookies)
- Can route based on URL path, hostname, user agent, etc.
- More powerful but slightly more overhead
- Example: AWS Application Load Balancer, Nginx

```
Layer 7 path-based routing example:
/api/*    → Route to API servers
/images/* → Route to image servers (optimized for static files)
/admin/*  → Route to admin servers (with IP allowlist)
```

---

### 📐 Theory: Routing Algorithms

```java
public class LoadBalancer {
    private final List<String> servers;
    private final Map<String, Integer> connectionCount = new ConcurrentHashMap<>();
    private final AtomicInteger index = new AtomicInteger(0);

    // 1. Round Robin
    // Each request goes to the next server in order: 1, 2, 3, 1, 2, 3...
    // Best when: servers are identical and requests take similar time
    public String roundRobin() {
        int i = index.getAndIncrement() % servers.size();
        return servers.get(i);
    }

    // 2. Least Connections
    // Send to the server currently handling the fewest active connections
    // Best when: requests have very different durations (some fast, some slow)
    public String leastConnections() {
        return servers.stream()
            .min(Comparator.comparingInt(s -> connectionCount.getOrDefault(s, 0)))
            .orElseThrow();
    }

    // 3. IP Hash
    // Hash the client IP to always route the same client to the same server
    // Best when: you want session affinity without sticky session cookies
    public String ipHash(String clientIp) {
        int hash = Math.abs(clientIp.hashCode());
        return servers.get(hash % servers.size());
    }

    // 4. Random — simple, effective when servers are identical
    public String random() {
        return servers.get((int)(Math.random() * servers.size()));
    }
}
```

---

### 📐 Theory: Health Checks

A load balancer must continuously verify which servers are healthy. Two types:

- **Passive health checks** — observe real traffic; if a server returns 5xx errors repeatedly, mark it unhealthy
- **Active health checks** — proactively ping each server every few seconds with a test request

```java
@Scheduled(fixedRate = 5000)  // Every 5 seconds
public void runHealthChecks() {
    for (String server : allRegisteredServers) {
        boolean healthy = checkHealth(server);
        if (healthy) {
            activeServers.add(server);
        } else {
            activeServers.remove(server);
            alertingService.notify("Server " + server + " is DOWN — removed from rotation");
        }
    }
}

private boolean checkHealth(String server) {
    try {
        HttpResponse<String> response = httpClient.send(
            HttpRequest.newBuilder()
                .uri(URI.create(server + "/health"))
                .timeout(Duration.ofSeconds(2))  // Must respond within 2s
                .build(),
            HttpResponse.BodyHandlers.ofString()
        );
        return response.statusCode() == 200;
    } catch (Exception e) {
        return false;  // Timeout or connection refused
    }
}
```

---

### 📐 Theory: Session Persistence (Sticky Sessions)

**The problem**: HTTP is stateless, but some applications store user session data in **server memory** (e.g. shopping cart, login state). If a user's requests go to different servers, each server has a different view of the session.

```
User logs in → Server 1 stores session in memory
Next request → Load balancer sends to Server 2
Server 2: "I don't have your session!" → User gets logged out
```

**Solution 1 — Sticky Sessions (short-term fix):**
The load balancer always routes the same user to the same server using a cookie.

```java
public String getServerForRequest(HttpServletRequest request) {
    Cookie stickyCookie = getCookie(request, "LB_SERVER");

    if (stickyCookie != null) {
        String assignedServer = stickyCookie.getValue();
        if (activeServers.contains(assignedServer)) {
            return assignedServer;  // Same server as last time (still healthy)
        }
    }

    // No cookie, or assigned server is down → assign a new server
    String newServer = leastConnections();
    response.addCookie(new Cookie("LB_SERVER", newServer));
    return newServer;
}
```

**Problem with sticky sessions**: If the assigned server goes down, the user's session data is lost. Not truly highly available.

**Solution 2 — Shared Session Store (proper fix):** Store sessions in Redis. Any server can read any session. No stickiness needed.

```java
// Spring Session + Redis: one annotation and sessions are stored in Redis automatically
@EnableRedisHttpSession
@Configuration
public class SessionConfig {
    // All server instances read and write sessions from the same Redis cluster
    // Any server can handle any user's request — complete flexibility
}
```

---

### 📐 Theory: SSL Termination

HTTPS connections are encrypted with TLS. Decrypting TLS at every backend server wastes CPU. Instead, the load balancer handles decryption once (**SSL termination**), then forwards plain HTTP to backend servers on the trusted internal network.

```
Client ──HTTPS──► Load Balancer (decrypts TLS here) ──HTTP──► Server 1
                                                     ──HTTP──► Server 2
                                                     ──HTTP──► Server 3

Benefits:
- Backend servers do not need SSL certificates
- Backend CPU is used for business logic, not cryptography
- SSL certificates managed in one place (easier to rotate)
- Internal network is trusted (private VPC / subnet)
```

---

### 📊 Routing Algorithm Summary

| Algorithm | Best For | Trade-off |
|-----------|----------|-----------|
| **Round Robin** | Identical servers, uniform requests | Ignores actual server load |
| **Weighted Round Robin** | Mixed-capacity servers | Requires manual weight tuning |
| **Least Connections** | Long-lived, variable-duration requests | Slightly more tracking overhead |
| **IP Hash** | Session affinity without cookies | Can cause uneven distribution |
| **Random** | Simple, low-traffic systems | Not optimal under heavy load |

---

*End of Document*
