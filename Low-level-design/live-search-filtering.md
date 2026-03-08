# 02 - Live Search & Filtering in React

## ❓ Interview Question
> "How would you build a search functionality with live filtering in a React application? Walk me through how you'd handle user input, optimise filtering for large datasets, and manage results when interacting with an API."

---

## 🙋 My Answer (What I Said)
- Controlled input with `value` and `onChange`
- Debounce in `handleSearch` to avoid API call on every keystroke
- Fetch top 10 results only for large datasets
- Reset dropdown state on each new API response
- Show results in a toggle dropdown
- Implement infinite scroll inside dropdown for loading more

---

## ✅ What I Got Right
- Controlled input pattern
- Debounce concept for optimizing API calls
- Paginating results (top 10 only)
- Resetting dropdown on each new search
- Dropdown UI for results
- Infinite scroll inside dropdown (bonus - mature real-world thinking!)

## ❌ What Was Wrong / Missing
- Didn't explain HOW debounce is implemented (setTimeout pattern)
- Didn't mention client-side filtering for small/local datasets
- Missing: loading & error states
- Missing: AbortController to cancel previous in-flight requests
- Missing: empty state handling (no results found)
- Missing: minimum character threshold before searching

---

## 💡 Model Answer

### Basic Live Search with Debounce

```jsx
import { useState, useEffect } from "react";

const SearchComponent = () => {
  const [search, setSearch] = useState("");
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [showDropdown, setShowDropdown] = useState(false);

  useEffect(() => {
    // Minimum character threshold
    if (search.length < 2) {
      setResults([]);
      setShowDropdown(false);
      return;
    }

    const controller = new AbortController(); // cancel previous request

    const timer = setTimeout(async () => {
      setLoading(true);
      setError(null);
      try {
        const res = await fetch(`/api/search?q=${search}&limit=10`, {
          signal: controller.signal,
        });
        if (!res.ok) throw new Error("Something went wrong");
        const data = await res.json();
        setResults(data);
        setShowDropdown(true);
      } catch (err) {
        if (err.name !== "AbortError") {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    }, 500); // debounce delay

    return () => {
      clearTimeout(timer);       // clear debounce
      controller.abort();        // cancel previous fetch
    };
  }, [search]);

  return (
    <div style={{ position: "relative" }}>
      <input
        value={search}
        onChange={(e) => setSearch(e.target.value)}
        placeholder="Search..."
      />

      {loading && <p>Searching...</p>}
      {error && <p style={{ color: "red" }}>{error}</p>}

      {showDropdown && (
        <div className="dropdown">
          {results.length === 0 ? (
            <p>No results found</p>
          ) : (
            results.map((item) => (
              <div key={item.id} className="dropdown-item">
                {item.name}
              </div>
            ))
          )}
        </div>
      )}
    </div>
  );
};
```

### Client-side Filtering (for small/local datasets)

```jsx
const LocalSearch = ({ data }) => {
  const [search, setSearch] = useState("");

  const filtered = data.filter((item) =>
    item.name.toLowerCase().includes(search.toLowerCase())
  );

  return (
    <div>
      <input
        value={search}
        onChange={(e) => setSearch(e.target.value)}
        placeholder="Filter..."
      />
      {filtered.map((item) => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
};
```

---

## 🔑 Key Concepts to Remember
| Concept | Detail |
|---|---|
| Debounce | `setTimeout` inside `useEffect`, clear with `clearTimeout` on cleanup |
| AbortController | Cancel in-flight requests when new search fires |
| Min threshold | Don't search until 2-3 chars typed |
| Empty state | Always handle "no results found" UI |
| Client vs API | Client-side `.filter()` for small data, API for large datasets |
