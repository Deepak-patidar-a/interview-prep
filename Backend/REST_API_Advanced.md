# 🌐 REST API & Advanced Node.js — Interview Revision Notes

> Topic: REST principles, HTTP methods, status codes, auth, scaling, performance  
> Covers both REST API design and Advanced Node.js concepts

---

## 🧠 What is REST?

**REST** (Representational State Transfer) is an **architectural style** for building APIs using HTTP. It defines a set of constraints that make APIs scalable, stateless, and uniform.

- Not a protocol, not a library — just a **design standard**
- Uses standard **HTTP methods** (GET, POST, PUT, PATCH, DELETE)
- Data exchanged in **JSON** (most common) or XML
- Each request is **independent** (stateless)

---

## 📐 6 REST Constraints (Principles)

| # | Constraint | Meaning |
|---|---|---|
| 1 | **Stateless** | Each request must contain all info. Server stores no session state. |
| 2 | **Client-Server** | UI and backend are separated, communicate only via API. |
| 3 | **Cacheable** | Responses must declare if they can/can't be cached. |
| 4 | **Uniform Interface** | Consistent URL structure, HTTP methods, response format. |
| 5 | **Layered System** | Client doesn't know if it talks to server directly or via load balancer. |
| 6 | **Code on Demand** | (Optional) Server can send executable code (e.g., JS scripts). |

> Most important for interviews: **Stateless** and **Uniform Interface**

---

## 🔧 HTTP Methods — CRUD Mapping

| Method | CRUD | Use Case | Idempotent? |
|---|---|---|---|
| `GET` | Read | Fetch data. No body. | ✅ Yes |
| `POST` | Create | Create new resource. Has body. | ❌ No |
| `PUT` | Update (full) | Replace entire resource. | ✅ Yes |
| `PATCH` | Update (partial) | Update only specific fields. | ⚠️ Usually |
| `DELETE` | Delete | Remove resource. | ✅ Yes |

> **Idempotent** = calling it multiple times gives the same result.  
> Example: DELETE /users/1 multiple times → user is deleted, same result each time ✅  
> POST /users multiple times → creates multiple users ❌

### PUT vs PATCH:
```javascript
// PUT — replaces entire user (missing fields become null/default)
PUT /users/1
Body: { "name": "John" }
// Result: { "name": "John", "email": null, "age": null }

// PATCH — only updates specified fields
PATCH /users/1
Body: { "name": "John" }
// Result: { "name": "John", "email": "old@email.com", "age": 25 }
```

---

## 📊 HTTP Status Codes

### 2xx — Success
| Code | Name | Use When |
|---|---|---|
| `200` | OK | Successful GET, PUT, PATCH |
| `201` | Created | Successful POST (new resource created) |
| `204` | No Content | Successful DELETE (no body to return) |

### 3xx — Redirection
| Code | Name | Use When |
|---|---|---|
| `301` | Moved Permanently | URL has changed permanently |
| `304` | Not Modified | Cached version is still valid |

### 4xx — Client Errors
| Code | Name | Use When |
|---|---|---|
| `400` | Bad Request | Invalid data sent |
| `401` | Unauthorized | Not logged in / no valid token |
| `403` | Forbidden | Logged in but no permission |
| `404` | Not Found | Resource doesn't exist |
| `409` | Conflict | Duplicate resource (e.g., email already exists) |
| `422` | Unprocessable Entity | Validation failed |
| `429` | Too Many Requests | Rate limit exceeded |

### 5xx — Server Errors
| Code | Name | Use When |
|---|---|---|
| `500` | Internal Server Error | Something broke on the server |
| `502` | Bad Gateway | Upstream server error |
| `503` | Service Unavailable | Server overloaded or down |

---

## 🗺️ REST API Design — Best Practices

### URL Design Rules:
```
✅ Good:                          ❌ Bad:
GET    /api/users                 GET  /api/getUsers
POST   /api/users                 POST /api/createUser
GET    /api/users/123             GET  /api/getUserById?id=123
PUT    /api/users/123             POST /api/updateUser/123
DELETE /api/users/123             GET  /api/deleteUser/123
GET    /api/users/123/posts       GET  /api/getUserPosts/123
```

**Rules:**
- Use **nouns**, not verbs in URLs
- Use **plural** resource names (`/users` not `/user`)
- Use **HTTP method** to indicate the action
- Use **nested routes** for relationships (`/users/:id/posts`)
- Use **lowercase** and hyphens for multi-word (`/blog-posts` not `/blogPosts`)

