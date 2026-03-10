# 04 - `this` Keyword + call, apply, bind

## ❓ Interview Question
> "Explain `this` keyword in JavaScript. How does its value change across regular functions, arrow functions, object methods, and event handlers? And how do `call`, `apply`, and `bind` relate to it?"

---

## 🙋 My Answer (What I Said)
- `this` refers to an object depending on where/how it's used
- Regular function called independently → undefined (got strict mode backwards)
- Arrow functions inherit `this` from lexical scope
- Object methods → refers to the object
- Event handlers → work like normal functions
- `call`/`apply` same but call takes individual params, apply takes array
- `bind` returns new function, doesn't execute immediately

---

## ✅ What I Got Right
- `this` depends on how/where it's called
- Arrow functions don't have own `this`
- Object methods `this` refers to the object
- `call` vs `apply` params difference correct
- `bind` returns new function — correct

## ❌ What Was Missing
- Strict mode behavior was swapped
- Event handlers — `this` refers to DOM element, not like normal function
- `new` keyword context
- Losing `this` context problem
- Method borrowing with call/apply

---

## 💡 Model Answer

### 1. Regular Function

```js
// Non-strict mode → this = window (global)
function test() {
  console.log(this); // Window object
}
test();

// Strict mode → this = undefined
"use strict";
function test() {
  console.log(this); // undefined
}
test();
```

### 2. Arrow Function

```js
// Arrow functions inherit this from lexical scope
const obj = {
  name: "Deepak",
  regular: function() {
    console.log(this.name); // "Deepak" — this = obj
  },
  arrow: () => {
    console.log(this.name); // undefined — this = window
  }
};

// Common React pattern
class Timer {
  constructor() {
    this.seconds = 0;
  }
  start() {
    // Arrow function inherits this from start()
    setInterval(() => {
      this.seconds++; // ✅ works because arrow function
    }, 1000);
  }
}
```

### 3. Object Methods

```js
const person = {
  name: "Deepak",
  greet() {
    console.log(`Hi, I'm ${this.name}`); // this = person
  }
};
person.greet(); // "Hi, I'm Deepak"

// Losing context!
const fn = person.greet;
fn(); // "Hi, I'm undefined" — lost this!
```

### 4. Event Handlers

```js
const button = document.querySelector("button");

// Regular function — this = DOM element
button.addEventListener("click", function() {
  console.log(this); // <button> element
  this.style.color = "red"; // works!
});

// Arrow function — this = window (lexical)
button.addEventListener("click", () => {
  console.log(this); // Window — not the button!
});
```

### 5. `new` Keyword

```js
function Person(name) {
  this.name = name; // this = new object being created
  this.greet = function() {
    return `Hi, I'm ${this.name}`;
  };
}

const p = new Person("Deepak");
console.log(p.name);    // "Deepak"
console.log(p.greet()); // "Hi, I'm Deepak"
```

### call, apply, bind

```js
const person1 = { name: "Deepak" };
const person2 = { name: "John" };

function greet(city, country) {
  return `Hi, I'm ${this.name} from ${city}, ${country}`;
}

// call — args as individual values, executes immediately
greet.call(person1, "Indore", "India");
// "Hi, I'm Deepak from Indore, India"

// apply — args as array, executes immediately
greet.apply(person2, ["Mumbai", "India"]);
// "Hi, I'm John from Mumbai, India"

// bind — returns NEW function, execute later
const greetDeeaak = greet.bind(person1, "Indore");
greetDeeaak("India"); // "Hi, I'm Deepak from Indore, India"
```

### Fixing Lost Context with bind

```js
const obj = {
  name: "Deepak",
  greet() {
    console.log(this.name);
  }
};

// ❌ Lost context
const fn = obj.greet;
fn(); // undefined

// ✅ Fix with bind
const fixed = obj.greet.bind(obj);
fixed(); // "Deepak"

// ✅ Fix with arrow in class
class MyClass {
  constructor() {
    this.name = "Deepak";
    this.greet = this.greet.bind(this); // bind in constructor
  }
  greet() {
    console.log(this.name);
  }
}
```

---

## 🔑 Key Concepts to Remember
| Context | `this` value |
|---|---|
| Regular function (non-strict) | `window` |
| Regular function (strict) | `undefined` |
| Arrow function | Inherited from lexical scope |
| Object method | The object itself |
| Event handler (regular) | DOM element |
| Event handler (arrow) | `window` |
| `new` keyword | New instance being created |
| `call/apply` | Whatever you pass as first arg |
| `bind` | Returns new function with fixed `this` |
