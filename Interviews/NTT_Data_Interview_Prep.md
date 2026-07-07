# NTT Data Interview Prep — React / JS / Node.js

Good luck tomorrow. You already went through one round and know their style now:
**they want working code + correct output, not lecture-style explanations.** So for every
answer below, lead with the code/output first, and only use the explanation to make sure
*you* understand it well enough to answer follow-up questions confidently.

---

## PART 1 — Questions from your last round (corrected + answered)

### Q1. What is the purpose of the `useReducer` hook in React? Explain with code.

**Purpose:** `useReducer` is used for managing complex state logic — especially when the
next state depends on the previous state, or when you have multiple sub-values that change
together (e.g., a form, a cart, a multi-step flow). It's React's built-in alternative to
`useState` for that case, and it follows the same reducer pattern as Redux (`(state, action) => newState`).

```jsx
import { useReducer } from "react";

const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    case "reset":
      return initialState;
    default:
      throw new Error("Unknown action: " + action.type);
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      <button onClick={() => dispatch({ type: "reset" })}>Reset</button>
    </div>
  );
}
```

**One-liner if asked "useState vs useReducer":** Use `useState` for simple, independent
values. Use `useReducer` when state transitions are complex, interrelated, or when the
update logic itself should be testable/reusable outside the component.

---

### Q2. When do we need a Higher-Order Component (HOC)? Example with code.

**Corrected question:** *"When do we need a Higher-Order Component? Give an example of a HOC in React with code."*

**When you need one:** When you want to reuse component *logic* (not UI) across multiple
components — e.g., authentication checks, logging, injecting shared props, permission
gating — without repeating that logic in every component. A HOC is a function that takes a
component and returns a new enhanced component.

```jsx
// HOC that protects a route based on auth status
function withAuth(WrappedComponent) {
  return function AuthenticatedComponent(props) {
    const isLoggedIn = Boolean(localStorage.getItem("token"));

    if (!isLoggedIn) {
      return <p>Please log in to continue.</p>;
    }
    return <WrappedComponent {...props} />;
  };
}

function Dashboard({ user }) {
  return <h1>Welcome, {user}</h1>;
}

const ProtectedDashboard = withAuth(Dashboard);

// usage: <ProtectedDashboard user="Rahul" />
```

**Follow-up they may ask: HOC vs Custom Hook?**
- HOC wraps a *component* and returns a new component (affects the render tree, can inject props).
- Custom Hook shares *logic* only, doesn't touch the render tree.
- Use a HOC when you need to wrap/replace rendering itself (e.g., route guarding, layout
  injection, conditionally rendering nothing). Use a hook when you just need shared stateful
  logic inside a component you still fully control.

---

### Q3. Write code for `useRef` hook in React.

```jsx
import { useRef, useEffect } from "react";

function TextInputFocus() {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current.focus(); // runs once, focuses input on mount
  }, []);

  return <input ref={inputRef} placeholder="Auto-focused input" />;
}
```

Second common variant — using `useRef` to store a mutable value that does **not** trigger re-render:

```jsx
function RenderCounter() {
  const renderCount = useRef(0);
  renderCount.current += 1; // mutating ref does NOT cause re-render

  return <p>This component has rendered {renderCount.current} times</p>;
}
```

---

### Q4. How do you pass data between sibling components using React Router?

**Corrected question:** *"How do you share/pass data between sibling route components using React Router?"* — React Router doesn't move data between siblings directly; siblings don't have a parent-child relationship to each other. You use one of these patterns:

**Option A — Lift state up to the common parent (most common answer expected):**
```jsx
function Parent() {
  const [sharedData, setSharedData] = useState("");
  return (
    <>
      <SiblingA data={sharedData} setData={setSharedData} />
      <SiblingB data={sharedData} />
    </>
  );
}
```

**Option B — `useOutletContext` when siblings are nested routes under a shared layout route:**
```jsx
// Parent route element
<Outlet context={{ user: currentUser }} />

// In either sibling route component:
import { useOutletContext } from "react-router-dom";
const { user } = useOutletContext();
```

**Option C — Pass via `navigate` state when routing from one page to the next:**
```jsx
navigate("/checkout", { state: { cartTotal: 499 } });

// In the destination component:
import { useLocation } from "react-router-dom";
const { state } = useLocation();
console.log(state.cartTotal);
```

**Option D — Shared URL search params** (both siblings read/write the same query string) or a
**global store** (Context API / Redux / Zustand) when the components aren't in a direct
parent-child relationship at all.

Mention all four briefly if asked "how would you do it" — it shows you know the trade-offs
instead of just one trick.

