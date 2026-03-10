# ⚛️ Todo List — Add, Delete, Update, Checkbox

## Requirements
- Add a new todo
- Delete a todo
- Edit/Update an existing todo
- Mark todo as complete via checkbox
- Count should never go below 0

---

## My Code
```jsx
import { useEffect, useState } from "react";

export default function App() {
  const [value, setValue] = useState("");
  const [todo, setTodo] = useState([]);
  const [update, setUpdate] = useState(null);
  const [read, setRead] = useState(false);

  const handleButton = () => {
    if (!value.trim()) return;
    if (update) {
      const updated = todo.filter((f) => f.id !== update);
      setTodo([...updated, { id: update, title: value, checked: false }]);
    } else {
      const addTodo = { id: Date.now(), title: value, checked: false };
      setTodo((prev) => [...prev, addTodo]);
    }
    setValue("");
  };

  const handleDelete = (id) => {
    const filtered = todo.filter((f) => f.id !== id);
    setTodo(filtered);
  };

  const handleUpdate = (id) => {
    setUpdate(id.id);
    setValue(id.title);
  };

  const handleCheckbox = (id) => {
    setTodo((prev) => {
      prev.map((c) => {
        c.id === id ? { ...c, c.checked: !checked } : { ...c };
      });
    });
  };

  return (
    <div>
      <h1>Todo List</h1>
      <input
        type="text"
        value={value}
        onChange={(e) => setValue(e.target.value)}
      />
      <button onClick={handleButton}>{update ? "Update" : "Add"}</button>
      {todo?.length > 0 &&
        todo.map((id) => (
          <div key={id.id}>
            <input
              type="checkbox"
              checked={read}
              onChange={() => handleCheckbox(id.id)}
            />
            {id.title}
            <button onClick={() => handleUpdate(id)}>Update</button>
            <button onClick={() => handleDelete(id.id)}>Delete</button>
          </div>
        ))}
    </div>
  );
}
```

---

## ✅ What I Did Well
- Add, Delete, Update all implemented — complete feature set
- `if (!value.trim()) return` — good guard, prevents empty todos
- `Date.now()` for unique id — correct approach
- Dynamic button text `update ? "Update" : "Add"` — clean
- Functional update `setTodo(prev => [...prev, addTodo])` — correct

---

## 🚨 Critical Bugs

**1. handleCheckbox — syntax error + wrong reference**
```javascript
// ❌ Three problems — wrong key syntax, wrong variable, missing return
const handleCheckbox = (id) => {
  setTodo((prev) => {
    prev.map((c) => {                          // missing return
      c.id === id
        ? { ...c, c.checked: !checked }        // wrong syntax + checked undefined
        : { ...c }                             // missing return
    })                                         // missing return of map result
  })
}

// ✅ Fixed — arrow function shorthand handles returns automatically
const handleCheckbox = (id) => {
  setTodo((prev) =>
    prev.map((c) =>
      c.id === id ? { ...c, checked: !c.checked } : c
    )
  );
};
```

**2. Checkbox tied to wrong state**
```jsx
// ❌ All checkboxes share one shared "read" state — all toggle together
const [read, setRead] = useState(false);
<input checked={read} ... />

// ✅ Each todo has its own checked field — use that directly
<input checked={item.checked} ... />

// Also remove this line — not needed at all
const [read, setRead] = useState(false);
```

---

## ⚠️ Two More Issues

**1. Update resets checked to false**
```javascript
// ❌ Loses the checked state when updating title
setTodo([...updated, { id: update, title: value, checked: false }]);

// ✅ Preserve existing checked value
const existing = todo.find((f) => f.id === update);
setTodo([...updated, { id: update, title: value, checked: existing.checked }]);
```

**2. update state not cleared after saving**
```javascript
// ❌ Button stays as "Update" forever after saving
setValue("");

// ✅ Reset update state too
setValue("");
setUpdate(null);
```

---

## ✅ Clean Final Version
```jsx
import { useState } from "react";

export default function App() {
  const [value, setValue] = useState("");
  const [todo, setTodo] = useState([]);
  const [update, setUpdate] = useState(null);

  const handleButton = () => {
    if (!value.trim()) return;

    if (update) {
      const existing = todo.find((f) => f.id === update);  // ✅ preserve checked
      const updated = todo.filter((f) => f.id !== update);
      setTodo([...updated, { id: update, title: value, checked: existing.checked }]);
    } else {
      setTodo((prev) => [...prev, { id: Date.now(), title: value, checked: false }]);
    }

    setValue("");
    setUpdate(null);                                        // ✅ reset update state
  };

  const handleDelete = (id) => {
    setTodo(todo.filter((f) => f.id !== id));
  };

  const handleUpdate = (item) => {
    setUpdate(item.id);
    setValue(item.title);
  };

  const handleCheckbox = (id) => {
    setTodo((prev) =>
      prev.map((c) =>
        c.id === id ? { ...c, checked: !c.checked } : c   // ✅ correct syntax
      )
    );
  };

  return (
    <div>
      <h1>Todo List</h1>
      <div>
        <input
          type="text"
          value={value}
          onChange={(e) => setValue(e.target.value)}
          placeholder="Enter todo..."
        />
        <button onClick={handleButton}>{update ? "Update" : "Add"}</button>
        {update && (                                        // ✅ cancel button
          <button onClick={() => { setUpdate(null); setValue(""); }}>
            Cancel
          </button>
        )}
      </div>

      {todo.length === 0 && <p>No todos yet. Add one!</p>}

      {todo.map((item) => (
        <div
          key={item.id}
          style={{ textDecoration: item.checked ? "line-through" : "none" }}
        >
          <input
            type="checkbox"
            checked={item.checked}                         // ✅ each todo's own state
            onChange={() => handleCheckbox(item.id)}
          />
          {item.title}
          <button onClick={() => handleUpdate(item)}>Edit</button>
          <button onClick={() => handleDelete(item.id)}>Delete</button>
        </div>
      ))}
    </div>
  );
}
```

---

## 🔑 Key Things To Remember

**Toggle a field inside array — memorize this pattern:**
```javascript
// Comes up in every interview
setState(prev =>
  prev.map(item =>
    item.id === id ? { ...item, field: !item.field } : item
  )
);
```

**Spread syntax for updating one field:**
```javascript
{ ...item, checked: !item.checked }  // ✅ correct
{ ...item, item.checked: !checked }  // ❌ wrong key syntax
```

- Always reset `update` state to `null` after saving
- Always add a Cancel button when in edit mode
- Each item manages its own state — never use shared state for per-item behavior
- `find()` to get existing item before update — preserves other fields

## Final Score: 7.5/10
