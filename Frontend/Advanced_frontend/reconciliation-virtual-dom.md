# ⚛️ React Reconciliation + Virtual DOM Diffing

## What is Virtual DOM?

```
Real DOM updates are SLOW:
  Browser recalculates layout → repaints → slow 😤

React's solution — Virtual DOM:
  Keep a lightweight JS copy of DOM in memory
  State changes → create new Virtual DOM
  Compare old vs new (diffing)
  Only update parts that actually changed in real DOM ✅
```

---

## Reconciliation — How React Diffs

React follows 3 rules when comparing old vs new Virtual DOM:

### Rule 1 — Different element type = rebuild entire subtree
```jsx
// Old
<div>
  <Counter />
</div>

// New — div changed to span
<span>
  <Counter />
</span>

// React destroys the div and ALL its children
// Creates span and Counter fresh from scratch
// Counter loses all its state
```

### Rule 2 — Same element type = update only changed attributes
```jsx
// Old
<div className="old" style={{ color: 'red' }}>Hello</div>

// New
<div className="new" style={{ color: 'red' }}>Hello</div>

// React keeps the div — only updates className
// style stays the same — not touched
```

### Rule 3 — Keys help identify list items
```jsx
// ❌ No key — React re-renders entire list on any change
{items.map(item => <Item item={item} />)}

// ✅ With key — React knows exactly which item changed
{items.map(item => <Item key={item.id} item={item} />)}
```

---

## Why Keys Matter — Visual Example

```
List: [A, B, C]
Insert D at beginning: [D, A, B, C]

Without key:
  React compares by position
  Position 1: A → D (different) → re-render
  Position 2: B → A (different) → re-render
  Position 3: C → B (different) → re-render
  Position 4: nothing → C (new)  → create
  Result: 4 operations 😤

With key:
  React compares by key
  key=D: new → create
  key=A: same → keep
  key=B: same → keep
  key=C: same → keep
  Result: 1 operation ✅
```

---

## Never Use Index as Key
```jsx
// ❌ Wrong — if list order changes, keys change → wrong components update
{items.map((item, index) => <Item key={index} item={item} />)}

// ✅ Right — stable unique id
{items.map(item => <Item key={item.id} item={item} />)}
```

---

## Full Flow Summary
```
setState called
  ↓
React creates new Virtual DOM
  ↓
Diffing algorithm compares old vs new Virtual DOM
  ↓
React calculates minimum set of changes (patch)
  ↓
Only those changes applied to real DOM
  ↓
Browser repaints only changed parts
```

---

## 🔑 Key Points To Remember
- Virtual DOM = lightweight JS copy of real DOM — fast to update
- Reconciliation = process of comparing old and new Virtual DOM
- Different element type → full subtree rebuild (expensive)
- Same element type → only attribute updates (cheap)
- Keys = help React identify list items without full re-render
- Never use array index as key for dynamic/reorderable lists
