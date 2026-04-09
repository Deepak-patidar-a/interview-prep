# 🏢 Interview Revision — GoTo Company Round

---

## Q1. Sort Array of 0s, 1s, 2s in O(n)

### Problem
```js
const arr = [0, 1, 2, 0, 2, 1, 0, 1, 0]
// Expected output: [0, 0, 0, 0, 1, 1, 1, 2, 2]
```

### Algorithm — Dutch National Flag (Dijkstra)
Three pointers — single pass, no extra space.

```
low  → boundary for 0s (start)
mid  → current element being checked
high → boundary for 2s (end)
```

### Solution
```js
const arr = [0, 1, 2, 0, 2, 1, 0, 1, 0];

function sortColors(arr) {
  let low = 0;
  let mid = 0;
  let high = arr.length - 1;

  while (mid <= high) {
    if (arr[mid] === 0) {
      // swap low and mid, move both forward
      [arr[low], arr[mid]] = [arr[mid], arr[low]];
      low++;
      mid++;
    } else if (arr[mid] === 1) {
      // 1 is already in correct place, just move mid
      mid++;
    } else {
      // swap mid and high, move high backward
      // don't move mid — need to recheck swapped value
      [arr[mid], arr[high]] = [arr[high], arr[mid]];
      high--;
    }
  }

  return arr;
}

console.log(sortColors(arr));
// [0, 0, 0, 0, 1, 1, 1, 2, 2]
```

### Complexity
| | Value |
|---|---|
| Time | O(n) — single pass |
| Space | O(1) — no extra array |

### What to Say in Interview
*"Since there are only 3 distinct values, we don't need a comparison-based sort. We can partition the array in a single pass using 3 pointers — Dutch National Flag algorithm."*

---

## Q2. Equilibrium Index — Left Average equals Right Average

### Problem
Find all indices where `left_sum * left_count === right_sum * right_count`
(i.e. left average equals right average)

```js
const arr1 = [1, 4, 5, -4, -5]
```

---

### Solution 1 — Your Original O(n²) Approach

```js
const arr1 = [1, 4, 5, -4, -5];

const equil = (arr, index) => {
  let left_sum = 0;
  let left_count = 0;
  let right_sum = 0;
  let right_count = 0;

  // loop right side
  for (let i = index + 1; i < arr.length; i++) {
    left_sum = left_sum + arr[i];
    left_count = left_count + 1;
  }

  // loop left side
  for (let i = index - 1; i >= 0; i--) {
    right_sum = right_sum + arr[i];
    right_count = right_count + 1;
  }

  if (left_sum * left_count === right_sum * right_count) {
    return [index, left_sum * left_count];
  }
  return null;
};

const result = () => {
  let res = [];
  for (let i = 0; i < arr1.length; i++) {
    let data = equil(arr1, i);
    data ? res.push(data) : res;
  }
  return res;
};

console.log(result());
```

### Why O(n²)?
```
result() loops       → n times
  equil() loops      → n times inside
= n × n              = O(n²) ❌
```

---

### Solution 2 — Optimized O(n) Using Prefix Sum

### Key Insight
> Instead of recalculating right sum by looping every time,
> use: `rightSum = totalSum - leftSum - currentElement`
> Derive it instantly — no second loop needed.

```js
const arr1 = [1, 4, 5, -4, -5];

const equilOptimized = (arr) => {
  const totalSum = arr.reduce((acc, val) => acc + val, 0);
  const res = [];

  let leftSum = 0;
  let leftCount = 0;

  for (let i = 0; i < arr.length; i++) {
    const rightCount = arr.length - i - 1;

    // derive rightSum instantly — no loop needed
    const rightSum = totalSum - leftSum - arr[i];

    if (
      leftCount > 0 &&   // at least one element on left
      rightCount > 0 &&  // at least one element on right
      leftSum * leftCount === rightSum * rightCount
    ) {
      res.push([i, leftSum * leftCount]);
    }

    // update for next iteration
    leftSum += arr[i];
    leftCount++;
  }

  return res;
};

console.log(equilOptimized(arr1));
```

### Why O(n)?
```
totalSum calculation  → one pass  O(n)
main loop             → one pass  O(n)
No nested loops                   ✅
```

### Comparison
| | O(n²) Solution | O(n) Solution |
|---|---|---|
| Time | O(n²) | O(n) |
| Space | O(1) | O(1) |
| Loops | Nested | Single pass |
| Key idea | Recalculate every time | Precompute total sum |

