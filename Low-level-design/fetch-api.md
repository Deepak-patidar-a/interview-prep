# ⚛️ Fetch API — Loading, Error, Success States

## Requirements
- Fetch data from an API
- Show loading state while fetching
- Show error state if fetch fails
- Show data on success

---

## My Code
```jsx
import { useEffect, useState } from "react";

export default function App() {
  const [data, setData] = useState([]);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    const fetch = async () => {
      try {
        setLoading(true);
        const res = await fetch("https://dummyjson.com/products");
        if (!res.ok) {
          setError("Some http error coming");
        }
        const result = await res.json();
        setData(result.products);
      } catch (e) {
        setError(e.message);
      } finally {
        setLoading(false);
      }
    };
    fetch();
  }, []);

  return (
    <div>
      <h1>Fetch API</h1>
      {loading && <p>Loading...</p>}
      {error && <p>{error}</p>}
      {data.length > 0 &&
        data.map((m) => {
          return <div key={m.id}>{m.name}</div>;
        })}
    </div>
  );
}
```

---

## ✅ What I Did Well
- `finally` block for `setLoading(false)` — perfect, loading stops whether success or error
- `try/catch` structure — correct
- All 3 states managed — loading, error, data
- `res.ok` check for HTTP errors — great, most people forget this
- Correct conditional rendering order — loading → error → data

---

## 🚨 Critical Bug — Naming Conflict

```javascript
// ❌ Named inner function "fetch" — conflicts with browser's built-in fetch
const fetch = async () => {
  const res = await fetch("..."); // tries to call itself recursively 💥
}

// ✅ Fix — rename the function
const fetchData = async () => {
  const res = await fetch("..."); // now correctly calls browser fetch
}
fetchData();
```

---

## ⚠️ Two More Issues

**1. HTTP error doesn't stop execution**
```javascript
// ❌ Sets error but code keeps running — still tries to parse response
if (!res.ok) {
  setError("Some http error coming");
}
const result = await res.json(); // still runs!

// ✅ throw to jump directly to catch block
if (!res.ok) {
  throw new Error(`HTTP error! status: ${res.status}`);
}
const result = await res.json(); // only runs if res.ok ✅
```

**2. Wrong field name for dummyjson API**
```jsx
// ❌ products don't have "name" field in dummyjson
<div key={m.id}>{m.name}</div>

// ✅ correct field is "title"
<div key={m.id}>{m.title}</div>
```

---

## ✅ Clean Final Version
```jsx
import { useEffect, useState } from "react";

export default function App() {
  const [data, setData] = useState([]);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    const fetchData = async () => {              // ✅ renamed
      try {
        setLoading(true);
        const res = await fetch("https://dummyjson.com/products");

        if (!res.ok) {
          throw new Error(`HTTP error! status: ${res.status}`); // ✅ throw
        }

        const result = await res.json();
        setData(result.products);
      } catch (e) {
        setError(e.message);
      } finally {
        setLoading(false);                       // ✅ always runs
      }
    };

    fetchData();
  }, []);

  return (
    <div>
      <h1>Fetch API</h1>
      {loading && <p>Loading...</p>}
      {error && <p style={{ color: "red" }}>{error}</p>}
      {data.length > 0 &&
        data.map((m) => (
          <div key={m.id}>{m.title}</div>        // ✅ title not name
        ))}
    </div>
  );
}
```

---

## 🔑 Key Things To Remember
- Never name a function the same as a built-in — `fetch`, `map`, `filter` etc.
- `throw new Error()` inside try block jumps directly to catch — use it for HTTP errors
- `finally` always runs — perfect place for cleanup like `setLoading(false)`
- `res.ok` checks if status is 200-299 — always check this for HTTP errors

## Final Score: 7.5/10
