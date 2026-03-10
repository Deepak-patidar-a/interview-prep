# 10 - Higher Order Functions: map, filter, reduce

## ❓ Interview Question
> "What are higher order functions in JavaScript? Can you explain map, filter, reduce with examples — and also write a reduce that does what map does and a reduce that does what filter does?"

---

## 🙋 My Answer (What I Said)
- HOF takes one or more functions as arguments or returns a function
- map — iterate, perform logic, returns new array
- filter — iterate, filter by condition, returns new array
- reduce — iterate, returns new array or single value, use for sum
- Couldn't implement map/filter using reduce

---

## ✅ What I Got Right
- HOF definition correct
- map/filter/reduce explanations correct
- Basic reduce signature correct
- Sum use case for reduce correct

## ❌ What Was Missing
- map and filter implemented using reduce
- Real world reduce examples (groupBy, count, flatten)
- map vs forEach difference
- Always provide initial value to reduce

---

## 💡 Model Answer

### Higher Order Functions

```js
// HOF — takes function as argument
[1, 2, 3].map(x => x * 2);

// HOF — returns a function
const multiply = (factor) => (number) => number * factor;
const double = multiply(2);
const triple = multiply(3);
double(5); // 10
triple(5); // 15
```

### map

```js
// Transforms each element, returns NEW array of SAME length
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(n => n * 2);        // [2, 4, 6, 8, 10]
const strings = numbers.map(n => `item-${n}`);  // ["item-1", ...]
const objects = numbers.map(n => ({ id: n }));  // [{id:1}, ...]

// Real world
const users = [{ name: "Deepak", age: 25 }, { name: "John", age: 30 }];
const names = users.map(user => user.name); // ["Deepak", "John"]
```

### filter

```js
// Returns NEW array with elements that pass condition
const numbers = [1, 2, 3, 4, 5, 6];
const evens = numbers.filter(n => n % 2 === 0);  // [2, 4, 6]
const odds  = numbers.filter(n => n % 2 !== 0);  // [1, 3, 5]

// Real world
const users = [
  { name: "Deepak", active: true },
  { name: "John", active: false },
  { name: "Alice", active: true }
];
const activeUsers = users.filter(u => u.active);
// [{ name: "Deepak" }, { name: "Alice" }]
```

### reduce

```js
// Reduces array to single value (or new array/object)
// ALWAYS provide initial value!
const numbers = [1, 2, 3, 4, 5];

// Sum
const sum = numbers.reduce((acc, curr) => acc + curr, 0); // 15

// Product
const product = numbers.reduce((acc, curr) => acc * curr, 1); // 120

// Max value
const max = numbers.reduce((acc, curr) => Math.max(acc, curr), -Infinity); // 5
```

### map using reduce ✅

```js
const myMap = (arr, cb) => {
  return arr.reduce((acc, curr) => {
    acc.push(cb(curr)); // transform and push
    return acc;
  }, []); // start with empty array
};

myMap([1, 2, 3], x => x * 2); // [2, 4, 6]
```

### filter using reduce ✅

```js
const myFilter = (arr, cb) => {
  return arr.reduce((acc, curr) => {
    if (cb(curr)) acc.push(curr); // only push if condition met
    return acc;
  }, []); // start with empty array
};

myFilter([1, 2, 3, 4, 5], x => x % 2 === 0); // [2, 4]
```

### Advanced reduce Patterns

```js
const orders = [
  { product: "shoes", category: "footwear", price: 999 },
  { product: "shirt", category: "clothing", price: 499 },
  { product: "boots", category: "footwear", price: 1499 },
  { product: "pants", category: "clothing", price: 799 },
];

// Group by category
const grouped = orders.reduce((acc, order) => {
  const key = order.category;
  if (!acc[key]) acc[key] = [];
  acc[key].push(order);
  return acc;
}, {});
// { footwear: [...], clothing: [...] }

// Count occurrences
const words = ["apple", "banana", "apple", "cherry", "banana", "apple"];
const count = words.reduce((acc, word) => {
  acc[word] = (acc[word] || 0) + 1;
  return acc;
}, {});
// { apple: 3, banana: 2, cherry: 1 }

// Flatten nested array
const nested = [[1, 2], [3, 4], [5, 6]];
const flat = nested.reduce((acc, curr) => [...acc, ...curr], []);
// [1, 2, 3, 4, 5, 6]

// Sum of specific field
const total = orders.reduce((sum, order) => sum + order.price, 0); // 3796
```

### map vs forEach

```js
// map — returns NEW array, use when you need result
const doubled = [1, 2, 3].map(x => x * 2); // [2, 4, 6]

// forEach — returns undefined, use for side effects only
[1, 2, 3].forEach(x => console.log(x)); // undefined returned

// ❌ Don't do this — pointless
const result = [1, 2, 3].forEach(x => x * 2); // undefined!

// ✅ Chain map + filter + reduce
const result2 = [1, 2, 3, 4, 5]
  .filter(n => n % 2 === 0)    // [2, 4]
  .map(n => n * 3)              // [6, 12]
  .reduce((sum, n) => sum + n, 0); // 18
```

---

## 🔑 Key Concepts to Remember
| Method | Returns | Use when |
|---|---|---|
| `map` | New array (same length) | Transform each element |
| `filter` | New array (shorter/same) | Keep elements matching condition |
| `reduce` | Anything | Accumulate to single value/object |
| `forEach` | `undefined` | Side effects only |

### reduce Tip
Always provide initial value as second argument:
```js
[].reduce((acc, curr) => acc + curr)      // ❌ TypeError on empty array
[].reduce((acc, curr) => acc + curr, 0)   // ✅ returns 0
```
