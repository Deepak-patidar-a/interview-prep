# 05 - Custom useFetch Hook in React

## ❓ Interview Question
> "How would you implement a custom `useFetch` hook in React? Walk me through the design — what states it should manage, how it should handle loading, success and error scenarios, and what would make it truly reusable across multiple components."

---

## 🙋 My Answer (What I Said)
- Hook signature: `useFetch(url, options)`
- Validate URL — return error if not provided
- Use AbortController to cancel duplicate/stale requests
- Set `loading = true` and `error = null` before fetch
- Use `try/catch/finally` — fetch in try, handle errors in catch, set loading false in finally
- Check `!response.ok` and throw HTTP error
- In catch, ignore AbortError — only set error for real errors
- Return cleanup using `abortController.abort()`
- Return `data` at the end

---

## ✅ What I Got Right
- Reusability motivation
- Correct hook signature
- URL validation
- AbortController usage
- Setting loading/error before fetch
- try/catch/finally structure
- `!response.ok` check
- Ignoring AbortError in catch
- Loading false in finally
- Cleanup with abort

## ❌ What Was Wrong / Missing
- Didn't explicitly wrap fetch inside `useEffect`
- Return statement should be `{ data, loading, error }` — not just `data`
- Missing: initial state declarations
- Missing: `refetch` function for manual re-triggering
- Missing: `url` as `useEffect` dependency so hook re-runs when URL changes

---

## 💡 Model Answer

### Complete useFetch Hook

```jsx
import { useState, useEffect, useCallback } from "react";

const useFetch = (url, options = {}) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchData = useCallback(async () => {
    // URL validation
    if (!url) {
      setError("Please provide a valid URL");
      return;
    }

    const controller = new AbortController();

    setLoading(true);
    setError(null);

    try {
      const response = await fetch(url, {
        ...options,
        signal: controller.signal,
      });

      if (!response.ok) {
        throw new Error(`HTTP Error: ${response.status}`);
      }

      const result = await response.json();
      setData(result);
    } catch (err) {
      if (err.name !== "AbortError") {
        setError(err.message);
      }
    } finally {
      setLoading(false);
    }

    return () => controller.abort(); // cleanup
  }, [url]);

  useEffect(() => {
    const cleanup = fetchData();
    return () => cleanup?.(); // run cleanup on unmount or url change
  }, [fetchData]);

  return { data, loading, error, refetch: fetchData };
};

export default useFetch;
```

### Usage in a Component

```jsx
const UserProfile = ({ userId }) => {
  const { data, loading, error, refetch } = useFetch(
    `https://api.example.com/users/${userId}`
  );

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;
  if (!data) return null;

  return (
    <div>
      <h2>{data.name}</h2>
      <p>{data.email}</p>
      <button onClick={refetch}>Refresh</button>
    </div>
  );
};
```

### Advanced Version with POST support

```jsx
const useFetch = (url, options = {}) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const execute = useCallback(
    async (body = null) => {
      if (!url) return;

      const controller = new AbortController();
      setLoading(true);
      setError(null);

      try {
        const config = {
          ...options,
          signal: controller.signal,
          ...(body && {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify(body),
          }),
        };

        const res = await fetch(url, config);
        if (!res.ok) throw new Error(`HTTP Error: ${res.status}`);

        const result = await res.json();
        setData(result);
        return result;
      } catch (err) {
        if (err.name !== "AbortError") setError(err.message);
      } finally {
        setLoading(false);
      }
    },
    [url]
  );

  useEffect(() => {
    execute();
  }, [execute]);

  return { data, loading, error, refetch: execute };
};
```

---

## 🔑 Key Concepts to Remember
| Concept | Detail |
|---|---|
| Initial states | `data: null`, `loading: false`, `error: null` |
| useEffect | Wrap fetch call inside useEffect, add `url` as dependency |
| AbortController | Cancel stale requests on URL change or unmount |
| `!response.ok` | fetch doesn't throw on 4xx/5xx — must check manually |
| AbortError | Skip setting error for intentional cancellations |
| finally | Always runs — perfect place to set `loading: false` |
| Return | Always return `{ data, loading, error, refetch }` |
| refetch | Expose manual trigger for refresh functionality |
