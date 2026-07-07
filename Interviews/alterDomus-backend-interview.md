# AlterDomus Backend Interview — Revision Notes

Topics: Node.js Concurrency, Backpressure, Event Loop, AI Tool Hallucinations, Token Optimization, Race Conditions, Concurrent Request Handling.

---

## 1. Handling Concurrency in Node.js (Without Scaling Infra)

Node.js is single-threaded, but you can handle high concurrency through:

### A) Request Queuing with Concurrency Limits
```javascript
import pLimit from 'p-limit';

const limit = pLimit(10); // max 10 concurrent operations

app.post('/process', async (req, res) => {
  const result = await limit(() => heavyAsyncOperation(req.body));
  res.json(result);
});
```

### B) Worker Threads (CPU-bound tasks)
```javascript
import { Worker, isMainThread, parentPort, workerData } from 'worker_threads';

// main.js
function runInWorker(data) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./worker.js', { workerData: data });
    worker.on('message', resolve);
    worker.on('error', reject);
  });
}

// worker.js
const result = heavyCpuTask(workerData);
parentPort.postMessage(result);
```

### C) Cluster Module (multi-core utilization)
```javascript
import cluster from 'cluster';
import os from 'os';

if (cluster.isPrimary) {
  const cpus = os.cpus().length;
  for (let i = 0; i < cpus; i++) cluster.fork();

  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} died, restarting...`);
    cluster.fork(); // auto-restart
  });
} else {
  startServer(); // each worker runs its own server
}
```

### D) Rate Limiting + Circuit Breaker
```javascript
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 1 * 60 * 1000, // 1 minute
  max: 100,                  // 100 req/min per IP
  handler: (req, res) => res.status(429).json({ error: 'Too many requests' })
});

app.use('/api/', limiter);
```

**One-liner for interview:** *"Node handles concurrency via the event loop for I/O-bound work; for CPU-bound work I'd offload to worker threads or scale horizontally with cluster; and I'd protect the app with concurrency limiters and rate limiting so it degrades gracefully instead of crashing under load."*

---

## 2. Backpressure in Node.js

**Backpressure** = when a writable stream can't consume data as fast as a readable stream produces it — causing memory bloat or crashes.

### Where it helps
- File uploads/downloads
- Piping large DB result sets to HTTP response
- Log processing pipelines

### ❌ Without Backpressure (dangerous)
```javascript
// This floods memory — readable doesn't wait for writable
readable.on('data', (chunk) => {
  writable.write(chunk); // ignores return value!
});
```

### ✅ With Backpressure using `.pipe()`
```javascript
// pipe() handles backpressure automatically
fs.createReadStream('huge-file.csv')
  .pipe(transformStream)
  .pipe(res); // pauses reading if res is slow
```

### ✅ Manual Backpressure Control
```javascript
readable.on('data', (chunk) => {
  const canContinue = writable.write(chunk);

  if (!canContinue) {
    readable.pause(); // stop reading — consumer is overwhelmed

    writable.once('drain', () => {
      readable.resume(); // resume when consumer catches up
    });
  }
});
```

### ✅ Using Async Generators (modern approach)
```javascript
async function* readChunks(readable) {
  for await (const chunk of readable) {
    yield chunk; // naturally handles backpressure
  }
}

for await (const chunk of readChunks(fileStream)) {
  await writeToDB(chunk); // awaits each write before reading next
}
```

**One-liner for interview:** *"Backpressure is the mechanism that keeps a fast producer from overwhelming a slow consumer. In Node I handle it by using `.pipe()` (which manages pause/resume + drain automatically) or manually checking the return value of `write()` and pausing the readable stream until `drain` fires."*

---

## 3. Node.js Event Loop Phases

```
   ┌──────────────────────────┐
   │         timers           │  → setTimeout, setInterval callbacks
   └──────────────┬───────────┘
   ┌──────────────┴───────────┐
   │     pending callbacks    │  → I/O errors from previous iteration
   └──────────────┬───────────┘
   ┌──────────────┴───────────┐
   │       idle, prepare      │  → internal use only
   └──────────────┬───────────┘
   ┌──────────────┴───────────┐
   │          poll            │  → fetch new I/O events (core phase)
   └──────────────┬───────────┘
   ┌──────────────┴───────────┐
   │          check           │  → setImmediate() callbacks
   └──────────────┬───────────┘
   ┌──────────────┴───────────┐
   │     close callbacks      │  → socket.on('close'), etc.
   └──────────────────────────┘
```

**Between each phase:** `process.nextTick()` and Promise microtasks run first — always, before moving to the next phase.

```javascript
setTimeout(() => console.log('1. timer'), 0);
setImmediate(() => console.log('2. check'));
Promise.resolve().then(() => console.log('3. microtask'));
process.nextTick(() => console.log('4. nextTick'));

