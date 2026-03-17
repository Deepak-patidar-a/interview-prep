# 🏢 React Interview Revision — Wipro Round

---

## Q1. What is Microfrontend?

Think of it like **microservices but for the frontend**.

In a normal app, the entire frontend is one big codebase — one team, one deploy. In microfrontend architecture, you **split the frontend into smaller independent apps**, each owned by a different team, that come together to look like one app.

```
Normal App:
[ One big React app — all features together ]

Microfrontend:
[ Header App ] + [ Cart App ] + [ Product App ] + [ Checkout App ]
     ↓                ↓               ↓                  ↓
  Team A           Team B          Team C             Team D
```

**Key points to say in interview:**
- Each micro app can use **different frameworks** (one in React, one in Vue)
- Each can be **deployed independently** — no need to redeploy the whole app
- They communicate via **custom events or shared state**
- Popular implementation: **Module Federation** (Webpack 5 feature)

> 💡 Real world example: Think of Amazon — the navbar, product listing, cart, and recommendations are likely all separate teams/apps stitched together.

---

## Q2. What is Batching in React?

Batching means React **groups multiple state updates together** and does **one single re-render** instead of re-rendering for each update.

```js
// Without batching — would cause 2 re-renders
setCount(c => c + 1);
setName("Deepak");

// With batching — React groups both, causes only 1 re-render ✅
```

**React 17 and before** — batching only happened inside React event handlers. If you updated state inside `setTimeout` or a `Promise`, it would re-render separately for each update.

**React 18** introduced **Automatic Batching** — now batching works everywhere, including `setTimeout`, `fetch`, and `Promises`.

```js
// React 18 — all three updates = only ONE re-render
setTimeout(() => {
  setCount(1);
  setName("Deepak");
  setLoading(false);
}, 1000);
```

> 💡 If you ever need to **opt out** of batching (rare), React 18 provides `flushSync()`.

```js
import { flushSync } from "react-dom";

flushSync(() => setCount(1));       // re-renders immediately
flushSync(() => setName("Deepak")); // re-renders immediately
```

---

## Q3. useReducer — What it is and When to Use it

`useReducer` is an alternative to `useState` for managing **complex state logic**.

```js
const [state, dispatch] = useReducer(reducer, initialState);
```

The **reducer** is a pure function that takes current state + action, and returns new state — exactly like Redux.

```js
const initialState = { count: 0, name: "" };

function reducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return { ...state, count: state.count + 1 };
    case "SET_NAME":
      return { ...state, name: action.payload };
    default:
      return state;
  }
}

function App() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <button onClick={() => dispatch({ type: "INCREMENT" })}>
      {state.count}
    </button>
  );
}
```

**When to use useReducer over useState:**

| Situation | Use |
|---|---|
| Simple toggle, single value | `useState` |
| Multiple related state values | `useReducer` |
| Next state depends on previous | `useReducer` |
| Complex update logic | `useReducer` |
| State looks like a Redux slice | `useReducer` |

> 💡 Think of it this way — `useReducer` is basically **Redux inside a single component**.

---

## Q4. What Files Get Generated in `dist` After Build?

When you run `yarn build` or `npm run build`, the build tool (Vite / Webpack) **compiles, bundles and optimizes** your code into static files ready for production.

**Files generated in `/dist`:**

```
/dist
  /assets
    index-3f2a1b.js     → Your entire app JS (minified + bundled)
    index-8c4d2e.css    → All your CSS (minified)
    logo-7a3b1c.png     → Static assets (images, fonts)
  index.html            → Entry HTML file
```

**What happens during build:**
- **Bundling** — all JS files merged into one (or few) files
- **Minification** — removes spaces, comments, shortens variable names to reduce file size
- **Tree shaking** — removes unused/dead code from the bundle
- **Code splitting** — splits into chunks so browser loads only what's needed
- **Hashing** — filenames get a hash (like `index-3f2a1b.js`) for **cache busting**

> 💡 **Cache busting** is the most impressive thing to mention — the hash in the filename forces the browser to download a fresh file on every new deploy, instead of using the old cached version.

---

## 🔑 Quick Cheat Sheet

| Topic | One Line to Remember |
|---|---|
| Microfrontend | Split frontend into independent deployable apps — like microservices for UI |
| Module Federation | Webpack 5 feature that enables microfrontends |
| Batching | Multiple state updates → one single re-render |
| React 18 Batching | Automatic batching everywhere — setTimeout, fetch, Promises |
| flushSync | Opt out of batching when you need immediate re-render |
| useReducer | useState for complex/related state — uses reducer pure function + dispatch |
| useReducer vs useState | Multiple related states or complex logic → useReducer |
| dist folder | Bundled, minified, tree-shaken, hashed files ready for production |
| Cache busting | Hash in filename forces browser to fetch fresh file on new deploy |
