# 🚀 Deloitte Round 2 — JD-Specific Interview Prep

> Focus: Architecture, tooling, design patterns, AI integrations
> Strategy: You don't need to have *built* everything — you need to explain it confidently and correctly.

---

## Table of Contents

1. [What is Vite, and what are the alternatives?](#1-what-is-vite-and-what-are-the-alternatives)
2. [What is Express.js and why do we need it?](#2-what-is-expressjs-and-why-do-we-need-it)
3. [What is Prisma ORM and how is it different from TypeORM?](#3-what-is-prisma-orm-and-how-is-it-different-from-typeorm)
4. [What is PostgreSQL and how is it different from Oracle SQL?](#4-what-is-postgresql-and-how-is-it-different-from-oracle-sql)
5. [What is a RESTful API and how do you build one with OpenAPI docs?](#5-what-is-a-restful-api-and-how-do-you-build-one-with-openapi-docs)
6. [What is rate limiting and how do you implement it?](#6-what-is-rate-limiting-and-how-do-you-implement-it)
7. [Caching strategies, DB query optimisation, frontend performance tuning](#7-caching-strategies-db-query-optimisation-frontend-performance-tuning)
8. [Frontend ↔ AI services, CRM integrations, MCP server protocols](#8-frontend--ai-services-crm-integrations-mcp-server-protocols)
9. [How to do REST API design and database design](#9-how-to-do-rest-api-design-and-database-design)
10. [npm/yarn, PM2, OpenAPI/Swagger](#10-npmyarn-pm2-openapiswagger)
11. [Jest, React Testing Library, integration testing](#11-jest-react-testing-library-integration-testing)
12. [How to design and architect agentic AI solutions](#12-how-to-design-and-architect-agentic-ai-solutions)

---

## 1. What is Vite, and What Are the Alternatives?

### 🎤 Model Interview Answer

> "Vite is a **modern frontend build tool** that replaces the traditional Webpack-based setup. The key difference is that Vite uses **native ES Modules** in the browser during development — so instead of bundling the entire app before serving it, it serves files on-demand. This makes the dev server start almost instantly regardless of app size, and hot module replacement (HMR) is nearly instant too.
>
> For production builds, Vite uses **Rollup** under the hood, which generates highly optimised, tree-shaken bundles.
>
> The JD mentions React 18 + TypeScript + Vite — this is the current industry-standard stack for new React projects."

### 📖 How Vite Differs from Webpack

```
Webpack (old way):
  Start dev server → Bundle EVERYTHING → Serve bundle to browser
  → Cold start: 30–60 seconds on large apps 😩

Vite (new way):
  Start dev server → Serve files as native ES Modules directly
  → Browser requests only what it needs, when it needs it
  → Cold start: < 300ms ⚡

Production:
  Both produce optimised bundles
  Vite uses Rollup → smaller, faster output
```

### Alternatives to Vite

| Tool | What It Is | When to Use |
|------|-----------|-------------|
| **Webpack** | Oldest, most configurable bundler | Legacy projects, complex custom configs |
| **Create React App (CRA)** | Old React starter — uses Webpack | Deprecated, avoid for new projects |
| **Turbopack** | Rust-based, built for Next.js | Next.js 13+ projects (Vercel ecosystem) |
| **Parcel** | Zero-config bundler | Quick prototypes, no config needed |
| **esbuild** | Extremely fast Go-based bundler | Used inside Vite; also standalone for libraries |
| **React Compiler** (new) | Compiler that auto-memoises components | Upcoming React 19 feature — replaces manual `useMemo`/`useCallback` |

### The React Compiler Distinction (Important!)

> "The React Compiler is different from a build tool — it's a **Babel transform** that analyses your React component code at build time and automatically adds memoisation. It will eventually make `useMemo` and `useCallback` largely unnecessary. It can be used alongside Vite."

---

## 2. What is Express.js and Why Do We Need It?

### 🎤 Model Interview Answer

> "Express.js is a **minimal web framework for Node.js**. Node.js gives you raw HTTP capabilities, but Express adds structure — routing, middleware, request/response helpers, and error handling. Without Express, you'd be writing verbose `if (req.url === '/users' && req.method === 'GET')` checks manually.
>
> Express is the foundation most Node.js REST APIs are built on. The JD also mentions building Express APIs with Prisma ORM and PostgreSQL — so this is the core backend stack."

### 📖 Node.js Alone vs with Express

```javascript
// ❌ Pure Node.js HTTP — verbose, no structure
const http = require('http');
http.createServer((req, res) => {
  if (req.url === '/users' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ users: [] }));
  } else if (req.url === '/users' && req.method === 'POST') {
    // parse body manually...
  }
}).listen(3000);

// ✅ Express — clean, structured, scalable
const express = require('express');
const app = express();
app.use(express.json()); // body parsing middleware

app.get('/users', (req, res) => {
  res.json({ users: [] });
});

app.post('/users', async (req, res) => {
  const user = await createUser(req.body);
  res.status(201).json(user);
});

app.listen(3000);
```

### Key Express Concepts for the Interview

```javascript
// Middleware — runs on every request
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next(); // pass to next middleware
});

// Route params
app.get('/users/:id', (req, res) => {
  const { id } = req.params; // from URL
  const { fields } = req.query; // from ?fields=name,email
});

// Error handling middleware — 4 params, always last
app.use((err, req, res, next) => {
  res.status(err.status || 500).json({ error: err.message });
});
```

### Alternatives to Express

| Framework | Why Use It |
|-----------|-----------|
| **Fastify** | 2x faster than Express, built-in schema validation |
| **NestJS** | TypeScript-first, Angular-like structure, great for large teams (JD mentions this style) |
| **Hono** | Ultra-lightweight, edge-runtime compatible |
| **Koa** | By Express creators, cleaner async/await support |

---

## 3. What is Prisma ORM and How is it Different from TypeORM?

### 🎤 Model Interview Answer

> "Prisma is a **next-generation ORM** for Node.js and TypeScript. The key difference from TypeORM is how you define your data model — Prisma uses a dedicated **schema file** (`schema.prisma`) with its own DSL, and from that it auto-generates a fully type-safe client. TypeORM uses **decorators on TypeScript classes**.
>
> Prisma's generated client is extremely type-safe — if you query a field that doesn't exist, TypeScript catches it at compile time. The developer experience is generally considered better. The JD specifically mentions Prisma with PostgreSQL, so this is the stack they're using."

### 📖 Side-by-Side Comparison

```
TYPEORM — Decorator approach
---------------------------------
@Entity()
class User {
  @PrimaryGeneratedColumn() id: number;
  @Column() email: string;
  @Column() name: string;
}

// Query
const user = await userRepo.findOne({ where: { email } });


PRISMA — Schema + Generated client
---------------------------------
// schema.prisma
model User {
  id    Int    @id @default(autoincrement())
  email String @unique
  name  String
}

// Query (fully type-safe, auto-generated)
const user = await prisma.user.findUnique({ where: { email } });
```

### Feature Comparison Table

| Feature | Prisma | TypeORM |
|---------|--------|---------|
| **Schema definition** | `.prisma` DSL file | TypeScript decorators |
| **Type safety** | Excellent — generated client | Good — but can be bypassed |
| **Migrations** | `prisma migrate dev` — clean, version-controlled | `synchronize: true` (dev) or migration files |
| **Query builder** | Intuitive, nested `include`/`select` | More verbose, requires joins manually |
| **Raw SQL** | `prisma.$queryRaw` | `query()` method |
| **Learning curve** | Gentler | Steeper |
| **Active Record** | No | Yes (`model.save()`) |
| **DB support** | PostgreSQL, MySQL, SQLite, MongoDB | Same + Oracle, MSSQL |

### Prisma Example That Impresses Interviewers

```javascript
// Fetch user with their posts and comments — one line, fully typed
const user = await prisma.user.findUnique({
  where:   { id: userId },
  include: {
    posts: {
      include: { comments: true },
      orderBy: { createdAt: 'desc' },
      take: 10  // pagination
    }
  }
});
// TypeScript KNOWS the shape of user.posts[0].comments[0].body ✅
```

---

## 4. What is PostgreSQL and How is it Different from Oracle SQL?

### 🎤 Model Interview Answer

> "PostgreSQL is an **open-source, enterprise-grade relational database**. It's ACID-compliant, supports complex queries, JSON columns (making it hybrid relational/document), full-text search, and has excellent support for advanced data types. It's the most popular open-source RDBMS for new projects.
>
> Oracle is also a relational database but it's **proprietary, very expensive, and common in large enterprises** — especially banking, telecom, and government. The core SQL is similar, but they differ in syntax for things like sequences, pagination, stored procedures, and proprietary functions."

### 📖 Key Differences Table

| Feature | PostgreSQL | Oracle SQL |
|---------|-----------|------------|
| **Cost** | Free, open-source | Very expensive licensing |
| **Owner** | Community (PostgreSQL Global Development Group) | Oracle Corporation |
| **Used by** | Startups, modern tech companies, AWS RDS | Banks, telecom, government, SAP |
| **JSON support** | Native `jsonb` column type — excellent | Limited (added later) |
| **Syntax quirks** | `SERIAL` / `GENERATED ALWAYS AS IDENTITY` | `SEQUENCE` objects |
| **Pagination** | `LIMIT 10 OFFSET 20` | `ROWNUM` or `FETCH FIRST 10 ROWS ONLY` |
| **String concat** | `\|\|` operator | `\|\|` or `CONCAT()` |
| **NULL handling** | Standard | Standard (but `NVL()` instead of `COALESCE()`) |
| **Explain plan** | `EXPLAIN ANALYZE` | `EXPLAIN PLAN FOR` |
| **Partitioning** | Built-in | Built-in (more mature) |
| **Cloud** | AWS RDS, Aurora, Supabase, Neon | AWS RDS Oracle, Oracle Cloud |

### PostgreSQL Features Worth Mentioning

```sql
-- JSON column (hybrid relational + document)
CREATE TABLE products (
  id      SERIAL PRIMARY KEY,
  name    TEXT,
  metadata JSONB  -- queryable JSON!
);

-- Query inside JSON
SELECT * FROM products WHERE metadata->>'category' = 'electronics';

-- Full-text search (built-in, no Elasticsearch needed for basic use)
SELECT * FROM articles
WHERE to_tsvector('english', body) @@ plainto_tsquery('nodejs performance');

-- Window functions (very powerful for analytics)
SELECT name, salary,
  RANK() OVER (PARTITION BY department ORDER BY salary DESC)
FROM employees;
```

---

## 5. What is a RESTful API and How Do You Build One with OpenAPI Docs?

### 🎤 Model Interview Answer

> "REST (Representational State Transfer) is an **architectural style** for APIs — not a protocol. A RESTful API uses HTTP methods (GET, POST, PUT, PATCH, DELETE) as verbs and URLs as nouns (resources). It's **stateless** — each request contains all information needed; the server holds no session.
>
> OpenAPI (formerly Swagger) is a **specification for documenting REST APIs** in a machine-readable JSON/YAML format. From that spec you can auto-generate docs, client SDKs, and even server stubs. In practice, we either write the spec first (design-first) or generate it from code annotations (code-first)."

### 📖 REST Design Principles

```
The 6 constraints of REST:
1. Stateless         — No session on server; each request is self-contained
2. Client-Server     — Frontend and backend are separate concerns
3. Cacheable         — Responses can be cached (GET requests)
4. Uniform Interface — Consistent URL + HTTP method conventions
5. Layered System    — Client doesn't know if it's talking to a load balancer, cache, etc.
6. Code on Demand    — (Optional) Server can send executable code (e.g. JS)
```

### REST URL Design — Good vs Bad

```
Resource: Users

❌ Bad (RPC style)      ✅ Good (REST style)
GET  /getUsers         GET    /users           — list all
GET  /getUserById/1    GET    /users/1         — get one
POST /createUser       POST   /users           — create
PUT  /updateUser/1     PUT    /users/1         — full replace
POST /deleteUser/1     DELETE /users/1         — delete
                       PATCH  /users/1         — partial update

Nested resources:
GET  /users/1/posts           — posts by user 1
GET  /users/1/posts/5         — specific post by user 1
POST /users/1/posts           — create post for user 1

With query params:
GET /users?role=admin&page=2&limit=20&sort=createdAt:desc
```

### OpenAPI (Swagger) Example

```yaml
# openapi.yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0

paths:
  /users:
    get:
      summary: List all users
      parameters:
        - name: page
          in: query
          schema: { type: integer, default: 1 }
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'

  /users/{id}:
    get:
      summary: Get a user by ID
      parameters:
        - name: id
          in: path
          required: true
          schema: { type: integer }
      responses:
        '200':
          description: User found
        '404':
          description: User not found

components:
  schemas:
    User:
      type: object
      properties:
        id:    { type: integer }
        email: { type: string, format: email }
        name:  { type: string }
```

### Code-First Approach with swagger-jsdoc (Express)

```javascript
/**
 * @swagger
 * /users:
 *   get:
 *     summary: Get all users
 *     tags: [Users]
 *     responses:
 *       200:
 *         description: List of users
 */
app.get('/users', async (req, res) => { ... });

// Then serve docs at /api-docs
const swaggerUi = require('swagger-ui-express');
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));
```

---

## 6. What is Rate Limiting and How Do You Implement It?

### 🎤 Model Interview Answer

> "Rate limiting restricts how many requests a client can make in a given time window. It protects APIs from **brute force attacks, DDoS, and abuse**. For example, limiting login attempts to 5 per minute per IP prevents password brute-forcing.
>
> In Express, the simplest implementation is the `express-rate-limit` middleware. For distributed systems (multiple servers), we use **Redis** as a shared store so all servers count from the same bucket."

### 📖 Implementation Levels

```
Level 1: Simple — express-rate-limit (in-memory, single server)
Level 2: Distributed — express-rate-limit + Redis store
Level 3: Infrastructure — Cloudflare, AWS WAF, NGINX rate limiting
```

### Level 1 — Basic Implementation

```javascript
const rateLimit = require('express-rate-limit');

// General API limit
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,                  // 100 requests per window
  message: { error: 'Too many requests, please try again later.' },
  standardHeaders: true,     // Return rate limit info in headers
  legacyHeaders: false,
});

// Stricter limit for auth endpoints
const authLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 5,              // 5 login attempts per minute
  message: { error: 'Too many login attempts.' },
  skipSuccessfulRequests: true, // Don't count successful logins
});

app.use('/api', apiLimiter);
app.post('/auth/login', authLimiter, loginHandler);
```

### Level 2 — Redis for Distributed Systems

```javascript
const { RedisStore } = require('rate-limit-redis');
const redis = require('redis');

const client = redis.createClient({ url: process.env.REDIS_URL });

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  store: new RedisStore({
    sendCommand: (...args) => client.sendCommand(args),
  }),
  // Key by IP — can also key by userId for authenticated routes
  keyGenerator: (req) => req.ip,
});
```

### Rate Limit Algorithms — Know These!

| Algorithm | How It Works | Best For |
|-----------|-------------|----------|
| **Fixed window** | Count resets every N seconds | Simple, but allows burst at window edge |
| **Sliding window** | Rolling count over last N seconds | More accurate, prevents edge bursts |
| **Token bucket** | Tokens refill at fixed rate, each request costs 1 token | Allows controlled bursts |
| **Leaky bucket** | Requests processed at fixed rate, excess queued/dropped | Smooths out traffic spikes |

---

## 7. Caching Strategies, DB Query Optimisation, and Frontend Performance Tuning

### 🎤 Model Interview Answer (Overview)

> "These are three distinct but related performance concerns. Caching reduces redundant computation. Query optimisation reduces database load. Frontend tuning reduces time-to-interactive for users."

---

### Part A — Caching Strategies

```
Where to cache:
  1. Browser cache     — HTTP Cache-Control headers (static assets, API responses)
  2. CDN cache         — Cloudflare, AWS CloudFront (global edge caching)
  3. Server-side cache — Redis/Memcached (DB query results, session data)
  4. In-memory cache   — Node.js Map/object (small, frequently-read config data)
```

```javascript
// Redis caching pattern — Cache-Aside (most common)
async function getUser(userId) {
  const cacheKey = `user:${userId}`;

  // 1. Check cache first
  const cached = await redis.get(cacheKey);
  if (cached) return JSON.parse(cached); // Cache HIT ⚡

  // 2. Cache MISS — query DB
  const user = await prisma.user.findUnique({ where: { id: userId } });

  // 3. Store in cache with TTL (10 minutes)
  await redis.setEx(cacheKey, 600, JSON.stringify(user));

  return user;
}

// Invalidate cache when data changes
async function updateUser(userId, data) {
  await prisma.user.update({ where: { id: userId }, data });
  await redis.del(`user:${userId}`); // Bust the cache
}
```

### HTTP Cache Headers (frontend-facing)

```javascript
// Cache static assets for 1 year (they're content-hashed by Vite)
app.use('/assets', express.static('dist/assets', {
  maxAge: '1y',
  immutable: true
}));

// Cache API responses for 5 minutes
app.get('/api/products', (req, res) => {
  res.set('Cache-Control', 'public, max-age=300');
  res.json(products);
});
```

---

### Part B — Database Query Optimisation

```sql
-- 1. INDEXES — most impactful change
CREATE INDEX idx_users_email ON users(email);       -- single column
CREATE INDEX idx_posts_user_date ON posts(user_id, created_at DESC); -- composite

-- 2. EXPLAIN ANALYZE — find slow queries
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 5 AND status = 'pending';
-- Look for: "Seq Scan" (bad) vs "Index Scan" (good)

-- 3. SELECT only what you need — never SELECT *
-- ❌ Bad
SELECT * FROM users;
-- ✅ Good
SELECT id, name, email FROM users;

-- 4. Pagination — ALWAYS paginate large lists
SELECT id, name FROM products
ORDER BY created_at DESC
LIMIT 20 OFFSET 40;  -- page 3, 20 per page

-- 5. N+1 Problem — the most common ORM bug
-- ❌ N+1: 1 query for posts + N queries for each author
const posts = await prisma.post.findMany();
for (const post of posts) {
  const author = await prisma.user.findUnique({ where: { id: post.userId } }); // N queries!
}

-- ✅ Fixed: single query with JOIN
const posts = await prisma.post.findMany({
  include: { author: true }  // Prisma does a JOIN — 1 query total
});
```

---

### Part C — Frontend Performance Tuning

```
Key techniques:

1. Code splitting — load only what's needed
   React.lazy() + Suspense → per-route bundles

2. Image optimisation
   WebP format, lazy loading (loading="lazy"), correct sizes

3. Bundle analysis
   npx vite-bundle-visualizer → see what's bloating your bundle

4. Tree shaking
   Vite/Rollup automatically removes unused exports
   Avoid: import * as lodash — use: import { debounce } from 'lodash-es'

5. Memoisation
   React.memo() → skip re-render if props unchanged
   useMemo() → skip expensive recalculation
   useCallback() → stable function reference for child components

6. Virtual scrolling
   For long lists (1000+ items) — only render visible items
   Libraries: react-virtual, react-window

7. Web Vitals (what Google/Deloitte cares about)
   LCP (Largest Contentful Paint) — < 2.5s  (page load speed)
   FID (First Input Delay)        — < 100ms (interactivity)
   CLS (Cumulative Layout Shift)  — < 0.1  (visual stability)
```

---

## 8. Frontend ↔ AI Services, CRM Integrations, and MCP Server Protocols

### 🎤 Model Interview Answer

> "These are three integration patterns mentioned in the JD that relate to connecting the frontend application to external intelligent services."

---

### Part A — Connecting Frontend with AI Services

```
Pattern: Frontend → Your Backend → AI Service (never directly)
Reason: API keys must stay on the server; streaming responses need a proxy

Common AI services:
  - OpenAI / Anthropic (Claude) — LLM completions, embeddings
  - AWS Bedrock — managed AI models
  - Salesforce Einstein — AI within Salesforce (relevant to Deloitte/Agentforce)
```

```javascript
// Backend: proxy to OpenAI with streaming
app.post('/api/chat', authMiddleware, async (req, res) => {
  const { messages } = req.body;

  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');

  const stream = await openai.chat.completions.create({
    model: 'gpt-4',
    messages,
    stream: true,
  });

  for await (const chunk of stream) {
    const content = chunk.choices[0]?.delta?.content || '';
    res.write(`data: ${JSON.stringify({ content })}\n\n`);
  }
  res.end();
});

// Frontend: consume streaming response
const response = await fetch('/api/chat', {
  method: 'POST',
  body: JSON.stringify({ messages }),
  headers: { 'Content-Type': 'application/json' }
});
const reader = response.body.getReader();
// Process chunks as they arrive...
```

---

### Part B — CRM Integrations

> "CRM (Customer Relationship Management) integration means connecting our app to platforms like **Salesforce, HubSpot, or Microsoft Dynamics** to sync customer data. In the Deloitte context, this is almost certainly **Salesforce** since they're a top Salesforce partner."

```
Common CRM integration patterns:

1. REST API calls
   → Call Salesforce REST API to create/read/update records
   → Use OAuth 2.0 for auth (Salesforce uses Connected Apps)

2. Webhooks
   → Salesforce sends events to our backend when records change
   → e.g., "deal closed" → trigger onboarding workflow in our system

3. Agentforce (Salesforce AI)
   → Configure AI agents in Salesforce that can call your backend APIs
   → JD specifically mentions this — it's Salesforce's AI platform

Salesforce-specific terms to know:
  Object   = Table (Account, Contact, Opportunity, Lead are standard)
  Record   = Row
  SOQL     = Salesforce query language (like SQL but for Salesforce objects)
  Apex     = Salesforce's Java-like server-side language
  LWC      = Lightning Web Components (Salesforce frontend framework)
```

```javascript
// Example: Create a contact in Salesforce via REST API
const sfToken = await getSalesforceToken(); // OAuth 2.0

await axios.post(
  `${SF_INSTANCE_URL}/services/data/v58.0/sobjects/Contact`,
  {
    FirstName: 'John',
    LastName:  'Doe',
    Email:     'john@example.com',
    AccountId: '0015g00000AbcDef'
  },
  { headers: { Authorization: `Bearer ${sfToken}` } }
);
```

---

### Part C — MCP Server Protocols

> "MCP stands for **Model Context Protocol** — it's an open standard (introduced by Anthropic in 2024) that allows AI models to connect to external tools and data sources in a standardised way. Instead of every AI integration being custom-built, MCP provides a common protocol so AI agents can use tools like databases, APIs, and file systems through a unified interface."

```
Think of it like this:
  USB-C is a standard connector — any device, any charger
  MCP is like USB-C for AI — any AI model, any tool/data source

Architecture:
  AI Model (Claude, GPT) ↔ MCP Client ↔ MCP Server ↔ External Resource
                                                        (DB, API, filesystem)

MCP Server exposes:
  Tools     — actions the AI can take (e.g. "createSalesforceContact")
  Resources — data the AI can read (e.g. "getCustomerList")
  Prompts   — reusable prompt templates

Relevance to JD:
  "Connect frontend with AI services and MCP server protocols" means
  building Node.js services that implement MCP so AI agents (Agentforce/Claude)
  can call your backend tools in a standardised way.
```

```javascript
// Simple MCP server concept in Node.js
// The server exposes "tools" that an AI agent can call

const tools = {
  getCustomerById: async ({ customerId }) => {
    return await prisma.customer.findUnique({ where: { id: customerId } });
  },
  createSalesforceContact: async ({ name, email }) => {
    return await salesforce.createContact({ name, email });
  }
};

// AI agent calls your MCP server, gets back structured results
// No custom integration code needed per AI model
```

---

## 9. How to Do REST API Design and Database Design

### Part A — REST API Design Principles

```
Step 1: Identify your resources (nouns)
  Users, Products, Orders, Reviews, Categories

Step 2: Define relationships
  User HAS MANY Orders
  Order HAS MANY Products (through OrderItems)
  Product BELONGS TO Category

Step 3: Design URLs
  /users
  /users/:id
  /users/:id/orders
  /orders/:id/items

Step 4: Assign HTTP methods
  GET    → Read (safe, idempotent)
  POST   → Create (not idempotent)
  PUT    → Full replace (idempotent)
  PATCH  → Partial update (idempotent)
  DELETE → Remove (idempotent)

Step 5: Design request/response shapes
  Always return { data, meta } envelope
  Include pagination: { data: [...], meta: { total, page, limit } }

Step 6: Error responses — consistent shape
  { error: { code: "USER_NOT_FOUND", message: "...", details: {} } }

Step 7: Versioning
  /api/v1/users  ← version in URL (most common)
```

### API Response Design

```javascript
// ✅ Consistent envelope pattern
{
  "success": true,
  "data": { "id": 1, "name": "John" },
  "meta": { "requestId": "abc-123", "timestamp": "2025-04-14T10:00:00Z" }
}

// ✅ Paginated list
{
  "success": true,
  "data": [ {...}, {...} ],
  "pagination": { "total": 150, "page": 2, "limit": 20, "totalPages": 8 }
}

// ✅ Error response
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email is required",
    "fields": { "email": "Required field" }
  }
}
```

---

### Part B — Database Design Principles

```
Step 1: Identify entities → become tables
Step 2: Identify attributes → become columns
Step 3: Identify relationships → foreign keys or junction tables
Step 4: Normalise (remove redundancy)
Step 5: Add indexes for query patterns
```

### Normalisation (1NF → 3NF)

```sql
-- Bad: storing multiple values in one column (not 1NF)
users: id | name  | phone_numbers
       1  | Alice | "0901,0902"    ❌

-- Good: separate table (1NF)
users:  id | name
phones: id | user_id | number

-- Bad: redundant data (not 2NF / 3NF)
orders: id | user_id | user_email | user_name | product_id | product_name | price
-- user and product data is duplicated across rows ❌

-- Good: normalised
users:    id | email | name
products: id | name  | price
orders:   id | user_id (FK) | created_at
order_items: id | order_id (FK) | product_id (FK) | quantity | unit_price
```

### Junction Table for Many-to-Many

```sql
-- Students and Courses — many-to-many
CREATE TABLE enrollments (
  student_id INT REFERENCES students(id),
  course_id  INT REFERENCES courses(id),
  enrolled_at TIMESTAMP DEFAULT NOW(),
  PRIMARY KEY (student_id, course_id)
);
```

---

## 10. npm/yarn, PM2, OpenAPI/Swagger

### npm vs yarn

```
Both are package managers for Node.js projects.

npm  — comes with Node.js, default, most widely used
yarn — Facebook's alternative, historically faster (now comparable)

Key commands (both do the same things):

           npm                        yarn
Install:   npm install                yarn
Add pkg:   npm install express        yarn add express
Dev dep:   npm install -D jest        yarn add -D jest
Remove:    npm uninstall express      yarn remove express
Run script: npm run build             yarn build
Lock file: package-lock.json         yarn.lock
```

> **Interview tip:** "I use npm in most projects. Yarn was preferred historically for its speed and deterministic installs, but npm 7+ addressed those gaps. The lock file is critical either way — it ensures everyone installs the exact same versions."

---

### PM2 — Process Manager

> "PM2 is a **production process manager for Node.js**. When you run `node app.js`, if the process crashes, your server is down. PM2 automatically restarts crashed processes, manages logs, supports zero-downtime restarts, and can run multiple instances (cluster mode) to utilise all CPU cores."

```bash
# Start app
pm2 start app.js --name "my-api"

# Cluster mode — use all CPU cores
pm2 start app.js -i max  # max = number of CPU cores

# Auto-restart on crash
pm2 start app.js --restart-delay=3000

# Zero-downtime reload (for deployments)
pm2 reload my-api

# Monitoring dashboard
pm2 monit

# View logs
pm2 logs my-api

# Auto-start on server reboot
pm2 startup
pm2 save
```

```
Without PM2:
  node app.js → crashes → server is DOWN until someone manually restarts it

With PM2:
  pm2 start app.js → crashes → PM2 restarts it automatically in < 1 second
  Server is always running ✅
```

---

### OpenAPI / Swagger

> "OpenAPI is the **specification**. Swagger is the **tooling ecosystem** built around it — Swagger UI renders the spec as interactive docs, Swagger Editor lets you write the spec visually."

```
The flow:
  1. Write openapi.yaml (or annotate code with @swagger JSDoc)
  2. swagger-jsdoc generates the spec from annotations
  3. swagger-ui-express serves it at /api-docs
  4. Developers can test endpoints directly from the browser
  5. Frontend devs / third parties use it to understand the API
  6. Can auto-generate client SDKs in any language

Why it matters in enterprise (Deloitte context):
  → Teams working across onshore/offshore need clear API contracts
  → Client handoff requires documentation
  → Automated testing can be generated from the spec
```

---

## 11. Jest, React Testing Library, Integration Testing

### 🎤 Model Interview Answer

> "These represent the three layers of frontend testing. Jest is the **test runner and assertion library** — it runs test files, provides `describe`, `it`, `expect`. React Testing Library (RTL) is used for **component testing** — it renders components and lets you interact with them the way a user would, not by inspecting internal state. Integration testing tests that **multiple units working together** produce the correct outcome."

---

### Jest — The Foundation

```javascript
// Basic Jest test
describe('freqCheck function', () => {
  it('counts frequency of each element', () => {
    const result = freqCheck([1, 2, 2, 3]);
    expect(result).toEqual({ 1: 1, 2: 2, 3: 1 });
  });

  it('returns empty object for empty array', () => {
    expect(freqCheck([])).toEqual({});
  });
});

// Mocking — isolate the unit under test
jest.mock('../services/emailService');
jest.fn().mockResolvedValue({ id: 1, email: 'test@test.com' });

// Async tests
it('fetches user from API', async () => {
  const user = await getUser(1);
  expect(user).toHaveProperty('email');
});
```

---

### React Testing Library — Component Testing

```javascript
import { render, screen, fireEvent, waitFor } from '@testing-library/react';

// RTL philosophy: test behaviour, not implementation
// "What does a USER see and do?" — not "what is the component's internal state?"

describe('LoginForm', () => {
  it('shows error on invalid email', async () => {
    render(<LoginForm />);

    // Find elements like a user would (by role/label/text)
    const emailInput = screen.getByLabelText('Email');
    const submitBtn  = screen.getByRole('button', { name: /sign in/i });

    fireEvent.change(emailInput, { target: { value: 'notanemail' } });
    fireEvent.click(submitBtn);

    // Assert what the USER sees
    await waitFor(() => {
      expect(screen.getByText('Please enter a valid email')).toBeInTheDocument();
    });
  });
});
```

---

### Integration Testing

```javascript
// Integration test: does the route, middleware, controller, DB all work together?
// Uses supertest to make real HTTP requests to your Express app

const request = require('supertest');
const app = require('../app');

describe('POST /auth/login', () => {
  it('returns 401 for wrong password', async () => {
    const res = await request(app)
      .post('/auth/login')
      .send({ email: 'user@test.com', password: 'wrongpassword' });

    expect(res.status).toBe(401);
    expect(res.body.error).toBe('Invalid credentials');
  });

  it('returns access token for correct credentials', async () => {
    const res = await request(app)
      .post('/auth/login')
      .send({ email: 'user@test.com', password: 'correctpassword' });

    expect(res.status).toBe(200);
    expect(res.body).toHaveProperty('accessToken');
  });
});
```

### The Testing Pyramid

```
         /\
        /E2E\        Cypress, Playwright — few, slow, expensive
       /------\
      /  Integr \    supertest, MSW — moderate
     /------------\
    /  Unit Tests  \ Jest, RTL — many, fast, cheap
   /----------------\

JD says "80%+ test coverage" — means mostly unit + integration
```

---

## 12. How to Design and Architect Agentic AI Solutions

### 🎤 Model Interview Answer

> "An agentic AI solution is one where the AI doesn't just respond to a single prompt — it **autonomously plans and executes a sequence of steps** to achieve a goal, using tools along the way.
>
> The key components are: an **LLM as the reasoning engine**, a set of **tools** the agent can call, a **memory system** to retain context, and an **orchestration layer** that manages the loop of plan → act → observe → plan again.
>
> In the Salesforce context, Agentforce is Salesforce's platform for building these agents. A Deloitte consultant configures AI agents that can retrieve customer data, run approvals, and take actions in Salesforce — all autonomously."

### 📖 The Agent Loop

```
User Goal: "Find all overdue invoices and email the customers"

Agent Loop:
  Step 1 — PLAN:   "I need to query the invoices table for overdue ones"
  Step 2 — ACT:    Calls tool: getOverdueInvoices()
  Step 3 — OBSERVE: Returns: [{ id: 1, customer: "Acme", amount: 5000 }, ...]
  Step 4 — PLAN:   "Now I need to get email for each customer"
  Step 5 — ACT:    Calls tool: getCustomerEmail("Acme")
  Step 6 — ACT:    Calls tool: sendEmail(email, subject, body)
  Step 7 — OBSERVE: "Email sent successfully"
  Step 8 — PLAN:   "No more customers, task complete"
  → Done ✅
```

### Architecture Components

```
1. LLM (Reasoning Engine)
   → Claude, GPT-4, Llama — takes context + available tools, decides what to do

2. Tools / Functions
   → What the agent can DO: query DB, call APIs, send emails, create records
   → Defined as JSON schema so LLM understands inputs/outputs

3. Memory
   → Short-term: conversation history (in-context)
   → Long-term: vector DB (Pinecone, pgvector) for semantic search over past data

4. Orchestration Layer
   → The loop that runs: LLM call → tool execution → result back to LLM → repeat
   → Frameworks: LangChain, LlamaIndex, Vercel AI SDK, Salesforce Agentforce

5. Human-in-the-Loop (HITL)
   → For high-stakes actions (sending bulk emails, financial transactions)
   → Agent pauses and requests human approval before proceeding
```

### Simple Agentic Pattern in Code

```javascript
// Tool definition (OpenAI function calling style — same concept in all frameworks)
const tools = [
  {
    name: "getOverdueInvoices",
    description: "Returns all invoices past their due date",
    parameters: {
      type: "object",
      properties: {
        daysOverdue: { type: "number", description: "Minimum days overdue" }
      }
    }
  },
  {
    name: "sendEmail",
    description: "Sends an email to a customer",
    parameters: {
      type: "object",
      properties: {
        to:      { type: "string" },
        subject: { type: "string" },
        body:    { type: "string" }
      },
      required: ["to", "subject", "body"]
    }
  }
];

// Agent loop
async function runAgent(userGoal) {
  const messages = [{ role: "user", content: userGoal }];

  while (true) {
    const response = await llm.chat({ messages, tools });

    if (response.finishReason === "tool_use") {
      // Execute the tool the LLM chose
      const toolResult = await executeTool(response.toolCall);
      messages.push({ role: "tool", content: toolResult });
      // Loop: give result back to LLM, it decides next step
    } else {
      // LLM finished — return final answer
      return response.content;
    }
  }
}
```

### Agentforce (Salesforce) — Specific to JD

```
Agentforce is Salesforce's platform for building AI agents:

  Einstein Copilot  → Conversational AI in Salesforce UI
  Agent Builder     → Low-code tool to define agent skills/actions
  Apex Actions      → Custom code agents can call (backend logic)
  Flow Automation   → Existing Salesforce automations agents can trigger

As a consultant configuring Agentforce:
  1. Define the agent's TOPIC (what it handles — e.g. "Invoice Management")
  2. Define ACTIONS (tools — e.g. queryInvoices, sendReminder)
  3. Write INSTRUCTIONS (system prompt for the agent)
  4. Configure guardrails (what it can/cannot do autonomously)
  5. Test with Einstein Trust Layer (data stays in Salesforce, not sent externally)
```

---

## Quick Revision — One-Liner Answers

| Topic | Say This |
|-------|----------|
| **Vite** | "Modern build tool — uses native ES Modules for instant dev server startup, Rollup for production bundles" |
| **Express** | "Minimal Node.js web framework — adds routing, middleware, and request handling on top of raw Node HTTP" |
| **Prisma vs TypeORM** | "Prisma uses a schema file + auto-generated type-safe client; TypeORM uses decorators on classes. Prisma has better DX and type safety." |
| **PostgreSQL vs Oracle** | "PostgreSQL is open-source, free, great JSON support, used by modern teams. Oracle is proprietary, expensive, dominant in enterprise/banking." |
| **RESTful API** | "Architectural style using HTTP verbs on resource URLs — stateless, consistent, uses OpenAPI for documentation" |
| **Rate limiting** | "Restricts requests per time window — express-rate-limit for single server, Redis store for distributed; protects against brute force and DDoS" |
| **Caching** | "Cache-aside pattern with Redis — check cache first, on miss fetch from DB and store with TTL" |
| **DB optimisation** | "Add indexes on queried columns, use EXPLAIN ANALYZE, avoid SELECT *, fix N+1 with includes/joins" |
| **MCP** | "Anthropic's open protocol for AI-to-tool connectivity — standardises how AI agents call external tools and data sources" |
| **PM2** | "Node.js process manager — auto-restart on crash, cluster mode for multi-core, zero-downtime deploys" |
| **Jest** | "Test runner + assertion library — runs test files, provides expect(), mocking, coverage reports" |
| **RTL** | "Renders React components and tests them from a user's perspective — by role, label, text — not internal state" |
| **Agentic AI** | "AI that autonomously plans and executes multi-step tasks using tools — LLM + tools + memory + orchestration loop" |

---

## Phrases That Show Seniority

Use these naturally in answers:

- *"In a distributed system, I'd use Redis to share the rate limit state across instances..."*
- *"The N+1 problem is one of the most common ORM pitfalls — solved with eager loading via `include`..."*
- *"For Agentforce, the Einstein Trust Layer ensures customer data doesn't leave the Salesforce boundary..."*
- *"OpenAPI serves as the contract between frontend and backend teams — especially important in onshore/offshore delivery..."*
- *"I'd recommend Prisma over TypeORM for new projects — the generated client's type safety catches schema mismatches at compile time..."*
- *"Rate limiting at the application layer is a first line of defence — in production I'd layer it with Cloudflare WAF..."*

---

*Deloitte Round 2 Prep — April 2026 | Customer Team — Sales & Service*
