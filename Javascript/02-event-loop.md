# 02 - JavaScript Event Loop

## ❓ Interview Question
> "How does the JavaScript Event Loop work? Walk me through the Call Stack, Web APIs, Callback Queue, and Microtask Queue — and tell me which one takes priority and why."

---

## 🙋 My Answer (What I Said)
- Event loop manages async and sync operations
- Direct execution code runs first in call stack
- Web APIs include setTimeout, setInterval, observers
- Callback Queue handles setTimeout/setInterval after timer completes
- Microtask Queue handles promises — runs before Callback Queue
- Priority: sync code → microtask → callback queue

---

## ✅ What I Got Right
- Sync code runs first
- setTimeout goes to Web APIs then Callback Queue
- Microtask Queue has higher priority than Callback Queue
- Overall execution order correct

## ❌ What Was Missing
- Promises are NOT Web APIs — they're JS engine features
- Event loop's actual job — checking if call stack is empty
- Call stack is LIFO structure
- `queueMicrotask()` API
- Microtasks drain completely before next callback

---

## 💡 Model Answer

### How It All Works Together

```
┌─────────────────────────────────────────────┐
│              CALL STACK (LIFO)               │
│  [ function3 ]                               │
│  [ function2 ]                               │
│  [ function1 ]  ← executes top first        │
└─────────────────────────────────────────────┘
         ↓ Web APIs (browser provided)
┌─────────────────────────────────────────────┐
│  setTimeout | fetch | DOM events | etc       │
└─────────────────────────────────────────────┘
         ↓ splits into two queues
┌───────────────────┐    ┌───────────────────┐
│  MICROTASK QUEUE  │    │  CALLBACK QUEUE   │
│  - Promises       │    │  - setTimeout     │
│  - queueMicrotask │    │  - setInterval    │
│  - MutationObserver    │  - DOM events     │
│  HIGH PRIORITY ✅ │    │  LOW PRIORITY     │
└───────────────────┘    └───────────────────┘
```

### Event Loop's Job
```js
// Event loop pseudocode
while (true) {
  // 1. Execute all synchronous code
  executeSyncCode();

  // 2. Drain microtask queue completely
  while (microtaskQueue.length) {
    microtaskQueue.shift()();
  }

  // 3. Take ONE task from callback queue
  if (callbackQueue.length) {
    callbackQueue.shift()();
  }
}
```

### Classic Output Question
```js
console.log("1");                          // sync

setTimeout(() => console.log("2"), 0);    // callback queue

Promise.resolve().then(() => console.log("3")); // microtask queue

console.log("4");                          // sync

// Output: 1, 4, 3, 2
// Why?
// 1 → sync, runs immediately
// 4 → sync, runs immediately
// 3 → microtask, runs before callback queue
// 2 → callback queue, runs last
```

### Advanced Example
```js
console.log("start");

setTimeout(() => console.log("timeout 1"), 0);

Promise.resolve()
  .then(() => {
    console.log("promise 1");
    return Promise.resolve();
  })
  .then(() => console.log("promise 2"));

setTimeout(() => console.log("timeout 2"), 0);

console.log("end");

// Output: start, end, promise 1, promise 2, timeout 1, timeout 2
```

### Web APIs vs JS Engine Features
```js
// Web APIs (browser provided) — NOT part of JS engine
setTimeout()        // Web API
setInterval()       // Web API
fetch()             // Web API
addEventListener()  // Web API
IntersectionObserver // Web API

// JS Engine features
Promise             // JS engine — goes to MICROTASK queue
queueMicrotask()    // JS engine — explicitly queue a microtask
```

---

## 🔑 Key Concepts to Remember
| Concept | Detail |
|---|---|
| Call Stack | LIFO — executes sync code |
| Web APIs | Browser provided — setTimeout, fetch, DOM events |
| Microtask Queue | Promises, MutationObserver — HIGH priority |
| Callback Queue | setTimeout, setInterval — LOW priority |
| Priority Order | Sync → Microtasks (all) → One Callback → repeat |
| Microtask drain | ALL microtasks run before next callback |
