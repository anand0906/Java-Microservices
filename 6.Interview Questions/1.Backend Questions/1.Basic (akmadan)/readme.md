# 🖥️ Backend Developer Interview Guide
> Theory-focused answers with simple real-world analogies. Clear, concise, easy to revise.

---

## 📑 Index

| #  | Question |
|----|----------|
| 1  | [Role of Backend vs Frontend](#1-role-of-backend-vs-frontend) |
| 2  | [HTTP Requests & Methods](#2-http-requests--methods) |
| 3  | [What is a Server?](#3-what-is-a-server) |
| 4  | [Relational vs NoSQL Database](#4-relational-vs-nosql-database) |
| 5  | [Environment Variables](#5-environment-variables) |
| 6  | [What is an API?](#6-what-is-an-api) |
| 7  | [Primary Key in a Database](#7-primary-key-in-a-database) |
| 8  | [Stateless Server & REST](#8-stateless-server--rest) |
| 9  | [Storing Passwords Securely](#9-storing-passwords-securely) |
| 10 | [Configuration Files](#10-configuration-files) |
| 11 | [Handling Invalid Client Data](#11-handling-invalid-client-data) |
| 12 | [Authentication vs Authorization](#12-authentication-vs-authorization) |
| 13 | [Middleware](#13-middleware) |
| 14 | [Designing a Blog Database Schema](#14-designing-a-blog-database-schema) |
| 15 | [HTTP Status Codes](#15-http-status-codes) |
| 16 | [Basic API Endpoint for Users](#16-basic-api-endpoint-for-users) |
| 17 | [GET vs POST — Data Handling](#17-get-vs-post--data-handling) |
| 18 | [Handling Database Connection Failure](#18-handling-database-connection-failure) |
| 19 | [Caching](#19-caching) |
| 20 | [Logging Errors](#20-logging-errors) |

---

## 1. Role of Backend vs Frontend

### What is it?
A web application has two main parts that work together — the **frontend** and the **backend**.

```
User  →  Frontend (what you see)  →  Backend (the engine)  →  Database (storage)
```

### Frontend
- Everything the user sees and interacts with — buttons, forms, pages.
- Runs in the **browser**.
- Built with HTML, CSS, JavaScript.
- Its job: display data and send user actions to the backend.

### Backend
- The brain behind the scenes — processes logic, talks to the database, enforces rules.
- Runs on a **server**.
- Built with Java, Node.js, Python, etc.
- Its job: receive requests, process them, send responses.

### 🍕 Real-world analogy
Think of a restaurant.
- The **frontend** is the menu and the waiter — what you see and interact with.
- The **backend** is the kitchen — where the actual work happens.
- The **database** is the pantry — where ingredients (data) are stored.

---

## 2. HTTP Requests & Methods

### What is an HTTP Request?
When your browser needs data from a server, it sends an **HTTP request** — a structured message that says *"Hey server, do this for me."*

Every request has:
- A **method** — what action to perform
- A **URL** — where to send it
- **Headers** — extra info (like who you are)
- A **body** — data you're sending (only for some methods)

### Common HTTP Methods

| Method | Purpose | Real-world analogy |
|--------|---------|-------------------|
| `GET` | Fetch/read data | Asking to see the menu |
| `POST` | Create new data | Placing an order |
| `PUT` | Replace existing data | Replacing your whole order |
| `PATCH` | Update part of data | Changing one item in your order |
| `DELETE` | Remove data | Cancelling your order |

### Flow diagram
```
Browser                         Server
  |                               |
  |--- GET /products ----------->|
  |                               |  (looks up products in DB)
  |<-- 200 OK + product list ----|
  |                               |
```

---

## 3. What is a Server?

### What is it?
A **server** is a program (or a machine) that:
1. Listens for incoming requests on a network
2. Processes the request (runs logic, queries DB)
3. Sends back a response

### How it handles a request — step by step

```
1. Client sends:   GET /users/5
2. Server receives the request
3. Server parses it: "They want user with ID 5"
4. Server queries the database: SELECT * FROM users WHERE id = 5
5. Server formats the result as JSON
6. Server sends: 200 OK + { "id": 5, "name": "Alice" }
```

### 🏨 Real-world analogy
A server is like a hotel receptionist:
- Guests (clients) come with requests ("I need a room", "What's the Wi-Fi password?")
- The receptionist (server) processes each request and responds
- They check internal records (database) when needed

### Key points
- A server can handle **many clients at the same time**
- It listens on a specific **port** (e.g., port 8080)
- It speaks the **HTTP protocol** to communicate with clients

---

## 4. Relational vs NoSQL Database

### Relational Database (SQL)
- Stores data in **tables** with rows and columns — like a spreadsheet
- Has a **fixed schema** — you define columns before inserting data
- Tables can be **related** to each other using foreign keys
- Uses **SQL** to query data
- Examples: MySQL, PostgreSQL, Oracle

```
users table:
| id | name  | email           |
|----|-------|-----------------|
|  1 | Alice | alice@email.com |
|  2 | Bob   | bob@email.com   |
```

### NoSQL Database
- Stores data as **documents, key-value pairs, or graphs** — very flexible
- **No fixed schema** — each record can have different fields
- Built for **scale and speed**
- Examples: MongoDB (documents), Redis (key-value), Cassandra

```json
// MongoDB document — flexible structure
{
  "name": "Alice",
  "email": "alice@email.com",
  "hobbies": ["coding", "reading"],
  "address": { "city": "Hyderabad" }
}
```

### When to use which?

| Use Relational when... | Use NoSQL when... |
|------------------------|-------------------|
| Data has clear structure | Data structure varies |
| Relationships matter (orders ↔ users) | You need massive scale |
| You need transactions (banking) | Speed is top priority |
| Example: E-commerce, Banking | Example: Social feeds, Chat apps |

---

## 5. Environment Variables

### What are they?
Environment variables are **key-value settings stored outside your code**, in the operating system or deployment environment.

They hold sensitive or environment-specific values like:
- Database passwords
- API keys
- Server port numbers
- Feature flags

### Why are they important?

| Problem | Solution |
|---------|----------|
| Hardcoded passwords in code get leaked | Store them as env variables |
| Different DB for dev vs production | Each environment has its own values |
| Changing a value requires redeployment | Change the env variable — no code change needed |

### 🔑 Golden rule
```
Never hardcode secrets in your source code.
If your code goes to GitHub, your secrets go with it.
```

### How it works
```
OS / Server
  └── ENV: DB_PASSWORD = "secretABC"
  └── ENV: APP_PORT    = "8080"

Your App
  └── reads DB_PASSWORD at startup → connects to DB
  └── reads APP_PORT → starts server on that port
```

---

## 6. What is an API?

### What is it?
An **API (Application Programming Interface)** is a **contract** between two systems that defines:
- What requests can be made
- In what format
- What responses to expect

In backend development, an API is the **front door** of your server — clients (browsers, mobile apps, other services) call it to get or send data.

### 🍔 Real-world analogy
Think of a restaurant menu — it tells you:
- What you can order (available endpoints)
- How to order (request format)
- What you'll receive (response format)

You don't need to know how the kitchen works. You just follow the menu.

### Why use APIs?
- **Separation of concerns** — frontend and backend can be developed independently
- **Reusability** — the same API can serve a web app, mobile app, and third-party partners
- **Security** — internal business logic stays hidden; only the API is exposed

### Request → Response flow
```
Mobile App  ──── GET /api/weather?city=Hyderabad ────▶  Backend API
Mobile App  ◀─── { "temp": "32°C", "condition": "Sunny" } ──  Backend API
```

---

## 7. Primary Key in a Database

### What is it?
A **primary key** is a column (or set of columns) in a database table that **uniquely identifies each row**.

### Rules of a Primary Key
- Must be **unique** — no two rows can have the same value
- Cannot be **null** — every row must have one
- Should be **stable** — it shouldn't change over time

### Why is it necessary?
Without a primary key, you can't:
- Find a specific record reliably
- Update the right row
- Delete a specific entry
- Link tables together (relationships)

```
users table — WITH primary key:
| id (PK) | name  | email           |
|---------|-------|-----------------|
|    1    | Alice | alice@email.com |
|    2    | Alice | alice2@email.com|
    ↑
Even two users named "Alice" are uniquely identified by their id
```

### Types of primary keys
| Type | Description | Example |
|------|-------------|---------|
| Auto-increment integer | DB generates it automatically | `1, 2, 3, 4...` |
| UUID | Random unique string | `a3f2-bc91-...` |
| Natural key | A real-world unique value | Email, National ID |

---

## 8. Stateless Server & REST

### What does "stateless" mean?
A **stateless** server does not store any memory of previous requests. Every request is treated as **brand new** — the server has no idea who talked to it before.

### Stateful vs Stateless

```
STATEFUL (like a phone call):
  Request 1: "Hi, I'm Alice"       → Server remembers Alice
  Request 2: "Show me my orders"   → Server knows it's Alice ✅

STATELESS (like sending letters):
  Request 1: "Hi, I'm Alice. Show me my orders. Token: abc123"
  Request 2: "Hi, I'm Alice. Show me my profile. Token: abc123"
  → Each letter carries all the information needed ✅
```

### How does REST handle this?
In REST APIs, the client sends a **token** (like a JWT) with every request. The server reads the token to identify the user — no memory needed.

### Benefits of stateless design
- **Scalability** — any server in a cluster can handle any request
- **Simplicity** — no session storage to manage
- **Reliability** — no shared state that can go out of sync

---

## 9. Storing Passwords Securely

### What NOT to do
- ❌ Store plain text: `password = "myPass123"` — if DB is hacked, all passwords are exposed
- ❌ Use basic encryption (reversible) — can be decoded
- ❌ Use MD5 or SHA-1 — they're fast, which makes them easy to brute-force

### The right approach — Hashing
**Hashing** is a one-way transformation. You can never go back from the hash to the original password.

```
User types:    "myPass123"
                    ↓ BCrypt
Stored in DB:  "$2a$10$eW5bJ3K...xyz..."

When logging in:
  User types "myPass123" again
  BCrypt hashes it → compares with stored hash
  Match? ✅ Login success. No match? ❌ Rejected.
```

### Best practices
- Use **BCrypt**, **Argon2**, or **PBKDF2** — they are slow by design, making brute-force attacks harder
- BCrypt automatically adds a **salt** (random noise) to each hash, so two users with the same password get different hashes
- Never log passwords, even in error messages
- Never send passwords in URLs

---

## 10. Configuration Files

### What is it?
A **configuration file** is a separate file (not your Java code) that holds all the settings your application needs to run — like port numbers, database URLs, timeouts, and feature flags.

### Why separate config from code?
- Code defines *how* the app works
- Config defines *where* and *with what settings* it runs
- You can deploy the same code to dev, staging, and production — just swap the config file

### What goes in a config file?

| Setting type | Example |
|-------------|---------|
| Server settings | Port number, hostname |
| Database settings | URL, username, connection pool size |
| External services | Email server address, SMS gateway URL |
| App behaviour | Max file upload size, session timeout |
| Feature flags | `feature.dark-mode=true` |

### 🏗️ Real-world analogy
Think of a config file as the **settings panel** of your app. The engine (code) stays the same — you just change the settings for each environment.

```
application-dev.properties   → points to local DB, debug logs ON
application-prod.properties  → points to cloud DB, debug logs OFF
```

---

## 11. Handling Invalid Client Data

### Why is this important?
Clients can send anything — missing fields, wrong data types, strings where numbers are expected, or even malicious input. Your API must **validate all incoming data** before processing it.

### What can go wrong without validation?
- Saving garbage data to the database
- Application crashes (NullPointerException, etc.)
- Security vulnerabilities (SQL injection, XSS)

### How to handle it — the right way

**Step 1 — Validate the input**
Check that all required fields are present, types are correct, and values are within allowed ranges.

**Step 2 — Return a clear error response**
If validation fails, return `400 Bad Request` with a descriptive message.

**Step 3 — Never trust the client**
Even if the frontend validates inputs, always re-validate on the backend. The frontend can be bypassed.

### Response structure for invalid input
```
HTTP 400 Bad Request
{
  "error": "Validation failed",
  "details": [
    "Name is required",
    "Email format is invalid",
    "Age must be at least 18"
  ]
}
```

---

## 12. Authentication vs Authorization

### The difference

| | Authentication | Authorization |
|--|---------------|---------------|
| Question | "Who are you?" | "What can you do?" |
| When | At login | On every protected action |
| Result | Identity confirmed | Access granted or denied |
| Example | Checking username + password | Checking if you're an admin |

### 🏨 Real-world analogy
Imagine a hotel:
- **Authentication** = Showing your ID at check-in to prove who you are
- **Authorization** = Your room key only opens *your* room, not the penthouse

### How they work together
```
1. User logs in with email + password
         ↓ Authentication
2. Server verifies identity → issues a token (JWT)
         ↓
3. User calls DELETE /users/5
         ↓ Authorization
4. Server checks: "Does this user have the ADMIN role?"
   YES → proceeds   |   NO → 403 Forbidden
```

### Common HTTP responses
- `401 Unauthorized` — Authentication failed (you're not logged in)
- `403 Forbidden` — Authenticated but not authorized (logged in, but no permission)

---

## 13. Middleware

### What is it?
**Middleware** is a layer of code that sits between the incoming request and your controller (handler). It intercepts every request before it reaches the business logic.

```
Incoming Request
      ↓
[ Middleware 1: Log the request ]
      ↓
[ Middleware 2: Check authentication token ]
      ↓
[ Middleware 3: Check rate limit ]
      ↓
  Controller (actual logic)
      ↓
Outgoing Response
```

### Why use middleware?
Without middleware, you'd have to repeat the same code (like auth checks or logging) in every single endpoint. Middleware lets you write it once and apply it everywhere.

### Common uses of middleware

| Middleware | What it does |
|-----------|-------------|
| **Logging** | Records every incoming request and response |
| **Authentication** | Validates the user's token on every request |
| **CORS** | Allows/blocks requests from specific domains |
| **Rate Limiting** | Blocks a client that sends too many requests |
| **Request Parsing** | Parses JSON body before it reaches the controller |

### 🏢 Real-world analogy
Middleware is like security checkpoints at an office building:
- Every visitor (request) must pass through reception (log), show ID (auth), and sign in (rate limit) before reaching the floor they want (controller).

---

## 14. Designing a Blog Database Schema

### Entities and their relationships

```
┌───────────┐        ┌──────────────┐        ┌────────────────┐
│   users   │        │    posts     │        │    comments    │
├───────────┤        ├──────────────┤        ├────────────────┤
│ id (PK)   │───┐    │ id (PK)      │───┐    │ id (PK)        │
│ name      │   └──▶ │ user_id (FK) │   └──▶ │ post_id (FK)   │
│ email     │        │ title        │        │ user_id (FK) ──┘
│ password  │        │ content      │        │ body           │
│ joined_at │        │ created_at   │        │ created_at     │
└───────────┘        └──────────────┘        └────────────────┘

Relationships:
  - One user can write many posts         (1 : Many)
  - One post can have many comments       (1 : Many)
  - One user can write many comments      (1 : Many)
```

### Design decisions explained

| Decision | Why |
|----------|-----|
| `user_id` in posts | Links every post to its author |
| `post_id` in comments | Links every comment to its post |
| `user_id` in comments | Tracks who wrote each comment |
| `created_at` everywhere | Allows sorting by date |
| Separate tables | Avoids duplicating data (normalization) |

---

## 15. HTTP Status Codes

### What are they?
HTTP status codes are **3-digit numbers** the server sends back with every response to tell the client what happened.

### The 5 categories

| Range | Meaning |
|-------|---------|
| `1xx` | Informational — request received, processing |
| `2xx` | ✅ Success — request completed |
| `3xx` | 🔀 Redirect — go somewhere else |
| `4xx` | ❌ Client error — you did something wrong |
| `5xx` | 💥 Server error — we did something wrong |

### Most important codes to know

| Code | Name | When it's used |
|------|------|----------------|
| `200` | OK | Request succeeded, data is returned |
| `201` | Created | A new resource was successfully created |
| `204` | No Content | Success, but nothing to return (e.g., DELETE) |
| `400` | Bad Request | Client sent invalid or missing data |
| `401` | Unauthorized | Not logged in / invalid token |
| `403` | Forbidden | Logged in but no permission |
| `404` | Not Found | The resource doesn't exist |
| `409` | Conflict | Duplicate entry (e.g., email already registered) |
| `500` | Internal Server Error | Unexpected crash on the server |
| `503` | Service Unavailable | Server is down or overloaded |

---

## 16. Basic API Endpoint for Users

### What it involves
A well-structured API endpoint to fetch users follows a **layered architecture**:

```
HTTP Request (GET /api/users)
          ↓
    Controller Layer
    (receives request, sends response)
          ↓
    Service Layer
    (business logic — e.g., filters, sorting)
          ↓
    Repository Layer
    (talks to the database)
          ↓
    Database (returns user records)
```

### What each layer is responsible for

| Layer | Responsibility |
|-------|---------------|
| **Controller** | Receive HTTP request, call service, return HTTP response |
| **Service** | Apply business rules (e.g., exclude inactive users) |
| **Repository** | Run the actual database query |
| **Model/Entity** | Represent the data structure (User object) |

### What the response looks like
```json
GET /api/users

200 OK
[
  { "id": 1, "name": "Alice", "email": "alice@example.com" },
  { "id": 2, "name": "Bob",   "email": "bob@example.com"   }
]
```

### Why separate layers?
- Each layer has **one job** (Single Responsibility Principle)
- Easy to test each layer independently
- Changing the database doesn't affect the controller

---

## 17. GET vs POST — Data Handling

### The core difference
- **GET** — asks the server for data. Sends parameters in the **URL**.
- **POST** — sends data to the server to create something. Sends data in the **request body**.

### Side-by-side comparison

| Feature | GET | POST |
|---------|-----|------|
| Data location | URL (query string) | Request body |
| Visible in browser? | ✅ Yes | ❌ No |
| Saved in history? | ✅ Yes | ❌ No |
| Can be bookmarked? | ✅ Yes | ❌ No |
| Cached by browser? | ✅ Often | ❌ No |
| Data size limit? | ✅ Yes (URL length) | ❌ No practical limit |
| Use case | Read data | Create / submit data |

### Examples in practice
```
GET  /users?page=2&limit=10
     → Fetch users, page 2, 10 per page
     → URL is shareable and bookmarkable

POST /users
Body: { "name": "Alice", "email": "alice@email.com" }
     → Create a new user
     → Data is hidden from the URL
```

### ⚠️ Important rule
Never send **passwords, tokens, or sensitive data** in a GET request. They appear in:
- Browser address bar
- Browser history
- Server access logs
- Referrer headers

---

## 18. Handling Database Connection Failure

### Why does this matter?
Databases can go down — restarts, network issues, overload. If your backend doesn't handle this gracefully, the entire application crashes and every client gets an ugly, unhandled error.

### The right behaviour
When a DB connection fails, the server should:
1. **Catch** the error before it crashes the app
2. **Log** the full error details internally (for debugging)
3. **Return** a clean, safe message to the client (not the raw error)
4. **Respond** with `503 Service Unavailable`

### Flow
```
Request arrives
      ↓
Try to connect to DB
      ↓
❌ Connection fails
      ↓
Catch the exception
      ↓
Log: "DB connection failed at 14:32 — connection refused on port 5432"
      ↓
Respond to client: 503 "Service temporarily unavailable. Please try again later."
      ↑
      Client gets a clean message — app doesn't crash
```

### Best practices
- Use a **global exception handler** so you don't repeat try/catch in every endpoint
- Set **connection timeouts** — don't let a request hang forever
- Use **connection pooling** (HikariCP in Spring Boot) to manage DB connections efficiently
- Consider **retry logic** for transient (temporary) failures

---

## 19. Caching

### What is it?
**Caching** is the practice of storing the result of an expensive operation (like a DB query) in a faster storage location, so future requests for the same data are served instantly — without hitting the database again.

### The problem it solves
```
Without cache:
  100 users request /products
  → 100 database queries
  → DB is under heavy load
  → Responses are slow

With cache:
  1st user requests /products → DB query → result stored in cache
  Users 2–100 request /products → served from cache instantly
  → Only 1 DB query total!
```

### Types of caching

| Type | Where data is stored | Example |
|------|---------------------|---------|
| **In-memory cache** | Inside the application's RAM | Spring Cache (local) |
| **Distributed cache** | Separate fast server shared by all app instances | Redis, Memcached |
| **HTTP cache** | Browser or CDN caches the response | `Cache-Control` headers |

### When to use caching
✅ Good candidates:
- Data that is read very frequently
- Data that doesn't change often (product catalogue, categories, config)
- Expensive computations or aggregations

❌ Bad candidates:
- User-specific or real-time data (bank balance, live scores)
- Data that changes every second

### Cache invalidation
The hardest part of caching is knowing *when to clear the cache*. If data changes in the DB but the cache still holds the old version, clients get stale (outdated) data. Always clear (evict) the cache when the underlying data changes.

---

## 20. Logging Errors

### What is logging?
**Logging** is the practice of recording events that happen in your application — what requests came in, what operations ran, what errors occurred, and when.

### Why is logging important?
- In production, you can't attach a debugger — logs are your only window into what's happening
- Helps you diagnose bugs and errors after they occur
- Tracks unusual behaviour (too many failed logins = possible attack)
- Required for compliance and auditing in many industries

### Log levels — from least to most severe

| Level | When to use |
|-------|-------------|
| `DEBUG` | Very detailed info — only useful during development |
| `INFO` | Normal events — "Server started", "User logged in" |
| `WARN` | Something unexpected happened but the app continued |
| `ERROR` | A real failure occurred — needs investigation |
| `FATAL` | Critical failure — app cannot continue |

### What a good log entry contains
```
[TIMESTAMP]  [LEVEL]  [CLASS]  [Message]

2024-08-15 14:32:01  INFO   UserController   User Alice logged in successfully
2024-08-15 14:33:10  WARN   OrderService     Order ID 99 not found
2024-08-15 14:35:22  ERROR  PaymentService   Payment failed: Connection timeout to payment gateway
```

### Best practices
- Log **what happened**, **when**, and **where** (which class/method)
- Use structured logs (JSON format) in production — easier to search and filter
- Never log **passwords**, **tokens**, or **personal data**
- Write logs to a **file or log management system** (e.g., ELK Stack, Datadog) — not just the console
- Don't show raw error details to the client — log them internally and return a safe generic message

---

*End of document — 20 questions answered.*
