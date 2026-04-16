# 📘 DSA Revision Sheet — Arrays (Part 2)

---

## 1. Remove Duplicates from Sorted Array II (In-Place)

**Problem:** Each unique element can appear **at most twice**. Remove extra duplicates in-place, return the new length.

```
Input:  [1, 1, 1, 2, 2, 3]
Output: 5  →  [1, 1, 2, 2, 3, _]
```

```js
const removeDuplicateInPlace = (nums) => {
  let i = 0;
  for (let j = 0; j < nums.length; j++) {
    if (i < 2 || nums[j] !== nums[i - 2]) {
      nums[i] = nums[j];
      i++;
    }
  }
  return i;
};
```

**Key Points:**
- `i` is the write pointer; `j` is the read pointer.
- The condition `i < 2` handles the first two elements unconditionally — they're always valid.
- `nums[j] !== nums[i - 2]` is the core trick: if the current element differs from the element **two spots behind the write head**, it's safe to include (means we haven't placed it twice yet).
- Works for "at most K times" by replacing `2` with `K`.
- Time: O(n) | Space: O(1)

---

## 2. Best Time to Buy and Sell Stock I

**Problem:** One transaction only (buy once, sell once). Find the maximum profit.

```
Input:  [7, 1, 5, 3, 6, 4]
Output: 5  (buy at 1, sell at 6)

Input:  [7, 6, 4, 3, 1]
Output: 0  (no profitable transaction)
```

```js
const bestTime = (arr) => {
  let profit = 0;
  let min = Infinity;
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] < min) {
      min = arr[i];
    }
    if (arr[i] - min > profit) {
      profit = arr[i] - min;
    }
  }
  return profit;
};
```

**Key Points:**
- Greedy: always try to buy at the lowest price seen so far.
- Use `Infinity` (not `0`) as initial `min` so the first price is always captured.
- If no profit is possible, returns `0` — never goes negative.
- Time: O(n) | Space: O(1)

---

## 3. Best Time to Buy and Sell Stock II

**Problem:** Multiple transactions allowed. You can buy and sell on the same day but can hold **at most one share** at a time.

```
Input:  [7, 1, 5, 3, 6, 4]
Output: 7  (buy at 1 sell at 5 = +4, buy at 3 sell at 6 = +3)

Input:  [1, 2, 3, 4, 5]
Output: 4  (buy at 1, sell at 5)
```

```js
const maxProfit = (prices) => {
  let profit = 0;
  for (let i = 1; i < prices.length; i++) {
    if (prices[i] > prices[i - 1]) {
      profit += prices[i] - prices[i - 1];
    }
  }
  return profit;
};
```

**Key Points:**
- Greedy insight: collecting every upward "step" equals the total peak-to-valley profit.
  - e.g., `[1, 2, 3]` → `(2-1) + (3-2) = 2`, same as buying at 1 and selling at 3.
- No need to simulate actual buy/sell days — just sum all positive differences.
- Time: O(n) | Space: O(1)

**Stock I vs Stock II:**

| | Stock I | Stock II |
|---|---------|----------|
| Transactions | 1 | Unlimited |
| Strategy | Track min, max profit | Sum all upward moves |

---

## 4. Product of Array Except Self

**Problem:** Return an array where `result[i]` is the product of all elements except `nums[i]`. No division allowed, O(n) time.

```
Input:  [1, 2, 3, 4]
Output: [24, 12, 8, 6]
```

```js
const productExceptSelf = (nums) => {
  const n = nums.length;
  const result = new Array(n).fill(1);

  // Pass 1: fill result[i] with product of everything to the LEFT
  let left = 1;
  for (let i = 0; i < n; i++) {
    result[i] = left;
    left *= nums[i];
  }

  // Pass 2: multiply result[i] by product of everything to the RIGHT
  let right = 1;
  for (let i = n - 1; i >= 0; i--) {
    result[i] *= right;
    right *= nums[i];
  }

  return result;
};
```

**Step-by-step trace for `[1, 2, 3, 4]`:**

After **left pass** (result holds prefix products):
```
result = [1, 1, 2, 6]
          ↑  ↑  ↑  ↑
          L  L  L  L  (nothing to left of index 0, so 1)
```

After **right pass** (multiply in suffix products):
```
i=3: result[3] = 6  * 1  = 6,   right = 4
i=2: result[2] = 2  * 4  = 8,   right = 12
i=1: result[1] = 1  * 12 = 12,  right = 24
i=0: result[0] = 1  * 24 = 24

result = [24, 12, 8, 6] ✅
```

**Key Points:**
- Two-pass prefix/suffix trick avoids division entirely.
- `result[i] = leftProduct[i] × rightProduct[i]`
- The same output array is reused for both passes — O(1) extra space (excluding output).
- Time: O(n) | Space: O(1) extra

---

## 🔁 Quick Complexity Summary

| # | Problem | Time | Space | Technique |
|---|---------|------|-------|-----------|
| 1 | Remove Duplicates II | O(n) | O(1) | Two pointers, `i - 2` check |
| 2 | Buy & Sell Stock I | O(n) | O(1) | Greedy, track min |
| 3 | Buy & Sell Stock II | O(n) | O(1) | Greedy, sum upward steps |
| 4 | Product Except Self | O(n) | O(1) | Prefix × Suffix pass |
