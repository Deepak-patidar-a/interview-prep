# 🔁 Event Loop — Interview Revision Notes

> Source: Namaste Node.js (ep-9) + Interview Prep  
> Topic: Event Loop, phases, tick, process.nextTick, setImmediate, setTimeout

---

## 🧠 What is the Event Loop?

The Event Loop is the **heart of Node.js async behavior**. It constantly checks:
- Is the **Call Stack empty**?
- If yes → pick next callback from **Callback Queue** → push to Call Stack → execute

> From your notes: *"The job of the event loop is to keep checking the Call Stack and Callback Queue."*

> **Tick** = One full cycle of the event loop is known as a **tick**.

---

## 🗺️ Big Picture — How it all fits

```
┌─────────────────────────────────────────────────────┐
│                      Node.js                        │
│                                                     │
│  ┌─────────────┐          ┌─────────────────────┐   │
│  │ V8 JS Engine│          │        libuv        │   │
│  │             │          │  ┌──────────────┐   │   │
│  │ Memory Heap │          │  │  EVENT LOOP  │   │   │
│  │ Call Stack  │◄─────────│  └──────────────┘   │   │
│  │ GEC         │          │  ┌──────────────┐   │   │
│  │ Garbage Col.│          │  │ THREAD POOL  │   │   │
│  └─────────────┘          │  │ (4 threads)  │   │   │
│                           │  └──────────────┘   │   │
│                           │  Callback Queues     │   │
│                           └─────────────────────┘   │
└─────────────────────────────────────────────────────┘
                    ↕ (offloads to)
┌─────────────────────────────────────────────────────┐
│               OS (Asynchronous I/O)                 │
│        File System   WWW   Timer   DB               │
└─────────────────────────────────────────────────────┘
```

---

## 🔄 Event Loop Phases (in order)

The Event Loop runs in **phases**. Each phase has its own callback queue.

```
   ┌──────────────────────────┐
   │         Timers           │  → setTimeout, setInterval callbacks
   └──────────────┬───────────┘
                  │
   ┌──────────────▼───────────┐
   │     Pending Callbacks    │  → I/O callbacks from previous iteration
   └──────────────┬───────────┘
                  │
   ┌──────────────▼───────────┐
   │      Idle / Prepare      │  → internal use only
   └──────────────┬───────────┘
                  │
   ┌──────────────▼───────────┐
   │           Poll           │  → new I/O events: fs, http.get, crypto
   └──────────────┬───────────┘
                  │
   ┌──────────────▼───────────┐
   │          Check           │  → setImmediate callbacks run here
   └──────────────┬───────────┘
                  │
   ┌──────────────▼───────────┐
   │      Close Callbacks     │  → socket.on('close'), etc.
   └──────────────┬───────────┘
                  │
                  └──── loops back to top ↑
```

> **Important:** Between each phase, Node.js checks the **microtask queue** (process.nextTick + Promises) first before moving to the next phase.

---

## ⚡ Callback Queue Priority Order

From your notes — execution priority (highest to lowest):

```
1. process.nextTick(cb)       ← runs after current op, before I/O
2. Promise.resolve(cb)         ← microtask queue
3. setTimeout(cb, 0)           ← timer phase
4. setImmediate(cb)            ← check phase
5. fs.readFile(path, cb)       ← poll/I/O phase
6. https.get(url, cb)          ← poll/I/O phase
```

### Live Example:
```javascript
console.log("start");

process.nextTick(() => console.log("1 - nextTick"));
Promise.resolve().then(() => console.log("2 - Promise"));
setTimeout(() => console.log("3 - setTimeout"), 0);
setImmediate(() => console.log("4 - setImmediate"));

console.log("end");

// Output:
// start
// end
// 1 - nextTick
// 2 - Promise
// 3 - setTimeout
// 4 - setImmediate
```

---

## 🔁 Step-by-Step: How Async Code Actually Runs

Using example from your notes:

```javascript
http.get("https://api.fbi.com", (res) => {
  console.log(res?.secret);
});

fs.readFile("./gossip.txt", (data) => {
  console.log("File Data", data);
});

setTimeout(() => {
  console.log("wait here for 5 min");
}, 5000);
```

### Execution Flow:
```
Step 1: GEC created → pushed to Call Stack
Step 2: http.get seen → offloaded to libuv → V8 moves to next line
Step 3: fs.readFile seen → offloaded to libuv → V8 moves to next line
Step 4: setTimeout seen → offloaded to libuv → V8 moves to next line
Step 5: Call Stack becomes EMPTY
Step 6: libuv completes tasks → puts callbacks in Callback Queue (FIFO order)
Step 7: Event Loop sees empty Call Stack → picks callbacks → executes
```

