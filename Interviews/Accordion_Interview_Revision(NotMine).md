# 🎯 Accordion Interview – Revision Guide

> A complete study guide with explanations, analogies, and code examples to help you understand and remember each topic.

---

## Table of Contents

1. [Flattening a Nested Array](#1-flattening-a-nested-array)
2. [Feature Flagging](#2-feature-flagging)
3. [Microfrontends](#3-microfrontends)
4. [Data Structures & Algorithms – Real World Scenarios](#4-data-structures--algorithms)
5. [Merge Sort](#5-merge-sort)
6. [Kth Largest Element in an Array](#6-kth-largest-element-in-an-array)
7. [SOLID Principles](#7-solid-principles)
8. [OOP in JavaScript](#8-oop-in-javascript)
9. [Semantic HTML Elements](#9-semantic-html-elements)
10. [display: none vs visibility: hidden](#10-display-none-vs-visibility-hidden)
11. [Architecting for Multiple Devices](#11-architecting-for-multiple-devices)

---

## 1. Flattening a Nested Array

### 🧠 What is it?
Flattening means converting a deeply nested array like `[1, [2, [3, [4]]]]` into a flat array `[1, 2, 3, 4]`.

### 🔑 Key Methods

#### Method 1: `Array.flat(depth)`
```js
const arr = [1, [2, [3, [4]]]];

arr.flat();       // [1, 2, [3, [4]]]  → only 1 level deep
arr.flat(2);      // [1, 2, 3, [4]]    → 2 levels deep
arr.flat(Infinity); // [1, 2, 3, 4]   → fully flat ✅
```

#### Method 2: Recursive approach (custom flatten)
```js
function flatten(arr) {
  return arr.reduce((acc, item) => {
    return Array.isArray(item)
      ? acc.concat(flatten(item))  // recurse if it's an array
      : acc.concat(item);          // otherwise just add it
  }, []);
}

flatten([1, [2, [3, [4]]]]); // [1, 2, 3, 4]
```

#### Method 3: Using a stack (iterative, interview favourite)
```js
function flattenIterative(arr) {
  const stack = [...arr];
  const result = [];

  while (stack.length) {
    const item = stack.pop();
    if (Array.isArray(item)) {
      stack.push(...item); // spread nested array back on stack
    } else {
      result.unshift(item);
    }
  }
  return result;
}
```

### 💡 Remember
- `flat(Infinity)` is the simplest one-liner
- Recursive is great for interviews to show you understand recursion
- Interviewer may ask: *"What if the array is 1000 levels deep?"* → Iterative stack avoids call stack overflow

---

## 2. Feature Flagging

### 🧠 What is it?
Feature flags (also called feature toggles) let you **turn features on or off without deploying new code**. Think of it like a light switch in your app.

### 🎯 Why use it?
- Roll out features gradually (to 10% of users first, then 100%)
- Quickly disable a broken feature in production
- A/B test new UI
- Let QA test unreleased features in production safely

### 🔧 How it works

#### Simple approach: Config / Environment Variables
```js
// config.js
const featureFlags = {
  newCheckout: false,
  darkMode: true,
  betaSearch: false,
};

// In your component
if (featureFlags.newCheckout) {
  renderNewCheckout();
} else {
  renderOldCheckout();
}
```

#### Advanced: Remote Feature Flag Service
Services like **LaunchDarkly**, **Unleash**, **Firebase Remote Config**, or even a simple API endpoint serve flags from a server.

```js
async function getFlags(userId) {
  const response = await fetch(`/api/flags?userId=${userId}`);
  return response.json();
  // Returns: { newCheckout: true, darkMode: false }
}
```

**Flow:**
```
User visits app
      ↓
App fetches flags from server (no deployment needed!)
      ↓
Flag "newCheckout" = false → Old checkout shown
      ↓
Ops team sets flag to true → New checkout shown instantly
```

### 🏗️ Types of Feature Flags
| Type | Use Case |
|------|----------|
| **Release flags** | Gradually roll out new features |
| **Ops flags** | Kill switch for broken features |
| **Experiment flags** | A/B testing |
| **Permission flags** | Premium-only features |

### 💡 Key Interview Point
> "We can disable a feature **without deployment** by using a remote feature flag — we just update the flag value on the server/dashboard, and all users instantly see the change on their next request."

---

## 3. Microfrontends

### 🧠 What is it?
Microfrontends apply the **microservices concept to the frontend**. Instead of one giant React/Angular app, you split the UI into smaller, independently deployable apps owned by different teams.

### 🏠 Analogy
Think of a shopping mall:
- **Monolith** = One giant department store managing everything
- **Microfrontends** = Individual stores (Nike, Apple, H&M) each managing their own section, but they all exist under the same mall roof

### 📦 Types of Microfrontends

#### 1. Build-time Integration (NPM packages)
Each team publishes their feature as an npm package.
```bash
npm install @team-auth/login-widget
npm install @team-cart/cart-component
```
**Pro:** Simple. **Con:** All teams must redeploy when any package updates.

#### 2. Run-time Integration via iFrames
Each microfrontend is a separate page loaded in an `<iframe>`.
```html
<iframe src="https://cart.myapp.com" />
```
**Pro:** Perfect isolation. **Con:** Hard to share state, styling, and communication.

#### 3. Run-time Integration via JavaScript (Module Federation)
Webpack 5's **Module Federation** is the most popular approach.
```js
// webpack.config.js of the HOST app
remotes: {
  cartApp: 'cartApp@https://cart.myapp.com/remoteEntry.js',
}

// Use it like a local component
import CartWidget from 'cartApp/CartWidget';
```
**Pro:** Teams deploy independently, runtime loading. **Con:** Complex setup.

#### 4. Web Components
Each team builds framework-agnostic custom elements.
```html
<cart-widget user-id="123"></cart-widget>
<auth-header></auth-header>
```

### 💡 Key Points
- **Team autonomy**: Each team owns their deployment pipeline
- **Tech agnostic**: One team can use React, another Vue
- **Challenges**: Shared state, consistent design system, performance (loading multiple JS bundles)

---

## 4. Data Structures & Algorithms

### 🧠 How to choose the right one?

Always ask: *"What operations are most frequent — insert, search, delete, or traverse?"*

### 🗂️ Quick Reference Table

| Data Structure | Best For | Real-World Use |
|---|---|---|
| **Array** | Index-based access, iteration | Lists, matrices |
| **HashMap** | O(1) lookups by key | Caching, frequency count |
| **Queue (FIFO)** | Processing in order | Print queue, BFS |
| **Stack (LIFO)** | Undo/redo, backtracking | Browser history, call stack |
| **Heap (Priority Queue)** | Always access min/max fast | Scheduling, Dijkstra's |
| **Graph** | Connections between nodes | Maps, social networks |
| **Trie** | Prefix-based search | Autocomplete |

---

### 🛗 Real-World Example: Elevator / Lift System

This is a classic DS&A design question. Let's break it down:

#### Problem
Multiple floors request the elevator. How do you decide which floor to go to next?

#### Naive Approach: Queue (FIFO)
```
Requests: [5, 2, 8, 1]
Elevator visits: 5 → 2 → 8 → 1
```
Simple, but very inefficient — elevator moves up and down wastefully.

#### Better Approach: Min-Heap + Scan Algorithm (SCAN / Elevator Algorithm)
The real elevator algorithm works like a scanner: go in one direction, serve all floors in that direction, then reverse.

```
Current floor: 3, going UP
Requests: [1, 5, 7, 2, 9]

Going UP → serve: 5, 7, 9
Reverse DOWN → serve: 2, 1
```

**Data Structure Used:**
- Two **min-heaps** (or sorted sets): one for floors above, one for floors below
- A **direction flag**: `UP` or `DOWN`

```js
// Pseudocode
class Elevator {
  constructor() {
    this.currentFloor = 0;
    this.direction = 'UP';
    this.upQueue = new MinHeap();   // floors above
    this.downQueue = new MaxHeap(); // floors below
  }

  request(floor) {
    if (floor >= this.currentFloor) this.upQueue.insert(floor);
    else this.downQueue.insert(floor);
  }

  next() {
    if (this.direction === 'UP' && !this.upQueue.isEmpty()) {
      this.currentFloor = this.upQueue.extractMin();
    } else {
      this.direction = 'DOWN';
      this.currentFloor = this.downQueue.extractMax();
    }
  }
}
```

**Why Heap?** → Because we always need the **nearest floor** in the current direction. A heap gives us min/max in O(log n).

### 💡 Key Interview Point
> "I'd use a priority queue / heap to always serve the nearest floor in the current direction — this mimics the real elevator SCAN algorithm and minimizes travel distance."

---

## 5. Merge Sort

### 🧠 What is it?
Merge Sort is a **divide and conquer** sorting algorithm. It splits the array in half, sorts each half, and **merges** them back together.

### 📊 Properties
| Property | Value |
|---|---|
| **Time Complexity** | O(n log n) — always |
| **Space Complexity** | O(n) — needs extra space |
| **Stable?** | ✅ Yes (preserves order of equal elements) |
| **Best for** | Linked lists, large datasets, external sorting |

### 🎨 Visual Walkthrough
```
Input: [38, 27, 43, 3]

Step 1 – Divide:
[38, 27, 43, 3]
[38, 27]   [43, 3]
[38] [27]  [43] [3]

Step 2 – Merge (sorted):
[27, 38]   [3, 43]
[3, 27, 38, 43]  ✅
```

### 💻 Code
```js
function mergeSort(arr) {
  // Base case: array of 1 is already sorted
  if (arr.length <= 1) return arr;

  // Step 1: Divide
  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));

  // Step 2: Merge sorted halves
  return merge(left, right);
}

function merge(left, right) {
  const result = [];
  let i = 0, j = 0;

  // Compare elements from both arrays, pick the smaller one
  while (i < left.length && j < right.length) {
    if (left[i] <= right[j]) {
      result.push(left[i++]);
    } else {
      result.push(right[j++]);
    }
  }

  // Add remaining elements
  return result.concat(left.slice(i)).concat(right.slice(j));
}

// Test
mergeSort([38, 27, 43, 3, 9, 82, 10]);
// Output: [3, 9, 10, 27, 38, 43, 82]
```

### 💡 Remember with this analogy
> Imagine sorting a deck of cards: split the deck in half, sort each half (recursively), then merge the two sorted halves by always picking the smaller top card.

---

## 6. Kth Largest Element in an Array

### 🧠 Problem
Given `[3, 2, 1, 5, 6, 4]` and `k = 2`, find the **2nd largest** element → Answer: `5`

### 🔑 Approaches

#### Approach 1: Sort (Simple but slow)
```js
function kthLargest(nums, k) {
  nums.sort((a, b) => b - a); // sort descending
  return nums[k - 1];
}
// Time: O(n log n) | Space: O(1)
```

#### Approach 2: Min-Heap of size K (Optimal for streaming data) ⭐
Keep a min-heap of size `k`. The root (minimum) of this heap is always the Kth largest.

```js
function kthLargest(nums, k) {
  // Use a simple sorted array as a min-heap simulation
  let heap = [];

  for (let num of nums) {
    heap.push(num);
    heap.sort((a, b) => a - b); // sort ascending

    if (heap.length > k) {
      heap.shift(); // remove the smallest
    }
  }

  return heap[0]; // smallest in heap = Kth largest overall
}

// Example:
// nums = [3,2,1,5,6,4], k = 2
// After processing: heap = [5, 6]
// heap[0] = 5 ✅
```

#### Approach 3: QuickSelect (Best average O(n))
Based on QuickSort's partition logic. Average O(n), worst case O(n²).

```js
function kthLargest(nums, k) {
  const targetIndex = nums.length - k; // kth largest = (n-k)th smallest

  function partition(left, right) {
    const pivot = nums[right];
    let i = left;

    for (let j = left; j < right; j++) {
      if (nums[j] <= pivot) {
        [nums[i], nums[j]] = [nums[j], nums[i]];
        i++;
      }
    }
    [nums[i], nums[right]] = [nums[right], nums[i]];
    return i;
  }

  function quickSelect(left, right) {
    const pivotIndex = partition(left, right);
    if (pivotIndex === targetIndex) return nums[pivotIndex];
    else if (pivotIndex < targetIndex) return quickSelect(pivotIndex + 1, right);
    else return quickSelect(left, pivotIndex - 1);
  }

  return quickSelect(0, nums.length - 1);
}
```

### 📊 Comparison
| Approach | Time | Space | Notes |
|---|---|---|---|
| Sort | O(n log n) | O(1) | Simple, fine for small arrays |
| Min-Heap | O(n log k) | O(k) | Best when k is small or streaming |
| QuickSelect | O(n) avg | O(1) | Best overall average performance |

---

## 7. SOLID Principles

### 🧠 What is it?
SOLID is a set of 5 design principles that make OOP code **maintainable, scalable, and flexible**.

---

### S – Single Responsibility Principle (SRP)
> **"A class should have only ONE reason to change."**

❌ Bad: One class doing too much
```js
class UserService {
  getUser(id) { /* fetch from DB */ }
  formatUserEmail(user) { /* format email */ } // ← not UserService's job
  sendEmail(user) { /* send email */ }          // ← not UserService's job
}
```

✅ Good: Split responsibilities
```js
class UserService { getUser(id) { } }
class EmailFormatter { format(user) { } }
class EmailService { send(user) { } }
```

---

### O – Open/Closed Principle (OCP)
> **"Open for extension, closed for modification."**

Don't edit existing code — extend it instead.

❌ Bad: Adding new shape means editing existing code
```js
function getArea(shape) {
  if (shape.type === 'circle') return Math.PI * shape.r ** 2;
  if (shape.type === 'square') return shape.side ** 2;
  // Adding triangle means editing this function ❌
}
```

✅ Good: Each shape knows its own area
```js
class Circle { area() { return Math.PI * this.r ** 2; } }
class Square { area() { return this.side ** 2; } }
class Triangle { area() { return 0.5 * this.base * this.height; } }
// New shape? Just add a new class — no existing code modified ✅
```

---

### L – Liskov Substitution Principle (LSP)
> **"Subclasses should be replaceable for their parent class without breaking the app."**

```js
class Bird { fly() { console.log("Flying"); } }

// ❌ Penguin can't fly — breaks LSP!
class Penguin extends Bird {
  fly() { throw new Error("I can't fly!"); }
}

// ✅ Fix: Separate flying and non-flying birds
class FlyingBird { fly() {} }
class NonFlyingBird { walk() {} }
class Penguin extends NonFlyingBird {}
```

---

### I – Interface Segregation Principle (ISP)
> **"Don't force a class to implement interfaces it doesn't use."**

```js
// ❌ Bad: Fat interface forces classes to implement unused methods
class Animal {
  eat() {}
  fly() {}   // Not all animals fly!
  swim() {}  // Not all animals swim!
}

// ✅ Good: Smaller, focused interfaces
class Eater { eat() {} }
class Flyer { fly() {} }
class Swimmer { swim() {} }

class Duck extends Eater { } // Mixin what's needed
```

---

### D – Dependency Inversion Principle (DIP)
> **"Depend on abstractions, not concrete implementations."**

```js
// ❌ Bad: Tightly coupled to MySQL
class UserService {
  constructor() {
    this.db = new MySQLDatabase(); // hardcoded dependency
  }
}

// ✅ Good: Inject the dependency (any DB works!)
class UserService {
  constructor(database) { // accepts any database
    this.db = database;
  }
}

const service = new UserService(new MySQLDatabase());
const testService = new UserService(new MockDatabase()); // easy testing!
```

### 💡 Memory Trick: **S**tephen **O**ften **L**ikes **I**nteresting **D**esigns

---

## 8. OOP in JavaScript

### 🧠 The 4 Pillars of OOP

#### 1. Encapsulation – Bundle data + behaviour together, hide internals
```js
class BankAccount {
  #balance = 0; // private field (ES2022)

  deposit(amount) {
    if (amount > 0) this.#balance += amount;
  }

  getBalance() {
    return this.#balance; // controlled access
  }
}

const acc = new BankAccount();
acc.deposit(100);
console.log(acc.getBalance()); // 100
console.log(acc.#balance); // ❌ SyntaxError – can't access private field
```

#### 2. Inheritance – Child class reuses parent class features
```js
class Animal {
  constructor(name) { this.name = name; }
  speak() { console.log(`${this.name} makes a sound.`); }
}

class Dog extends Animal {
  speak() { console.log(`${this.name} barks!`); } // override
}

const d = new Dog('Rex');
d.speak(); // "Rex barks!"
```

#### 3. Polymorphism – Same method name, different behavior
```js
class Shape {
  area() { return 0; }
}
class Circle extends Shape {
  area() { return Math.PI * this.r ** 2; }
}
class Square extends Shape {
  area() { return this.side ** 2; }
}

const shapes = [new Circle(5), new Square(4)];
shapes.forEach(s => console.log(s.area())); // each behaves differently ✅
```

#### 4. Abstraction – Hide complexity, expose only what's needed
```js
class CoffeeMachine {
  // Public: simple interface
  makeCoffee() {
    this.#heatWater();
    this.#grindBeans();
    this.#brew();
    console.log("Here's your coffee ☕");
  }

  // Private: hidden complexity
  #heatWater() { /* complex logic */ }
  #grindBeans() { /* complex logic */ }
  #brew() { /* complex logic */ }
}
```

### 🏗️ Class Syntax in JS
```js
class Person {
  // Static property (shared by all instances)
  static count = 0;

  constructor(name, age) {
    this.name = name;
    this.age = age;
    Person.count++;
  }

  // Instance method
  greet() {
    return `Hi, I'm ${this.name}`;
  }

  // Static method (called on class, not instance)
  static getCount() {
    return Person.count;
  }
}

const p1 = new Person('Alice', 25);
const p2 = new Person('Bob', 30);
console.log(Person.getCount()); // 2
console.log(p1.greet()); // "Hi, I'm Alice"
```

---

## 9. Semantic HTML Elements

### 🧠 What is it?
Semantic HTML uses **meaningful tags** that describe the content's purpose — not just how it looks.

### 🆚 Non-semantic vs Semantic
```html
<!-- ❌ Non-semantic: just boxes, no meaning -->
<div id="header">...</div>
<div id="nav">...</div>
<div id="content">...</div>
<div id="footer">...</div>

<!-- ✅ Semantic: self-describing -->
<header>...</header>
<nav>...</nav>
<main>...</main>
<footer>...</footer>
```

### 📋 Key Semantic Elements

| Tag | Purpose |
|-----|---------|
| `<header>` | Top section of page or section |
| `<nav>` | Navigation links |
| `<main>` | Primary content (only one per page) |
| `<section>` | Thematic grouping of content |
| `<article>` | Standalone content (blog post, news article) |
| `<aside>` | Sidebar / tangentially related content |
| `<footer>` | Bottom section |
| `<figure>` | Image with optional caption |
| `<figcaption>` | Caption for `<figure>` |
| `<time>` | Dates and times |
| `<mark>` | Highlighted text |
| `<details>` + `<summary>` | Expandable/collapsible content |

### 🎯 Why it matters
1. **Accessibility** – Screen readers use semantic tags to navigate
2. **SEO** – Search engines better understand your content structure
3. **Readability** – Code is easier to maintain
4. **Default styling** – Browsers apply some default styles (headings, etc.)

```html
<!-- A proper semantic page structure -->
<body>
  <header>
    <nav>...</nav>
  </header>

  <main>
    <article>
      <h1>Blog Post Title</h1>
      <time datetime="2024-01-15">Jan 15, 2024</time>
      <section>...</section>
    </article>
    <aside>Related Posts</aside>
  </main>

  <footer>
    <p>© 2024 MyApp</p>
  </footer>
</body>
```

---

## 10. `display: none` vs `visibility: hidden`

### 🧠 Quick Answer
| | `display: none` | `visibility: hidden` |
|---|---|---|
| **Visible?** | ❌ No | ❌ No |
| **Takes up space?** | ❌ No (removed from layout) | ✅ Yes (space preserved) |
| **Accessible to screen readers?** | ❌ No | ❌ No |
| **Children affected?** | Yes – children hidden | Can show children with `visibility: visible` |
| **Triggers reflow?** | ✅ Yes (layout recalculated) | ❌ No |
| **CSS transitions?** | ❌ Can't animate | ✅ Can animate opacity with it |

### 🎨 Visual Example
```
Layout: [Box A] [Box B] [Box C]

display: none on Box B:
→ [Box A] [Box C]        ← Box B is gone, C moves left

visibility: hidden on Box B:
→ [Box A] [     ] [Box C] ← Empty space remains
```

### 💻 Code Example
```html
<style>
  .hidden-display { display: none; }
  .hidden-visibility { visibility: hidden; }
</style>

<div>Box A</div>
<div class="hidden-display">Box B (display: none)</div>
<div>Box C</div>
<br>
<div>Box A</div>
<div class="hidden-visibility">Box B (visibility: hidden)</div>
<div>Box C</div>
```

### 🤓 Bonus: `opacity: 0`
A third option — element is invisible but **still takes space AND is still interactive** (clickable!).

| | `display: none` | `visibility: hidden` | `opacity: 0` |
|---|---|---|---|
| **Visible** | No | No | No |
| **Takes space** | No | Yes | Yes |
| **Interactive** | No | No | ✅ Yes |
| **Animatable** | No | No | ✅ Yes |

### 💡 Use cases
- `display: none` → Toggle menus, modals, conditional content
- `visibility: hidden` → Preserve layout, placeholder-like hiding
- `opacity: 0` + transition → Fade animations

---

## 11. Architecting for Multiple Devices

### 🧠 What is it?
Designing an application that works well across **mobile, tablet, desktop, smart TV**, etc.

### 🏗️ The Full Architecture

#### 1. Responsive Design (CSS Layer)
```css
/* Mobile-first approach */
.container { width: 100%; padding: 16px; }

/* Tablet */
@media (min-width: 768px) {
  .container { max-width: 768px; margin: 0 auto; }
}

/* Desktop */
@media (min-width: 1024px) {
  .container { max-width: 1200px; }
}
```

Use **CSS Grid + Flexbox** for fluid layouts. Avoid fixed pixel widths.

#### 2. Adaptive Components
Serve different components based on screen size:
```js
function ProductCard({ isMobile }) {
  return isMobile
    ? <MobileProductCard />
    : <DesktopProductCard />;
}

// Detect device
const isMobile = window.innerWidth < 768;
```

Or use a hook:
```js
function useBreakpoint() {
  const [width, setWidth] = useState(window.innerWidth);
  useEffect(() => {
    const handler = () => setWidth(window.innerWidth);
    window.addEventListener('resize', handler);
    return () => window.removeEventListener('resize', handler);
  }, []);
  return { isMobile: width < 768, isTablet: width < 1024, isDesktop: width >= 1024 };
}
```

#### 3. Design Tokens / Theme System
Use CSS variables so spacing, font sizes, etc. adapt:
```css
:root {
  --font-size-base: 16px;
  --spacing-md: 16px;
}

@media (max-width: 768px) {
  :root {
    --font-size-base: 14px;
    --spacing-md: 12px;
  }
}
```

#### 4. Performance for Mobile
Mobile devices have slower CPUs and connections:
- **Lazy load** images and components
- Use **next-gen image formats** (WebP, AVIF)
- **Code splitting** – don't send desktop bundles to mobile
- Use a **CDN** for static assets
- Implement **Service Workers** for offline support (PWA)

```js
// Lazy load a heavy component
const HeavyChart = React.lazy(() => import('./HeavyChart'));

// Responsive images
<img
  srcSet="img-400.jpg 400w, img-800.jpg 800w, img-1200.jpg 1200w"
  sizes="(max-width: 600px) 400px, (max-width: 1200px) 800px, 1200px"
  src="img-800.jpg"
  alt="Product"
/>
```

#### 5. Backend Considerations
- **BFF (Backend for Frontend)** pattern: Separate API endpoints for mobile vs desktop — mobile gets lighter payloads
- Use **GraphQL** so clients request only what they need
- Return device-appropriate data sizes

```
Mobile Client ──→ BFF (Mobile API) ──→ Core Services
Desktop Client ──→ BFF (Desktop API) ──→ Core Services
```

#### 6. Touch vs Mouse Interactions
```css
/* Increase tap targets for mobile (min 44x44px) */
@media (pointer: coarse) {
  button { min-height: 44px; min-width: 44px; }
}

/* Hover effects only for mouse users */
@media (hover: hover) {
  .card:hover { box-shadow: 0 4px 12px rgba(0,0,0,0.1); }
}
```

### 🗺️ Architecture Diagram
```
┌─────────────────────────────────────────┐
│           Client Layer                   │
│  Mobile App   Web App   Desktop App      │
│  (React Native) (React)  (Electron)      │
└────────────────┬────────────────────────┘
                 │ API Calls
┌────────────────▼────────────────────────┐
│     BFF (Backend for Frontend)           │
│  /api/mobile     /api/web                │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│          Core Services / APIs            │
│    Auth  │  Products  │  Orders  │ ...   │
└─────────────────────────────────────────┘
```

### 💡 Key Points for Interview
1. **Mobile-first CSS** – Start with mobile styles, use `min-width` breakpoints
2. **Responsive + Adaptive** – CSS handles layout, JS handles logic differences
3. **Performance is different per device** – lazy load, code split, responsive images
4. **Touch targets** – Minimum 44x44px on mobile
5. **BFF pattern** – Tailored APIs reduce over-fetching on mobile

---

## 🎓 Quick Revision Summary

| Topic | 1-line Answer |
|---|---|
| Flatten array | `arr.flat(Infinity)` or recursion with `reduce` |
| Feature flags | Toggle features via remote config — no deployment needed |
| Microfrontends | Split UI into independently deployable mini-apps |
| Elevator DSA | Min-Heap + SCAN algorithm for nearest floor in direction |
| Merge Sort | Divide array, sort halves recursively, merge — O(n log n) |
| Kth Largest | Min-Heap of size K or QuickSelect |
| SOLID | SRP, OCP, LSP, ISP, DIP — principles for clean OOP design |
| OOP in JS | Classes, `extends`, `#private`, Encapsulation/Inheritance/Polymorphism |
| Semantic HTML | Tags with meaning: `<article>`, `<nav>`, `<main>` etc. |
| display vs visibility | `display:none` removes space; `visibility:hidden` preserves it |
| Multi-device arch | Mobile-first CSS + responsive images + BFF pattern + lazy loading |

---

> 💪 **Good luck with your interview!** Review code examples out loud to simulate the interview environment.
