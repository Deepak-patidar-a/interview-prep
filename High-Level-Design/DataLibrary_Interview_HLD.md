# 📘 Data Library Project — Interview Q&A Revision Guide

> **Project:** Data Library API — Node.js middleware between The Hub and MerchOps Platform  
> **Your Role:** Only Node.js API Developer  
> **Stack:** Node.js, Express, IO-TS, Basic Auth, SDI Staging Tables, Stored Procedures

---

## 🏗️ Architecture Overview

```
The Hub  ──(invokes)──►  API (Node Server)  ──►  MerchOps Platform
                          │
                          ├── Basic Auth (MerchOps DB)
                          ├── Type Checking (IO-TS)
                          ├── Payload Modification
                          ├── SDI Staging Table Insert
                          ├── Pre-process Stored Procedure
                          └── Response (success / error with code)
```

---

## 🔐 1. Authentication

### Q: Did you use any API Gateway?
**A:** No. We used our existing Node server that was already built for the MerchOps platform. It acts as the entry point and handles all middleware logic including auth.

---

### Q: How are you doing Authentication?
**A:** Using Basic Auth — the caller (The Hub) sends a username and password with every API request in the Authorization header, encoded in Base64.

---

### Q: From where are you validating the username and password?
**A:** We validate against the MerchOps user database. On every request, we decode the Base64 credentials from the header and check them against the DB.

---

### Q: How would you detect an invalid user?
**A:** If the username doesn't exist in the MerchOps user database, or if the password doesn't match, we return a `401 Unauthorized` response immediately before any further processing.

---

### Q: Is Basic Auth safe? Why not use JWT?
**A:** Basic Auth is simpler and works well for internal service-to-service communication where the network is trusted and controlled. However, it has limitations:
- Credentials are sent with every request (Base64 is encoding, not encryption)
- No token expiry mechanism
- Requires a DB hit on every request

JWT would be better for user-facing APIs because:
- It's stateless — no DB hit needed for every request
- Has built-in expiry (`exp` claim)
- Can carry roles/claims for authorization

In our case, since this is an **internal API on a secured network**, Basic Auth was acceptable. If it were public-facing, I'd recommend JWT or OAuth2.

---

### Q: How are you sending Username and Password?
**A:** In the `Authorization` HTTP header with every API call:
```
Authorization: Basic <base64(username:password)>
```
The value is Base64 encoded (not encrypted), so HTTPS/TLS is essential to ensure the credentials are not exposed in transit.

---

### Q: Base64 is not encryption — what if someone intercepts the header?
**A:** That's correct — Base64 is just encoding. The protection comes from **HTTPS/TLS**, which encrypts the entire HTTP request including headers. Without HTTPS, Basic Auth would be highly insecure.

---

### Q: How would you prevent brute force attacks on Basic Auth?
**A:** A few approaches:
- Implement **rate limiting** per IP (e.g., using `express-rate-limit`)
- Lock account after N failed attempts
- Add IP whitelisting since it's an internal API
- Log and alert on repeated auth failures

---

### Q: What's the difference between Authentication and Authorization? Did you implement Authorization?
**A:** 
- **Authentication** = verifying *who* you are (username/password check)
- **Authorization** = verifying *what* you're allowed to do (roles/permissions)

In this project, I primarily handled Authentication. Authorization was implicitly handled by the fact that only The Hub (a trusted internal system) was the caller, so role-based access wasn't a requirement here.

---

### Q: How would you migrate to JWT without breaking existing callers?
**A:** 
1. Add a new `/auth/token` endpoint that accepts username/password and returns a JWT
2. Run both Basic Auth and JWT auth middleware in parallel (support both during transition)
3. Notify The Hub team to migrate their calls to use the token
4. Once all callers are migrated, deprecate Basic Auth

---

## 🧪 2. Type Validation (IO-TS)

