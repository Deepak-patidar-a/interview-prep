# Code Review — Spotting Problems in useEffect

## Question
A junior engineer on your team wrote this code. What's wrong with it and how would you fix it?

```javascript
useEffect(() => {
  fetch(`/api/products?search=${searchTerm}`)
    .then(res => res.json())
    .then(data => setProducts(data));
}, [searchTerm]);
```

---

## My Answer
1. Check if searchTerm is undefined, null or falsy — if so, return early and skip the fetch
2. Move fetch into a separate function instead of inline in useEffect
3. Wrap in try/catch — if fetch fails we can catch the error
4. Add loading state — while fetching, show a loader so user doesn't see a blank screen
5. Use AbortController for performance — cancel in-flight requests when searchTerm changes fast
6. Debounce — since this is a search query, debounce searchTerm to avoid fetching on every keystroke

## What I Got Right
- Found all the major issues — null check, error handling, loading state, AbortController, debounce
- AbortController is the one most candidates miss — got it
- Debounce for search is correct and connected to real Blue Yonder experience
- Clean code instinct — separate function for fetch logic

## What I Got Wrong
- Didn't explain WHY AbortController matters — the race condition problem
- Could mention React Query as the modern solution that handles all of this out of the box

---

## Model Answer

**Problems in the original code:**
1. **No null/empty check** — fetches even when searchTerm is empty, wasting API calls
2. **No error handling** — if fetch fails, it fails silently. No try/catch, no error state.
3. **No loading state** — user sees nothing while data loads
4. **Race condition** — if user types fast, older requests can resolve after newer ones, showing stale results
5. **No debounce** — fires a fetch on every single keystroke
6. **No cleanup** — when component unmounts or searchTerm changes, in-flight requests aren't cancelled

**Fixed version:**
```javascript
useEffect(() => {
  if (!searchTerm) return;

  const controller = new AbortController();
  setLoading(true);

  const fetchProducts = async () => {
    try {
      const res = await fetch(
        `/api/products?search=${searchTerm}`,
        { signal: controller.signal }
      );
      const data = await res.json();
      setProducts(data);
    } catch (err) {
      if (err.name !== 'AbortError') setError(err);
    } finally {
      setLoading(false);
    }
  };

  fetchProducts();
  return () => controller.abort(); // cancel on cleanup

}, [searchTerm]); // searchTerm should be debounced upstream
```

**Even better — use React Query:**
```javascript
const { data, isLoading, error } = useQuery({
  queryKey: ['products', debouncedSearchTerm],
  queryFn: () => fetch(`/api/products?search=${debouncedSearchTerm}`).then(r => r.json()),
  enabled: !!debouncedSearchTerm,
})
```
React Query handles caching, loading state, error state, and deduplication out of the box.

---

## Key Things to Remember

**The 6 problems checklist for useEffect + fetch:**
1. ✅ Guard clause for empty/null values
2. ✅ try/catch + error state
3. ✅ loading state
4. ✅ AbortController for race conditions
5. ✅ Debounce for search inputs
6. ✅ Cleanup function returning controller.abort()

**Race condition explained:**
- User types "mac" → request A starts
- User types "macbook" → request B starts
- Request B resolves first (faster server) → shows "macbook" results
- Request A resolves second (slower) → OVERWRITES with "mac" results
- User sees wrong data
- AbortController cancels request A when "macbook" search starts — problem solved

**AbortController pattern:**
```javascript
const controller = new AbortController()
fetch(url, { signal: controller.signal })
return () => controller.abort() // useEffect cleanup
```

**Debounce vs Throttle:**
- **Debounce** — wait until user stops typing for X ms, then fire. Best for search.
- **Throttle** — fire at most once every X ms regardless. Best for scroll events.

**Memory trick:** For any fetch in useEffect, ask ELEGANT — Empty check, Loading state, Error handling, Guard cleanup, AbortController, No-keystroke-spam (debounce).

---

## Things to Improve
- Memorize the AbortController pattern — write it from scratch without looking
- Practice explaining the race condition clearly in plain English
- Lead with React Query as the modern answer, then show the manual version as the "how it works under the hood" explanation