---

### Q5. Prototype output question

```js
function Obj() {}
Obj.prototype.x = 10;
const obj1 = new Obj();
const obj2 = new Obj();
Obj.prototype = { x: 20 };
const obj3 = new Obj();
console.log(obj1.x, obj2.x, obj3.x);
Obj.prototype.x = 30;
console.log(obj1.x, obj2.x, obj3.x);
```

**Output:**
```
10 10 20
10 10 30
```

**Why:** `obj1` and `obj2` are created *before* `Obj.prototype` is reassigned, so their
internal `[[Prototype]]` link points to the **original** prototype object (the one with
`x: 10`). Reassigning `Obj.prototype = { x: 20 }` only changes what the constructor's
`.prototype` property points to going forward — it does not retroactively change the
prototype link of already-created instances. `obj3` is created *after* the reassignment, so
it links to the new object (`x: 20`).

For the second log: `Obj.prototype.x = 30` mutates the **new** prototype object (the one
`obj3` is linked to), so `obj3.x` becomes `30`. `obj1` and `obj2` are still linked to the
old prototype object, which is untouched, so they remain `10`.

---

### Q6. Async/await + event loop ordering

```js
async function async1() {
  console.log('async1 start');
  await async2();
  console.log('async1 end');
}
async function async2() {
  console.log('async2');
}
console.log('script start');
setTimeout(function() { console.log('setTimeout'); }, 0);
async1();
new Promise(function(resolve) {
  console.log('promise1');
  resolve();
}).then(function() { console.log('promise2'); });
console.log('script end');
```

**Output, in order:**
```
script start
async1 start
async2
promise1
script end
async1 end
promise2
setTimeout
```

**Why (walk through this out loud in the interview):**
1. `script start` — first synchronous line.
2. `setTimeout` callback is registered (macrotask queue) — doesn't run yet.
3. `async1()` runs synchronously up to its first `await`: logs `async1 start`, then calls
   `async2()`, which synchronously logs `async2` and returns. The `await` then pauses
   `async1` and schedules its remainder (`console.log('async1 end')`) as a **microtask**,
   handing control back to the caller.
4. Back in the main script, the `new Promise(...)` executor runs **synchronously**, logging
   `promise1`, then calls `resolve()` — this schedules the `.then` callback (`promise2`) as
   a microtask.
5. `script end` logs — synchronous code is now finished.
6. Event loop drains the **microtask queue** in FIFO order: `async1 end` was queued before
   `promise2`, so it runs first, then `promise2`.
7. Only after the microtask queue is empty does the event loop move to the **macrotask
   queue**, running the `setTimeout` callback last — `setTimeout`.

**Golden rule to state confidently:** microtasks (promises, async/await continuations)
always fully drain before the next macrotask (setTimeout, setInterval, I/O) runs.

---

### Q7. Object key coercion

```js
let a = {};
let b = { key: "b" };
let c = { key: "c" };
a[b] = 123;
a[c] = 456;
console.log(a[b]);
```

**Output:** `456`

**Why:** Object property keys must be strings (or Symbols). When you use an object as a
key (`a[b]`), JavaScript implicitly calls `.toString()` on it. Both `b` and `c` are plain
objects, so both stringify to the same value: `"[object Object]"`. That means `a[b] = 123`
and `a[c] = 456` are actually writing to the **same key** — the second write overwrites the
first. So `a` ends up as `{ "[object Object]": 456 }`, and `a[b]` (which also coerces `b` to
`"[object Object]"`) retrieves `456`.

---

## PART 2 — Extra questions likely to come up (React)

### `React.memo` vs `useMemo` vs `useCallback`

| | What it wraps | What it prevents | Returns |
|---|---|---|---|
| `React.memo` | A component | Re-render of that component if props are shallow-equal to last render | A memoized component |
| `useMemo` | A computed value | Recomputation of an expensive value on every render | A memoized value |
| `useCallback` | A function | Recreation of a new function reference on every render | A memoized function reference |

```jsx
const MemoChild = React.memo(function Child({ value }) {
  console.log("Child rendered");
  return <p>{value}</p>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState("");

  // Without useMemo, this recalculates every render
  const expensiveValue = useMemo(() => {
    console.log("Recalculating...");
    return count * 1000;
  }, [count]);

  // Without useCallback, a new function reference breaks React.memo on Child
  const handleClick = useCallback(() => {
    console.log("Clicked", count);
  }, [count]);

  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <MemoChild value={expensiveValue} onClick={handleClick} />
    </>
  );
}
```