### Q: Why IO-TS over Zod or Joi?
**A:** IO-TS integrates well with TypeScript's type system — it provides **runtime type safety** that mirrors compile-time types, which makes it consistent. The project was already using TypeScript, so IO-TS felt like a natural fit. That said, Zod is more ergonomic and increasingly popular — if starting fresh today I'd evaluate Zod seriously.

---

### Q: Where in the middleware chain did you place type validation?
**A:** After authentication, before business logic:
```
Auth → Type Validation → Payload Modification → DB Insert → Response
```
This ensures we don't waste DB resources on invalid payloads, and only authenticated + valid requests proceed further.

---

### Q: What HTTP status code do you return on type validation failure?
**A:** `400 Bad Request` — because it's a client error (they sent wrong data). The response body includes a descriptive error message indicating which field failed and why.

---

### Q: Did you validate nested objects?
**A:** Yes, IO-TS supports recursive/nested type definitions, so we validated the full shape of the payload including nested objects, not just top-level fields.

---

## 📦 3. Payload Handling & Chunking

### Q: What was the biggest challenge you faced?
**A:** During testing, testers were sending large JSON payloads. Express has a default JSON body size limit of **5KB**, which caused the API to timeout. I researched this, increased the limit to **1MB** in Express config:
```js
app.use(express.json({ limit: '1mb' }));
```
Additionally, instead of inserting the full payload at once, I implemented **chunking** — splitting the payload into smaller parts and inserting them sequentially into the staging tables. This made DB inserts more manageable and reliable.

---

### Q: How did you implement chunking?
**A:** I split the array/payload in-memory into fixed-size chunks (e.g., 100 records per chunk) using a utility function, then iterated and inserted each chunk in sequence.
```js
const chunkArray = (arr, size) =>
  Array.from({ length: Math.ceil(arr.length / size) },
    (_, i) => arr.slice(i * size, i * size + size));
```

---

### Q: How did you handle partial failures — if chunk 3 of 5 fails, what happens to chunks 1 and 2?
**A:** This is an important edge case. Ideally, the entire operation should be wrapped in a **database transaction** so that if any chunk fails, all previous inserts are rolled back. If transactions weren't used, we'd need to implement a cleanup/rollback step manually or mark records with a status flag so failed batches can be retried cleanly.

---

### Q: Did you consider other solutions besides increasing the limit?
**A:** Yes, alternatives I considered:
- **Streaming** the request body instead of buffering it all at once
- **Compression** (gzip) on the client side to reduce payload size
- Asking callers to paginate/batch their requests
- Using a message queue (like RabbitMQ) so large payloads don't block the API

Increasing the limit + chunking was the quickest pragmatic fix given the timeline.

---

## 🗄️ 4. SDI Staging Tables & Stored Procedure

### Q: Why use staging tables instead of inserting directly into main tables?
**A:** Staging tables act as a **buffer/validation layer**:
- Raw data lands in staging first without affecting production data
- The stored procedure then validates and transforms it (business rules)
- Only clean, validated data moves to the main tables
- Easier to debug, retry, or rollback if something goes wrong

---

### Q: What does the pre-process stored procedure validate?
**A:** Business validation rules — for example: checking mandatory fields have valid values, checking referential integrity (does this ID exist in related tables), applying fixed value mappings or sequence generation. The exact rules were defined by the senior consultants and DB team.

---

### Q: Who owned the stored procedure — you or the DB team?
**A:** The DB team owned the stored procedure. My role was to call it correctly after insertion and handle its response — success or error codes — and translate those into API responses for the caller.

---

### Q: What happens if the stored procedure times out?
**A:** We should handle it with a try/catch, return a `500 Internal Server Error`, and log the failure. Ideally, a timeout threshold should be configured and the staging records should be marked with a "failed" status so they can be retried or investigated.

---

### Q: How do you handle duplicate data sent to the staging table?
**A:** The stored procedure's business validation can catch duplicates using unique constraints or by checking existing records. At the API level, we can also implement idempotency keys — if the same request ID is received twice, we return the cached result without re-inserting.

---

## ⚠️ 5. Error Handling