### Response Format (always be consistent):
```javascript
// Success response
{
  "success": true,
  "data": { ... },
  "message": "User created successfully"
}

// Error response
{
  "success": false,
  "message": "User not found",
  "error": "NOT_FOUND"
}

// List response with pagination
{
  "success": true,
  "data": [...],
  "pagination": {
    "total": 100,
    "page": 2,
    "limit": 10,
    "totalPages": 10
  }
}
```

---

## 🔐 Authentication — JWT vs Sessions

### JWT (JSON Web Token) — Stateless ✅ Preferred for REST
```javascript
// Structure: header.payload.signature
// eyJhbGc... . eyJ1c2VySW... . signature

// Login
const token = jwt.sign(
  { id: user._id, role: user.role },
  process.env.JWT_SECRET,
  { expiresIn: '15m' }           // short expiry!
);

// Client sends: Authorization: Bearer <token>
// Server verifies without DB lookup — fast!
```

### Sessions — Stateful
```javascript
// Server stores session in DB/memory
// Client sends session cookie
// Doesn't scale horizontally without sticky sessions or shared session store
```

| Feature | JWT | Sessions |
|---|---|---|
| State | Stateless | Stateful |
| Storage | Client (token) | Server (DB/memory) |
| Scalability | ✅ Easy | ⚠️ Needs shared store |
| Revocation | Hard (needs blocklist) | Easy (delete from DB) |
| Best for | REST APIs, microservices | Traditional web apps |

> **Best Practice:** JWT access token (15min expiry) + Refresh token (7 days, stored in httpOnly cookie)

---

## 📄 Pagination

### Offset Pagination (most common):
```javascript
// GET /api/users?page=2&limit=10
app.get('/api/users', async (req, res) => {
  const page = parseInt(req.query.page) || 1;
  const limit = parseInt(req.query.limit) || 10;
  const skip = (page - 1) * limit;

  const users = await User.find().skip(skip).limit(limit);
  const total = await User.countDocuments();

  res.json({
    success: true,
    data: users,
    pagination: {
      total,
      page,
      limit,
      totalPages: Math.ceil(total / limit)
    }
  });
});
```

### Cursor Pagination (for real-time data):
```javascript
// GET /api/users?cursor=lastId&limit=10
// Better for feeds — no duplicate/missing items when data changes
const users = await User.find({ _id: { $gt: cursor } }).limit(limit);
```

---

## 🔢 API Versioning

When you update your API, don't break existing clients — version it!

```javascript
// URI versioning (most common + recommended in interviews)
app.use('/api/v1/users', v1UserRoutes);
app.use('/api/v2/users', v2UserRoutes);

// Header versioning (cleaner URLs)
// Client sends: Accept: application/vnd.myapi.v2+json

// Query param versioning
// GET /api/users?version=2
```

---

## 🛡️ Rate Limiting

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,                  // 100 requests per window per IP
  standardHeaders: true,     // Return rate limit info in headers
  message: {
    success: false,
    message: 'Too many requests, please try again after 15 minutes'
  }
});

app.use('/api/', limiter);
```

---

## ⚡ REST vs GraphQL vs WebSocket

| Feature | REST | GraphQL | WebSocket |
|---|---|---|---|
| Endpoint | Multiple | Single (`/graphql`) | Persistent connection |
| Data fetching | Fixed response | Client defines what to get | Real-time push |
| Over-fetching | ✅ Can happen | ❌ Never | N/A |
| Use case | CRUD APIs | Complex data needs | Chat, live updates |
| Caching | ✅ Easy (HTTP cache) | ⚠️ Complex | ❌ No |

---

## 🚀 Scaling Node.js Applications

### Cluster Module:
```javascript
const cluster = require('cluster');
const os = require('os');
const numCPUs = os.cpus().length;

if (cluster.isMaster) {
  console.log(`Master ${process.pid} running`);
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork(); // create worker per CPU core
  }
  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork(); // restart dead worker
  });
} else {
  require('./app'); // each worker runs the app
  console.log(`Worker ${process.pid} started`);
}
```

### Worker Threads (CPU-intensive tasks):
```javascript
const { Worker, isMainThread, parentPort } = require('worker_threads');