### What to Say in Interview
*"My first approach was O(n²) — two loops inside each other. I optimized it using prefix sum — since I know the total sum upfront, I can calculate rightSum as totalSum - leftSum - currentElement without a second loop, bringing it down to O(n)."*

---

## Q3. OOP — Animal & Cat Class, Type Compatibility

### Problem
```js
let animal = new Animal("cat")       // Case 1
let cat    = new Cat("white","breed") // Case 2
let animal = new Cat("white","breed") // Case 3
let cat    = new Animal("cat")        // Case 4
```
Which works? Which fails? Why?

---

### Class Setup
```js
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    return `${this.name} makes a sound`;
  }
}

class Cat extends Animal {
  constructor(color, breed) {
    super("Cat"); // calls Animal constructor → sets this.name = "Cat"
    this.color = color;
    this.breed = breed;
  }

  speak() {
    return `${this.name} meows`; // overrides Animal's speak
  }

  purr() {
    return `${this.color} ${this.breed} is purring...`;
  }
}
```

---

### Case 1 — `let animal = new Animal("cat")` ✅
```js
let animal = new Animal("cat");
animal.speak(); // "cat makes a sound" ✅
animal.purr();  // ❌ TypeError — purr() doesn't exist on Animal
```
Normal and straightforward. Animal only knows Animal things.

---

### Case 2 — `let cat = new Cat("white", "persian")` ✅
```js
let cat = new Cat("white", "persian");
cat.speak(); // "Cat meows" ✅ — Cat's own speak()
cat.purr();  // "white persian is purring..." ✅
cat.name;    // "Cat" ✅ — inherited from Animal via super()
```
Cat has everything — its own properties AND Animal's via `super()`.

---

### Case 3 — `let animal = new Cat("white", "persian")` ✅
**Parent reference holding a Child object — Upcasting**
```js
let animal = new Cat("white", "persian");
animal.speak(); // "Cat meows" ✅ — runs Cat's speak, not Animal's
animal.name;    // "Cat" ✅
animal.purr();  // "white persian is purring..." ✅
// Variable named 'animal' but it IS a Cat instance
// so ALL Cat methods are accessible
```
> ✅ This always works — Child can always be assigned to Parent reference.
> Cat IS an Animal, so upcasting is always safe.

---

### Case 4 — `let cat = new Animal("cat")` ⚠️
**Parent object in Child reference — Downcasting — UNSAFE**
```js
let cat = new Animal("cat");
cat.speak(); // "cat makes a sound" — Animal's speak only
cat.purr();  // ❌ TypeError: cat.purr is not a function
cat.color;   // undefined — Animal has no color
```
Animal has no idea about Cat's properties.
Animal is NOT necessarily a Cat.

---

### How `this` Works Here
`this` always refers to the **actual instance** being created, not the class it's defined in.

```js
const c = new Cat("white", "persian");

// 'this' at runtime looks like one single object:
// {
//   name: "Cat",       ← set by Animal via super()
//   color: "white",    ← set by Cat
//   breed: "persian"   ← set by Cat
// }
```

`super()` in Cat says: *"Run Animal's constructor but use the same `this` object I'm building."*
So both parent and child properties end up on the **same object**.

---

### Summary Table

| Assignment | Works? | Name | Rule |
|---|---|---|---|
| `let animal = new Animal("cat")` | ✅ | Normal | Straightforward |
| `let cat = new Cat("white","breed")` | ✅ | Normal | Has all properties |
| `let animal = new Cat("white","breed")` | ✅ | Upcasting | Child → Parent ref, always safe |
| `let cat = new Animal("cat")` | ⚠️ | Downcasting | Parent → Child ref, child methods missing |

### What to Say in Interview
*"A child can always be assigned to a parent reference because child IS a parent — but not the other way around. A Cat is always an Animal, but an Animal is not always a Cat. This is the core of Polymorphism."*

---

## 🔑 Quick Cheat Sheet

| Topic | Key Point to Remember |
|---|---|
| Dutch National Flag | 3 pointers — low, mid, high — single pass O(n) |
| Sort 0,1,2 | No comparison sort needed — only 3 distinct values |
| Equilibrium O(n²) | Two loops inside each other — recalculates every time |
| Equilibrium O(n) | Precompute totalSum → rightSum = totalSum - leftSum - current |
| Upcasting | Child → Parent reference — always safe ✅ |
| Downcasting | Parent → Child reference — child methods unavailable ⚠️ |
| `this` in classes | Always refers to actual instance, not the class |
| `super()` | Runs parent constructor on same `this` object |
| Polymorphism | Cat IS an Animal. Animal is NOT always a Cat |
