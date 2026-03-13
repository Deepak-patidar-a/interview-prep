# ⚛️ Circular Dependencies in useEffect

## What Is a Circular Dependency?

When two effects depend on each other — A updates B, B updates A — creating an infinite loop.

```javascript
// 💥 Infinite loop
const [a, setA] = useState(0);
const [b, setB] = useState(0);

useEffect(() => {
  setB(a + 1);   // when a changes → update b
}, [a]);

useEffect(() => {
  setA(b + 1);   // when b changes → update a → triggers first effect again 💥
}, [b]);
```

---

## How React Handles It

**React does NOT automatically detect or prevent circular dependencies.**

It will run infinitely until the browser crashes or runs out of memory. It is 100% the developer's responsibility to prevent them.

---

## Three Ways To Fix

### Fix 1 — Derive instead of sync (best solution)
```javascript
// If b is always a+1, don't store it in state
// ✅ Derive it — no useEffect needed at all
const [a, setA] = useState(0);
const b = a + 1;   // derived value — always in sync, no loop possible
```

### Fix 2 — Add a condition to break the cycle
```javascript
useEffect(() => {
  if (b !== a + 1) {   // only update if actually needed
    setB(a + 1);
  }
}, [a, b]);
```

### Fix 3 — useRef to track previous value
```javascript
const prevA = useRef(a);

useEffect(() => {
  if (prevA.current !== a) {   // only run if a actually changed
    setB(a + 1);
    prevA.current = a;          // update ref to latest value
  }
}, [a]);
```

---

## Interview Answer
> *"React doesn't handle circular dependencies automatically — it's the developer's responsibility to prevent them. The best solution is to not sync state that can be derived. If b is always calculated from a, derive it directly instead of storing it separately and creating a sync loop."*

---

## 🔑 Key Points To Remember
- React has NO automatic circular dependency detection
- Infinite loop → browser crash or max call stack exceeded
- Best fix = derive the value, don't store it in state
- If you must sync — use a condition or useRef to break the cycle
- ESLint react-hooks/exhaustive-deps helps catch some of these issues