### Q: What error structure do you return?
**A:** A consistent JSON error structure:
```json
{
  "success": false,
  "errorCode": "VALIDATION_FAILED",
  "message": "Field 'productId' is required and must be a string"
}
```
This makes it easy for the caller (The Hub) to handle errors programmatically.

---

### Q: How do you differentiate client errors vs server errors?
**A:**
- `400` — Bad request (wrong payload, type mismatch)
- `401` — Unauthorized (auth failure)
- `422` — Unprocessable entity (business validation failure from stored procedure)
- `500` — Internal server error (DB failure, timeout, unexpected error)

---

### Q: Do you log errors?
**A:** Yes, errors are logged at the application level (console/logger). Ideally a tool like Winston or a centralized logging service should be used so errors can be monitored and alerted on in production.

---

### Q: Is there a retry mechanism if The Hub doesn't get a response?
**A:** That responsibility lies on The Hub side — they should implement retry with exponential backoff for network failures. On our API side, we ensure **idempotency** so retried requests don't cause duplicate inserts.

---

## 📐 6. Architecture & Scalability

### Q: What if traffic increases 10x?
**A:** The current single Node server would become a bottleneck. Solutions:
- **Horizontal scaling** — run multiple instances behind a load balancer
- **PM2 cluster mode** — use all CPU cores on the same machine
- Add a **message queue** (RabbitMQ/Kafka) between The Hub and API to decouple and buffer traffic
- **Connection pooling** for DB to handle more concurrent queries

---

### Q: Why no message queue between The Hub and API?
**A:** For the current scale and internal nature of the project, synchronous HTTP was sufficient and simpler. A queue would add operational complexity. However, for higher throughput or guaranteed delivery requirements, a queue like RabbitMQ would be the right call.

---

### Q: Is your API idempotent?
**A:** Idempotency means calling the API with the same payload multiple times produces the same result without side effects (e.g., no duplicate inserts). This should be implemented using a unique request ID — if the same ID is seen again, return the previous result without re-processing.

---

## 🟨 7. Node.js / Express Deep Dives

### Q: How did you structure your Express middleware chain?
**A:**
```
Request
  → Auth Middleware (Basic Auth validation)
  → Type Validation Middleware (IO-TS)
  → Payload Modification (sequence, fixed values)
  → Route Handler (DB insert + stored procedure call)
  → Response (success or error)
```
Each middleware calls `next()` on success or `next(error)` on failure, which routes to a centralized error handler.

---

### Q: How do you handle async errors in Express?
**A:** Express doesn't catch async errors automatically. You need to either:
1. Wrap async route handlers with a `try/catch` and call `next(err)`
2. Use a wrapper utility:
```js
const asyncHandler = (fn) => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);
```
Without this, unhandled promise rejections can crash the server silently.

---

### Q: Did you write unit tests?
**A:** Unit tests for critical middlewares like auth validation and type checking are important. Tools like Jest + Supertest can be used to test Express middleware in isolation. In this project given the fast pace, full coverage wasn't done but it's something I'd prioritize in a more stable phase.

---

## 💬 8. Soft Skills / Behavioural

### Q: What did you learn most from this project?
**A:** Beyond the technical side, I learned **defensive API design** — validating inputs early, failing fast with clear error messages, and building APIs that are forgiving for callers but strict internally. That shift from "making it work" to "making it production-ready" was the biggest growth.

I was the **only Node.js developer** on the project, working closely with senior consultants and a foreign client. This taught me a lot about communication, managing time zones, and delivering fixes quickly under pressure. Consulting-style work — where you need to understand requirements fast and translate them to technical solutions — was a major learning.

---

### Q: The project seems simple — is there really enough depth here?
**A:** The architecture is straightforward, but the **depth is in the details**: handling large payloads and chunking, understanding why Basic Auth is acceptable in some contexts but not others, coordinating between API and DB layers via stored procedures, designing middleware chains, and operating as the sole developer on a fast-paced client project. These are real engineering decisions, not toy problems.

---

*Last updated: May 2026 | Revision guide for interview prep*