// Output order:
// 4. nextTick  (highest priority)
// 3. microtask
// 1. timer
// 2. check
```

**One-liner for interview:** *"The event loop has 6 phases — timers, pending callbacks, idle/prepare, poll, check, and close callbacks. Between every phase (and every callback within a phase), Node drains the `process.nextTick` queue first, then the microtask/Promise queue, before continuing."*

---

## 4. Handling Hallucinations in AI Tool Calls

### A) Schema Validation (Zod)
```javascript
import { z } from 'zod';

const toolResponseSchema = z.object({
  action: z.enum(['search', 'create', 'update', 'delete']),
  entityId: z.string().uuid(),
  fields: z.record(z.string())
});

async function callAITool(prompt) {
  const raw = await llm.invoke(prompt);

  const parsed = toolResponseSchema.safeParse(JSON.parse(raw));
  if (!parsed.success) {
    return await retryWithCorrection(prompt, parsed.error);
  }
  return parsed.data;
}
```

### B) Retry with Self-Correction
```javascript
async function retryWithCorrection(prompt, error, retries = 2) {
  if (retries === 0) throw new Error('AI failed to produce valid output');

  const correctionPrompt = `
    Your previous response was invalid: ${error.message}
    Original task: ${prompt}
    Respond ONLY with valid JSON matching this schema: ${schema}
  `;

  const result = await llm.invoke(correctionPrompt);
  return validateAndRetry(result, retries - 1);
}
```

### C) Constrain Output with Structured Prompting
```javascript
const systemPrompt = `
  You are a tool executor. You MUST respond ONLY with a JSON object.
  No explanation. No markdown. No extra text.
  Schema: { "tool": string, "args": object }
  Valid tools: ["searchUser", "createTicket", "updateStatus"]
`;
```

### D) Confidence Scoring + Fallback
```javascript
const response = await llm.invoke(prompt);

if (response.confidence < 0.8 || response.containsHedging) {
  return fallbackHandler(prompt); // route to human review or deterministic fallback
}
```

**One-liner for interview:** *"I treat LLM output as untrusted input — validate against a strict schema (Zod), retry with a self-correction prompt on failure, constrain the model to structured JSON output only, and fall back to a deterministic path or human review when confidence is low."*

---

## 5. Decreasing Token Usage in AI Tools

### A) Trim System Prompts Aggressively
```javascript
// ❌ Bad — verbose
const prompt = `You are a helpful assistant that helps users manage their tasks.
Please make sure to be polite and thorough in your responses...`;

// ✅ Good — dense
const prompt = `Task manager AI. Return JSON only. Tools: [searchTask, createTask, deleteTask]`;
```

### B) Summarize Conversation History
```javascript
async function getCompressedHistory(messages) {
  if (messages.length < 10) return messages;

  const toSummarize = messages.slice(0, -5);
  const recent = messages.slice(-5);

  const summary = await llm.invoke(
    `Summarize this conversation in 3 sentences: ${JSON.stringify(toSummarize)}`
  );

  return [{ role: 'system', content: `History: ${summary}` }, ...recent];
}
```

### C) Cache Repeated Tool Descriptions
```javascript
const TOOL_REGISTRY = {
  searchUser: { description: '...', params: '...' }
};

// First call: send full schema. Subsequent calls: send only tool names.
const tools = isFirstCall ? TOOL_REGISTRY : Object.keys(TOOL_REGISTRY);
```

### D) Use Prompt Caching (Anthropic)
```javascript
{
  role: "user",
  content: [
    {
      type: "text",
      text: longStaticSystemContext,
      cache_control: { type: "ephemeral" } // billed at ~10% cost on cache hit
    },
    { type: "text", text: dynamicUserQuery }
  ]
}
```

**One-liner for interview:** *"I reduce token usage by trimming verbose prompts, summarizing older conversation history instead of sending it all, deduplicating tool schema descriptions after the first call, and using prompt caching for static context that doesn't change between requests."*

---

## 6. Handling Race Conditions at API/Server Level

### A) Database-Level Locking (Pessimistic)
```javascript
await db.transaction(async (trx) => {
  const user = await trx('users')
    .where({ id: userId })
    .forUpdate() // row-level lock — no other tx can touch this row
    .first();

  if (user.balance >= amount) {
    await trx('users').where({ id: userId }).decrement('balance', amount);
    await trx('transactions').insert({ userId, amount });
  }
});
```

### B) Optimistic Locking (version field)
```javascript
const user = await User.findById(userId);

const updated = await User.updateOne(
  { _id: userId, version: user.version },
  { $inc: { balance: -amount, version: 1 } }
);

if (updated.modifiedCount === 0) {
  throw new ConflictError('Stale data — please retry');
}
```

### C) Redis Distributed Lock (across instances)
```javascript
import Redlock from 'redlock';

const redlock = new Redlock([redisClient]);

