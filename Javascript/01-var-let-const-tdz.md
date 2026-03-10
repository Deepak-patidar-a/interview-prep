# 01 - var, let, const & Temporal Dead Zone

## ❓ Interview Question
> "Can you explain the difference between `var`, `let`, and `const`? And more specifically — what is the Temporal Dead Zone, and how does it behave differently across these three?"

---

## 🙋 My Answer (What I Said)
- `var` is function scoped, can be changed
- `let` is block scoped, can be changed
- `const` is block scoped, cannot be reassigned unless it's an object
- TDZ is the zone between declaration and initialization of `let`/`const`
- Accessing `let`/`const` before declaration throws ReferenceError
- `var` gives `undefined` before declaration

---

## ✅ What I Got Right
- `var` function scoped, `let`/`const` block scoped
- `const` object can be mutated
- TDZ explanation correct
- `var` gives `undefined`, `let`/`const` give ReferenceError

## ❌ What Was Missing
- `var` leaks out of blocks like `if/for`
- `let`/`const` ARE hoisted but not initialized (creates TDZ)
- `Object.freeze()` for true const immutability

---

## 💡 Model Answer

### Scoping Differences

```js
// var — function scoped, leaks out of blocks
function example() {
  if (true) {
    var x = 10;
  }
  console.log(x); // 10 — leaks out of if block!
}

// let — block scoped
function example2() {
  if (true) {
    let y = 10;
  }
  console.log(y); // ReferenceError — block scoped
}

// Classic var loop problem
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 1000); // 3, 3, 3
}

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 1000); // 0, 1, 2
}
```

### Hoisting Behavior

```js
// var — hoisted AND initialized as undefined
console.log(a); // undefined
var a = 5;

// let/const — hoisted but NOT initialized (TDZ!)
console.log(b); // ReferenceError: Cannot access 'b' before initialization
let b = 10;
```

### Temporal Dead Zone (TDZ)

```js
// TDZ = zone between start of block and variable declaration
{
  // TDZ for 'name' starts here ↓
  console.log(name); // ReferenceError — in TDZ!
  let name = "Deepak"; // TDZ ends here
  console.log(name); // "Deepak"
}
```

### const with Objects

```js
// const prevents reassignment, not mutation
const obj = { name: "Deepak" };
obj.name = "John";   // ✅ allowed — mutation
obj = {};            // ❌ TypeError — reassignment blocked

// True immutability with Object.freeze()
const frozen = Object.freeze({ name: "Deepak" });
frozen.name = "John"; // ❌ silently fails (or throws in strict mode)
```

---

## 🔑 Key Concepts to Remember
| Feature | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Function | Block | Block |
| Hoisted | ✅ (undefined) | ✅ (TDZ) | ✅ (TDZ) |
| Reassign | ✅ | ✅ | ❌ |
| Redeclare | ✅ | ❌ | ❌ |
| Block leak | ✅ leaks | ❌ | ❌ |
