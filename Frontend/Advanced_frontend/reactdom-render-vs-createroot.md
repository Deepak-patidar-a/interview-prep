# ⚛️ ReactDOM.render() vs createRoot() — React 18

## The Difference

```javascript
// Old way — React 17 and below
import ReactDOM from 'react-dom';
ReactDOM.render(<App />, document.getElementById('root'));

// New way — React 18
import { createRoot } from 'react-dom/client';
const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

---

## Why React 18 Introduced createRoot()

| Feature | ReactDOM.render | createRoot |
|---------|----------------|------------|
| Concurrent Mode | ❌ No | ✅ Yes |
| Automatic Batching | ❌ Partial | ✅ Full |
| useTransition | ❌ No | ✅ Yes |
| useDeferredValue | ❌ No | ✅ Yes |
| Suspense (full) | ❌ Limited | ✅ Yes |
| Streaming SSR | ❌ No | ✅ Yes |

---

## Automatic Batching — Biggest Practical Difference

```javascript
// Old React — ReactDOM.render — 2 separate re-renders
setTimeout(() => {
  setCount(c => c + 1);   // re-render 1
  setName("Deepak");      // re-render 2
}, 1000);

// React 18 — createRoot — batched into 1 re-render
setTimeout(() => {
  setCount(c => c + 1);   // \
  setName("Deepak");      //  → batched → only 1 re-render ✅
}, 1000);
```

Old React only batched inside React event handlers.
React 18 batches **everywhere** — setTimeout, Promises, native events.

---

## Concurrent Mode — What It Unlocks

```javascript
// useTransition — mark update as non-urgent
const [isPending, startTransition] = useTransition();

startTransition(() => {
  setSearchResults(results);  // non-urgent — React can pause this
});

// useDeferredValue — defer expensive renders
const deferredQuery = useDeferredValue(query);
// deferredQuery updates after urgent updates are done
```

---

## Interview Answer
> *"createRoot is the new React 18 way to mount your app. The key difference is it unlocks Concurrent Mode — which enables automatic batching of all state updates, useTransition for non-urgent updates, and full Suspense support. With ReactDOM.render you're locked into legacy mode and can't use any React 18 concurrent features."*

---

## 🔑 Key Points To Remember
- `createRoot` = required for React 18 concurrent features
- `ReactDOM.render` = legacy mode — still works but deprecated
- Automatic batching = multiple setState calls → only 1 re-render
- createRoot enables: useTransition, useDeferredValue, Suspense streaming
- Always use `createRoot` in new React 18 projects
