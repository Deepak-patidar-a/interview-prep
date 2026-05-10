# DSA Revision — Key Problems

A quick-reference guide covering four classic algorithm problems with code, complexity, and key insights.

---

## Table of Contents

1. [Longest Substring Without Repeating Characters](#1-longest-substring-without-repeating-characters)
2. [Next Greater Element](#2-next-greater-element)
3. [Reverse a Linked List](#3-reverse-a-linked-list)
4. [Two Sum & Three Sum](#4-two-sum--three-sum)

---

## 1. Longest Substring Without Repeating Characters

**Problem:** Given a string, find the longest substring that contains no duplicate characters.

**Examples:**
| Input | Output | Length |
|-------|--------|--------|
| `"abcabcbb"` | `"abc"` | 3 |
| `"bbbbb"` | `"b"` | 1 |
| `"pwwkew"` | `"wke"` | 3 |

**Approach: Sliding Window + Set**

Use two pointers (`left`, `right`) to define a window. Expand `right` each step. If the new character already exists in the window, shrink from `left` until the duplicate is removed.

```javascript
function longestSubstringWithoutRepeating(s) {
  let left = 0;
  let maxLen = 0;
  let bestStart = 0;
  const seen = new Set();

  for (let right = 0; right < s.length; right++) {
    // Shrink window from left until no duplicate
    while (seen.has(s[right])) {
      seen.delete(s[left]);
      left++;
    }

    seen.add(s[right]);

    if (right - left + 1 > maxLen) {
      maxLen = right - left + 1;
      bestStart = left;
    }
  }

  const result = s.substring(bestStart, bestStart + maxLen);
  return { substring: result, length: maxLen };
}
```

**Complexity:**
- Time: **O(n)** — each character visited at most twice
- Space: **O(min(n, m))** — where `m` is the character set size

**Key Insight:** The window always contains unique characters. The `Set` gives O(1) lookup to check for duplicates instantly.

---

## 2. Next Greater Element

**Problem:** For each element in an array, find the first greater element to its right. If none exists, return `-1`.

**Examples:**
| Input | Output |
|-------|--------|
| `[4, 5, 2, 10, 8]` | `[5, 10, 10, -1, -1]` |
| `[3, 1, 4, 1, 5, 9, 2, 6]` | `[4, 4, 5, 5, 9, -1, 6, -1]` |
| `[5, 4, 3, 2, 1]` | `[-1, -1, -1, -1, -1]` |

**Approach: Monotonic Stack**

Use a stack to keep track of indices whose "next greater" answer is still unknown. For every new element, pop everything from the stack that is smaller — those elements have found their answer.

```javascript
function nextGreaterElement(arr) {
  const n = arr.length;
  const result = new Array(n).fill(-1);
  const stack = []; // stores indices

  for (let i = 0; i < n; i++) {
    // Current element is the answer for all smaller elements on the stack
    while (stack.length > 0 && arr[stack[stack.length - 1]] < arr[i]) {
      const idx = stack.pop();
      result[idx] = arr[i];
    }
    stack.push(i);
  }
  // Remaining indices in stack have no greater element → stay -1

  return result;
}
```

**Complexity:**
- Time: **O(n)** — each element pushed and popped at most once
- Space: **O(n)** — for the stack and result array

**Key Insight:** The stack is always in decreasing order (monotonic). The moment you see a larger number, you resolve all the smaller pending elements in one sweep.

---

## 3. Reverse a Linked List

**Problem:** Given the head of a singly linked list, reverse it in place and return the new head.

**Example:**
```
Input:  1 → 2 → 3 → 4 → 5 → null
Output: 5 → 4 → 3 → 2 → 1 → null
```

### Iterative Approach (Recommended)

Walk through the list and flip each `next` pointer to point backwards. Three pointers do all the work: `prev`, `current`, `next`.

```javascript
function reverseLinkedList(head) {
  let prev = null;
  let current = head;

  while (current !== null) {
    let next = current.next;  // 1. Save next node
    current.next = prev;      // 2. Flip the pointer
    prev = current;           // 3. Advance prev
    current = next;           // 4. Advance current
  }

  return prev; // new head
}
```

**The four-step dance at every node:**
```
next    = current.next   ← save before overwriting
current.next = prev      ← THE flip
prev    = current        ← slide forward
current = next           ← slide forward
```

### Recursive Approach

```javascript
function reverseRecursive(head) {
  if (!head || !head.next) return head; // base case

  const newHead = reverseRecursive(head.next); // reverse the rest
  head.next.next = head; // make next node point back
  head.next = null;      // cut forward pointer
  return newHead;
}
```

**Complexity:**
| | Iterative | Recursive |
|--|-----------|-----------|
| Time | O(n) | O(n) |
| Space | **O(1)** ✅ | O(n) call stack |

**Key Insight:** Iterative is preferred — same time but constant space. Recursive risks stack overflow on very long lists.

---

## 4. Two Sum & Three Sum

### Two Sum

**Problem:** Given an array and a target, return the indices of the two numbers that add up to the target.

**Examples:**
| Input | Target | Output |
|-------|--------|--------|
| `[2, 7, 11, 15]` | `9` | `[0, 1]` |
| `[3, 2, 4]` | `6` | `[1, 2]` |
| `[1, 5, 3, 7]` | `8` | `[0, 3]` |

**Approach: Hash Map**

For each element, calculate its complement (`target - num`). Check if that complement is already stored in the map. If yes — done. If no — store the current value and move on.

```javascript
function twoSum(arr, target) {
  const map = new Map(); // value → index

  for (let i = 0; i < arr.length; i++) {
    const complement = target - arr[i];

    if (map.has(complement)) {
      return [map.get(complement), i];
    }
    map.set(arr[i], i);
  }

  return null;
}
```

**Complexity:**
- Time: **O(n)** — single pass
- Space: **O(n)** — for the hash map

---

### Three Sum (calls Two Sum internally)

**Problem:** Find all unique triplets `[a, b, c]` in the array such that `a + b + c = 0`.

**Examples:**
| Input | Output |
|-------|--------|
| `[-1, 0, 1, 2, -1, -4]` | `[[-1,-1,2], [-1,0,1]]` |
| `[0, 0, 0]` | `[[0,0,0]]` |
| `[1, 2, 3]` | `[]` |

**Approach: Sort + Fix Anchor + Call twoSum**

Sort the array. For each element as the "anchor" (`a`), call `twoSum` on the remaining subarray with `target = -a`. Sorting lets us skip duplicate anchors cleanly.

```javascript
function threeSum(arr) {
  arr.sort((a, b) => a - b);
  const result = [];

  for (let i = 0; i < arr.length - 2; i++) {
    // Skip duplicate anchors
    if (i > 0 && arr[i] === arr[i - 1]) continue;

    const anchor = arr[i];
    const target = -anchor;       // need b + c = -anchor
    const subArr = arr.slice(i + 1);

    const pair = twoSum(subArr, target); // ← reusing twoSum!

    if (pair !== null) {
      const [j, k] = pair;
      const triplet = [anchor, subArr[j], subArr[k]];
      const key = triplet.join(',');
      if (!result.some(t => t.join(',') === key)) {
        result.push(triplet);
      }
    }
  }

  return result;
}
```

**Complexity:**
- Time: **O(n²)** — outer loop O(n) × inner twoSum O(n)
- Space: **O(n)** — for the map inside each twoSum call

**Key Insight:** Three Sum reduces to Two Sum by fixing one element. You cannot do better than O(n²) for the general Three Sum problem.

---

## Quick Complexity Cheatsheet

| Problem | Approach | Time | Space |
|---------|----------|------|-------|
| Longest Substring | Sliding Window + Set | O(n) | O(n) |
| Next Greater Element | Monotonic Stack | O(n) | O(n) |
| Reverse Linked List | Iterative (3 pointers) | O(n) | O(1) |
| Two Sum | Hash Map | O(n) | O(n) |
| Three Sum | Sort + Fix + Two Sum | O(n²) | O(n) |

---

## Patterns to Remember

| Pattern | Problems it solves |
|---------|--------------------|
| **Sliding Window** | Substring problems, max/min subarrays |
| **Monotonic Stack** | Next greater/smaller, histogram problems |
| **Three pointers** | Reversals, in-place linked list operations |
| **Hash Map** | Two Sum, frequency counting, anagram detection |
| **Reduce to simpler** | Three Sum → Two Sum, k-Sum → (k-1)-Sum |
