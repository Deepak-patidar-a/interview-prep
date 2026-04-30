# React & JavaScript Interview Questions

> A comprehensive guide covering React internals, hooks, CSS layout, and JavaScript concepts commonly asked in frontend interviews.

---

## Table of Contents

1. [Concurrent React](#1-concurrent-react)
2. [React Fiber](#2-react-fiber)
3. [Reconciliation vs React Fiber](#3-reconciliation-vs-react-fiber)
4. [useEffect vs useLayoutEffect](#4-useeffect-vs-uselayouteffect)
5. [useTransition vs useDeferredValue](#5-usetransition-vs-usedeferredvalue)
6. [Rest vs Spread Operator](#6-rest-vs-spread-operator)
7. [useDebounce Hook](#7-usedebounce-hook)
8. [Flexbox vs CSS Grid](#8-flexbox-vs-css-grid)
9. [Context API — Why Use It and Why Not](#9-context-api--why-use-it-and-why-not)

---

## 1. Concurrent React

Concurrent React is a set of features that allow React to **interrupt, pause, resume, or abandon renders** — instead of blocking the main thread until rendering is complete.

Before Concurrent mode, rendering was **synchronous**. Once React started rendering, it couldn't stop until it finished, blocking user interactions.

### With Concurrent React:
- React can work on multiple versions of the UI simultaneously
- It can **prioritize urgent updates** (like typing) over non-urgent ones (like loading data)
- Enables features like `useTransition`, `useDeferredValue`, and `Suspense`

### Key Benefit:
> Keeps your UI responsive even during heavy rendering work.

---

## 2. React Fiber

Fiber is the **internal reconciliation engine** rewritten in React 16. It is the algorithm that powers how React renders and updates the UI.

### Key Concepts:
- Each component becomes a **"fiber node"** — a JavaScript object containing info about the component, its state, props, and links to parent/child/sibling fibers
- Fiber breaks rendering into small **units of work** that can be paused and resumed
- Introduced a two-phase render:
  - **Render phase** → can be interrupted
  - **Commit phase** → always synchronous

### Summary:
> Fiber is the **foundation** that made Concurrent React possible.

---

## 3. Reconciliation vs React Fiber

| | Reconciliation | React Fiber |
|---|---|---|
| **What it is** | The *algorithm/concept* of diffing the virtual DOM | The *implementation/engine* that executes reconciliation |
| **Purpose** | Determine *what* changed | Determine *how and when* to apply changes |
| **Introduced** | Since React's beginning | React 16 rewrite |
| **Nature** | Abstract strategy | Concrete data structure + scheduler |

### Simple Analogy:
> Reconciliation is the **plan**, Fiber is the **engine** that executes the plan — with the added ability to pause and resume.

---

## 4. useEffect vs useLayoutEffect

```js
// useEffect — runs AFTER paint
useEffect(() => {
  document.title = "Hello"; // User sees the page first, then this runs
}, []);

// useLayoutEffect — runs BEFORE paint
useLayoutEffect(() => {
  // DOM is ready but browser hasn't painted yet
  const height = ref.current.getBoundingClientRect().height;
  setHeight(height); // No visual flicker
}, []);
```

### Comparison Table

| | `useEffect` | `useLayoutEffect` |
|---|---|---|
| **Timing** | After browser paints | After DOM update, before paint |
| **Blocking** | Non-blocking | Blocks paint |
| **SSR** | Safe | Warns on SSR |

### Use Cases

**Use `useEffect` for:**
- API calls and data fetching
- Setting up subscriptions
- Logging and analytics
- Setting document title

**Use `useLayoutEffect` for:**
- Reading DOM measurements (width, height, position)
- Preventing visual flicker
- Tooltip / popover positioning
- Animations that depend on DOM size

---

## 5. useTransition vs useDeferredValue

Both are Concurrent React tools for **marking updates as non-urgent**.

### useTransition — wraps the state setter

```jsx
const [isPending, startTransition] = useTransition();

const handleSearch = (val) => {
  setInput(val); // urgent — updates input immediately
  startTransition(() => {
    setResults(filter(val)); // non-urgent — can be interrupted
  });
};

// Show loading indicator using isPending
{isPending && <Spinner />}
```

### useDeferredValue — wraps the value itself

```jsx
const deferredQuery = useDeferredValue(query);

// deferredQuery lags behind query during fast typing
// Good when you don't control the state setter
return <HeavyList filter={deferredQuery} />;
```

### Comparison Table

| | `useTransition` | `useDeferredValue` |
|---|---|---|
| **Wraps** | The state update (setter) | The value |
| **Gives you** | `isPending` flag | Deferred copy of value |
| **Use when** | You control the state update | You receive a prop/value you can't control |

---

## 6. Rest vs Spread Operator

Same syntax (`...`), different context.

### Spread — expands elements OUT

```js
// Arrays
const arr1 = [1, 2];
const arr2 = [...arr1, 3, 4]; // [1, 2, 3, 4]

// Objects
const obj1 = { a: 1 };
const obj2 = { ...obj1, b: 2 }; // { a: 1, b: 2 }

// Function call
Math.max(...[1, 5, 3]); // same as Math.max(1, 5, 3)
```

### Rest — collects remaining elements INTO an array

```js
// Function parameters
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}

// Array destructuring
const [first, ...rest] = [1, 2, 3, 4];
// first = 1, rest = [2, 3, 4]

// Object destructuring
const { a, ...remaining } = { a: 1, b: 2, c: 3 };
// a = 1, remaining = { b: 2, c: 3 }
```

### Rule of Thumb:
> **Spread** is used in *expressions*. **Rest** is used in *parameters and destructuring*.

---

## 7. useDebounce Hook

A custom hook that delays updating a value until the user has stopped changing it for a specified duration.

```js
import { useState, useEffect } from "react";

function useDebounce(value, delay = 500) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    // Cleanup cancels the previous timer on every keystroke
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}
```

### Usage Example

```jsx
function SearchBar() {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebounce(query, 500);

  useEffect(() => {
    if (debouncedQuery) {
      fetchResults(debouncedQuery); // fires only 500ms after user stops typing
    }
  }, [debouncedQuery]);

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

### How It Works:
1. User types → `query` updates instantly (responsive input)
2. A 500ms timer starts
3. If the user types again before 500ms, the timer **resets**
4. After 500ms of no typing, `debouncedQuery` updates → API call fires

---

## 8. Flexbox vs CSS Grid

```css
/* FLEXBOX — one-dimensional (row OR column) */
.flex-container {
  display: flex;
  justify-content: space-between; /* main axis */
  align-items: center;            /* cross axis */
  flex-wrap: wrap;
}

/* GRID — two-dimensional (rows AND columns) */
.grid-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto;
  gap: 16px;
}
```

### Comparison Table

| | Flexbox | Grid |
|---|---|---|
| **Dimensions** | 1D (row or column) | 2D (rows AND columns) |
| **Direction** | Content drives layout | Layout drives content |
| **Best for** | Navbars, button groups, centering | Page layouts, card grids, dashboards |
| **Control** | Child items control themselves | Parent controls everything |

### When to Use Which

**Use Flexbox for:**
- Navigation bars
- Aligning items in a single row or column
- Centering content vertically/horizontally
- Wrapping items dynamically

**Use Grid for:**
- Overall page layout
- Image galleries
- Complex 2D layouts (rows AND columns matter)
- Dashboard layouts

> **Pro Tip:** They're not mutually exclusive — use Grid for the outer layout, Flexbox inside the grid cells.

---

## 9. Context API — Why Use It and Why Not

### ✅ Why Use Context API

- Avoid **prop drilling** (passing props through many layers)
- Share **global state**: theme, auth user, locale/language, feature flags
- Simple to set up — no extra library needed

```jsx
const ThemeContext = createContext();

function App() {
  return (
    <ThemeContext.Provider value={{ theme: "dark" }}>
      <DeepChild /> {/* No prop drilling needed */}
    </ThemeContext.Provider>
  );
}

function DeepChild() {
  const { theme } = useContext(ThemeContext); // Direct access anywhere in tree
}
```

### ❌ Why NOT Use Context API

- **Performance:** Every consumer re-renders when context value changes — even if they only use one field of a large object
- **Not for frequent updates:** State that changes often (search input, scroll position) causes widespread re-renders
- **No selector support:** Unlike Redux/Zustand, you can't subscribe to only a slice of context
- **Harder to debug** compared to dedicated state management tools with DevTools

### When to Prefer Something Else

| Situation | Better Choice |
|---|---|
| Frequent updates (forms, animations) | Local state or Zustand |
| Complex global state with many updates | Redux Toolkit / Zustand |
| Server state (API data, caching) | React Query / SWR |
| Simple global read-mostly state | Context API ✅ |

---

## Quick Reference Summary

| Topic | One-Liner |
|---|---|
| Concurrent React | React can pause/resume renders to keep UI responsive |
| React Fiber | Internal engine (data structure) that powers rendering |
| Reconciliation | The diffing algorithm to find what changed |
| useEffect | Runs after paint — for side effects |
| useLayoutEffect | Runs before paint — for DOM measurements |
| useTransition | Marks a state update as non-urgent |
| useDeferredValue | Defers a value to avoid blocking urgent renders |
| Spread (`...`) | Expands elements outward |
| Rest (`...`) | Collects remaining elements inward |
| useDebounce | Delays value update until user stops changing it |
| Flexbox | 1D layout — row or column |
| Grid | 2D layout — rows and columns together |
| Context API | Good for static global state, bad for frequent updates |

---

*Prepared for frontend interview preparation — React & JavaScript core concepts.*
