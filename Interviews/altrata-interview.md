# ⚛️ React Interview Revision — Altrata Round

---

## Q1. What is `useState` and Props?

### Your Answer
- useState = internal data, component level only
- Props = external data, passed from parent

### ✅ Model Answer
**useState** is a React hook that stores internal/local state of a component. Only that component owns and updates it using the setter function.

**Props** are external data passed from a parent component to a child. They are **read-only** — the child cannot modify them directly.

```jsx
// useState - internal
const [count, setCount] = useState(0);

// Props - external, read-only in child
function Child({ name }) {
  return <h1>{name}</h1>;
}
```

> 💡 Key addition: Always mention **props are read-only** from the child's perspective.

---

## Q2. What is Derived State?

### Your Answer
- Derived state is calculated from other state and props

### ✅ Model Answer
Derived state is a value **computed from existing state or props** rather than stored separately in `useState`. If a value can be calculated, don't store it — derive it.

```jsx
const [firstName, setFirstName] = useState("Deepak");
const [lastName, setLastName] = useState("Patidar");

// Derived state — no need for separate useState
const fullName = `${firstName} ${lastName}`;
```

> 💡 Rule of thumb: If you can calculate it, don't store it in state.

---

## Q3. What Causes a Component to Re-render?

### Your Answer
- State change, event fire, useEffect, useState

### ✅ Model Answer (All 4 causes)
A React component re-renders when:

1. **State changes** — `useState` setter is called
2. **Props change** — parent passes new/different prop values
3. **Parent re-renders** — child re-renders when parent does (even if props didn't change)
4. **Context changes** — if component consumes a context that updated

```jsx
// All 4 triggers
const [count, setCount] = useState(0);     // 1. state
<Child name={count} />                      // 2. props
// Parent re-renders → Child re-renders    // 3. parent
const value = useContext(MyContext);        // 4. context
```

> 💡 Mentioning all 4 causes shows strong understanding.

---

## Q4. Real Project Challenge in React?

### Your Answer
- Rendering 10,000 products was sluggish → moved to pagination / virtualization

### ✅ Model Answer
*"In our current project, the landing page was rendering 10,000+ products at once which caused sluggish performance and high memory usage. We solved this by implementing **pagination** to limit items per page, and also explored **virtualization** using libraries like `react-window` which only renders items visible in the viewport."*

> 💡 Name-drop libraries: **react-window** or **react-virtual** for virtualization.

---

## Q5. Toggle Feature — Code Walkthrough

### Code Given
```js
const arr = [
  { id: 1, name: "deepak", enabled: true },
  { id: 2, name: "patidar", enabled: false },
  { id: 3, name: "uma", enabled: true },
];

const [isToggled, setIsToggled] = useState(arr);

const toggle = (id) => {
  setIsToggled(prev =>
    prev.map(key =>
      key.id === id ? { ...key, enabled: !key.enabled } : key
    )
  );
};
```

### ✅ Model Answer

**What does it do?**
It toggles the `enabled` field of an object in the array whose `id` matches the passed `id`.

**Why use `map`?**
`map` returns a **new array** without mutating the original. This is critical in React — we never mutate state directly. (`forEach` mutates in place and doesn't return a new array.)

**What is `...` (spread operator)?**
It copies all existing properties of the object. So `{ ...key, enabled: !key.enabled }` keeps `id` and `name` the same, and only overrides `enabled`.

**Why not just write `{ enabled: !key.enabled }`?**
Because that would create an object with *only* `enabled` — losing `id` and `name` entirely.

> ⚠️ Your mistake: You said "map iterates and changes the same array" — **map always returns a NEW array**. Remember this.

---

## Q6. Custom Hooks — Rules

### Your Answer
- Use "use" prefix
- Don't use inside useEffect, loops, or conditions
- Call at top of component

### ✅ Model Answer

**Two official rules of hooks:**

1. **Only call hooks at the top level** — never inside loops, conditions, or nested functions
2. **Only call hooks from React functions** — function components or other custom hooks (not regular JS functions)

**Rules for creating a custom hook:**
- Name must start with `use` (e.g., `useFetch`, `useDebounce`)
- Can use other hooks inside it
- Should return values the component needs

```jsx
// Custom hook example
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(url).then(res => res.json()).then(data => {
      setData(data);
      setLoading(false);
    });
  }, [url]);

  return { data, loading };
}
```

> 💡 When consumed: always call at the top of the component, not inside conditions or loops.

---

## 🔑 Quick Cheat Sheet

| Topic | Key Point to Remember |
|---|---|
| useState vs Props | Props are **read-only** in child |
| Derived State | If you can calculate it, **don't store it** |
| Re-render causes | State, Props, Parent re-render, Context |
| map vs forEach | `map` returns **new array**, forEach doesn't |
| Spread operator | Copies all props, then **override specific ones** |
| Hook rules | Top level only + React functions only |
| Virtualization | Use **react-window** library name |
