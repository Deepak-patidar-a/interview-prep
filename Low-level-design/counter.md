# ⚛️ Counter — Increment, Decrement, Reset

## Requirements
- Show count on screen
- 3 buttons — Increment, Decrement, Reset
- Count should never go below 0

---

## My Code
```jsx
import { useState } from "react";

export default function App() {
  const [count, setCount] = useState(0);

  const handleCounter = (val) => {
    if (val === "+") {
      setCount((count) => count + 1);
    } else if (val === "-") {
      setCount((count) => count - 1);
    } else {
      setCount(0);
    }
  };

  return (
    <div>
      <h1>Counter</h1>
      <div>{count}</div>
      <button onClick={() => handleCounter("+")}>Incremenet</button>
      <button onClick={() => handleCounter("-")} disabled={count < 1}>Decrement</button>
      <button onClick={() => handleCounter(0)}>Reset</button>
    </div>
  );
}
```

---

## ✅ What I Did Well
- Functional update pattern `setCount((count) => count + 1)` — correct, avoids stale state
- Edge case handled via `disabled={count < 1}` — prevents going below 0
- Single handler `handleCounter` instead of 3 separate functions — clean

---

## ⚠️ Issues Found

**1. Typo in button text**
```jsx
// ❌ Wrong
Incremenet

// ✅ Correct
Increment
```

**2. Reset passes number but other checks use strings — inconsistent**
```jsx
// ❌ Inconsistent — passing 0 (number) but checking "+" and "-" (strings)
<button onClick={() => handleCounter(0)}>Reset</button>

// ✅ Option 1 — keep strings consistent
<button onClick={() => handleCounter("reset")}>Reset</button>

// ✅ Option 2 — simplest, call setter directly
<button onClick={() => setCount(0)}>Reset</button>
```

**3. No guard inside decrement function**
```jsx
// ❌ disabled on button is UI only — function can still go negative if called directly
setCount((count) => count - 1);

// ✅ Add double protection inside logic
setCount((count) => Math.max(0, count - 1));
```

---

## ✅ Clean Final Version
```jsx
import { useState } from "react";

export default function App() {
  const [count, setCount] = useState(0);

  const handleCounter = (val) => {
    if (val === "+") {
      setCount((prev) => prev + 1);
    } else if (val === "-") {
      setCount((prev) => Math.max(0, prev - 1));  // double protection
    } else {
      setCount(0);
    }
  };

  return (
    <div>
      <h1>Counter</h1>
      <div>{count}</div>
      <button onClick={() => handleCounter("+")}>Increment</button>
      <button onClick={() => handleCounter("-")} disabled={count < 1}>Decrement</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

---

## 🔑 Key Things To Remember
- Always use functional update `setCount(prev => prev + 1)` — avoids stale state issues
- `disabled` on button = UI protection only. Add logic guard inside function too for full protection
- Mention in interview: *"I'm using functional update form because if multiple state updates happen in same render cycle, it always uses the latest state value"*

## Final Score: 8/10
