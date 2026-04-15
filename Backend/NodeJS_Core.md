# 📦 Node.js Core — Interview Revision Notes

> Source: Namaste Node.js (ep-8, ep-9) + Interview Prep  
> Topic: Node.js fundamentals, V8 Engine, libuv, async/sync execution

---

## 🧠 What is Node.js?

Node.js is a **runtime environment** built on Chrome's **V8 JavaScript Engine** that lets you run JavaScript on the server side.

- JavaScript is **synchronous + single-threaded** by nature
- Node.js becomes **asynchronous** because of `libuv` — which offloads I/O operations so the main thread is never blocked
- Node.js = V8 Engine + libuv + other libs (fs, http, crypto, etc.)

```
Node.js Architecture:
┌─────────────────────────────────────────┐
│                Node.js                  │
│  ┌──────────────┐   ┌────────────────┐  │
│  │ V8 JS Engine │   │     libuv      │  │
│  │  (Google)    │   │  (Thread Pool  │  │
│  │              │   │  + Event Loop) │  │
│  └──────────────┘   └────────────────┘  │
│       fs    http    crypto   ...        │
└─────────────────────────────────────────┘
```

---

## ⚙️ V8 JS Engine — What happens when code runs?

### Step 1 — Parsing
Code enters the parser → **Lexical Analysis** breaks code into **Tokens** → **Syntax Analysis** converts tokens into an **AST (Abstract Syntax Tree)**

### Step 2 — Compilation (JIT)
V8 uses **Ignition** (interpreter) + **TurboFan** (optimizing compiler) to convert AST into machine code at runtime (Just In Time compilation)

```
Code → PARSING → Lexical Analysis (Tokenization) → Tokens
                                                        ↓
                              Syntax Analysis (Parsing) → AST → Machine Code
```

### V8 Engine Internals
| Component | Role |
|---|---|
| Memory Heap | Stores variables and objects |
| Call Stack | Executes functions one by one |
| Garbage Collector | Frees unused memory automatically |
| JIT Compiler | Converts JS to machine code at runtime |

---

## 🔄 How Synchronous Code Executes

When you run code, a **GEC (Global Execution Context)** is created and pushed onto the Call Stack.

```javascript
var a = 407429;
var b = 172410;

function multiplyFn(x, y) {
  const result = a * b;  // stored in memory heap
  return result;
}

var c = multiplyFn(a, b); // pushed to call stack → executed → popped
console.log(c);
```

**Flow:**
1. GEC created → pushed to Call Stack
2. Variables `a`, `b` allocated in Memory Heap
3. `multiplyFn` called → new execution context pushed to stack
4. Calculation done → result returned → function pops off stack
5. When all code runs → Call Stack becomes **empty**

---

## ⚡ libuv — The Superpower of Node.js

**libuv** is a multi-platform C library that gives Node.js async I/O capability.

> "libuv is a multiplatform C library that provides support for asynchronous I/O-based event loops." — Your notes

### What libuv manages:
- **Thread Pool** (default 4 threads, max 1024) — for fs, crypto, DNS, zlib
- **Event Loop** — coordinates everything
- **Callback Queues** — stores completed async callbacks

### When does libuv kick in?
Whenever V8 sees: API call, File load, setTimeout, setInterval, https.get — it **offloads** to libuv. libuv manages it and Node.js continues executing the next line without waiting.

```
V8 Engine → sees async call → offloads to libuv
                                      ↓
                              libuv → OS / Thread Pool
                                      ↓
                              Task done → Callback Queue
                                      ↓
                              Event Loop → Call Stack
```

---

## 🔀 How Asynchronous Code Executes

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

**Flow:**
1. GEC created, pushed to Call Stack
2. `http.get` encountered → **offloaded to libuv** → V8 moves to next line
3. `fs.readFile` encountered → **offloaded to libuv** → V8 moves to next line
4. `setTimeout` encountered → **offloaded to libuv** → V8 moves to next line
5. Call Stack becomes empty
6. libuv finishes tasks → callbacks go into **Callback Queue**
7. Event Loop sees Call Stack is empty → picks callbacks → executes them

---

## 📋 Key Concepts Quick Reference

### require() vs import
| Feature | `require()` (CJS) | `import` (ESM) |
|---|---|---|
| Default in Node.js | ✅ Yes | ❌ Needs config |
| Loading | Synchronous | Asynchronous |
| When parsed | Runtime | Parse time (static) |
| Tree-shaking | ❌ No | ✅ Yes |
| File extension | `.js` | `.mjs` or `"type":"module"` |

### REPL
**R**ead → **E**valuate → **P**rint → **L**oop  
Run by typing `node` in terminal. Like browser console but for Node.js.

### package.json
Metadata file for a Node.js project — holds name, version, scripts, dependencies.

| Field | Meaning |
|---|---|
| `dependencies` | Needed in production (express, mongoose) |
| `devDependencies` | Only for development (nodemon, jest, eslint) |
| `scripts` | Shortcut commands (`npm start`, `npm test`) |

---

## 🔁 Callbacks vs Promises vs Async/Await

### Callbacks (old way — avoid)
```javascript
fs.readFile('./file.txt', (err, data) => {
  if (err) throw err;
  console.log(data); // callback hell if nested
});
```

### Promises (cleaner)
```javascript
fetch('https://api.example.com/data')
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.error(err));
```

### Async/Await (modern standard ✅)
```javascript
async function getData() {
  try {
    const res = await fetch('https://api.example.com/data');
    const data = await res.json();
    return data;
  } catch (err) {
    console.error(err);
  }
}
```

---

## 📡 Streams in Node.js

Streams handle data **piece by piece (chunks)** instead of loading everything into memory.

| Type | Description |
|---|---|
| Readable | Read data (fs.createReadStream) |
| Writable | Write data (fs.createWriteStream) |
| Duplex | Read + Write (TCP sockets) |
| Transform | Modify data as it passes through (zlib) |

```javascript
const readable = fs.createReadStream('./bigfile.txt');
readable.pipe(res); // pipe readable stream directly to HTTP response
```

---

## ❓ Interview Questions — Node.js Core

| # | Question | Difficulty |
|---|---|---|
| 1 | What is Node.js and why is it non-blocking? | 🟢 Easy |
| 2 | What is libuv and what is its role in Node.js? | 🟡 Medium |
| 3 | What happens inside V8 engine when code runs? | 🔴 Hard |
| 4 | What is GEC and how does the Call Stack work? | 🟡 Medium |
| 5 | Difference between require() and import? | 🟢 Easy |
| 6 | What is REPL in Node.js? | 🟢 Easy |
| 7 | Difference between dependencies and devDependencies? | 🟢 Easy |
| 8 | What are Streams? Name the 4 types. | 🟡 Medium |
| 9 | Callbacks vs Promises vs Async/Await — which to use and why? | 🟡 Medium |
| 10 | How does Node.js handle multiple requests with one thread? | 🔴 Hard |

---

## 💡 Key Interview Answer (memorize this)

> **"JavaScript is synchronous and single-threaded, but Node.js is asynchronous because libuv offloads I/O operations (file reads, HTTP calls, timers) to the OS and Thread Pool. The main thread never waits — it keeps executing. Once libuv is done, it pushes the callback into the queue, and the Event Loop picks it up when the Call Stack is empty."**

---

*Next: [02_Event_Loop.md](./02_Event_Loop.md)*
