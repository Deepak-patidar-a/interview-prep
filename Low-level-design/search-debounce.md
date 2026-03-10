# ⚛️ Search with Debounce

## Requirements
- Search input that filters a list
- Debounce the search — filter only after user stops typing (500ms)
- Show all items when search is empty
- Case insensitive search

---

## My Code
```jsx
import { useEffect, useState } from "react";

const products = [
  { id: 1, title: "deepak" },
  { id: 2, title: "orange" },
  { id: 3, title: "banana" },
  { id: 4, title: "kiwi" },
  { id: 5, title: "grapes" },
  { id: 6, title: "waterM" },
  { id: 7, title: "mango" },
  { id: 8, title: "onion" },
  { id: 9, title: "sugar" },
];

export default function App() {
  const [data, setData] = useState([]);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(false);
  const [search, setSearch] = useState("");
  const debounceQuery = useDebounce(search, 500);

  useEffect(() => {
    setLoading(true);
    const filtered = products.filter((prod) =>
      prod.title.startsWith(debounceQuery)
    );
    setData(filtered);
    setLoading(false);
  }, [debounceQuery]);

  return (
    <div>
      <h1>Search</h1>
      <input
        type="text"
        value={search}
        onChange={(e) => setSearch(e.target.value)}
      />
      {loading && <p>Loading...</p>}
      {error && <p>{error}</p>}
      {data?.length > 0 &&
        data.map((m) => {
          return <div key={m.id}>{m.title}</div>;
        })}
    </div>
  );
}

const useDebounce = (val, delay = 500) => {
  const [debounce, setDebounce] = useState(val);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebounce(val);
    }, delay);

    return () => clearTimeout(timer);
  }, [val, delay]);

  return debounce;
};
```

---

## ✅ What I Did Well
- `useDebounce` hook written correctly from memory
- Cleanup function `return () => clearTimeout(timer)` — perfect
- Filtering using `debounceQuery` not `search` — exactly right
- `data?.length` optional chaining — good defensive coding

---

## 🚨 Critical Bug — Hook Defined After Use

```javascript
// ❌ useDebounce used on line 15 but defined at the bottom
const debounceQuery = useDebounce(search, 500);  // line 15 — used here

const useDebounce = (val, delay) => { ... }      // defined after — ReferenceError!

// ✅ Always define custom hooks BEFORE the component
const useDebounce = (val, delay) => { ... }      // hook first

export default function App() { ... }            // component second
```

---

## ⚠️ Three More Issues

**1. Unnecessary loading state for sync operations**
```javascript
// ❌ Filtering local array is instant — loading is true for 0ms, user never sees it
setLoading(true);
const filtered = products.filter(...);  // sync — instant
setLoading(false);

// ✅ Remove loading for local filter — only use loading for real API calls
```

**2. Case sensitive search**
```javascript
// ❌ "Mango" won't match if user types "mango"
prod.title.startsWith(debounceQuery)

// ✅ Lowercase both sides
prod.title.toLowerCase().startsWith(debounceQuery.toLowerCase())
```

**3. Empty search shows nothing**
```javascript
// ❌ Empty string filters out everything
products.filter(prod => prod.title.startsWith(""))  // returns nothing

// ✅ Show all when search is empty
const filtered = debounceQuery
  ? products.filter(prod =>
      prod.title.toLowerCase().startsWith(debounceQuery.toLowerCase())
    )
  : products;
```

---

## ✅ Clean Final Version
```jsx
import { useEffect, useState } from "react";

const products = [
  { id: 1, title: "deepak" },
  { id: 2, title: "orange" },
  { id: 3, title: "banana" },
  { id: 4, title: "kiwi" },
  { id: 5, title: "grapes" },
  { id: 6, title: "waterM" },
  { id: 7, title: "mango" },
  { id: 8, title: "onion" },
  { id: 9, title: "sugar" },
];

// ✅ Hook defined BEFORE component
const useDebounce = (val, delay = 500) => {
  const [debounced, setDebounced] = useState(val);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebounced(val);
    }, delay);

    return () => clearTimeout(timer);
  }, [val, delay]);

  return debounced;
};

export default function App() {
  const [data, setData] = useState(products);   // ✅ show all initially
  const [search, setSearch] = useState("");
  const debounceQuery = useDebounce(search, 500);

  useEffect(() => {
    const filtered = debounceQuery
      ? products.filter((prod) =>               // ✅ filter only if query exists
          prod.title.toLowerCase().startsWith(debounceQuery.toLowerCase())
        )
      : products;                               // ✅ show all if empty

    setData(filtered);
  }, [debounceQuery]);

  return (
    <div>
      <h1>Search with Debounce</h1>
      <input
        type="text"
        value={search}
        onChange={(e) => setSearch(e.target.value)}
        placeholder="Search products..."
      />
      {data.length === 0 && <p>No results found</p>}
      {data.map((m) => (
        <div key={m.id}>{m.title}</div>
      ))}
    </div>
  );
}
```

---

## 🔑 Key Things To Remember
- Always define custom hooks **before** the component — `const` is not hoisted
- Remove loading state for sync operations — only use it for async API calls
- Always lowercase both sides for case insensitive search
- Empty search should show all items — add a ternary check
- Debounce cleanup `clearTimeout` is the most important part — cancels previous timer

## Final Score: 8.5/10