**When `useMemo` actually hurts performance:**
- The computation is cheap — the overhead of memoization (storing, comparing dependencies)
  costs more than just recalculating.
- The dependency array changes almost every render anyway, so the cache is invalidated
  constantly and you pay for memoization with none of the benefit.
- It's used on primitive values or trivial JSX that React already re-renders cheaply.
- Overusing it bloats memory (every memoized value stays in the fiber) and hurts readability
  without measurable gain — profile first, memoize second.

---

### `useRef` for persisting data / tracking previous value

```jsx
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value; // runs AFTER render, so during render ref.current still holds the old value
  }, [value]);
  return ref.current;
}

function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  return (
    <div>
      <p>Now: {count}, Before: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

Key point to say out loud: updating `ref.current` does **not** trigger a re-render (unlike
`useState`), which is exactly why it's the right tool for tracking values across renders
without causing extra render cycles.

---

### `useReducer` + Context vs Redux

Use `useReducer` + Context when:
- State is local to a feature/section of the app, not truly global.
- You don't need middleware, time-travel debugging, or dev-tools-level tracing.
- Bundle size matters and you want to avoid an extra dependency.

Use Redux (or Zustand/Recoil) when:
- State needs to be shared across many unrelated parts of a large app.
- You need middleware (logging, async side effects via thunks/sagas), or strict predictable
  patterns across a big team.

**Performance trade-off to mention:** Context has no built-in selector mechanism — *any*
value change in the context triggers a re-render of **every** component consuming that
context, even if they only care about one field. Redux (with `useSelector`) only re-renders
components whose selected slice actually changed. Mitigation with Context: split state into
multiple smaller contexts, or memoize the provider's value with `useMemo`.

---

### HOC — real scenario where it's mandatory over a custom hook

**RBAC route locking:**
```jsx
function withRoleAccess(WrappedComponent, allowedRoles) {
  return function (props) {
    const { user } = useAuth();
    if (!allowedRoles.includes(user.role)) {
      return <Navigate to="/unauthorized" />;
    }
    return <WrappedComponent {...props} />;
  };
}

const AdminPanel = withRoleAccess(AdminPanelComponent, ["admin", "superadmin"]);
```

**Why a HOC and not a hook here:** the decision to render *nothing* (or redirect) has to
happen **before** the wrapped component mounts and runs its own hooks/effects. A custom hook
runs *inside* the component — you can't use it to stop that component from rendering at all
without messy early-return logic duplicated in every guarded component. A HOC intercepts at
the render-tree level, so the protected component's code never even executes for
unauthorized users. Same logic applies to layout-wrapping (injecting a consistent
header/sidebar around many different page components).

---

## PART 3 — Node.js questions

### Event loop: `process.nextTick()` vs `setImmediate()` vs `setTimeout()`

```js
console.log('start');

setTimeout(() => console.log('setTimeout'), 0);
setImmediate(() => console.log('setImmediate'));
process.nextTick(() => console.log('nextTick'));

console.log('end');
```

**Typical output:**
```
start
end
nextTick
setTimeout   // or setImmediate — order between these two can vary depending on context
setImmediate // (see note below)
```

**Explain the phases:** Node's event loop has phases — timers → pending callbacks → idle/prepare → poll → check → close callbacks. `process.nextTick()` isn't technically part of the phases at all — it runs in a special microtask-like queue that's fully drained **after the current operation completes and before the event loop continues to the next phase**, so it always beats everything else, including native Promises. `setTimeout(fn, 0)` runs in the **timers** phase; `setImmediate()` runs in the **check** phase (right after poll). From the main module, their relative order isn't guaranteed. But **inside an I/O callback**, `setImmediate()` is always guaranteed to run before `setTimeout(fn, 0)`, because the poll phase transitions directly into the check phase.

---

### `cluster` vs `worker_threads` vs `spawn()` vs `fork()`

| API | Use case |
|---|---|
| `cluster` | Spin up multiple **Node processes** (one per CPU core), each running a full copy of the app, sharing the same server port. Best for scaling a stateless HTTP server across cores. |
| `worker_threads` | Run JS in a separate **thread** within the same process, sharing memory via `SharedArrayBuffer`. Best for CPU-heavy tasks (image processing, parsing) without spawning a whole new process. |
| `child_process.spawn()` | Launch an external command/process (e.g., a shell script, ffmpeg), streaming stdout/stderr. No Node-specific IPC by default. |
| `child_process.fork()` | Special case of `spawn()` specifically for launching **another Node.js script** as a child process, with a built-in IPC channel for message passing. |

**When to use `cluster` over child processes in an enterprise multi-core server:** when the
goal is horizontal scaling of a stateless web server to use all CPU cores, with automatic
load-balancing across workers and simple restart-on-crash semantics — `cluster` is built
exactly for this. Use `worker_threads` instead when the bottleneck is CPU-bound computation
within a single request that you don't want blocking the event loop, and you want to share
memory rather than pay IPC serialization costs.

---

### Streams/Buffers vs `fs.readFile` for large files

```js
const fs = require("fs");

