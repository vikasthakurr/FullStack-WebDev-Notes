# Arrays & Strings

## Array Basics in JavaScript

Arrays in JavaScript are **dynamic** — they automatically resize as elements are added or removed. Under the hood, JavaScript engines optimize arrays differently depending on usage (dense vs sparse, homogeneous vs heterogeneous).

```javascript
// Creating arrays
const nums = [1, 2, 3, 4, 5];
const mixed = [1, "hello", true, null]; // JS allows mixed types (avoid in interviews)
const empty = new Array(10); // Creates array with 10 empty slots

// Dynamic sizing
const arr = [];
arr.push(1); // [1]
arr.push(2); // [1, 2] — grows automatically
```

---

## Common Operations and Their Complexities

| Operation         | Method                  | Time Complexity | Why                              |
| ----------------- | ----------------------- | --------------- | -------------------------------- |
| Access by index   | `arr[i]`                | O(1)            | Direct memory offset calculation |
| Push (end)        | `arr.push(val)`         | O(1) amortized  | Append to end, occasional resize |
| Pop (end)         | `arr.pop()`             | O(1)            | Remove last element              |
| Unshift (start)   | `arr.unshift(val)`      | O(n)            | Must shift all elements right    |
| Shift (start)     | `arr.shift()`           | O(n)            | Must shift all elements left     |
| Insert (middle)   | `arr.splice(i, 0, val)` | O(n)            | Must shift elements after index  |
| Delete (middle)   | `arr.splice(i, 1)`      | O(n)            | Must shift elements to fill gap  |
| Search (unsorted) | `arr.indexOf(val)`      | O(n)            | Must check each element          |
| Search (sorted)   | Binary search           | O(log n)        | Halve search space each step     |
| Copy              | `arr.slice()`           | O(n)            | Must copy each element           |
| Sort              | `arr.sort()`            | O(n log n)      | TimSort algorithm                |

---

## String Methods and Immutability

Strings in JavaScript are **immutable** — every operation creates a new string.

```javascript
let str = "hello";
str[0] = "H"; // Does nothing — strings are immutable
str = "H" + str.slice(1); // Creates a new string "Hello"
```

**Important for interviews:** String concatenation in a loop is O(n²) because each concatenation creates a new string.

```javascript
// Bad: O(n²) — creates a new string each iteration
let result = "";
for (let i = 0; i < n; i++) {
  result += chars[i]; // New string allocation every time
}

// Good: O(n) — join at the end
const parts = [];
for (let i = 0; i < n; i++) {
  parts.push(chars[i]);
}
const result = parts.join(""); // Single string creation
```

### Useful String Methods

```javascript
str.charAt(0); // "h" — access character
str.charCodeAt(0); // 104 — ASCII/Unicode value
str.split(""); // ["h","e","l","l","o"] — to array
str.slice(1, 3); // "el" — substring
str.toLowerCase(); // "hello"
str.includes("ell"); // true
str.indexOf("l"); // 2 — first occurrence
str.lastIndexOf("l"); // 3 — last occurrence
str.replace("l", "L"); // "heLlo" — first match only
str.trim(); // Remove whitespace from both ends
```

---

## Two Pointer Technique

Use two pointers that move toward each other or in the same direction to reduce O(n²) brute force to O(n).

### Pattern: Opposite Direction (Sorted Array)

```javascript
// Two Sum II — Input Array Is Sorted
function twoSum(numbers, target) {
  let left = 0;
  let right = numbers.length - 1;

  while (left < right) {
    const sum = numbers[left] + numbers[right];
    if (sum === target) return [left, right];
    else if (sum < target)
      left++; // Need bigger sum
    else right--; // Need smaller sum
  }
  return [-1, -1];
}
// Time: O(n), Space: O(1)
```

### Pattern: Same Direction (Remove Duplicates)

```javascript
// Remove duplicates from sorted array in-place
function removeDuplicates(nums) {
  if (nums.length === 0) return 0;
  let slow = 0; // Points to last unique element

  for (let fast = 1; fast < nums.length; fast++) {
    if (nums[fast] !== nums[slow]) {
      slow++;
      nums[slow] = nums[fast];
    }
  }
  return slow + 1; // Length of unique portion
}
// Time: O(n), Space: O(1)
```

---

## Sliding Window Technique

Maintain a "window" (subarray/substring) that slides across the array to solve problems about contiguous sequences.

### Fixed-Size Window

```javascript
// Maximum sum of subarray of size k
function maxSubarraySum(arr, k) {
  if (arr.length < k) return null;

  let windowSum = 0;
  // Calculate sum of first window
  for (let i = 0; i < k; i++) {
    windowSum += arr[i];
  }

  let maxSum = windowSum;
  // Slide the window: add next element, remove first element
  for (let i = k; i < arr.length; i++) {
    windowSum += arr[i] - arr[i - k]; // Add new, remove old
    maxSum = Math.max(maxSum, windowSum);
  }
  return maxSum;
}
// Time: O(n), Space: O(1)
```

### Variable-Size Window

