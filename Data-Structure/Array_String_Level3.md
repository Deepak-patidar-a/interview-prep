# 💻 Coding Interview Questions – Revision Guide

> Deep explanations, visual walkthroughs, bug fixes, and optimisations for each problem.

---

## Table of Contents

1. [Valid Anagram](#1-valid-anagram)
2. [Valid Palindrome](#2-valid-palindrome)
3. [Jewels and Stones](#3-jewels-and-stones)
4. [Longest Common Prefix](#4-longest-common-prefix)
5. [Group Anagrams](#5-group-anagrams)
6. [Top K Frequent Elements](#6-top-k-frequent-elements)
7. [3Sum](#7-3sum)

---

## 1. Valid Anagram

### 🧠 Problem
Given two strings `s` and `t`, return `true` if `t` is an anagram of `s`.

```
s = "anagram"
t = "nagaram"
→ true  ✅ (same letters, different order)

s = "rat"
t = "car"
→ false ❌
```

### 💡 Intuition
An anagram uses **exactly the same letters, the same number of times**. So if we count the frequency of each character in both strings, they must match perfectly.

---

### 🔑 Approach 1: Two Maps (Brute Force)
Build a frequency map for `s`, build another for `t`, then compare.

```js
const anagram = (s, t) => {
  if (s.length !== t.length) return false;

  let sMap = {};
  let tMap = {};

  for (let char of s) sMap[char] = (sMap[char] || 0) + 1;
  for (let char of t) tMap[char] = (tMap[char] || 0) + 1;

  for (let key in sMap) {
    if (sMap[key] !== tMap[key]) return false;
  }
  return true;
};
```

**Time:** O(n) | **Space:** O(n) — but uses two maps

---

### ⚡ Approach 2: One Map (Efficient — your solution) ✅
Build only one map for `s`, then **decrement** it while iterating `t`. If any count goes missing or negative → not an anagram.

```js
const anagram = (s, t) => {
  if (s.length !== t.length) return false;

  let sMap = {};

  // Count frequency of each char in s
  for (let i = 0; i < s.length; i++) {
    sMap[s[i]] = (sMap[s[i]] || 0) + 1;
  }

  // Decrement for each char in t
  for (let i = 0; i < t.length; i++) {
    if (!sMap[t[i]]) return false;  // char not found or already 0
    sMap[t[i]] -= 1;
    if (sMap[t[i]] < 0) return false; // used more than available
  }

  return true;
};
```

### 🎨 Visual Walkthrough
```
s = "anagram", t = "nagaram"

Build sMap from s:
{ a: 3, n: 1, g: 1, r: 1, m: 1 }

Iterate t = "nagaram":
  n → sMap[n] = 0  ✅
  a → sMap[a] = 2  ✅
  g → sMap[g] = 0  ✅
  a → sMap[a] = 1  ✅
  r → sMap[r] = 0  ✅
  a → sMap[a] = 0  ✅
  m → sMap[m] = 0  ✅

All passed → return true ✅
```

**Time:** O(n) | **Space:** O(1) — at most 26 keys (alphabet)

---

### 🚀 Bonus: Cleanest One-liner
```js
const anagram = (s, t) =>
  s.split("").sort().join("") === t.split("").sort().join("");
// Time: O(n log n) — sorting is slower, but ultra-readable
```

---

## 2. Valid Palindrome

### 🧠 Problem
A phrase is a palindrome if it reads the same forwards and backwards, **ignoring case, spaces, and non-alphanumeric characters**.

```
"A man, a plan, a canal: Panama"
→ "amanaplanacanalpanama" (filtered)
→ true ✅

"race a car"
→ "raceacar"
→ false ❌
```

---

### 🐛 Bug Alert in Your Code!

Your original code has **two bugs**. Let's find and fix them.

```js
// ❌ BUGGY VERSION
const validPalindrom = (str) => {
  let str = s.toLowerCase()  // Bug 1: parameter is 'str' but you used 's'
                              //        also re-declaring 'str' with 'let' shadows param
  ...
}

const validPalindromWithTwoPointer = (str) => {
  let str = str.toLowerCase()  // Bug 2: same issue — can't re-declare 'str'
  while(i < j){
    ...
    if(!str[j].match(/[a-z0-9]/j)) {  // Bug 3: regex flag 'j' doesn't exist, should be 'i'
      --j
    }
    if(str[i] !== str[j]) return false
    // Bug 4: Missing i++ j-- after the char comparison!
    // Without this, the while loop is infinite when chars match
  }
}
```

---

### ✅ Approach 1: Filter + Reverse (Your approach, fixed)

```js
const validPalindrome = (s) => {
  let str = s.toLowerCase(); // ✅ use the parameter directly
  let filteredStr = "";
  let rev = "";

  for (let i = 0; i < str.length; i++) {
    if (str[i].match(/[a-z0-9]/i)) {
      filteredStr += str[i];
      rev = str[i] + rev; // prepend to build reverse
    }
  }

  return filteredStr === rev;
};

validPalindrome("A man, a plan, a canal: Panama"); // true ✅
```

**Time:** O(n) | **Space:** O(n)

---

### ⚡ Approach 2: Two Pointer (Optimal — your approach, fixed)

No extra strings needed. Use two pointers converging from both ends.

```js
const validPalindromeWithTwoPointer = (s) => {
  let str = s.toLowerCase(); // ✅ separate variable, no conflict
  let i = 0;
  let j = str.length - 1;

  while (i < j) {
    // Skip non-alphanumeric from left
    if (!str[i].match(/[a-z0-9]/i)) {
      i++;
      continue; // ✅ use continue to restart the loop check
    }

    // Skip non-alphanumeric from right
    if (!str[j].match(/[a-z0-9]/i)) { // ✅ flag is 'i' not 'j'
      j--;
      continue;
    }

    // Both pointers on valid chars — compare them
    if (str[i] !== str[j]) return false;

    i++; // ✅ MUST move both pointers after a match
    j--;
  }

  return true;
};

validPalindromeWithTwoPointer("A man, a plan, a canal: Panama"); // true ✅
```

### 🎨 Visual Walkthrough
```
"amanaplanacanalpanama"
 i                   j

Step 1: a === a ✅ → i++, j--
Step 2: m === m ✅ → i++, j--
Step 3: a === a ✅ → ...
...
i and j meet in the middle → return true ✅
```

**Time:** O(n) | **Space:** O(1) — best solution 🏆

---

### 📋 Bug Summary
| Bug | Problem | Fix |
|-----|---------|-----|
| `let str = s.toLowerCase()` with param `str` | `s` is undefined | Use `let cleaned = s.toLowerCase()` or rename param |
| `let str = str.toLowerCase()` | Can't re-declare a variable with same name | Use a different variable name |
| `/[a-z0-9]/j` | `j` is not a valid regex flag | Use `/[a-z0-9]/i` |
| Missing `i++`, `j--` after comparison | Infinite loop when chars match | Always move both pointers |

---

## 3. Jewels and Stones

### 🧠 Problem
`jewels` is a string of jewel types. `stones` is a string of stones you have. Return how many stones are jewels.

```
jewels = "aA", stones = "aAAbbbb"
→ 3  (the 'a' and two 'A's are jewels)
```

---

### 🔑 Approach 1: HashMap (Your solution)

```js
var numJewelsInStones = function(jewels, stones) {
  let jew = {};
  let count = 0;

  // Mark all jewel types
  for (let i = 0; i < jewels.length; i++) {
    jew[jewels[i]] = true; // we only care if it exists, not frequency
  }

  // Check each stone
  for (let i = 0; i < stones.length; i++) {
    if (jew[stones[i]]) count++;
  }

  return count;
};
```

**Time:** O(j + s) | **Space:** O(j) where j = jewels length, s = stones length

---

### ⚡ Approach 2: Set (Cleaner — optimal) ✅

```js
var numJewelsInStones = function(jewels, stones) {
  let jewelSet = new Set(jewels); // Set gives O(1) lookup
  let count = 0;

  for (let stone of stones) {
    if (jewelSet.has(stone)) count++;
  }

  return count;
};
```

### 💡 Why Set over HashMap here?
- We only need to know **if a jewel type exists** — not how many times
- `Set.has()` is O(1) and semantically cleaner than a map
- `new Set(jewels)` is a one-liner to build the lookup

### 🚀 One-liner version
```js
var numJewelsInStones = (jewels, stones) => {
  const s = new Set(jewels);
  return [...stones].filter(c => s.has(c)).length;
};
```

**Time:** O(j + s) | **Space:** O(j)

---

## 4. Longest Common Prefix

### 🧠 Problem
Find the longest common prefix string among an array of strings.

```
["flower", "flow", "flight"]
→ "fl"

["dog", "racecar", "car"]
→ ""  (no common prefix)
```

---

### 🔑 Your Solution (Vertical Scanning) ✅

Take the first string as the reference. For each character position, check if all other strings have the same character at that position.

```js
var longestCommonPrefix = function(strs) {
  let commonPrefix = "";

  // Iterate through each character of the first string
  for (let i = 0; i < strs[0].length; i++) {
    // Compare this character with all other strings at position i
    for (let j = 1; j < strs.length; j++) {
      if (strs[0][i] !== strs[j][i]) {
        return commonPrefix; // mismatch found, return what we have
      }
    }
    commonPrefix += strs[0][i]; // all matched, add to prefix
  }

  return commonPrefix;
};
```

### 🎨 Visual Walkthrough
```
strs = ["flower", "flow", "flight"]

i=0: 'f' === 'f' === 'f' ✅ → prefix = "f"
i=1: 'l' === 'l' === 'l' ✅ → prefix = "fl"
i=2: 'o' === 'o' but 'o' !== 'i' ❌ → return "fl"
```

**Time:** O(S) where S = total characters | **Space:** O(1)

---

### ⚡ Approach 2: Horizontal Scanning (Alternative)

```js
var longestCommonPrefix = function(strs) {
  let prefix = strs[0];

  for (let i = 1; i < strs.length; i++) {
    // Keep trimming the prefix until current string starts with it
    while (!strs[i].startsWith(prefix)) {
      prefix = prefix.slice(0, -1); // remove last character
      if (prefix === "") return "";
    }
  }

  return prefix;
};
```

### 🎨 Visual Walkthrough
```
prefix = "flower"

Compare with "flow":
  "flow".startsWith("flower")? No → prefix = "flowe"
  "flow".startsWith("flowe")? No → prefix = "flow"
  "flow".startsWith("flow")? Yes ✅

Compare with "flight":
  "flight".startsWith("flow")? No → prefix = "flo"
  "flight".startsWith("flo")? No → prefix = "fl"
  "flight".startsWith("fl")? Yes ✅

return "fl" ✅
```

### 💡 Edge Cases to Handle
```js
if (!strs || strs.length === 0) return "";
// Empty array guard
```

---

## 5. Group Anagrams

### 🧠 Problem
Group words that are anagrams of each other.

```
["eat","tea","tan","ate","nat","bat"]
→ [["eat","tea","ate"], ["tan","nat"], ["bat"]]
```

---

### 🔑 Your Solution ✅ (Sort as Key)

The key insight: **anagrams sort to the same string**. Use the sorted word as the map key.

```js
const groupAnagram = (strs) => {
  let map = {};

  for (let i = 0; i < strs.length; i++) {
    // Sort the word to create a canonical "anagram key"
    let sortedStr = strs[i].split("").sort().join("");

    if (!map[sortedStr]) {
      map[sortedStr] = [strs[i]];    // create new group
    } else {
      map[sortedStr].push(strs[i]);  // add to existing group
    }
  }

  return Object.values(map); // return only the grouped arrays
};
```

### 🎨 Visual Walkthrough
```
Word      Sorted    Map State
"eat"  →  "aet"   { aet: ["eat"] }
"tea"  →  "aet"   { aet: ["eat","tea"] }
"tan"  →  "ant"   { aet: ["eat","tea"], ant: ["tan"] }
"ate"  →  "aet"   { aet: ["eat","tea","ate"], ant: ["tan"] }
"nat"  →  "ant"   { aet: ["eat","tea","ate"], ant: ["tan","nat"] }
"bat"  →  "abt"   { aet: [...], ant: [...], abt: ["bat"] }

Object.values → [["eat","tea","ate"], ["tan","nat"], ["bat"]] ✅
```

**Time:** O(n × k log k) where k = max word length | **Space:** O(n × k)

---

### ⚡ Approach 2: Character Count as Key (Faster for long words)

Instead of sorting, use a frequency count array as the key. Avoids the O(k log k) sort.

```js
const groupAnagrams = (strs) => {
  const map = new Map();

  for (let word of strs) {
    // Create a count array for 26 letters
    const count = new Array(26).fill(0);

    for (let char of word) {
      count[char.charCodeAt(0) - 97]++; // 'a' = 97
    }

    const key = count.join("#"); // "1#0#0#...#1#..." — unique per anagram group

    if (!map.has(key)) map.set(key, []);
    map.get(key).push(word);
  }

  return [...map.values()];
};
```

**Time:** O(n × k) | **Space:** O(n × k) — optimal! 🏆

### 💡 Which to use in interview?
- **Sorting approach** → simpler to explain and code
- **Count approach** → more optimal, impressive if you can explain it

---

## 6. Top K Frequent Elements

### 🧠 Problem
Return the `k` most frequent elements.

```
nums = [1,1,1,2,2,3], k = 2
→ [1, 2]  (1 appears 3x, 2 appears 2x)
```

---

### 🔑 Your Solution ✅ (HashMap + Sort)

```js
var topKFrequent = function(nums, k) {
  let map = {};

  // Step 1: Count frequency of each number
  for (let num of nums) {
    map[num] = (map[num] || 0) + 1;
  }

  // Step 2: Convert to [num, freq] pairs
  // Object.entries gives [["1", 3], ["2", 2], ["3", 1]]
  let arr = Object.entries(map);

  // Step 3: Sort by frequency descending
  arr.sort((a, b) => b[1] - a[1]);

  // Step 4: Take top k and return just the numbers
  return arr.slice(0, k).map(item => Number(item[0]));
};
```

### 🎨 Visual Walkthrough
```
nums = [1,1,1,2,2,3], k = 2

Step 1: map = { 1: 3, 2: 2, 3: 1 }

Step 2: arr = [["1",3], ["2",2], ["3",1]]

Step 3: sorted = [["1",3], ["2",2], ["3",1]]

Step 4: slice(0,2) → [["1",3], ["2",2]]
        .map → [1, 2] ✅
```

**Time:** O(n log n) for sorting | **Space:** O(n)

---

### ⚡ Approach 2: Bucket Sort (Optimal — O(n)) 🏆

The key insight: frequency can be **at most `n`** (if all elements are the same). So we can use frequency as the array index!

```js
var topKFrequent = function(nums, k) {
  const freqMap = {};

  // Step 1: Count frequencies
  for (let num of nums) {
    freqMap[num] = (freqMap[num] || 0) + 1;
  }

  // Step 2: Create buckets indexed by frequency
  // bucket[i] = list of numbers that appear exactly i times
  const bucket = Array.from({ length: nums.length + 1 }, () => []);

  for (let num in freqMap) {
    const freq = freqMap[num];
    bucket[freq].push(Number(num));
  }

  // Step 3: Collect top k from highest frequency buckets
  const result = [];
  for (let i = bucket.length - 1; i >= 0 && result.length < k; i--) {
    result.push(...bucket[i]);
  }

  return result.slice(0, k);
};
```

### 🎨 Visual Walkthrough (Bucket Sort)
```
nums = [1,1,1,2,2,3], k = 2

freqMap = { 1:3, 2:2, 3:1 }

Buckets (index = frequency):
  bucket[0] = []
  bucket[1] = [3]   ← 3 appears 1 time
  bucket[2] = [2]   ← 2 appears 2 times
  bucket[3] = [1]   ← 1 appears 3 times

Scan from right (highest freq first):
  bucket[3] → [1]    result = [1]
  bucket[2] → [2]    result = [1, 2]  ← k=2, done!

→ [1, 2] ✅
```

**Time:** O(n) | **Space:** O(n) — optimal! 🏆

### 📊 Comparison
| Approach | Time | Space | Notes |
|---|---|---|---|
| HashMap + Sort | O(n log n) | O(n) | Simple, interview-safe |
| Bucket Sort | O(n) | O(n) | Optimal, impressive |
| Min-Heap | O(n log k) | O(k) | Good when k << n |

---

## 7. 3Sum

### 🧠 Problem
Find all unique triplets in the array that sum to **zero**.

```
nums = [-1, 0, 1, 2, -1, -4]
→ [[-1, -1, 2], [-1, 0, 1]]
```

---

### 💡 Intuition: Reduce to Two Sum
Fix one number (`nums[i]`), then use the **two-pointer technique** on the rest of the array to find pairs that sum to `-nums[i]`.

This is why it's called "using the Two Sum method" — 3Sum = fix one + Two Sum on the rest.

---

### 🔑 Solution: Sort + Two Pointers ✅

```js
var threeSum = function(nums) {
  const result = [];
  nums.sort((a, b) => a - b); // Step 1: Sort the array

  for (let i = 0; i < nums.length - 2; i++) {

    // Skip duplicate values for the fixed element
    if (i > 0 && nums[i] === nums[i - 1]) continue;

    // Early exit: smallest possible sum is already > 0
    if (nums[i] > 0) break;

    let left = i + 1;
    let right = nums.length - 1;

    while (left < right) {
      const sum = nums[i] + nums[left] + nums[right];

      if (sum === 0) {
        result.push([nums[i], nums[left], nums[right]]);

        // Skip duplicates for left pointer
        while (left < right && nums[left] === nums[left + 1]) left++;
        // Skip duplicates for right pointer
        while (left < right && nums[right] === nums[right - 1]) right--;

        left++;
        right--;

      } else if (sum < 0) {
        left++;  // need bigger sum → move left pointer right
      } else {
        right--; // need smaller sum → move right pointer left
      }
    }
  }

  return result;
};
```

### 🎨 Visual Walkthrough
```
nums = [-4, -1, -1, 0, 1, 2]  (after sort)

i=0, nums[i]=-4:
  left=1(-1), right=5(2): sum=-3 < 0 → left++
  left=2(-1), right=5(2): sum=-3 < 0 → left++
  left=3(0),  right=5(2): sum=-2 < 0 → left++
  left=4(1),  right=5(2): sum=-1 < 0 → left++
  left=5 not < right=5, stop

i=1, nums[i]=-1:
  left=2(-1), right=5(2): sum=0 ✅ → push [-1,-1,2]
    skip dups: left=3, right=4
  left=3(0),  right=4(1): sum=0 ✅ → push [-1,0,1]
    skip dups: left=4, right=3 → stop

i=2, nums[i]=-1: same as i=1, SKIP (duplicate)

i=3, nums[i]=0:
  left=4(1),  right=5(2): sum=3 > 0 → right--
  right=4, not < left=4, stop

Result: [[-1,-1,2], [-1,0,1]] ✅
```

**Time:** O(n²) — best possible for this problem | **Space:** O(1) excluding output

---

### 🔑 Why Sorting Helps
1. **Duplicates are adjacent** → easy to skip with `if nums[i] === nums[i-1]`
2. **Two-pointer works** → sorted array means we know which direction to move to increase/decrease the sum
3. **Early exit** → if `nums[i] > 0`, the smallest possible sum is already positive

---

### ⚠️ Common Mistakes
```js
// ❌ Mistake 1: Not skipping duplicates → duplicate triplets in output
// ❌ Mistake 2: Forgetting to move both pointers after finding a triplet
// ❌ Mistake 3: Not sorting first → two-pointer won't work
// ❌ Mistake 4: Using i < nums.length instead of i < nums.length - 2
```

---

## 🧩 Patterns Summary

Understanding patterns helps you solve new problems faster:

| Pattern | Problems | When to Use |
|---|---|---|
| **HashMap / Frequency Count** | Anagram, Jewels & Stones, Top K | Count occurrences, fast lookup |
| **Two Pointers** | Palindrome, 3Sum | Sorted array, find pairs/triplets |
| **Sort as Key** | Group Anagrams | Group items with same content |
| **Bucket Sort** | Top K Frequent | Frequency bounded by input size |
| **Fix + Two Sum** | 3Sum | Reduce n-dimension problem by 1 |

---

## ⚡ Complexity Cheat Sheet

| Problem | Time | Space | Best Approach |
|---|---|---|---|
| Valid Anagram | O(n) | O(1) | One HashMap |
| Valid Palindrome | O(n) | O(1) | Two Pointers |
| Jewels & Stones | O(j+s) | O(j) | Set |
| Longest Common Prefix | O(S) | O(1) | Vertical Scan |
| Group Anagrams | O(n·k) | O(n·k) | Char Count Key |
| Top K Frequent | O(n) | O(n) | Bucket Sort |
| 3Sum | O(n²) | O(1) | Sort + Two Pointers |

---

## 🐛 Bugs Fixed in Your Code

| Problem | Bug | Fix |
|---|---|---|
| Valid Palindrome | `let str = s.toLowerCase()` where param is `str` — `s` is undefined | Use `let cleaned = s.toLowerCase()` |
| Valid Palindrome | Re-declaring `str` with `let` in same scope | Use a different variable name |
| Valid Palindrome | Regex flag `/[a-z0-9]/j` — `j` is invalid | Use `/[a-z0-9]/i` |
| Valid Palindrome | Missing `i++; j--` after char comparison | Both pointers must advance after a match |

---

> 💪 **Tip:** When solving, always say your approach out loud first, then code it. Interviewers value clear thinking over fast typing.
