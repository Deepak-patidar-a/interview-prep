# 06 - Promises in JavaScript

## ❓ Interview Question
> "Can you explain Promises in JavaScript? Walk me through the three states, how Promise chaining works, and the difference between Promise.all, Promise.allSettled, Promise.race, and Promise.any — with use cases for each."

---

## 🙋 My Answer (What I Said)
- Promises used for async operations
- Three states: resolve, settle, reject (incorrect)
- `Promise.all` — settles when any promise fails
- `Promise.allSettled` — waits for all to settle, for independent APIs
- `Promise.race` — settles when first promise fails (incorrect)
- `Promise.any` — first to resolve, AggregateError if all fail

---

## ✅ What I Got Right
- `Promise.allSettled` behavior and use case correct
- `Promise.any` correct
- `Promise.all` array of promises concept correct

## ❌ What Was Missing
- Correct three states: Pending, Fulfilled, Rejected
- `Promise.all` — resolves when ALL resolve, rejects when ANY ONE fails
- `Promise.race` — first to settle (resolve OR reject)
- Promise chaining
- Comparison table

---

## 💡 Model Answer

### Three States

```js
// Pending → initial state
const p = new Promise((resolve, reject) => {
  // doing async work...
});

// Fulfilled → resolve was called
const p2 = new Promise((resolve, reject) => {
  resolve("success data"); // → fulfilled
});

// Rejected → reject was called
const p3 = new Promise((resolve, reject) => {
  reject(new Error("something failed")); // → rejected
});
```

### Creating Promises

```js
function fetchUser(id) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (id > 0) {
        resolve({ id, name: "Deepak" }); // success
      } else {
        reject(new Error("Invalid ID")); // failure
      }
    }, 1000);
  });
}
```

### Promise Chaining

```js
fetch("/api/user")
  .then(res => res.json())                    // parse response
  .then(user => fetch(`/api/posts/${user.id}`)) // use result
  .then(res => res.json())                    // parse again
  .then(posts => console.log(posts))          // use final result
  .catch(err => console.error(err))           // catch ANY error above
  .finally(() => setLoading(false));          // always runs
```

### Promise.all

```js
// Resolves when ALL resolve
// Rejects immediately when ANY ONE fails (fail fast)
const [user, posts, comments] = await Promise.all([
  fetch("/api/user").then(r => r.json()),
  fetch("/api/posts").then(r => r.json()),
  fetch("/api/comments").then(r => r.json()),
]);
// Use when APIs DEPEND on all succeeding together
// If any one fails → whole thing fails
```

### Promise.allSettled

```js
// Waits for ALL to settle (resolve OR reject)
// Never rejects — always gives results array
const results = await Promise.allSettled([
  fetch("/api/user").then(r => r.json()),
  fetch("/api/posts").then(r => r.json()),
  fetch("/api/comments").then(r => r.json()),
]);

results.forEach(result => {
  if (result.status === "fulfilled") {
    console.log("success:", result.value);
  } else {
    console.log("failed:", result.reason);
  }
});
// Use when APIs are INDEPENDENT — want all results even if some fail
```

### Promise.race

```js
// Settles with FIRST promise that settles (resolve OR reject)
const timeout = new Promise((_, reject) =>
  setTimeout(() => reject(new Error("Timeout!")), 5000)
);

try {
  const result = await Promise.race([fetchData(), timeout]);
  // if fetchData resolves first → success
  // if timeout rejects first → error
} catch(err) {
  console.log("Timed out!");
}
// Use case: implementing request timeouts
```

### Promise.any

```js
// Resolves with FIRST promise that resolves
// Only rejects if ALL promises reject (AggregateError)
const fastestServer = await Promise.any([
  fetch("https://server1.api.com/data"),
  fetch("https://server2.api.com/data"),
  fetch("https://server3.api.com/data"),
]);
// Use case: redundant requests — use whichever responds first
```

---

## 🔑 Key Concepts to Remember
| Method | Resolves when | Rejects when |
|---|---|---|
| `Promise.all` | ALL resolve | ANY ONE rejects |
| `Promise.allSettled` | ALL settle | Never rejects |
| `Promise.race` | FIRST settles (resolve/reject) | FIRST rejects |
| `Promise.any` | FIRST resolves | ALL reject (AggregateError) |

### Three States
| State | Meaning |
|---|---|
| Pending | Initial — neither fulfilled nor rejected |
| Fulfilled | `resolve()` was called — success |
| Rejected | `reject()` was called — failure |
| Settled | Either fulfilled OR rejected (not a state, just a term) |
