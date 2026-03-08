# 07 - Draggable List in React

## ❓ Interview Question
> "How would you implement a draggable list in React? Walk me through how you'd manage the drag state, how you'd update the order of items when a user drags and drops, and how you'd keep it performant."

---

## 🙋 My Answer (What I Said)
- Two states: `select` (left side) and `selected` (right side)
- Track which item is selected using mouse listener
- On drop, add to selected using `push()` or `shift()` and remove from source using `map()`
- All inside `useEffect` triggered when new DOM element comes

---

## ✅ What I Got Right
- Instinct to track dragged item in state
- Idea of removing item from source list after drop
- Two-list concept (transfer list) is valid

## ❌ What Was Wrong / Missing
- `array.push()` mutates state and returns length — not a new array. Use spread: `[...arr, item]`
- `array.shift()` removes first element — wrong usage
- `useEffect` is wrong here — drag logic belongs in event handlers not effects
- Missing: HTML5 Drag and Drop API (`draggable`, `onDragStart`, `onDragOver`, `onDrop`)
- Missing: `e.preventDefault()` on `onDragOver` — required to allow drop
- Missing: reordering within same list (index swapping)
- Missing: libraries like `react-beautiful-dnd` or `dnd-kit`
- Missing: performance with `useCallback` and `React.memo`

---

## 💡 Model Answer

### Approach 1: HTML5 Drag and Drop API (Native)

```jsx
import { useState, useCallback } from "react";

const DraggableList = () => {
  const [items, setItems] = useState([
    { id: 1, text: "Item 1" },
    { id: 2, text: "Item 2" },
    { id: 3, text: "Item 3" },
    { id: 4, text: "Item 4" },
  ]);

  const dragItem = useRef(null);     // index of item being dragged
  const dragOverItem = useRef(null); // index of item being dragged over

  const handleDragStart = (index) => {
    dragItem.current = index;
  };

  const handleDragOver = (e, index) => {
    e.preventDefault(); // REQUIRED — allows drop to happen
    dragOverItem.current = index;
  };

  const handleDrop = () => {
    const newItems = [...items];
    const draggedItem = newItems.splice(dragItem.current, 1)[0]; // remove from old position
    newItems.splice(dragOverItem.current, 0, draggedItem);        // insert at new position
    dragItem.current = null;
    dragOverItem.current = null;
    setItems(newItems);
  };

  return (
    <ul style={{ listStyle: "none", padding: 0 }}>
      {items.map((item, index) => (
        <li
          key={item.id}
          draggable
          onDragStart={() => handleDragStart(index)}
          onDragOver={(e) => handleDragOver(e, index)}
          onDrop={handleDrop}
          style={{
            padding: "10px",
            margin: "5px 0",
            background: "#f0f0f0",
            cursor: "grab",
            border: "1px solid #ccc",
            borderRadius: "4px",
          }}
        >
          {item.text}
        </li>
      ))}
    </ul>
  );
};

export default DraggableList;
```

### Approach 2: Transfer List (Two Lists)

```jsx
import { useState, useRef } from "react";

const TransferList = () => {
  const [left, setLeft] = useState(["Apple", "Banana", "Cherry"]);
  const [right, setRight] = useState(["Mango", "Grape"]);
  const draggedItem = useRef(null);
  const dragSource = useRef(null); // "left" or "right"

  const handleDragStart = (item, source) => {
    draggedItem.current = item;
    dragSource.current = source;
  };

  const handleDropOnRight = (e) => {
    e.preventDefault();
    if (dragSource.current === "left") {
      setLeft((prev) => prev.filter((i) => i !== draggedItem.current));
      setRight((prev) => [...prev, draggedItem.current]);
    }
  };

  const handleDropOnLeft = (e) => {
    e.preventDefault();
    if (dragSource.current === "right") {
      setRight((prev) => prev.filter((i) => i !== draggedItem.current));
      setLeft((prev) => [...prev, draggedItem.current]);
    }
  };

  const listStyle = {
    minHeight: "150px",
    width: "200px",
    border: "1px solid #ccc",
    padding: "10px",
    borderRadius: "4px",
  };

  return (
    <div style={{ display: "flex", gap: "20px" }}>
      <ul
        style={listStyle}
        onDragOver={(e) => e.preventDefault()}
        onDrop={handleDropOnLeft}
      >
        {left.map((item) => (
          <li
            key={item}
            draggable
            onDragStart={() => handleDragStart(item, "left")}
            style={{ cursor: "grab", padding: "5px" }}
          >
            {item}
          </li>
        ))}
      </ul>

      <ul
        style={listStyle}
        onDragOver={(e) => e.preventDefault()}
        onDrop={handleDropOnRight}
      >
        {right.map((item) => (
          <li
            key={item}
            draggable
            onDragStart={() => handleDragStart(item, "right")}
            style={{ cursor: "grab", padding: "5px" }}
          >
            {item}
          </li>
        ))}
      </ul>
    </div>
  );
};
```

### Approach 3: Using `dnd-kit` Library (Production Recommended)

```bash
npm install @dnd-kit/core @dnd-kit/sortable
```

```jsx
import {
  DndContext,
  closestCenter,
  PointerSensor,
  useSensor,
  useSensors,
} from "@dnd-kit/core";
import {
  SortableContext,
  verticalListSortingStrategy,
  useSortable,
  arrayMove,
} from "@dnd-kit/sortable";
import { CSS } from "@dnd-kit/utilities";
import { useState } from "react";

const SortableItem = ({ id }) => {
  const { attributes, listeners, setNodeRef, transform, transition } =
    useSortable({ id });

  const style = {
    transform: CSS.Transform.toString(transform),
    transition,
    padding: "10px",
    margin: "5px",
    background: "#f0f0f0",
    cursor: "grab",
  };

  return (
    <div ref={setNodeRef} style={style} {...attributes} {...listeners}>
      {id}
    </div>
  );
};

const SortableList = () => {
  const [items, setItems] = useState(["Item 1", "Item 2", "Item 3", "Item 4"]);
  const sensors = useSensors(useSensor(PointerSensor));

  const handleDragEnd = (event) => {
    const { active, over } = event;
    if (active.id !== over.id) {
      setItems((items) => {
        const oldIndex = items.indexOf(active.id);
        const newIndex = items.indexOf(over.id);
        return arrayMove(items, oldIndex, newIndex);
      });
    }
  };

  return (
    <DndContext
      sensors={sensors}
      collisionDetection={closestCenter}
      onDragEnd={handleDragEnd}
    >
      <SortableContext items={items} strategy={verticalListSortingStrategy}>
        {items.map((id) => (
          <SortableItem key={id} id={id} />
        ))}
      </SortableContext>
    </DndContext>
  );
};
```

---

## 🔑 Key Concepts to Remember
| Concept | Detail |
|---|---|
| `draggable` | HTML attribute to make element draggable |
| `onDragStart` | Store which item / index is being dragged |
| `onDragOver` | Must call `e.preventDefault()` to allow drop |
| `onDrop` | Reorder array using splice or arrayMove |
| State mutation | Never use `push/shift/splice` directly on state — use spread |
| `useRef` for drag | Store drag index in ref (not state) to avoid re-renders |
| Libraries | `dnd-kit` or `react-beautiful-dnd` for production |
| Performance | `useCallback` on handlers, `React.memo` on list items |
