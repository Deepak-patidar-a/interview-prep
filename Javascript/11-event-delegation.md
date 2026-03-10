# 11 - Event Delegation, Bubbling & Capturing

## ❓ Interview Question
> "What is event delegation in JavaScript? Why is it useful, how does it work under the hood with event bubbling and capturing, and can you give a real world example of where you'd use it?"

---

## 🙋 My Answer (What I Said)
- Event delegation — handle events at parent instead of individual children
- Useful for dynamically generated lists
- Bubbling — event flows from inner child to parent
- Capturing — event flows from parent to child
- Capturing use case: logging and analytics

---

## ✅ What I Got Right
- Event delegation definition correct
- Dynamic list use case correct
- Bubbling direction correct
- Capturing direction correct
- Capturing use case for analytics — good thinking!

## ❌ What Was Missing
- e.target vs e.currentTarget
- stopPropagation
- e.preventDefault vs stopPropagation
- How to enable capturing (third argument)
- Full event flow order

---

## 💡 Model Answer

### Event Bubbling

```js
// Events bubble UP from child to parent
document.querySelector("#child").addEventListener("click", () => {
  console.log("child clicked");
});
document.querySelector("#parent").addEventListener("click", () => {
  console.log("parent clicked");
});

// Clicking child logs:
// "child clicked"
// "parent clicked"  ← bubbles up!
```

### Event Capturing

```js
// Events capture DOWN from parent to child
// Enable with third argument: true
document.querySelector("#parent").addEventListener("click", () => {
  console.log("parent (capturing)");
}, true); // ← capturing phase

document.querySelector("#child").addEventListener("click", () => {
  console.log("child");
});

// Clicking child logs:
// "parent (capturing)"  ← fires first in capturing
// "child"
```

### Full Event Flow

```
Capturing (top → down):  document → body → parent → child
Target:                   child (event fires here)
Bubbling (bottom → up):  child → parent → body → document
```

### Event Delegation

```js
// ❌ Without delegation — 1000 listeners for 1000 items
const items = document.querySelectorAll(".item");
items.forEach(item => {
  item.addEventListener("click", handleClick); // 1000 listeners!
});

// ✅ With delegation — 1 listener on parent
const list = document.querySelector("#list");
list.addEventListener("click", (e) => {
  if (e.target.matches(".item")) {
    handleClick(e.target);
  }
});
// Works for dynamically added items too!
```

### e.target vs e.currentTarget

```js
const parent = document.querySelector("#parent");

parent.addEventListener("click", (e) => {
  console.log(e.target);        // element ACTUALLY clicked
  console.log(e.currentTarget); // element listener is attached to (parent)
});

// If you click a child inside parent:
// e.target → the child element
// e.currentTarget → parent (where listener is)
```

### Real World — Dynamic List

```js
// Items added dynamically — delegation handles new items automatically
const todoList = document.querySelector("#todo-list");

// Add items dynamically
function addTodo(text) {
  const li = document.createElement("li");
  li.className = "todo-item";
  li.dataset.id = Date.now();
  li.innerHTML = `
    <span>${text}</span>
    <button class="delete-btn">Delete</button>
    <button class="complete-btn">Complete</button>
  `;
  todoList.appendChild(li);
}

// ONE listener handles ALL items — even newly added ones
todoList.addEventListener("click", (e) => {
  const item = e.target.closest(".todo-item");
  if (!item) return;

  if (e.target.matches(".delete-btn")) {
    item.remove();
  }

  if (e.target.matches(".complete-btn")) {
    item.classList.toggle("completed");
  }
});
```

### stopPropagation vs preventDefault

```js
// stopPropagation — stops event travelling up/down DOM
child.addEventListener("click", (e) => {
  e.stopPropagation(); // parent click handler won't fire
  console.log("child only");
});

// preventDefault — stops DEFAULT browser behavior
// (doesn't stop propagation!)
link.addEventListener("click", (e) => {
  e.preventDefault(); // stops navigation
  console.log("link clicked but not navigating");
});

form.addEventListener("submit", (e) => {
  e.preventDefault(); // stops page reload
  handleSubmit();
});

// stopImmediatePropagation — stops other listeners on SAME element too
button.addEventListener("click", (e) => {
  e.stopImmediatePropagation();
});
button.addEventListener("click", () => {
  console.log("won't fire"); // blocked!
});
```

---

## 🔑 Key Concepts to Remember
| Concept | Detail |
|---|---|
| Bubbling | Child → Parent (default, bottom up) |
| Capturing | Parent → Child (top down, enable with `true`) |
| Delegation | One listener on parent handles all children |
| `e.target` | Element that was actually clicked |
| `e.currentTarget` | Element the listener is attached to |
| `stopPropagation` | Stops event from bubbling/capturing further |
| `preventDefault` | Stops default browser behavior (reload, navigate) |
| Why delegation | Performance + works for dynamic elements |
