# 03 - Dynamic Form Fields in React

## ❓ Interview Question
> "How would you design a form with dynamic fields in React? For example, a form where users can add or remove fields — walk me through how you'd structure the state, handle validation, and your thoughts on controlled vs uncontrolled components."

---

## 🙋 My Answer (What I Said)
- Explained controlled (useState + value) vs uncontrolled (useRef + DOM)
- Store fields as array of objects with `field_name`, `type`, `disable`, `value`, `onChange`
- Loop over fields array to render inputs
- Validation based on field types
- Map over fields in return

---

## ✅ What I Got Right
- Correct explanation of controlled vs uncontrolled components
- `useRef` for uncontrolled is correct
- Right instinct to use array of objects for fields
- Mapping over fields array to render inputs
- Mentioning type-based validation

## ❌ What Was Wrong / Missing
- Storing `onChange` functions inside state — this is an anti-pattern, functions should be derived in the map
- State structure was confused — mixing concerns
- Missing: how to **remove** a field (core part of question!)
- Missing: validation per field with error messages
- Missing: unique `key` prop when mapping
- Missing: form submission — collecting all values

---

## 💡 Model Answer

### Dynamic Fields with Add & Remove

```jsx
import { useState } from "react";

const DynamicForm = () => {
  const [fields, setFields] = useState([
    { id: Date.now(), value: "", error: "" },
  ]);

  // Add a new field
  const addField = () => {
    setFields([...fields, { id: Date.now(), value: "", error: "" }]);
  };

  // Remove a field by id
  const removeField = (id) => {
    setFields(fields.filter((field) => field.id !== id));
  };

  // Update field value
  const handleChange = (id, value) => {
    setFields(
      fields.map((field) =>
        field.id === id ? { ...field, value, error: "" } : field
      )
    );
  };

  // Validate all fields
  const validate = () => {
    let isValid = true;
    const updated = fields.map((field) => {
      if (!field.value.trim()) {
        isValid = false;
        return { ...field, error: "This field is required" };
      }
      return field;
    });
    setFields(updated);
    return isValid;
  };

  // Submit handler
  const handleSubmit = () => {
    if (!validate()) return;
    const values = fields.map((f) => f.value);
    console.log("Submitted values:", values);
    // call API here
  };

  return (
    <div>
      {fields.map((field) => (
        <div key={field.id} style={{ marginBottom: "10px" }}>
          <input
            value={field.value}
            onChange={(e) => handleChange(field.id, e.target.value)}
            placeholder="Enter value"
          />
          <button onClick={() => removeField(field.id)}>Remove</button>
          {field.error && <p style={{ color: "red" }}>{field.error}</p>}
        </div>
      ))}

      <button onClick={addField}>+ Add Field</button>
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
};

export default DynamicForm;
```

### Controlled vs Uncontrolled

```jsx
// ✅ Controlled — React controls the value
const Controlled = () => {
  const [value, setValue] = useState("");
  return <input value={value} onChange={(e) => setValue(e.target.value)} />;
};

// Uncontrolled — DOM controls the value, React reads via ref
const Uncontrolled = () => {
  const inputRef = useRef(null);
  const handleSubmit = () => console.log(inputRef.current.value);
  return <input ref={inputRef} />;
};
```

---

## 🔑 Key Concepts to Remember
| Concept | Detail |
|---|---|
| Add field | `setFields([...fields, { id: Date.now(), value: '', error: '' }])` |
| Remove field | `setFields(fields.filter(f => f.id !== id))` |
| Update field | `fields.map(f => f.id === id ? {...f, value} : f)` |
| key prop | Always use unique `key` when mapping — use `id`, not index |
| onChange in state | ❌ Anti-pattern — derive onChange in map, don't store in state |
| Controlled | React owns value via useState |
| Uncontrolled | DOM owns value, read via useRef |