async function transferFunds(fromId, toId, amount) {
  const lock = await redlock.acquire([`lock:user:${fromId}`], 5000); // 5s TTL
  try {
    await processTransfer(fromId, toId, amount);
  } finally {
    await lock.release();
  }
}
```

### D) Idempotency Keys (prevent duplicate POSTs)
```javascript
app.post('/payment', async (req, res) => {
  const idempotencyKey = req.headers['idempotency-key'];
  if (!idempotencyKey) return res.status(400).json({ error: 'Key required' });

  const cached = await redis.get(`idem:${idempotencyKey}`);
  if (cached) return res.json(JSON.parse(cached));

  const result = await processPayment(req.body);

  await redis.setex(`idem:${idempotencyKey}`, 86400, JSON.stringify(result));
  res.json(result);
});
```

**One-liner for interview:** *"For race conditions I reach for the least heavy tool that fits: DB row locks (`SELECT FOR UPDATE`) for strict consistency, optimistic locking with a version field when contention is low, Redis distributed locks (Redlock) when the race spans multiple server instances, and idempotency keys to make retried/duplicate requests safe."*

---

## 7. Concurrent Requests to the Same API — How Responses Are Routed Correctly

**Question:** 5 people call the same API at the same time — how do you make sure each person gets *their own* correct response?

### Core Concept: Each Request Gets Its Own Closure

Node doesn't process requests in parallel threads, but **each request has its own isolated execution context via closures**. The event loop interleaves execution during I/O waits, but never mixes data — because each request's `req`, `res`, and local variables are captured in a separate closure.

```javascript
app.get('/api/data', async (req, res) => {
  const userId = req.query.userId;       // unique per invocation
  const data = await fetchFromDB(userId); // Promise bound to THIS closure
  res.json(data);                         // writes to the exact socket that requested it
});
```

`res` is a reference tied to that specific TCP socket — there's no ambiguity about where the response goes.

### Visualizing the Interleaving

```javascript
app.get('/api/user/:id', async (req, res) => {
  console.log(`Request for ${req.params.id} started`);
  const user = await db.findUser(req.params.id); // <- yields control here
  console.log(`Request for ${req.params.id} resolved`);
  res.json(user);
});

// 5 simultaneous calls: /api/user/1 ... /api/user/5

// Execution order (interleaved, not parallel):
// Request for 1 started
// Request for 2 started
// Request for 3 started
// Request for 4 started
// Request for 5 started
// Request for 3 resolved   ← DB responded first for user 3
// Request for 1 resolved
// Request for 5 resolved
// Request for 2 resolved
// Request for 4 resolved
```

Each `await` creates a Promise tied to that specific closure. When the DB responds, the engine resumes *exactly* that function instance with *exactly* that `res` — guaranteed by closures + Promises, not something to manage manually.

### Where This CAN Go Wrong — Shared/Global State

### ❌ BAD — race condition via shared state
```javascript
let currentUserId; // module-level — SHARED across all requests!

app.get('/api/data', async (req, res) => {
  currentUserId = req.query.userId;  // request A sets this
  const data = await fetchFromDB();   // request B overwrites currentUserId
                                        // while A is still awaiting!
  res.json({ userId: currentUserId, data }); // A might send B's userId!
});
```

### ✅ GOOD — keep everything in local closure
```javascript
app.get('/api/data', async (req, res) => {
  const userId = req.query.userId; // local to this invocation — safe
  const data = await fetchFromDB(userId);
  res.json({ userId, data }); // always correct, no cross-contamination
});
```

**Rule of thumb:** Never store per-request state in module-level variables, singletons, or class instance fields unless that instance is created fresh per request.

### Advanced: Tracking Request Context Across Async Boundaries

Use `AsyncLocalStorage` to trace a request across deep async layers without manually threading an ID through every function:

```javascript
import { AsyncLocalStorage } from 'async_hooks';

const requestContext = new AsyncLocalStorage();

app.use((req, res, next) => {
  const requestId = crypto.randomUUID();
  requestContext.run({ requestId, userId: req.user?.id }, () => {
    next(); // context persists through the ENTIRE async chain below
  });
});

// Deep inside a service file, no need to pass requestId as a param:
function logSomething(message) {
  const ctx = requestContext.getStore();
  console.log(`[${ctx.requestId}] ${message}`);
}
```

**One-liner for interview:** *"Node doesn't process requests in parallel, but each request gets an isolated closure over its own `req`/`res`. The event loop interleaves execution during I/O waits, but Promises resolve back into the exact closure that created them — so `res.json()` always writes to the correct socket. The only way to accidentally mix up responses is shared/global state, which is a coding mistake, not a Node limitation. For tracing context across deep async call chains, I'd use `AsyncLocalStorage` instead of manually threading a requestId through every function."*

---

## Quick Reference Summary

| Topic | Key Technique |
|---|---|
| Concurrency | `p-limit`, Worker Threads, Cluster, rate limiting |
| Backpressure | `.pipe()`, pause/drain, async generators |
| Event Loop | 6 phases; nextTick > microtask > macrotask phases |
| Hallucinations | Zod validation, self-correction retry, confidence fallback |
| Token reduction | Compress history, cache prompts, trim system prompt |
| Race conditions | DB locks, optimistic versioning, Redis + idempotency keys |
| Concurrent request routing | Per-request closures, avoid shared state, `AsyncLocalStorage` |
