# 08 - async/await in JavaScript

## ❓ Interview Question
> "Can you explain async/await in JavaScript? How does it work under the hood, how do you handle errors with it, and what is the difference between Promise.all vs sequential await — and which is more performant?"

---

## 🙋 My Answer (What I Said)
- async/await for async operations, came with ES6+
- Returns a Promise under the hood
- Error handling with try/catch
- await must be inside async function
- Promise.all executes both promises together — more performant
- Sequential await executes one after another — slower

---

## ✅ What I Got Right
- Returns Promise under the hood
- try/catch for error handling
- await inside async function rule
- Promise.all is more performant
- Sequential await is slower

## ❌ What Was Missing
- Deeper explanation of how it pauses execution
- Error handling differences between parallel vs sequential
- When sequential is CORRECT (dependent calls)
- async function always returns Promise even for plain values

---

## 💡 Model Answer

### How async/await Works Under the Hood

```js
// async/await is syntactic sugar over Promises
// When JS hits await, it PAUSES that function and returns
// control to event loop, resuming when promise settles

// This:
async function getUser() {
  const user = await fetchUser();
  return user;
}

// Is essentially this:
function getUser() {
  return fetchUser().then(user => user);
}
```

### Basic Usage

```js
async function fetchData() {
  try {
    setLoading(true);
    const response = await fetch("/api/data");

    if (!response.ok) {
      throw new Error(`HTTP Error: ${response.status}`);
    }

    const data = await response.json();
    setData(data);
  } catch(err) {
    setError(err.message);
  } finally {
    setLoading(false); // always runs
  }
}
```

### async Always Returns Promise

```js
async function greet() {
  return "hello"; // plain value
}

greet() instanceof Promise // true!
greet().then(console.log)  // "hello"

// Even if you return nothing
async function doSomething() {}
doSomething() // Promise { undefined }
```

### Parallel vs Sequential — Performance

```js
// ❌ Sequential — SLOW (if fetchA=1s, fetchB=1s → total 2s)
const a = await fetchA();
const b = await fetchB();
// fetchB doesn't start until fetchA finishes

// ✅ Parallel — FAST (total ~1s)
const [a, b] = await Promise.all([fetchA(), fetchB()]);
// Both start at the same time

// Visual:
// Sequential: [--fetchA--][--fetchB--] = 2s
// Parallel:   [--fetchA--]
//             [--fetchB--]            = 1s
```

### When Sequential IS Correct

```js
// When second call DEPENDS on first result
const user = await fetchUser();
const posts = await fetchPosts(user.id); // needs user.id!
// Sequential is REQUIRED here — not a performance mistake
```

### Error Handling Differences

```js
// Promise.all — one catch for everything
// If any fails → all fail
try {
  const [user, posts] = await Promise.all([
    fetchUser(),
    fetchPosts()
  ]);
} catch(err) {
  // catches first error — don't know which one failed
  console.error(err);
}

// Sequential — handle errors individually
let user, posts;
try {
  user = await fetchUser();
} catch(err) {
  console.error("fetchUser failed:", err);
  return; // stop here if user fails
}

try {
  posts = await fetchPosts(user.id);
} catch(err) {
  console.error("fetchPosts failed:", err);
  // can continue without posts
}
```

### Common Mistakes

```js
// ❌ Mistake 1 — await in forEach (doesn't work!)
const ids = [1, 2, 3];
ids.forEach(async (id) => {
  const user = await fetchUser(id); // forEach doesn't wait!
});

// ✅ Fix — use for...of
for (const id of ids) {
  const user = await fetchUser(id); // properly awaited
}

// ✅ Or use Promise.all for parallel
const users = await Promise.all(ids.map(id => fetchUser(id)));

// ❌ Mistake 2 — forgetting error handling
async function fetchData() {
  const data = await fetch("/api"); // if this fails, unhandled!
}

// ✅ Always wrap in try/catch
async function fetchData() {
  try {
    const data = await fetch("/api");
  } catch(err) {
    console.error(err);
  }
}
```

### async/await with Promise combinators

```js
// allSettled — get all results even if some fail
const results = await Promise.allSettled([
  fetchUser(),
  fetchPosts(),
  fetchComments()
]);

results.forEach(({ status, value, reason }) => {
  if (status === "fulfilled") console.log(value);
  else console.error(reason);
});
```

---

## 🔑 Key Concepts to Remember
| Concept | Detail |
|---|---|
| Under the hood | Syntactic sugar over Promises + generators |
| Return value | async function ALWAYS returns a Promise |
| Error handling | try/catch/finally |
| Parallel | `Promise.all([...])` — starts all at once |
| Sequential | `await` one by one — use when calls depend on each other |
| await in loops | Use `for...of` not `forEach` |
| Performance | Parallel saves time equal to slowest - fastest request |
