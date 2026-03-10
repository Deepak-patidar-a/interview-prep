# 07 - == vs ===, Type Coercion, typeof & instanceof

## ❓ Interview Question
> "What is the difference between `==` and `===` in JavaScript? Can you explain type coercion and walk me through some examples that might surprise a developer? Also what is `typeof` and `instanceof` and when would you use each?"

---

## 🙋 My Answer (What I Said)
- `==` compares value only, `===` compares value AND type
- Couldn't recall type coercion
- `typeof` checks type of variable
- `instanceof` checks instances of function/class

---

## ✅ What I Got Right
- `==` vs `===` difference correct
- `typeof` usage correct
- `instanceof` concept correct

## ❌ What Was Missing
- Type coercion — what `==` does under the hood
- Surprising coercion examples
- `typeof null` bug
- `NaN === NaN` is false
- `typeof` unreliable for arrays/null

---

## 💡 Model Answer

### == vs ===

```js
// === strict equality — checks value AND type
5 === 5         // true
5 === "5"       // false — different types
null === null   // true
null === undefined // false

// == loose equality — converts types first (coercion)
5 == "5"        // true — string "5" becomes number 5
0 == false      // true — false becomes 0
"" == false     // true — both become 0
null == undefined // true — special rule
null == false   // false — null only == undefined
```

### Type Coercion — Surprising Examples

```js
// false, 0, "", null, undefined, NaN are all "falsy"
// JS converts these when using ==

0 == false          // true
"" == false         // true
"0" == false        // true  ← surprising!
[] == false         // true  ← [] becomes "" becomes 0
[] == 0             // true
"" == 0             // true
null == undefined   // true  ← only these two equal each other
null == 0           // false ← null only equals undefined!
null == ""          // false

// NaN is NEVER equal to anything — even itself!
NaN == NaN          // false ← unique in JS!
NaN === NaN         // false

// Always use === to avoid coercion surprises
```

### Number() Conversion Table — Memorize This!

```js
Number(null)        // 0      ← NOT NaN! Common mistake
Number(undefined)   // NaN
Number("")          // 0
Number(" ")         // 0
Number(false)       // 0
Number(true)        // 1
Number("123")       // 123
Number("123abc")    // NaN
Number([])          // 0
Number([1])         // 1
Number([1,2])       // NaN
```

### typeof

```js
typeof "hello"      // "string"
typeof 42           // "number"
typeof true         // "boolean"
typeof undefined    // "undefined"
typeof Symbol()     // "symbol"
typeof function(){} // "function"
typeof {}           // "object"
typeof []           // "object"  ← arrays are "object"!
typeof null         // "object"  ← famous JS bug!

// Reliable checks for edge cases
Array.isArray([])        // true  ← use this for arrays
value === null           // true  ← use this for null
Number.isNaN(NaN)        // true  ← use this for NaN
Object.prototype.toString.call([]) // "[object Array]" ← most reliable
```

### instanceof

```js
// Checks prototype chain — is X an instance of Y?
[] instanceof Array     // true
[] instanceof Object    // true  ← arrays are also Objects!
new Date() instanceof Date // true
new Error() instanceof Error // true

// Class instances
class Dog {}
const rex = new Dog();
rex instanceof Dog     // true
rex instanceof Object  // true

// When to use typeof vs instanceof
typeof "hello"          // "string"  ← use typeof for primitives
"hello" instanceof String // false   ← don't use instanceof for primitives

new String("hello") instanceof String // true ← only for wrapper objects
```

### NaN Checks

```js
// ❌ Wrong
NaN(value)        // NaN is not a function!
typeof NaN        // "number" ← NaN is typeof "number"!

// ✅ Reliable NaN check
Number.isNaN(NaN)       // true  ← recommended, no coercion
Number.isNaN("hello")   // false ← doesn't coerce

// ⚠️ isNaN — coerces first, less reliable
isNaN("hello")          // true  ← coerces "hello" to NaN first
isNaN(undefined)        // true  ← coerces undefined to NaN first
```

---

## 🔑 Key Concepts to Remember
| Check | Best Method |
|---|---|
| Null | `value === null` |
| Undefined | `value === undefined` or `typeof value === "undefined"` |
| NaN | `Number.isNaN(value)` |
| Array | `Array.isArray(value)` |
| null or undefined | `value == null` (loose equality trick) |
| Type of primitive | `typeof value` |
| Instance of class | `value instanceof ClassName` |

### Must Memorize
- `typeof null` → `"object"` (JS bug!)
- `NaN === NaN` → `false` (only value not equal to itself)
- `Number(null)` → `0` (NOT NaN!)
- `Number(undefined)` → `NaN`
