# 03 - Closures in JavaScript

## ❓ Interview Question
> "What is Closure in JavaScript? Can you explain it with a real-world use case — not just a textbook definition — and also tell me one potential problem closures can cause?"

---

## 🙋 My Answer (What I Said)
- Closures are functions that access outer lexically scoped variables
- Inner function accessing outer function's variable
- Use cases: data privacy, encapsulation, OOP, state management
- Potential problem: memory issues with large objects in nested functions

---

## ✅ What I Got Right
- Definition is correct
- Lexical scope explanation correct
- Data privacy use case correct
- Memory leak as potential problem correct

## ❌ What Was Missing
- Key phrase: "after outer function has finished executing"
- Classic `var` in loop closure problem
- Memoization use case
- How to fix memory leak (set to null)

---

## 💡 Model Answer

### Definition
> A closure is a function that **retains access to its outer scope even after the outer function has finished executing**

```js
function outer() {
  const name = "Deepak"; // outer variable

  function inner() {
    console.log(name); // accessing outer variable — CLOSURE
  }

  return inner;
}

const fn = outer(); // outer() has finished executing
fn(); // "Deepak" — inner still has access to name!
```

### Real World Use Case 1 — Data Privacy / Counter

```js
function createCounter() {
  let count = 0; // private — not accessible outside

  return {
    increment: () => ++count,
    decrement: () => --count,
    getCount: () => count
  };
}

const counter = createCounter();
counter.increment();
counter.increment();
console.log(counter.getCount()); // 2
console.log(counter.count);      // undefined — private!
```

### Real World Use Case 2 — Memoization

```js
function memoize(fn) {
  const cache = {}; // closure over cache

  return function(n) {
    if (cache[n] !== undefined) {
      console.log("from cache");
      return cache[n];
    }
    cache[n] = fn(n);
    return cache[n];
  };
}

const expensiveCalc = memoize((n) => n * n);
expensiveCalc(5); // calculates: 25
expensiveCalc(5); // from cache: 25
```

### Real World Use Case 3 — Partial Application

```js
function multiply(factor) {
  return (number) => number * factor; // closure over factor
}

const double = multiply(2);
const triple = multiply(3);

double(5);  // 10
triple(5);  // 15
```

### Classic Problem — var in Loop

```js
// ❌ Problem — all closures share same 'i' reference
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 1000);
}
// Output: 3, 3, 3 — because var is function scoped
// by the time setTimeout fires, loop is done and i = 3

// ✅ Fix 1 — use let (block scoped, new i each iteration)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 1000);
}
// Output: 0, 1, 2

// ✅ Fix 2 — IIFE to capture current value
for (var i = 0; i < 3; i++) {
  ((j) => {
    setTimeout(() => console.log(j), 1000);
  })(i);
}
// Output: 0, 1, 2
```

### Potential Problem — Memory Leak

```js
// ❌ Large object held in memory by closure
function outer() {
  const largeData = new Array(1000000).fill("data");

  return function inner() {
    console.log("inner called");
    // largeData never released — closure holds reference!
  };
}

const fn = outer();
// largeData stays in memory as long as fn exists

// ✅ Fix — release the closure reference when done
fn = null; // now largeData can be garbage collected
```

---

## 🔑 Key Concepts to Remember
| Concept | Detail |
|---|---|
| Definition | Function retains access to outer scope after outer function finishes |
| Lexical scope | Scope determined by where function is written, not called |
| Data privacy | Variables in closure are private — not accessible outside |
| var loop bug | All iterations share same var reference — use let or IIFE |
| Memory leak | Closures prevent GC — set to null when done |
| Memoization | Cache results using closure over cache object |