```javascript
// Longest Substring Without Repeating Characters
function lengthOfLongestSubstring(s) {
  const charSet = new Set();
  let left = 0;
  let maxLen = 0;

  for (let right = 0; right < s.length; right++) {
    while (charSet.has(s[right])) {
      charSet.delete(s[left]);
      left++; // Shrink window from left
    }
    charSet.add(s[right]);
    maxLen = Math.max(maxLen, right - left + 1);
  }
  return maxLen;
}
// Time: O(n), Space: O(min(n, alphabet_size))
```

---

## Frequency Counter / Hash Map Pattern

Use a hash map to count occurrences — often turns O(n²) brute force into O(n).

```javascript
// Valid Anagram
function isAnagram(s, t) {
  if (s.length !== t.length) return false;

  const freq = {};
  for (const char of s) {
    freq[char] = (freq[char] || 0) + 1;
  }
  for (const char of t) {
    if (!freq[char]) return false;
    freq[char]--;
  }
  return true;
}
// Time: O(n), Space: O(1) — at most 26 lowercase letters
```

---

## Common Interview Problems

### Two Sum (Hash Map — O(n))

```javascript
function twoSum(nums, target) {
  const map = new Map(); // value → index

  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];
    if (map.has(complement)) {
      return [map.get(complement), i];
    }
    map.set(nums[i], i);
  }
  return [];
}
// Time: O(n), Space: O(n)
```

### Maximum Subarray (Kadane's Algorithm)

```javascript
function maxSubArray(nums) {
  let currentMax = nums[0];
  let globalMax = nums[0];

  for (let i = 1; i < nums.length; i++) {
    // Either extend current subarray or start fresh
    currentMax = Math.max(nums[i], currentMax + nums[i]);
    globalMax = Math.max(globalMax, currentMax);
  }
  return globalMax;
}
// Time: O(n), Space: O(1)
```

### Palindrome Check

```javascript
function isPalindrome(s) {
  // Remove non-alphanumeric and lowercase
  const cleaned = s.replace(/[^a-zA-Z0-9]/g, "").toLowerCase();
  let left = 0;
  let right = cleaned.length - 1;

  while (left < right) {
    if (cleaned[left] !== cleaned[right]) return false;
    left++;
    right--;
  }
  return true;
}
// Time: O(n), Space: O(n) for cleaned string
```

### Reverse String (In-Place)

```javascript
function reverseString(s) {
  // s is a char array like ["h","e","l","l","o"]
  let left = 0;
  let right = s.length - 1;

  while (left < right) {
    [s[left], s[right]] = [s[right], s[left]]; // Swap
    left++;
    right--;
  }
}
// Time: O(n), Space: O(1)
```

---

## Matrix / 2D Array Traversal

```javascript
const matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9],
];

// Row-by-row traversal
for (let row = 0; row < matrix.length; row++) {
  for (let col = 0; col < matrix[0].length; col++) {
    console.log(matrix[row][col]);
  }
}
// Time: O(rows × cols) or O(n × m)

// Common directions for graph/grid problems
const directions = [
  [0, 1], // right
  [0, -1], // left
  [1, 0], // down
  [-1, 0], // up
];

// Check bounds
function isValid(row, col, matrix) {
  return row >= 0 && row < matrix.length && col >= 0 && col < matrix[0].length;
}
```

---

## When to Use Arrays vs Other Structures

| Use Arrays When                    | Consider Alternatives When                           |
| ---------------------------------- | ---------------------------------------------------- |
| Need random access by index        | Frequent insertions/deletions at start → Linked List |
| Data size is known or fixed        | Need key-value lookups → Hash Map                    |
| Iterating through all elements     | Need ordered unique elements → Set / BST             |
| Stack/queue with small data        | Need priority ordering → Heap / Priority Queue       |
| Sorting or binary search is needed | Need fast search + insert → Balanced BST             |

---

## Common Interview Questions

1. **Two Sum** — Use hash map for O(n) solution.
2. **Best Time to Buy and Sell Stock** — Track min price, calculate max profit.
3. **Contains Duplicate** — Use Set for O(n) time and space.
4. **Maximum Subarray** — Kadane's algorithm, O(n).
5. **Product of Array Except Self** — Prefix and suffix products, O(n) without division.
6. **Merge Intervals** — Sort by start, merge overlapping.
7. **Longest Substring Without Repeating Characters** — Sliding window with Set.
8. **Valid Anagram** — Frequency counter.
9. **Group Anagrams** — Sort each word as key in hash map.
10. **Rotate Array** — Reverse approach (three reverses).

---

## Summary

- Arrays provide O(1) access but O(n) insertions/deletions (except at the end).
- Strings are immutable in JavaScript — avoid concatenation in loops (use `Array.join()`).
- **Two Pointers** reduce O(n²) to O(n) for sorted array or palindrome problems.
- **Sliding Window** efficiently handles contiguous subarray/substring problems.
- **Frequency Counter** (hash map) is the go-to pattern for counting, grouping, and matching.
- Kadane's algorithm solves maximum subarray in O(n).
- Always consider the time-space trade-off: hash maps use O(n) space but often reduce time from O(n²) to O(n).
