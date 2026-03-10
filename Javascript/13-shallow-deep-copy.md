# 13 - Shallow Copy vs Deep Copy

## ❓ Interview Question
> "What is the difference between shallow copy and deep copy in JavaScript? What are the different ways to achieve each, and what are their limitations?"

---

## 🙋 My Answer (What I Said)
- Shallow copy copies only first level
- Deep copy copies nested levels too
- Shallow: spread operator, Object.assign()
- Deep: JSON.stringify + JSON.parse (loses functions, Map, Set, Date)
- structuredClone() — modern recommended way
- Recursive function — best/most flexible way

---

## ✅ What I Got Right
- Shallow vs deep distinction correct
- Spread and Object.assign for shallow — correct
- JSON.stringify/parse limitations — excellent!
- structuredClone() — impressive!
- Recursive approach — correct

## ❌ What Was Missing
- Demonstrating WHY shallow copy is a problem (reference sharing)
- structuredClone also has limitations (functions)
- Recursive implementation written out
- Lodash cloneDeep for production

---

## 💡 Model Answer

### Why Shallow Copy is a Problem

```js
const obj = {
  name: "Deepak",
  address: { city: "Indore" } // nested object
};

// Shallow copy — address still points to SAME reference!
const copy = { ...obj };

copy.name = "John";           // ✅ original unchanged
copy.address.city = "Mumbai"; // ❌ original ALSO changed!

console.log(obj.address.city); // "Mumbai" — oops!
```

### Shallow Copy Methods

```js
// 1. Spread operator
const copy1 = { ...original };
const arrCopy = [...originalArr];

// 2. Object.assign()
const copy2 = Object.assign({}, original);

// 3. Array methods (all shallow)
const arrCopy2 = originalArr.slice();
const arrCopy3 = Array.from(originalArr);

// All have same problem — nested objects are shared references
const arr = [[1, 2], [3, 4]];
const shallowArr = [...arr];
shallowArr[0].push(99);
console.log(arr[0]); // [1, 2, 99] — original affected!
```

### Deep Copy Methods

#### 1. JSON.stringify + JSON.parse

```js
const deepCopy = JSON.parse(JSON.stringify(original));

// ✅ Works for: strings, numbers, booleans, arrays, plain objects
// ❌ Fails for:
const obj = {
  name: "Deepak",
  fn: () => {},           // ❌ functions — becomes undefined/lost
  date: new Date(),       // ❌ Date — becomes string
  map: new Map(),         // ❌ Map — becomes {}
  set: new Set([1, 2]),   // ❌ Set — becomes {}
  undef: undefined,       // ❌ undefined — lost
  reg: /regex/,           // ❌ RegExp — becomes {}
  circular: null          // ❌ circular refs — throws error
};
```

#### 2. structuredClone() — Modern Recommended

```js
const deepCopy = structuredClone(original);

// ✅ Works for: most types including Date, Map, Set, ArrayBuffer
const obj = {
  date: new Date(),      // ✅ preserved as Date
  map: new Map(),        // ✅ preserved as Map
  set: new Set([1, 2])   // ✅ preserved as Set
};
const copy = structuredClone(obj);

// ❌ Still fails for:
structuredClone({ fn: () => {} }); // ❌ throws for functions
structuredClone({ dom: document.body }); // ❌ throws for DOM nodes
```

#### 3. Recursive Deep Copy — Most Flexible

```js
function deepCopy(obj) {
  // Handle primitives and null
  if (obj === null || typeof obj !== "object") return obj;

  // Handle Date
  if (obj instanceof Date) return new Date(obj.getTime());

  // Handle Array
  if (Array.isArray(obj)) return obj.map(deepCopy);

  // Handle Object
  return Object.keys(obj).reduce((acc, key) => {
    acc[key] = deepCopy(obj[key]);
    return acc;
  }, {});
}

// Usage
const original = {
  name: "Deepak",
  address: { city: "Indore" },
  hobbies: ["coding", "reading"],
  joined: new Date()
};

const copy = deepCopy(original);
copy.address.city = "Mumbai";
console.log(original.address.city); // "Indore" — original safe!
```

#### 4. Lodash cloneDeep — Production

```js
import cloneDeep from "lodash/cloneDeep";

const copy = cloneDeep(original);
// Handles all edge cases including circular references
```

### Comparison Table

```
Method              | Nested | Functions | Date | Map/Set | Circular
--------------------|--------|-----------|------|---------|----------
Spread {...}        | ❌     | ✅        | ✅   | ✅      | ✅
Object.assign()     | ❌     | ✅        | ✅   | ✅      | ✅
JSON.parse/stringify| ✅     | ❌        | ❌   | ❌      | ❌
structuredClone()   | ✅     | ❌        | ✅   | ✅      | ✅
Recursive custom    | ✅     | ✅*       | ✅   | ✅*     | ❌
lodash cloneDeep    | ✅     | ✅        | ✅   | ✅      | ✅
```

---

## 🔑 Key Concepts to Remember
| Situation | Best Method |
|---|---|
| Simple object, no nesting | Spread `{...obj}` |
| Deep copy, no functions/DOM | `structuredClone()` |
| Deep copy, all types | `lodash.cloneDeep()` |
| Custom logic needed | Recursive function |
| Quick and dirty (plain data) | `JSON.parse(JSON.stringify())` |

### One-liner to remember
> Shallow copy = same reference for nested objects
> Deep copy = completely independent clone at all levels
