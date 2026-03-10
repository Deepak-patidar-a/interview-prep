# 17 - Call Stack, Heap & Stack Overflow

## ❓ Interview Question
> "What is the difference between call stack and heap in JavaScript memory? What is a stack overflow, how does it happen, and can you give a real world example? Also explain tail call optimization."

---

## 💡 Full Answer

### Call Stack vs Heap

```
┌─────────────────────────────────────────────────────────┐
│                      CALL STACK                         │
│                                                         │
│  - Stores: function calls, local variables, primitives  │
│  - Structure: LIFO (Last In First Out)                  │
│  - Size: LIMITED (usually ~10,000 frames)               │
│  - Access: FAST                                         │
│  - Management: Automatic (push/pop on call/return)      │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                        HEAP                             │
│                                                         │
│  - Stores: objects, arrays, functions                   │
│  - Structure: Unstructured, dynamic                     │
│  - Size: LARGE (limited by RAM)                         │
│  - Access: SLOWER (via reference/pointer)               │
│  - Management: Garbage Collector                        │
└─────────────────────────────────────────────────────────┘
```

### How They Work Together

```js
function greet(name) {
  const greeting = "Hello"; // primitive → stored in STACK
  const obj = { msg: "hi" }; // object → stored in HEAP
  // obj variable in stack holds REFERENCE (pointer) to heap location

  return `${greeting}, ${name}`;
}

// Call stack when greet("Deepak") is called:
// [ greet("Deepak") ]  ← pushed
// [ main ]
//
// After greet returns:
// [ main ]             ← greet popped off
```

### Call Stack — LIFO Visualization

```js
function third() {
  console.log("third");
}

function second() {
  third();
  console.log("second");
}

function first() {
  second();
  console.log("first");
}

first();

// Call stack frames:
// Step 1: [first()]
// Step 2: [second(), first()]
// Step 3: [third(), second(), first()]
// Step 4: [second(), first()]   ← third() returned, popped
// Step 5: [first()]             ← second() returned, popped
// Step 6: []                    ← first() returned, popped
```

### Stack Overflow

```js
// Happens when call stack exceeds its size limit
// Most common cause: infinite recursion (missing base case)

// ❌ Stack overflow — infinite recursion
function recursive() {
  return recursive(); // calls itself forever
}
recursive(); // RangeError: Maximum call stack size exceeded

// ❌ Real world example — forgot base case
function factorial(n) {
  return n * factorial(n - 1); // no base case!
}
factorial(5); // RangeError!

// ✅ Fixed — with base case
function factorial(n) {
  if (n <= 1) return 1; // base case — stops recursion
  return n * factorial(n - 1);
}
factorial(5); // 120

// ❌ Mutual recursion — also causes overflow
function isEven(n) {
  if (n === 0) return true;
  return isOdd(n - 1); // calls isOdd
}
function isOdd(n) {
  if (n === 0) return false;
  return isEven(n - 1); // calls isEven
}
isEven(100000); // Stack overflow!
```

### Tail Call Optimization (TCO)

```js
// Regular recursion — new stack frame added each call
// Must keep previous frame to do multiplication after return
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
  //     ↑ must wait for recursive call to return, then multiply
  // Stack: factorial(5) → factorial(4) → factorial(3) → ... → factorial(1)
  // All frames kept in memory simultaneously!
}
factorial(10000); // Stack overflow!

// Tail call — recursive call is LAST operation
// Nothing to do after it returns → engine reuses same frame
function factorial(n, acc = 1) {
  if (n <= 1) return acc;
  return factorial(n - 1, n * acc); // last operation — tail call!
  // acc carries the result — no need to keep previous frame
}

// With TCO — only ONE stack frame needed:
// factorial(5, 1)
// factorial(4, 5)   ← reuses same frame
// factorial(3, 20)  ← reuses same frame
// factorial(2, 60)  ← reuses same frame
// factorial(1, 120) ← returns 120

factorial(100000); // works with TCO!
```

### Iterative Alternative (Most Reliable)

```js
// For very deep recursion, convert to iteration
// No stack overflow possible

function factorialIterative(n) {
  let result = 1;
  for (let i = 2; i <= n; i++) {
    result *= i;
  }
  return result;
}

// Fibonacci — iterative
function fibonacci(n) {
  if (n <= 1) return n;
  let prev = 0, curr = 1;
  for (let i = 2; i <= n; i++) {
    [prev, curr] = [curr, prev + curr];
  }
  return curr;
}
fibonacci(1000000); // works fine — no stack involved
```

### Stack vs Heap in Practice

```js
// Primitives — stored by VALUE in stack
let a = 5;
let b = a; // copy of value
b = 10;
console.log(a); // 5 — unchanged

// Objects — stored by REFERENCE in heap
let obj1 = { name: "Deepak" };
let obj2 = obj1; // copy of REFERENCE (both point to same heap object)
obj2.name = "John";
console.log(obj1.name); // "John" — changed! same reference
```

---

## 🔑 Key Concepts to Remember
| Concept | Detail |
|---|---|
| Call Stack | LIFO, function calls + primitives, limited size |
| Heap | Objects/arrays, large, garbage collected |
| Stack Overflow | Too many nested calls — usually infinite recursion |
| Base case | Required in every recursive function |
| Tail Call | Recursive call is last operation — engine can reuse frame |
| TCO | Tail Call Optimization — prevents stack overflow in tail calls |
| Primitives | Stored by value in stack |
| Objects | Stored by reference — variable in stack, data in heap |
