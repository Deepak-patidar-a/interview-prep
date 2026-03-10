# 15 - Generators & Iterators

## ❓ Interview Question
> "What are generators and iterators in JavaScript? How does the function* and yield keyword work, and can you give a real world use case where a generator would be more useful than a regular function?"

---

## 💡 Full Answer

### What is an Iterator?

```js
// An iterator is any object with a next() method
// next() returns { value, done }

// Arrays are iterable — they have a built-in iterator
const arr = [1, 2, 3];
const iterator = arr[Symbol.iterator]();

iterator.next(); // { value: 1, done: false }
iterator.next(); // { value: 2, done: false }
iterator.next(); // { value: 3, done: false }
iterator.next(); // { value: undefined, done: true }

// for...of uses iterator under the hood
for (const item of arr) {
  console.log(item); // 1, 2, 3
}
```

### What is a Generator?

```js
// function* creates a generator function
// yield pauses execution and returns a value
// .next() resumes execution until next yield

function* myGenerator() {
  console.log("start");
  yield 1;              // pause, return 1
  console.log("middle");
  yield 2;              // pause, return 2
  console.log("end");
  yield 3;              // pause, return 3
}

const gen = myGenerator();
gen.next(); // logs "start",  returns { value: 1, done: false }
gen.next(); // logs "middle", returns { value: 2, done: false }
gen.next(); // logs "end",    returns { value: 3, done: false }
gen.next(); //                returns { value: undefined, done: true }
```

### Real World Use Case 1 — Infinite Sequence

```js
// Regular function — can't do infinite without crashing
// Generator — computes lazily, only when asked

function* infiniteId() {
  let id = 1;
  while (true) {
    yield id++;
  }
}

const idGenerator = infiniteId();
idGenerator.next().value; // 1
idGenerator.next().value; // 2
idGenerator.next().value; // 3
// Never crashes — only computes when .next() is called

// Real world — unique ID generator
function* generateOrderId() {
  let n = 1000;
  while (true) {
    yield `ORD-${n++}`;
  }
}
const orderId = generateOrderId();
orderId.next().value; // "ORD-1000"
orderId.next().value; // "ORD-1001"
```

### Real World Use Case 2 — Paginated Data

```js
function* paginator(items, pageSize) {
  for (let i = 0; i < items.length; i += pageSize) {
    yield items.slice(i, i + pageSize);
  }
}

const data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const pages = paginator(data, 3);

pages.next().value; // [1, 2, 3]
pages.next().value; // [4, 5, 6]
pages.next().value; // [7, 8, 9]
pages.next().value; // [10]
pages.next().done;  // true
```

### Real World Use Case 3 — Lazy Range

```js
// Regular function loads ALL values into memory
function range(start, end) {
  const result = [];
  for (let i = start; i <= end; i++) result.push(i); // memory!
  return result;
}
range(1, 1000000); // creates array of 1M items in memory!

// Generator — lazy, only computes what you need
function* lazyRange(start, end) {
  for (let i = start; i <= end; i++) {
    yield i; // compute one at a time
  }
}

for (const num of lazyRange(1, 1000000)) {
  if (num > 5) break; // only computed 6 values!
  console.log(num);
}
```

### Passing Values into Generator

```js
function* calculator() {
  const x = yield "Enter first number:";
  const y = yield "Enter second number:";
  return x + y;
}

const calc = calculator();
calc.next();        // { value: "Enter first number:", done: false }
calc.next(10);      // { value: "Enter second number:", done: false } — passes 10 as x
calc.next(20);      // { value: 30, done: true } — passes 20 as y, returns 10+20
```

### async/await Under the Hood

```js
// async/await is built on generators + promises!

// This async/await code:
async function fetchUser() {
  const user = await getUser();
  const posts = await getPosts(user.id);
  return posts;
}

// Is conceptually similar to this generator:
function* fetchUser() {
  const user = yield getUser();       // pause until promise resolves
  const posts = yield getPosts(user.id); // pause until promise resolves
  return posts;
}
// (with a "runner" function handling the promises)
```

---

## 🔑 Key Concepts to Remember
| Concept | Detail |
|---|---|
| `function*` | Declares a generator function |
| `yield` | Pauses execution, returns value to caller |
| `.next()` | Resumes execution until next yield |
| `{ value, done }` | What .next() returns |
| `done: true` | Generator has finished |
| Iterator | Object with .next() method |
| Lazy evaluation | Generators compute only when asked |
| Use cases | Infinite sequences, pagination, lazy ranges |
| async/await | Built on generators + promises under the hood |
