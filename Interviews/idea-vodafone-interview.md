# 📝 JavaScript & React Interview Questions — Idea/Vodafone Round

---

## Table of Contents
- [Q1. Prime Numbers 1 to 1000](#q1-prime-numbers-1-to-1000)
- [Q2. Overall Topper + Subject-wise Topper](#q2-overall-topper--subject-wise-topper)
- [Q3. Search with Debounce in React](#q3-search-with-debounce-in-react)

---

## Q1. Prime Numbers 1 to 1000

### My Answer
- Was not able to solve it in the interview

### ✅ Simple Solution

```javascript
function getPrimes(limit) {
  const primes = [];

  for (let i = 2; i <= limit; i++) {    // 1 is not prime, start from 2
    let isPrime = true;

    for (let j = 2; j < i; j++) {       // check if anything divides i
      if (i % j === 0) {
        isPrime = false;
        break;                           // no need to check further
      }
    }

    if (isPrime) primes.push(i);
  }

  return primes;
}

console.log(getPrimes(1000));
```

### ✅ Optimized Solution (impress the interviewer)

```javascript
function getPrimes(limit) {
  const primes = [];

  for (let i = 2; i <= limit; i++) {
    let isPrime = true;

    for (let j = 2; j <= Math.sqrt(i); j++) {  // only check up to √i
      if (i % j === 0) {
        isPrime = false;
        break;
      }
    }

    if (isPrime) primes.push(i);
  }

  return primes;
}
```

**Why `Math.sqrt(i)` optimization works:**
If a number has a factor larger than its square root, it must also have one *smaller* than it. So we only need to check up to √i — cuts unnecessary iterations significantly.

---

## Q2. Overall Topper + Subject-wise Topper

### My Answer
- Solved subject-wise topper correctly
- Took time on overall topper — got confused on approach

### Input Data
```js
const list = [
  { name: "sam",  subject: "math",    marks: 97 },
  { name: "sam",  subject: "physics", marks: 37 },
  { name: "sam",  subject: "chem",    marks: 27 },
  { name: "sam1", subject: "math",    marks: 77 },
  { name: "sam1", subject: "physics", marks: 37 },
  { name: "sam1", subject: "chem",    marks: 67 },
  { name: "sam2", subject: "math",    marks: 47 },
  { name: "sam2", subject: "physics", marks: 67 },
  { name: "sam2", subject: "chem",    marks: 27 },
];
```

### ✅ Solution 1 — For Loop (interview friendly, easy to explain)

```javascript
// ── PART 1: Subject-wise topper ──────────────────────────

const subjectWiseTopper = {};

for (let i = 0; i < list.length; i++) {
  const { subject, name, marks } = list[i];

  // If subject not seen yet, or current marks are higher → update
  if (!subjectWiseTopper[subject] || marks > subjectWiseTopper[subject].marks) {
    subjectWiseTopper[subject] = { name, marks };
  }
}

console.log("Subject-wise Toppers:", subjectWiseTopper);
// { math: { name: 'sam', marks: 97 }, physics: { name: 'sam2', marks: 67 }, chem: { name: 'sam1', marks: 67 } }


// ── PART 2: Overall topper ────────────────────────────────

// Step 1 — sum total marks per student
const totalMarks = {};

for (let i = 0; i < list.length; i++) {
  const { name, marks } = list[i];

  if (!totalMarks[name]) {
    totalMarks[name] = 0;       // first time seeing this student
  }
  totalMarks[name] += marks;    // keep adding marks
}

console.log("Total Marks:", totalMarks);
// { sam: 161, sam1: 181, sam2: 141 }

// Step 2 — find who has the highest total
let overallTopper = { name: "", marks: 0 };

for (const name in totalMarks) {
  if (totalMarks[name] > overallTopper.marks) {
    overallTopper = { name, marks: totalMarks[name] };
  }
}

console.log("Overall Topper:", overallTopper);
// { name: 'sam1', marks: 181 }
```

### ✅ Solution 2 — Using `reduce` (cleaner, advanced)

```javascript
// ── PART 1: Subject-wise topper ──────────────────────────

const subjectWiseTopper = list.reduce((acc, curr) => {
  const { subject, name, marks } = curr;

  if (!acc[subject] || marks > acc[subject].marks) {
    acc[subject] = { name, marks };
  }

  return acc;
}, {});

console.log("Subject-wise Toppers:", subjectWiseTopper);


// ── PART 2: Overall topper ────────────────────────────────

// Step 1 — sum marks per student
const totalMarks = list.reduce((acc, curr) => {
  acc[curr.name] = (acc[curr.name] || 0) + curr.marks;
  return acc;
}, {});

// Step 2 — find highest total
const overallTopper = Object.entries(totalMarks).reduce(
  (best, [name, marks]) => marks > best.marks ? { name, marks } : best,
  { name: "", marks: 0 }
);

console.log("Overall Topper:", overallTopper);
// { name: 'sam1', marks: 181 }
```

### ⚠️ Key Insight — Why Overall Topper Needs Two Steps
```
Step 1 → Aggregate  (sum marks per student)   { sam: 161, sam1: 181, sam2: 141 }
Step 2 → Compare    (find max from totals)     { name: 'sam1', marks: 181 }
```
Trying to do it in one step is where most people get confused. Always split it.

> 💡 For loop is perfectly fine in interviews — use reduce only if you're comfortable explaining it.

---

## Q3. Search with Debounce in React

### My Answer
- Was able to solve it

### ✅ Solution

```jsx
import { useState, useEffect } from "react";

// Reusable debounce hook
function useDebounce(value, delay = 500) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    // Start timer — only updates debouncedValue after delay
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    // Cleanup — cancel previous timer if value changes again
    return () => clearTimeout(timer);

  }, [value, delay]);

  return debouncedValue;
}

// Main component
function SearchComponent() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);

  const debouncedQuery = useDebounce(query, 500);

  useEffect(() => {
    if (!debouncedQuery) {
      setResults([]);
      return;
    }

    const fetchResults = async () => {
      setLoading(true);
      const res = await fetch(`/api/search?q=${debouncedQuery}`);
      const data = await res.json();
      setResults(data);
      setLoading(false);
    };

    fetchResults();
  }, [debouncedQuery]); // only fires after user stops typing

  return (
    <div>
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      {loading && <p>Loading...</p>}
      <ul>
        {results.map((item) => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

### How Debounce Works — Full Workflow
```
User types "r"     → timer #1 starts (500ms)
User types "re"    → timer #1 cancelled ❌ → timer #2 starts
User types "rea"   → timer #2 cancelled ❌ → timer #3 starts
User stops typing  → timer #3 completes ✅
                   → debouncedQuery = "rea"
                   → API call fires ONCE 🚀
```

### Two Values — The Key Difference
```
query          → "r" → "re" → "rea" → "react"   (updates on every keystroke)
debouncedQuery → ""  →  ""  →  ""   → "react"   (updates only after 500ms silence)
```
Input shows `query` — fast and responsive.
API call uses `debouncedQuery` — controlled, fires once.

### Why Cleanup Function Matters
```js
return () => clearTimeout(timer);
```
Without this — old timers stack up and multiple API calls fire. The cleanup cancels the previous timer every time the user types a new character.

---

## 🔑 Quick Cheat Sheet

| Topic | Key Point |
|-------|-----------|
| Prime numbers | Check only up to **√i** for optimization |
| Overall topper | Always **two steps** — aggregate first, then compare |
| Debounce | `clearTimeout` cleanup is the key — cancels previous timer |
| For loop vs reduce | For loop is fine in interviews — readable and easy to explain |
