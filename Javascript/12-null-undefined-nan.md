# 12 - null, undefined & NaN

## ❓ Interview Question
> "Explain null, undefined, and NaN in JavaScript. What are the differences, how do you reliably check for each, and what does Number(null), Number(undefined), Number(''), Number(false) return — and why?"

---

## 🙋 My Answer (What I Said)
- null is an object, undefined is not defined, NaN is Not a Number
- Check undefined with === undefined
- Check NaN with NaN(a) — wrong syntax
- Number(null) → NaN — WRONG, it's 0
- Number(undefined) → NaN — correct
- Number('') → 0 — correct
- Number(false) → 0 — correct

---

## ✅ What I Got Right
- undefined explanation correct
- NaN = Not a Number correct
- Number('') = 0 correct
- Number(false) = 0 correct
- Reasoning for Number(false) correct

## ❌ What Was Missing
- null is NOT an object (JS bug with typeof)
- Number(null) = 0 NOT NaN — common mistake!
- NaN() is not a function — use Number.isNaN()
- null == undefined trick
- NaN !== NaN

---

## 💡 Model Answer

### null

```js
// null — intentional absence of value
// Developer explicitly sets it to "no value"
let user = null; // user doesn't exist yet

typeof null // "object" ← famous JS bug! null is actually a primitive
null === null // true ← best way to check for null

// null is falsy
Boolean(null) // false
!!null        // false
```

### undefined

```js
// undefined — variable declared but never assigned
let a;
console.log(a); // undefined

// Also returned when:
const obj = {};
obj.name;         // undefined — property doesn't exist

function fn() {}
fn();             // undefined — no return value

function greet(name) {
  console.log(name); // undefined if not passed
}
greet();

typeof undefined  // "undefined" ← reliable check
a === undefined   // true ← also reliable
```

### NaN

```js
// NaN — result of invalid math operation
"hello" * 2       // NaN
parseInt("abc")   // NaN
0 / 0             // NaN
Math.sqrt(-1)     // NaN

typeof NaN        // "number" ← NaN is typeof "number"! surprising
NaN === NaN       // false ← only value not equal to itself!

// ✅ Reliable NaN checks
Number.isNaN(NaN)       // true  ← recommended
Number.isNaN("hello")   // false ← doesn't coerce, strict
isNaN("hello")          // true  ← coerces first, less reliable
isNaN(undefined)        // true  ← coerces undefined → NaN
```

### null vs undefined

```js
// undefined — unintentional, JS assigned it
let x; // JS gives undefined

// null — intentional, developer assigned it
let user = null; // developer says "no user"

// Loose equality — they equal each other!
null == undefined   // true  ← useful pattern
null === undefined  // false ← strict check

// Check for BOTH at once
if (value == null) {
  // value is null OR undefined
}
```

### Number() Conversion — Memorize This!

```js
Number(null)        // 0      ← NOT NaN! Most common mistake
Number(undefined)   // NaN
Number("")          // 0
Number(" ")         // 0      ← whitespace becomes 0
Number(false)       // 0      ← false = 0
Number(true)        // 1      ← true = 1
Number("123")       // 123
Number("123abc")    // NaN
Number("abc")       // NaN
Number([])          // 0      ← empty array = 0
Number([5])         // 5      ← single element array
Number([1,2])       // NaN    ← multiple elements
Number({})          // NaN
```

### Reliable Checks Summary

```js
// null check
value === null

// undefined check
value === undefined
typeof value === "undefined" // safer for undeclared variables

// null OR undefined check (most useful!)
value == null

// NaN check
Number.isNaN(value)

// Falsy check (null, undefined, NaN, 0, "", false)
if (!value) { ... }

// Check if value exists and is not null/undefined
if (value != null) { ... }
```

### Surprising Comparisons

```js
null == false    // false ← null only == undefined
null == 0        // false
null == ""       // false
null == undefined // true ← only exception

undefined == false // false
undefined == 0     // false
undefined == ""    // false

NaN == NaN         // false ← always false!
NaN == false       // false
NaN == null        // false
```

---

## 🔑 Key Concepts to Remember
| Value | typeof | Falsy? | == undefined? | Common use |
|---|---|---|---|---|
| `null` | `"object"` (bug!) | ✅ | ✅ | Intentional empty |
| `undefined` | `"undefined"` | ✅ | ✅ | Unassigned variable |
| `NaN` | `"number"` | ✅ | ❌ | Invalid math |

### Must Memorize
- `typeof null` → `"object"` (JS bug from 1995!)
- `NaN === NaN` → `false` (use `Number.isNaN()`)
- `Number(null)` → `0` (NOT NaN!)
- `null == undefined` → `true` (useful pattern)
- `null === undefined` → `false`
