# React Interview Prep: Parent-Child Communication (Cart Question)

## The Question

> Build a Shopping Cart counter system with Parent-Child communication.
>
> - **Parent (`App`)** holds the product list and a `cartItems` state (quantity per product).
> - **Child (`Product`)** receives `name`, `price`, `quantity` as props, and has Add/Remove buttons.
> - Child must **NOT** hold its own quantity state — clicking Add/Remove calls a function passed down from the parent.
> - "Remove" button is disabled when quantity is `0`.
> - Parent has a "Clear Cart" button that resets everything.

This tests: **lifting state up**, **props for data flow down**, **callback props for events flowing up**, and spotting **state mutation bugs**.

---

## Dummy Data

```javascript
const products = [
  { id: 1, name: "Wireless Mouse", price: 799 },
  { id: 2, name: "Mechanical Keyboard", price: 2499 },
  { id: 3, name: "USB-C Hub", price: 1299 },
  { id: 4, name: "Laptop Stand", price: 999 },
  { id: 5, name: "Noise Cancelling Headphones", price: 4999 },
];
```

Cart state shape (object keyed by id, not an array):
```javascript
const [cartItems, setCartItems] = useState({});
// e.g. { 1: 2, 3: 1 } → 2 mice, 1 USB-C hub
```

**Why an object and not an array?** O(1) lookup/update by id instead of `.find()`/`.map()` through an array every time.

---

## Final Correct Solution

```javascriptreact
import "./styles.css";
import { useState } from "react";

export const Product = ({ id, name, price, quantity, onAdd, onRemove }) => {
  return (
    <div>
      <span>
        {name} - ₹{price} (Qty: {quantity})
      </span>
      <button onClick={onAdd}>Add</button>
      <button disabled={quantity <= 0} onClick={onRemove}>
        Remove
      </button>
    </div>
  );
};

export default function App() {
  const [cartItems, setCartItems] = useState({});

  const products = [
    { id: 1, name: "Wireless Mouse", price: 799 },
    { id: 2, name: "Mechanical Keyboard", price: 2499 },
    { id: 3, name: "USB-C Hub", price: 1299 },
    { id: 4, name: "Laptop Stand", price: 999 },
    { id: 5, name: "Noise Cancelling Headphones", price: 4999 },
  ];

  const handleAdd = (id) => {
    setCartItems((prev) => ({
      ...prev,
      [id]: (prev[id] || 0) + 1,
    }));
  };

  const handleRemove = (id) => {
    setCartItems((prev) => ({
      ...prev,
      [id]: Math.max((prev[id] || 0) - 1, 0),
    }));
  };

  const handleClearCart = () => setCartItems({});

  const totalCount = Object.values(cartItems).reduce((sum, qty) => sum + qty, 0);
  const totalPrice = products.reduce(
    (sum, p) => sum + p.price * (cartItems[p.id] || 0),
    0
  );

  return (
    <div className="App">
      <h1>Parent-Child Cart</h1>
      <h2>
        Items: {totalCount} | Total: ₹{totalPrice}
      </h2>
      <button onClick={handleClearCart}>Clear Cart</button>

      {products.map((item) => (
        <Product
          key={item.id}
          id={item.id}
          name={item.name}
          price={item.price}
          quantity={cartItems[item.id] || 0}
          onAdd={() => handleAdd(item.id)}
          onRemove={() => handleRemove(item.id)}
        />
      ))}
    </div>
  );
}
```

---

## Mistakes I Made (and Why They're Wrong)

### 1. Mutating state directly instead of creating a new object
```javascript
// ❌ Wrong
const cartItem = props.cartItems;
cartItem[id] = (cartItem[id] || 0) + 1;
props.setCartItems(cartItem);
```
`cartItem` is the **same object reference** as the parent's state. Mutating it in place and passing that same reference back to `setCartItems` means React's `Object.is` comparison sees "no change" → **no re-render**, even though data changed underneath.

```javascript
// ✅ Correct — always spread into a new object
setCartItems((prev) => ({
  ...prev,
  [id]: (prev[id] || 0) + 1,
}));
```

**Remember:** React state must always be treated as immutable. Never mutate objects/arrays in place — always create a new reference (spread, `.map()`, `.filter()`, etc).

---

### 2. Passing the whole `cartItems` object instead of just that product's quantity
```javascript
// ❌ Wrong — child gets the entire state blob
<Product cartItems={cartItems} setCartItems={setCartItems} />
```
```javascript
// ✅ Correct — child only gets what it needs
<Product quantity={cartItems[item.id] || 0} onAdd={...} onRemove={...} />
```
**Remember:** Pass down the **minimum data and behavior** a child needs — not your entire state shape. The child shouldn't know or care how the parent's state is structured internally. This is the actual point of "props" as a contract.

---

### 3. `disable` is not a real attribute — it's `disabled`
```javascript
// ❌ Wrong (typo, and wrong logic too)
<button disable={cartItem?.length <= 0 ? true : false}>
```
```javascript
// ✅ Correct
<button disabled={quantity <= 0}>
```
Also: `cartItem` was an **object** (`{1: 2, 3: 1}`), and objects don't have `.length` — so the original check was always `undefined`, meaning the button was never actually disabled correctly.