if (isMainThread) {
  const worker = new Worker('./heavy-task.js');
  worker.on('message', result => console.log(result));
  worker.postMessage({ data: 'process this' });
} else {
  parentPort.on('message', (data) => {
    // heavy CPU computation here — won't block main thread
    const result = heavyComputation(data);
    parentPort.postMessage(result);
  });
}
```

### Cluster vs Worker Threads:
| Feature | Cluster | Worker Threads |
|---|---|---|
| Creates | Multiple processes | Multiple threads (1 process) |
| Memory | Separate per process | Shared memory possible |
| Use for | Handling more HTTP requests | CPU-intensive computation |
| Communication | IPC (slower) | SharedArrayBuffer (faster) |

---

## 🔍 Memory Leak Prevention

### Common causes:
- Global variables that keep growing
- Event listeners not removed
- Unclosed DB connections
- Closures holding large references

```javascript
// ❌ Memory leak — listener accumulates
emitter.on('data', handler);

// ✅ Remove when done
emitter.on('data', handler);
// later...
emitter.removeListener('data', handler);
// or use once() for single-use listeners
emitter.once('data', handler);
```

### Debug memory leaks:
```bash
node --inspect app.js   # then open chrome://inspect
```

---

## ❓ Interview Questions — REST API & Advanced

| # | Question | Difficulty |
|---|---|---|
| 1 | What are the 6 REST constraints/principles? | 🔴 Most Asked |
| 2 | What is the difference between PUT and PATCH? | 🟡 Medium |
| 3 | What does idempotent mean? Which HTTP methods are idempotent? | 🔴 Most Asked |
| 4 | Design a RESTful API for a "Users" resource | 🔴 Most Asked |
| 5 | Explain important HTTP status codes | 🟡 Medium |
| 6 | JWT vs Sessions — which do you use for REST and why? | 🔴 Most Asked |
| 7 | How do you implement pagination in a REST API? | 🟡 Medium |
| 8 | What is API versioning? How do you implement it? | 🟡 Medium |
| 9 | REST vs GraphQL — when to use which? | 🟡 Medium |
| 10 | How do you scale a Node.js application? | 🔴 Hard |
| 11 | Cluster vs Worker Threads — what's the difference? | 🔴 Hard |
| 12 | How do you prevent memory leaks in Node.js? | 🔴 Hard |
| 13 | What is WebSocket and when to use over REST? | 🟡 Medium |
| 14 | How do you secure a REST API? | 🔴 Hard |

---

## 💡 Key Interview Answers

### REST Stateless:
> *"Each request must contain all information needed to process it. The server stores no session state. This is why we send JWT token in every request's Authorization header — the server can't remember who you are between requests."*

### Idempotent:
> *"An idempotent operation produces the same result no matter how many times you call it. GET, PUT, DELETE are idempotent. POST is not — calling POST /users three times creates three users."*

### REST API Design:
> *"Use nouns not verbs in URLs, use plural resource names, and use HTTP methods to indicate actions. GET /users fetches all users, POST /users creates one, GET /users/:id fetches one, PUT updates it, DELETE removes it."*

### Scaling:
> *"Node.js runs on a single thread by default, so to use all CPU cores I use the Cluster module which forks one worker process per core. In production I use PM2 with 'pm2 start app.js -i max'. For CPU-intensive tasks that would block the event loop, I use Worker Threads."*

---

*Previous: [03_ExpressJS.md](./03_ExpressJS.md)*

---

## 📚 Quick Revision Cheatsheet

```
Node.js = V8 + libuv (async I/O)
libuv   = Thread Pool + Event Loop + Callback Queues
Tick    = One full cycle of Event Loop

Priority: nextTick > Promise > setTimeout > setImmediate

REST constraints: Stateless, Client-Server, Cacheable,
                  Uniform Interface, Layered, Code-on-demand

HTTP Methods: GET(read) POST(create) PUT(replace) PATCH(update) DELETE(remove)
Idempotent:   GET ✅   POST ❌     PUT ✅       PATCH ⚠️    DELETE ✅

Status codes: 200 OK | 201 Created | 204 No Content
              400 Bad Request | 401 Unauth | 403 Forbidden | 404 Not Found
              500 Server Error | 503 Service Unavailable

Auth: JWT (stateless, send in Authorization header) > Sessions for REST APIs
Scale: Cluster (multiple processes) | Worker Threads (CPU tasks) | PM2
```
