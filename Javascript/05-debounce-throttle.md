# 05 - Debouncing & Throttling

## ❓ Interview Question
> "What is debouncing and throttling in JavaScript? Explain the difference between them, when you'd use each one, and can you write a basic implementation of both from scratch?"

---

## 🙋 My Answer (What I Said)
- Both used for performance optimization
- Debounce — skip calls during fast typing, fires after user stops
- Throttle — call function on fixed interval
- Debounce use case: search API
- Throttle use case: infinite scroll, window resize
- Wrote correct implementations for both (minor syntax issues)

---

## ✅ What I Got Right
- Correct explanation of both concepts
- Right use cases for both
- Debounce implementation logic correct
- Throttle timestamp approach correct

## ❌ What Was Missing
- `new Date.now()` should be `Date.now()` — no `new`
- Variable name mismatch in throttle (`lastCall` vs `last`)
- Leading vs trailing debounce concept

---

## 💡 Model Answer

### Key Difference
> **Debounce** — waits for user to **stop** doing something, then fires ONCE
> **Throttle** — fires at **regular intervals** regardless of how many times triggered

### Debounce Implementation

```js
const debounce = (cb, delay) => {
  let timer;

  return function(...args) {
    clearTimeout(timer); // clear previous timer on every call
    timer = setTimeout(() => {
      cb(...args);
    }, delay);
  };
};

// Usage — search input
const handleSearch = debounce((value) => {
  console.log("Searching for:", value);
  fetch(`/api/search?q=${value}`);
}, 500);

input.addEventListener("input", (e) => handleSearch(e.target.value));
// If user types fast, only fires 500ms after they STOP typing
```

### Throttle Implementation

```js
const throttle = (cb, delay) => {
  let lastCall = 0;

  return function(...args) {
    const now = Date.now(); // ✅ no "new" keyword
    if (now - lastCall < delay) return; // too soon — skip
    lastCall = now;
    return cb(...args);
  };
};

// Usage — scroll event
const handleScroll = throttle(() => {
  console.log("scroll position:", window.scrollY);
}, 300);

window.addEventListener("scroll", handleScroll);
// Fires at most once every 300ms no matter how fast user scrolls
```

### Visual Difference

```
User events:  --|--|--|--|--|--|--|----|------->
              (fast typing or scrolling)

Debounce:     -------------------------fire-->
              (only fires after events stop)

Throttle:     --fire-----fire-----fire------->
              (fires at regular intervals)
```

### Leading vs Trailing Debounce

```js
const debounce = (cb, delay, leading = false) => {
  let timer;

  return function(...args) {
    const callNow = leading && !timer;

    clearTimeout(timer);
    timer = setTimeout(() => {
      timer = null;
      if (!leading) cb(...args); // trailing — fire after delay
    }, delay);

    if (callNow) cb(...args); // leading — fire immediately
  };
};

// Trailing (default) — fires AFTER user stops
const trailingSearch = debounce(search, 500);

// Leading — fires IMMEDIATELY, then ignores for 500ms
const leadingSearch = debounce(search, 500, true);
```

### Using Lodash (Production)

```js
import { debounce, throttle } from "lodash";

// Debounce
const debouncedSearch = debounce(fetchResults, 500);

// Throttle
const throttledScroll = throttle(handleScroll, 300);

// Cleanup in React
useEffect(() => {
  return () => {
    debouncedSearch.cancel();
    throttledScroll.cancel();
  };
}, []);
```

### React Hook Pattern

```js
// Custom useDebounce hook
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timer); // cleanup
  }, [value, delay]);

  return debouncedValue;
}

// Usage
const debouncedSearch = useDebounce(searchTerm, 500);
useEffect(() => {
  if (debouncedSearch) fetchResults(debouncedSearch);
}, [debouncedSearch]);
```

---

## 🔑 Key Concepts to Remember
| Concept | Debounce | Throttle |
|---|---|---|
| When fires | After events STOP | At regular INTERVALS |
| Use case | Search input, form validation | Scroll, resize, mousemove |
| Mechanism | `clearTimeout` + `setTimeout` | Timestamp comparison |
| Result | One call after pause | Regular calls during activity |
| Leading | Fire immediately, then wait | N/A |
| Trailing | Fire after delay ends | N/A |
