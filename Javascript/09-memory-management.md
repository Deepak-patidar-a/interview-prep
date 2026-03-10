# 09 - Memory Management & Garbage Collection

## ❓ Interview Question
> "What is memory management in JavaScript? Explain how the Garbage Collector works, what a memory leak is, and give me at least 3 real-world examples of how memory leaks happen in JavaScript and how to fix them."

---

## 🙋 My Answer (What I Said)
- GC automatically frees memory after execution
- Memory leaks happen when JS can't garbage collect
- setTimeout, setInterval, eventListeners cause leaks without cleanup
- Fix: clearTimeout, clearInterval, removeEventListener in cleanup

---

## ✅ What I Got Right
- GC automatically frees memory — correct
- Cleanup pattern with clearTimeout/clearInterval — correct
- EventListeners causing leaks — correct
- return () => cleanup() pattern — correct

## ❌ What Was Missing
- Mark and Sweep algorithm
- GC removes objects when no references remain (not just after execution)
- Closures causing memory leaks
- Detached DOM nodes
- Global variables as leaks
- WeakMap/WeakRef

---

## 💡 Model Answer

### How Garbage Collection Works — Mark & Sweep

```js
// GC uses "Mark and Sweep" algorithm
// 1. MARK — start from roots (window/global), mark all reachable objects
// 2. SWEEP — remove everything NOT marked (unreachable)

let obj = { name: "Deepak" }; // reachable — referenced by obj
let ref = obj;                 // two references now

obj = null; // removed one reference — still reachable via ref
ref = null; // removed last reference — NOW eligible for GC
```

### Memory Leak Examples & Fixes

#### 1. Event Listeners Not Removed

```js
// ❌ Memory leak — listener added but never removed
function setupButton() {
  const button = document.getElementById("btn");
  button.addEventListener("click", handleClick);
  // handleClick stays in memory even after component unmounts!
}

// ✅ Fix — remove listener when done
function setupButton() {
  const button = document.getElementById("btn");
  button.addEventListener("click", handleClick);

  return () => {
    button.removeEventListener("click", handleClick); // cleanup
  };
}

// In React
useEffect(() => {
  window.addEventListener("resize", handleResize);
  return () => window.removeEventListener("resize", handleResize);
}, []);
```

#### 2. setInterval Not Cleared

```js
// ❌ Memory leak — interval runs forever
function startPolling() {
  setInterval(() => {
    fetchLatestData();
  }, 5000);
}

// ✅ Fix — store and clear
function startPolling() {
  const timer = setInterval(() => {
    fetchLatestData();
  }, 5000);

  return () => clearInterval(timer); // cleanup
}

// In React
useEffect(() => {
  const timer = setInterval(fetchData, 5000);
  return () => clearInterval(timer); // cleanup on unmount
}, []);
```

#### 3. Closures Holding Large Objects

```js
// ❌ Memory leak — largeData never released
function processData() {
  const largeData = new Array(1000000).fill("data");

  return function() {
    console.log("done"); // closure holds reference to largeData!
  };
}

const fn = processData();
// largeData stays in memory as long as fn exists

// ✅ Fix — release reference when done
fn = null; // largeData now eligible for GC
```

#### 4. Detached DOM Nodes

```js
// ❌ Memory leak — element removed from DOM but still referenced in JS
let element = document.getElementById("modal");
document.body.removeChild(element);
// element still referenced — DOM node stays in memory!

// ✅ Fix — null the reference after removing
document.body.removeChild(element);
element = null; // now eligible for GC
```

#### 5. Accidental Global Variables

```js
// ❌ Memory leak — accidental global
function calculate() {
  result = 1000; // no let/const/var — becomes window.result!
  // lives forever on global object
}

// ✅ Fix — always declare with let/const/var
function calculate() {
  const result = 1000; // scoped to function
}

// ✅ Use strict mode to catch this
"use strict";
function calculate() {
  result = 1000; // ReferenceError — caught!
}
```

### WeakMap & WeakRef — Advanced

```js
// WeakMap — holds WEAK references
// Keys can be garbage collected even if in WeakMap
const cache = new WeakMap();

function process(obj) {
  if (cache.has(obj)) return cache.get(obj);
  const result = expensiveCalculation(obj);
  cache.set(obj, result);
  return result;
}

// When obj is no longer referenced elsewhere,
// GC can collect it AND remove it from WeakMap automatically!

// Regular Map would prevent GC — memory leak!
const regularCache = new Map();
// obj in regularCache = never garbage collected
```

### Chrome DevTools — Finding Memory Leaks

```
1. Open DevTools → Memory tab
2. Take heap snapshot before action
3. Perform the action (e.g., open/close modal)
4. Take another heap snapshot
5. Compare — look for objects that should have been released
```

---

## 🔑 Key Concepts to Remember
| Concept | Detail |
|---|---|
| Mark & Sweep | GC marks reachable objects, sweeps unreachable |
| Reachability | Object is collected when NO references point to it |
| Event listeners | Always removeEventListener in cleanup |
| Timers | Always clearInterval/clearTimeout in cleanup |
| Closures | Large objects in closures — set to null when done |
| Detached DOM | null JS reference after removing from DOM |
| Global vars | Always use let/const/var, use strict mode |
| WeakMap | Weak references — allows GC to collect keys |