> From notes: *"First callback will be executed, order FIFO according to callback queue."*

---

## 🔍 process.nextTick() vs setImmediate() vs setTimeout()

| Method | Queue | When runs |
|---|---|---|
| `process.nextTick(cb)` | Microtask (nextTick) | After current op, before I/O |
| `Promise.resolve().then(cb)` | Microtask (Promise) | After nextTick |
| `setTimeout(cb, 0)` | Timer Phase | After microtasks, timer phase |
| `setImmediate(cb)` | Check Phase | After I/O callbacks |
| `fs.readFile(cb)` | Poll Phase | When I/O is complete |

### Key Difference — setTimeout(0) vs setImmediate:
```javascript
// Inside I/O callback → setImmediate ALWAYS runs first
fs.readFile('./file.txt', () => {
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate'));
});
// Output: immediate → timeout (guaranteed)

// Outside I/O → order is NOT guaranteed (depends on system)
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
// Output: either order possible
```

---

## 🧱 What Can Block the Event Loop?

CPU-intensive **synchronous** operations block the event loop — all other requests wait!

### Common blockers:
- `JSON.parse()` on very large data
- Complex math calculations in a loop
- `fs.readFileSync()` — synchronous file read
- Crypto operations without async

### Solutions:
| Problem | Solution |
|---|---|
| CPU-intensive tasks | Use **Worker Threads** |
| Multiple requests | Use **Cluster** module (multiple processes) |
| Blocking file reads | Always use **async** versions (`fs.readFile`) |
| Heavy computation | Offload to **child_process** |

```javascript
// ❌ Bad — blocks event loop
const data = fs.readFileSync('./bigfile.txt');

// ✅ Good — non-blocking
fs.readFile('./bigfile.txt', (err, data) => {
  console.log(data);
});

// ✅ Better — with async/await
const data = await fs.promises.readFile('./bigfile.txt');
```

---

## 🧵 Thread Pool Deep Dive

libuv's Thread Pool handles tasks that can't be done async at OS level:

| Uses Thread Pool | Doesn't Use Thread Pool |
|---|---|
| `fs` (file system) | `http`, `https` (uses OS async) |
| `crypto` | TCP/UDP sockets |
| `dns.lookup()` | `dns.resolve()` |
| `zlib` compression | — |

```javascript
// Increase thread pool size (default 4, max 1024)
process.env.UV_THREADPOOL_SIZE = 8;
```

---

## 🔄 Event Loop in Action — Close Callbacks Example

From your notes:
```javascript
// close event runs in Close Callbacks phase
socket.on("close", () => {
  console.log("socket closed");
});
```

---

## ❓ Interview Questions — Event Loop

| # | Question | Difficulty |
|---|---|---|
| 1 | Explain the Node.js Event Loop in detail | 🔴 Most Asked |
| 2 | What is a "tick" in Node.js? | 🟢 Easy |
| 3 | What is the difference between process.nextTick() and setImmediate()? | 🔴 Most Asked |
| 4 | What is the execution order of setTimeout(0), setImmediate, nextTick, Promise? | 🔴 Most Asked |
| 5 | What are the phases of the Event Loop? | 🟡 Medium |
| 6 | What happens when Call Stack is empty? | 🟢 Easy |
| 7 | What can block the Event Loop and how do you prevent it? | 🔴 Hard |
| 8 | What is the Thread Pool? When does Node.js use it? | 🟡 Medium |
| 9 | What is the difference between microtask and macrotask queue? | 🔴 Hard |
| 10 | Walk me through how an http.get() call executes asynchronously | 🔴 Most Asked |

---

## 💡 Key Interview Answers (memorize these)

### Answer 1 — Event Loop explanation:
> *"The Event Loop constantly checks if the Call Stack is empty. If yes, it picks the next callback from the Callback Queue and pushes it to the Call Stack. It runs in phases: Timers → Pending I/O → Poll → Check → Close. Between each phase, it first processes the microtask queue (nextTick + Promises)."*

### Answer 2 — Priority order:
> *"process.nextTick runs first, then Promise microtasks, then setTimeout, then setImmediate. But inside an I/O callback, setImmediate always runs before setTimeout."*

### Answer 3 — What is a tick:
> *"One full cycle of the event loop is called a tick. process.nextTick() means 'run this at the end of the current tick, before the next phase.'"*

---

*Previous: [01_NodeJS_Core.md](./01_NodeJS_Core.md) | Next: [03_ExpressJS.md](./03_ExpressJS.md)*