**Remember:** `.length` only works on arrays/strings. For object value counts, use `Object.keys(obj).length` or `Object.values(obj)`.

---

### 4. `cartItems?.length` to show item count — wrong, it's an object
```javascript
// ❌ Wrong
<h2>{cartItems?.length > 0 ? cartItems.length : 0}</h2>
```
```javascript
// ✅ Correct — sum the quantities
const totalCount = Object.values(cartItems).reduce((sum, qty) => sum + qty, 0);
<h2>Items: {totalCount}</h2>
```

---

### 5. Missing `key` prop in `.map()`
```javascript
// ❌ Wrong — no key
{products.map((item) => <Product id={item.id} ... />)}
```
```javascript
// ✅ Correct
{products.map((item) => <Product key={item.id} id={item.id} ... />)}
```
**Remember:** `key` must be on the element returned directly from `.map()`, must be stable and unique (use `id`, never array `index` if list order can change).

---

## Core Concepts to Remember

| Concept | One-liner |
|---|---|
| **Lifting state up** | When two siblings (or a parent UI element) need the same data, move state to their closest common parent. |
| **Props down** | Data flows parent → child via props. Child should treat props as **read-only**. |
| **Callbacks up** | Child → parent communication happens by calling a **function passed as a prop** (e.g. `onAdd`), not by the child managing its own state. |
| **Immutability** | Never mutate state directly. Always return a new object/array from setters (`{...prev}`, `[...prev]`). |
| **Functional setState** | `setState(prev => ...)` is safer than reading the state variable directly, especially with rapid updates. |
| **Minimal prop surface** | Pass only what the child needs (`quantity`, `onAdd`, `onRemove`) — not the whole state object. |
| **`key` prop** | Needed on any element from `.map()`, must be a stable unique id. |

---

## Likely Interviewer Follow-Up Questions

1. **"Why didn't you mutate `cartItems` directly?"**
   → Explain reference equality: React compares old vs new state reference to decide if a re-render is needed. Mutating in place keeps the same reference, so React thinks nothing changed.

2. **"Why pass `quantity` instead of the whole `cartItems` object to `Product`?"**
   → Separation of concerns. The child shouldn't be coupled to the parent's internal state shape — it should only know about its own slice of data.

3. **"If you had 1000 products, would this be efficient? What would re-render unnecessarily?"**
   → Every `Product` re-renders whenever **any** item's quantity changes, because `cartItems` is a new object every time, and inline arrows (`() => handleAdd(item.id)`) create new function references each render — this breaks `React.memo` even if added.
   → Fix: wrap `Product` in `React.memo`, and wrap `handleAdd`/`handleRemove` in `useCallback`.

4. **"How would you persist the cart across page refreshes?"**
   → `useEffect` to write `cartItems` to `localStorage` on change, and read it back with `useState(() => JSON.parse(localStorage.getItem(...)) || {})` on mount.

5. **"What's the difference between controlled and uncontrolled components — does this relate here?"**
   → Yes — `Product`'s quantity is "controlled" by the parent, similar to how a controlled `<input>` has its value owned by React state rather than the DOM.

6. **"What if `onAdd` needed to pass more than just an id — how would you scale this pattern?"**
   → Pass an object payload (`onAdd({ id, name })`), or use `useReducer` in the parent if action types start growing (ADD, REMOVE, CLEAR, etc.) — a natural segue to discussing `useReducer` vs `useState` for complex state logic.

---

## Bonus: Next-Level Version (if asked to optimize)

```javascriptreact
import { useState, useCallback, memo } from "react";

const Product = memo(({ id, name, price, quantity, onAdd, onRemove }) => {
  console.log("Rendering:", name); // use this to prove memo is working
  return (
    <div>
      <span>{name} - ₹{price} (Qty: {quantity})</span>
      <button onClick={() => onAdd(id)}>Add</button>
      <button disabled={quantity <= 0} onClick={() => onRemove(id)}>
        Remove
      </button>
    </div>
  );
});

export default function App() {
  const [cartItems, setCartItems] = useState({});

  const handleAdd = useCallback((id) => {
    setCartItems((prev) => ({ ...prev, [id]: (prev[id] || 0) + 1 }));
  }, []);

  const handleRemove = useCallback((id) => {
    setCartItems((prev) => ({ ...prev, [id]: Math.max((prev[id] || 0) - 1, 0) }));
  }, []);

  // ...rest same, pass handleAdd/handleRemove directly (not wrapped in new arrows)
}
```
**Key change:** `onAdd={handleAdd}` (not `onAdd={() => handleAdd(item.id)}`) — pass the `id` as an argument inside `Product` itself (`onClick={() => onAdd(id)}`), so the function reference passed to `Product` stays stable across renders, letting `React.memo` actually skip re-renders for unaffected products.
