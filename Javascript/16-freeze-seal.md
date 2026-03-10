# 16 - Object.freeze() vs Object.seal()

## ❓ Interview Question
> "What is the difference between Object.freeze() and Object.seal() in JavaScript? How do they differ, and how do they relate to const? Also — how would you make a truly deeply immutable object?"

---

## 🙋 My Answer (What I Said)
- seal — cannot add/delete properties, CAN modify existing
- freeze — cannot add, modify, or delete
- const object is mutable
- Object.freeze() makes truly deeply immutable — WRONG (only shallow)

---

## ✅ What I Got Right
- seal behavior correct
- freeze behavior correct
- const vs freeze relationship — correct

## ❌ What Was Missing
- Object.freeze is SHALLOW — nested objects NOT frozen
- Deep freeze requires recursive approach
- structuredClone for immutable copies
- Object.isFrozen / Object.isSealed

---

## 💡 Model Answer

### const vs seal vs freeze

```js
// const — prevents REASSIGNMENT, object is still mutable
const obj = { name: "Deepak", age: 25 };
obj.name = "John";  // ✅ mutation works
obj.city = "Indore"; // ✅ add property works
delete obj.age;      // ✅ delete works
obj = {};            // ❌ TypeError — reassignment blocked

// Object.seal() — no add/delete, CAN modify
const sealed = Object.seal({ name: "Deepak", age: 25 });
sealed.name = "John";   // ✅ modify existing — works
sealed.city = "Indore"; // ❌ silently fails — can't add
delete sealed.age;      // ❌ silently fails — can't delete
sealed = {};            // ✅ reassignment works (if let)

// Object.freeze() — no add/delete/modify
const frozen = Object.freeze({ name: "Deepak", age: 25 });
frozen.name = "John";   // ❌ silently fails
frozen.city = "Indore"; // ❌ silently fails
delete frozen.age;      // ❌ silently fails
frozen = {};            // ✅ reassignment works (if let)
```

### Comparison Table

```
Operation        | const | Object.seal() | Object.freeze()
-----------------|-------|---------------|----------------
Reassign var     | ❌    | ✅            | ✅
Add property     | ✅    | ❌            | ❌
Delete property  | ✅    | ❌            | ❌
Modify property  | ✅    | ✅            | ❌
Nested objects   | ✅    | ✅            | ✅ (still mutable!)
```

### Object.freeze is SHALLOW — Important!

```js
const obj = Object.freeze({
  name: "Deepak",
  address: { city: "Indore" } // nested object NOT frozen!
});

obj.name = "John";           // ❌ fails — top level frozen
obj.address.city = "Mumbai"; // ✅ WORKS — nested not frozen!

console.log(obj.address.city); // "Mumbai" — changed!
```

### Deep Freeze — Truly Immutable

```js
function deepFreeze(obj) {
  // Get all property names including non-enumerable
  Object.getOwnPropertyNames(obj).forEach(name => {
    const value = obj[name];
    // Recursively freeze nested objects
    if (value && typeof value === "object") {
      deepFreeze(value);
    }
  });
  return Object.freeze(obj);
}

const obj = deepFreeze({
  name: "Deepak",
  address: {
    city: "Indore",
    coordinates: { lat: 22.7, lng: 75.8 }
  },
  hobbies: ["coding", "reading"]
});

obj.name = "John";              // ❌ fails
obj.address.city = "Mumbai";    // ❌ fails — deep frozen!
obj.address.coordinates.lat = 0; // ❌ fails — deep frozen!
obj.hobbies.push("gaming");     // ❌ fails — array frozen!
```

### Checking Status

```js
const obj = Object.freeze({ name: "Deepak" });
Object.isFrozen(obj);  // true
Object.isSealed(obj);  // true — frozen objects are also sealed

const sealed = Object.seal({ name: "Deepak" });
Object.isSealed(sealed); // true
Object.isFrozen(sealed); // false — sealed but not frozen
```

### strict mode behavior

```js
// In non-strict mode — violations silently fail
const frozen = Object.freeze({ name: "Deepak" });
frozen.name = "John"; // silently ignored

// In strict mode — violations throw TypeError
"use strict";
const frozen2 = Object.freeze({ name: "Deepak" });
frozen2.name = "John"; // TypeError: Cannot assign to read only property
```

### Real World Use Cases

```js
// 1. Config object — should never change
const CONFIG = Object.freeze({
  API_URL: "https://api.example.com",
  MAX_RETRIES: 3,
  TIMEOUT: 5000
});

// 2. Constants/Enums
const STATUS = Object.freeze({
  PENDING: "pending",
  ACTIVE: "active",
  INACTIVE: "inactive"
});

// 3. Seal — allow updates but no new properties
const userProfile = Object.seal({
  name: "Deepak",
  age: 25,
  email: "deepak@example.com"
});
userProfile.age = 26;      // ✅ can update
userProfile.phone = "..."; // ❌ can't add new fields
```

---

## 🔑 Key Concepts to Remember
| Method | Add | Delete | Modify | Deep? |
|---|---|---|---|---|
| `const` | ✅ | ✅ | ✅ | N/A |
| `Object.seal()` | ❌ | ❌ | ✅ | ❌ |
| `Object.freeze()` | ❌ | ❌ | ❌ | ❌ (shallow!) |
| `deepFreeze()` | ❌ | ❌ | ❌ | ✅ |

### Most Important Thing to Remember
> `Object.freeze()` is **SHALLOW** — nested objects are still mutable!
> Use recursive `deepFreeze()` for true deep immutability
