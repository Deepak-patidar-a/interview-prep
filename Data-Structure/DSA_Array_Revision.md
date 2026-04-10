# 📘 DSA Revision Sheet — Array & Number Problems (JavaScript)

---

## 1. Largest Element in an Array

**Approach:** Iterate and track the max using `-Infinity` as the initial value (handles negative numbers).

```js
const arr = [5, 0, 7, 10, 8, 17, 1];
let largest = -Infinity;

for (let i = 0; i < arr.length; i++) {
  if (arr[i] > largest) {
    largest = arr[i];
  }
}
// largest = 17
```

**Key Points:**
- Use `-Infinity` (not `0`) to correctly handle arrays with all negative numbers.
- Time: O(n) | Space: O(1)

---

## 2. Second Largest in an Array

**Approach:** Single pass — update `secondLarg` whenever a value is greater than it but not equal to `largest`.

```js
const secondLargest = (arr) => {
  if (!arr.length) return "Empty array";
  if (arr.length === 1) return "Only one element in array";

  let largest = -Infinity;
  let secondLarg = -Infinity;

  for (let i = 0; i < arr.length; i++) {
    if (arr[i] > largest) {
      secondLarg = largest;
      largest = arr[i];
    } else if (arr[i] > secondLarg && arr[i] !== largest) {
      secondLarg = arr[i];
    }
  }
  return secondLarg;
};
```

**Key Points:**
- The `arr[i] !== largest` check handles duplicates (e.g., `[5, 5, 3]` → second largest is `3`).
- Time: O(n) | Space: O(1)

---

## 3. Count of Digits in a Number

**Approach:** Strip sign with `Math.abs()`, then divide by 10 repeatedly until the number reaches 0.

```js
const countDigit = (num) => {
  if (num === 0) return 1;
  num = Math.abs(num); // handles negatives
  let count = 0;
  while (num > 0) {
    num = Math.floor(num / 10);
    count++;
  }
  return count;
};

// countDigit(2345)  → 4
// countDigit(-21)   → 2
```

**Key Points:**
- Special case: `0` has 1 digit.
- `Math.abs()` handles negative numbers.
- Alternatively: `String(Math.abs(num)).length`
- Time: O(log n) | Space: O(1)

---

## 4. Palindrome Number

**Approach:** Reverse the digits and compare with the original.

```js
const palindromNumber = (num) => {
  let copy = num;
  let rev = 0;
  while (num > 0) {
    let lastDigit = num % 10;
    rev = rev * 10 + lastDigit;
    num = Math.floor(num / 10);
  }
  return rev === copy;
};

// palindromNumber(121) → true
// palindromNumber(123) → false
```

**Key Points:**
- Negative numbers are never palindromes (due to the `-` sign).
- Time: O(log n) | Space: O(1)

---

## 5. Remove Duplicates from a Sorted Array (In-Place)

**Approach:** Two-pointer technique — `x` marks the last unique position, `i` scans ahead.

```js
const removeDuplicate = (a) => {
  let x = 0;
  for (let i = 0; i < a.length; i++) {
    if (a[i] > a[x]) {
      x++;
      a[x] = a[i];
    }
  }
  return a;
};

// [0,0,1,1,1,2,2,3,3,4] → [0,1,2,3,4, ...]
```

**Alternatives:**
```js
// Using Set (order preserved)
[...new Set(arr)]

// Using filter
arr.filter((item, index) => arr.indexOf(item) === index)
```

**Key Points:**
- The two-pointer approach is O(n) time and O(1) space — best for in-place.
- `Set` and `filter` approaches work but use extra space.

---

## 6. Remove Element In-Place

**Approach:** Two-pointer — overwrite matching values by shifting non-matching ones to the front.

```js
const removeElement = (nums, val) => {
  let x = 0;
  for (let i = 0; i < nums.length; i++) {
    if (nums[i] !== val) {
      nums[x] = nums[i];
      x++;
    }
  }
  return nums;
};

// removeElement([0,1,1,2,3,4], 3) → [0,1,1,2,4,4] (first 5 elements valid)
```

**Alternative (mutates original):**
```js
nums.splice(nums.indexOf(val), 1);
```

**Key Points:**
- The in-place approach doesn't actually shorten the array; elements beyond index `x` are stale.
- `splice` is cleaner but O(n) for shifting.
- Time: O(n) | Space: O(1)

---

## 7. Best Time to Buy and Sell Stock

**Approach (Greedy):** Track the minimum price seen so far; at each step check if selling now beats the current best profit.

```js
const bestTime = (prices) => {
  let min = Infinity;
  let profit = 0;
  for (let i = 0; i < prices.length; i++) {
    if (prices[i] < min) {
      min = prices[i];
    }
    if (prices[i] - min > profit) {
      profit = prices[i] - min;
    }
  }
  return profit;
};

// bestTime([7,1,5,3,6,4]) → 5  (buy at 1, sell at 6)
// bestTime([7,6,4,3,1])   → 0  (no profit possible)
```

**Key Points:**
- Only one transaction allowed (buy once, sell once).
- Use `Infinity` as initial `min` (not `0`) so the first price is always captured.
- Time: O(n) | Space: O(1)

---

## 🔁 Quick Complexity Summary

| # | Problem | Time | Space | Technique |
|---|---------|------|-------|-----------|
| 1 | Largest element | O(n) | O(1) | Linear scan |
| 2 | Second largest | O(n) | O(1) | Linear scan, two vars |
| 3 | Count digits | O(log n) | O(1) | Divide by 10 |
| 4 | Palindrome number | O(log n) | O(1) | Reverse digits |
| 5 | Remove duplicates (sorted) | O(n) | O(1) | Two pointers |
| 6 | Remove element | O(n) | O(1) | Two pointers |
| 7 | Best time to buy/sell | O(n) | O(1) | Greedy |
