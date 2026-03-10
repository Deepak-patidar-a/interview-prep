# 14 - IIFE & Module Pattern

## ❓ Interview Question
> "What is IIFE in JavaScript, what problem does it solve, and where would you use it in modern JavaScript? Also explain the Module Pattern and how it relates to IIFEs."

---

## 🙋 My Answer (What I Said)
- IIFE = Immediately Invoked Function Expression
- Executes immediately after definition
- Couldn't recall use cases or Module Pattern

---

## ✅ What I Got Right
- Correct concept — executes immediately
- Basic syntax idea correct

## ❌ What Was Missing
- Correct syntax (needs wrapping parentheses)
- What problem it solves (global scope pollution)
- Module Pattern built on top of IIFE
- Real world use cases
- Modern alternatives (ES Modules)

---

## 💡 Model Answer

### IIFE Syntax

```js
// ❌ Wrong — parser confuses with function declaration
function() {
  console.log("hello");
}();

// ✅ Correct — wrap in parens to make it expression
(function() {
  console.log("runs immediately");
})();

// ✅ Arrow function version
(() => {
  console.log("runs immediately");
})();

// ✅ With parameters
((name) => {
  console.log(`Hello ${name}`);
})("Deepak");
```

### What Problem Does IIFE Solve?

```js
// Problem — var pollutes global scope
var count = 0;
var helper = function() {}; // these are now on window object
// Any other script can access and modify these!

// IIFE Solution — everything stays private
(function() {
  var count = 0;        // private
  var helper = function() {}; // private

  // only expose what you need to global
  window.myApp = {
    increment: function() { count++; },
    getCount: function() { return count; }
  };
})();

console.log(count);  // ReferenceError — private!
console.log(myApp.getCount()); // 0 — exposed public API
```

### Module Pattern Using IIFE

```js
// Classic Module Pattern — data privacy + public API
const ShoppingCart = (function() {
  // Private state
  let items = [];
  let discount = 0;

  // Private function
  function calculateTotal() {
    const subtotal = items.reduce((sum, item) => sum + item.price, 0);
    return subtotal - (subtotal * discount / 100);
  }

  // Public API — only expose what's needed
  return {
    addItem(item) {
      items.push(item);
      console.log(`${item.name} added`);
    },
    removeItem(id) {
      items = items.filter(item => item.id !== id);
    },
    setDiscount(percent) {
      discount = percent;
    },
    getTotal() {
      return calculateTotal(); // can call private function
    },
    getItems() {
      return [...items]; // return copy, not reference
    }
  };
})();

ShoppingCart.addItem({ id: 1, name: "Shoes", price: 999 });
ShoppingCart.setDiscount(10);
console.log(ShoppingCart.getTotal()); // 899.1
console.log(ShoppingCart.items);      // undefined — private!
console.log(ShoppingCart.discount);   // undefined — private!
```

### Real World Use Cases

```js
// 1. One-time initialization / Config
const AppConfig = (() => {
  const env = process.env.NODE_ENV;
  const isDev = env === "development";

  return {
    apiUrl: isDev ? "http://localhost:3000" : "https://api.prod.com",
    debug: isDev,
    version: "1.0.0"
  };
})();

console.log(AppConfig.apiUrl); // based on environment

// 2. Avoid polluting global in script tags
(function() {
  // entire library code here
  // nothing leaks to window object
  function privateHelper() {}
  const privateData = {};

  window.MyLibrary = {
    init() { /* ... */ },
    doSomething() { /* ... */ }
  };
})();

// 3. Classic var loop fix (before let)
for (var i = 0; i < 3; i++) {
  ((j) => {
    setTimeout(() => console.log(j), 1000);
  })(i); // capture current value of i
}
// Output: 0, 1, 2 (not 3, 3, 3)
```

### Modern Alternatives — ES Modules

```js
// Modern JS — each file is its own module scope
// No need for IIFE to protect scope!

// utils.js
const privateHelper = () => {}; // private — not exported
export const publicFunction = () => {}; // public — exported

// app.js
import { publicFunction } from "./utils.js";
// privateHelper not accessible here — module scope handles it
```

---

## 🔑 Key Concepts to Remember
| Concept | Detail |
|---|---|
| IIFE | Function that runs immediately — wrap in `()` |
| Problem solved | Prevents global scope pollution |
| Module Pattern | IIFE + return public API — data privacy |
| Private | Variables inside IIFE — not accessible outside |
| Public | Properties on returned object — accessible |
| Modern alternative | ES Modules — each file has its own scope |
| When still useful | Legacy code, script tags, one-time initialization |