app.get("/report.csv", (req, res) => {
  const stream = fs.createReadStream("./large-report.csv");
  stream.pipe(res); // streams chunks directly to the client
});
```

**Why not `fs.readFile`:** `fs.readFile` loads the **entire file into memory** as a single
`Buffer` before sending anything to the client. For a large PDF/CSV (say hundreds of MB),
this spikes memory usage per request, and with many concurrent requests that can crash the
process (OOM). A `Readable` stream reads and sends the file in small chunks, keeping memory
usage flat regardless of file size, and the client starts receiving data immediately instead
of waiting for the full read. `Buffer` still matters here — each chunk is a `Buffer` of raw
bytes — but only one chunk-sized buffer is in memory at a time, not the whole file, and
`pipe()` also automatically handles backpressure (pausing the read if the client can't keep up).

---

### Avoiding callback hell / `Promise.all` vs `allSettled` vs `race`

**Avoiding callback hell:** convert callback-based code to Promises (`util.promisify`), use
`async/await` for linear-looking asynchronous flow, and break logic into small named
functions instead of nesting.

```js
// Promise.all — fails fast: if ANY promise rejects, the whole thing rejects immediately
try {
  const [user, orders, inventory] = await Promise.all([
    getUser(id), getOrders(id), getInventory(id)
  ]);
} catch (err) {
  // one failure kills all results — use when all three are required together
}

// Promise.allSettled — waits for all, never rejects; gives status of each
const results = await Promise.allSettled([getUser(id), getOrders(id), getInventory(id)]);
results.forEach(r => r.status === "fulfilled" ? console.log(r.value) : console.error(r.reason));
// use when partial success is acceptable — e.g., a dashboard that can render with some data missing

// Promise.race — resolves/rejects as soon as the FIRST promise settles
const result = await Promise.race([
  fetchFromPrimaryDB(),
  new Promise((_, reject) => setTimeout(() => reject("timeout"), 3000))
]);
// use for timeout patterns, or picking the fastest of several redundant sources
```

**Production error-handling angle:** `Promise.all` is right when a partial result is
useless (e.g., you truly need all three pieces to render). `allSettled` is right for
resilience — don't let one failing non-critical call take down the whole response.
`race` is the standard pattern for implementing timeouts around slow external calls.

---

### Custom Express middleware — JWT validation

```js
const jwt = require("jsonwebtoken");

function authenticateJWT(req, res, next) {
  const authHeader = req.headers["authorization"]; // "Bearer <token>"
  const token = authHeader && authHeader.split(" ")[1];

  if (!token) {
    return res.status(401).json({ message: "No token provided" });
  }

  jwt.verify(token, process.env.JWT_SECRET, (err, decoded) => {
    if (err) {
      return res.status(403).json({ message: "Invalid or expired token" });
    }
    req.user = decoded; // inject user context payload into the request
    next(); // pass control to the next middleware/route handler
  });
}

// usage
app.get("/api/profile", authenticateJWT, (req, res) => {
  res.json({ message: `Hello ${req.user.name}` });
});
```

**Request-response lifecycle to mention:** Express processes a request through a chain of
middleware functions in the order they're registered. Each middleware receives
`(req, res, next)` and either ends the cycle (`res.send/json/end`) or calls `next()` to pass
control forward. If `next(err)` is called with an argument, Express skips straight to
error-handling middleware. This chain-of-responsibility structure is why middleware order
(e.g., body parser before auth, auth before the route handler) matters.

---

## A few tactical notes for tomorrow

- Given last time's format (chat box + live compiler, no explanation wanted), **practice
  typing these from memory** rather than just reading them — muscle memory matters more than
  understanding under that kind of time pressure.
- For the JS trivia questions (prototype, event loop, key coercion), if you get stuck, just
  write your best-guess output first, then adjust — a partial/wrong-but-reasoned answer often
  scores better than a blank chat box.
- Since you're going in wanting leverage for negotiation: perform first, negotiate after —
  once there's an offer in hand, it's fine to be upfront that you're comparing offers and
  ask for a few days or a revised number. That conversation goes with HR/recruiter, not in
  the technical round.

Good luck — you've clearly already learned a lot from round one; this round should feel more
familiar.
