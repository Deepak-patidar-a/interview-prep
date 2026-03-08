# 01 - Infinite Scrolling in React

## ❓ Interview Question
> "How would you implement infinite scrolling in React? Walk me through your approach — how would you detect when the user reaches the bottom of the page, how would you load more data, and how would you handle performance concerns like excessive API calls?"

---

## 🙋 My Answer (What I Said)
- Use `window.innerHeight` to detect page bottom
- Maintain a `page` state and increment it (`page + 1`)
- Use `useEffect` with `page` as dependency to trigger fetch with page query param
- Use throttling with `setInterval` to prevent excessive API calls

---

## ✅ What I Got Right
- Tracking a `page` state and incrementing it
- Using `useEffect` with `page` as dependency to trigger fetch
- Addressing performance concern with throttling concept

## ❌ What Was Wrong / Missing
- `window.innerHeight` alone doesn't detect bottom — need full formula
- Throttling with `setInterval` is incorrect — `setInterval` runs repeatedly regardless of user action
- Missing: where to attach scroll listener and cleanup
- Missing: `IntersectionObserver` (modern preferred approach)
- Missing: loading state and `hasMore` flag to prevent duplicate fetches

---

## 💡 Model Answer

### Approach 1: Scroll Event Listener

```jsx
import { useState, useEffect } from "react";

const useInfiniteScroll = (fetchMore) => {
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);

  const handleScroll = () => {
    const isBottom =
      window.innerHeight + window.scrollY >= document.body.offsetHeight - 100;

    if (isBottom && !loading && hasMore) {
      setPage((prev) => prev + 1);
    }
  };

  // Throttle helper
  const throttle = (fn, limit) => {
    let lastCall = 0;
    return (...args) => {
      const now = Date.now();
      if (now - lastCall >= limit) {
        lastCall = now;
        fn(...args);
      }
    };
  };

  useEffect(() => {
    const throttledScroll = throttle(handleScroll, 300);
    window.addEventListener("scroll", throttledScroll);
    return () => window.removeEventListener("scroll", throttledScroll); // cleanup
  }, [loading, hasMore]);

  useEffect(() => {
    const load = async () => {
      setLoading(true);
      const newData = await fetchMore(page);
      if (newData.length === 0) setHasMore(false);
      setLoading(false);
    };
    load();
  }, [page]);
};
```

### Approach 2: IntersectionObserver (Preferred ✅)

```jsx
import { useState, useEffect, useRef } from "react";

const InfiniteList = () => {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const [hasMore, setHasMore] = useState(true);
  const [loading, setLoading] = useState(false);
  const sentinelRef = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting && hasMore && !loading) {
          setPage((prev) => prev + 1);
        }
      },
      { threshold: 1.0 }
    );

    if (sentinelRef.current) observer.observe(sentinelRef.current);
    return () => observer.disconnect();
  }, [hasMore, loading]);

  useEffect(() => {
    const fetchItems = async () => {
      setLoading(true);
      const res = await fetch(`/api/items?page=${page}`);
      const data = await res.json();
      if (data.length === 0) setHasMore(false);
      setItems((prev) => [...prev, ...data]);
      setLoading(false);
    };
    fetchItems();
  }, [page]);

  return (
    <div>
      {items.map((item) => (
        <div key={item.id}>{item.name}</div>
      ))}
      {loading && <p>Loading...</p>}
      <div ref={sentinelRef} /> {/* invisible trigger element */}
    </div>
  );
};
```

---

## 🔑 Key Concepts to Remember
| Concept | Detail |
|---|---|
| Bottom detection | `window.innerHeight + window.scrollY >= document.body.offsetHeight` |
| Throttling | Limit how often scroll handler fires using timestamp diff |
| IntersectionObserver | Observe a sentinel div at bottom — modern & performant |
| hasMore flag | Prevent fetching when no more data exists |
| Cleanup | Always remove scroll listener or disconnect observer on unmount |
