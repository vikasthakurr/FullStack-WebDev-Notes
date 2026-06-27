# Searching Algorithms

## Linear Search — O(n)

Check every element one by one until you find the target or exhaust the array. Works on any array (sorted or unsorted).

```javascript
function linearSearch(arr, target) {
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === target) return i;
  }
  return -1; // Not found
}
// Time: O(n), Space: O(1)
```

**When to use:** Unsorted data, small arrays, or when you only need to search once (not worth sorting first).

---

## Binary Search — O(log n)

Divide the search space in half each step. **Requires sorted array.**

```
Target: 7    Array: [1, 3, 5, 7, 9, 11, 13]

Step 1: mid = 7 → found! Return index 3

Target: 9    Array: [1, 3, 5, 7, 9, 11, 13]

Step 1: mid = 7, 9 > 7 → search right [9, 11, 13]
Step 2: mid = 11, 9 < 11 → search left [9]
Step 3: mid = 9 → found! Return index 4
```

Each step eliminates half the array → log₂(n) steps total.

---

### Binary Search — Iterative (Preferred)

```javascript
function binarySearch(arr, target) {
  let left = 0;
  let right = arr.length - 1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);

    if (arr[mid] === target) return mid;
    else if (arr[mid] < target)
      left = mid + 1; // Search right half
    else right = mid - 1; // Search left half
  }
  return -1; // Not found
}
// Time: O(log n), Space: O(1)
```

### Binary Search — Recursive

```javascript
function binarySearchRecursive(arr, target, left = 0, right = arr.length - 1) {
  if (left > right) return -1;

  const mid = Math.floor((left + right) / 2);

  if (arr[mid] === target) return mid;
  if (arr[mid] < target)
    return binarySearchRecursive(arr, target, mid + 1, right);
  return binarySearchRecursive(arr, target, left, mid - 1);
}
// Time: O(log n), Space: O(log n) — recursion stack
```

> Prefer iterative — same logic, O(1) space, no stack overflow risk.

---

## Binary Search Variations

### Find First Occurrence

When duplicates exist, find the leftmost index of the target.

```javascript
function findFirst(arr, target) {
  let left = 0;
  let right = arr.length - 1;
  let result = -1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);

    if (arr[mid] === target) {
      result = mid; // Record this position
      right = mid - 1; // Keep searching left for earlier occurrence
    } else if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  return result;
}

// [1, 2, 2, 2, 3, 4]  target = 2
// Returns index 1 (first 2)
```

### Find Last Occurrence

```javascript
function findLast(arr, target) {
  let left = 0;
  let right = arr.length - 1;
  let result = -1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);

    if (arr[mid] === target) {
      result = mid; // Record this position
      left = mid + 1; // Keep searching right for later occurrence
    } else if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  return result;
}

// [1, 2, 2, 2, 3, 4]  target = 2
// Returns index 3 (last 2)
```

---

### Search in Rotated Sorted Array

Array was sorted then rotated at some pivot. Find the target.

```
Original: [1, 2, 3, 4, 5, 6, 7]
Rotated:  [4, 5, 6, 7, 1, 2, 3]  (rotated at index 4)
```

Key insight: One half is always sorted. Determine which half is sorted, then check if target is in that sorted half.

```javascript
function searchRotated(nums, target) {
  let left = 0;
  let right = nums.length - 1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);

    if (nums[mid] === target) return mid;

    // Left half is sorted
    if (nums[left] <= nums[mid]) {
      if (target >= nums[left] && target < nums[mid]) {
        right = mid - 1; // Target is in sorted left half
      } else {
        left = mid + 1; // Target is in right half
      }
    }
    // Right half is sorted
    else {
      if (target > nums[mid] && target <= nums[right]) {
        left = mid + 1; // Target is in sorted right half
      } else {
        right = mid - 1; // Target is in left half
      }
    }
  }
  return -1;
}
// Time: O(log n), Space: O(1)
```

---

### Find Peak Element

A peak is an element greater than its neighbors. Array may have multiple peaks — find any one.

```javascript
function findPeakElement(nums) {
  let left = 0;
  let right = nums.length - 1;

  while (left < right) {
    const mid = Math.floor((left + right) / 2);

    if (nums[mid] > nums[mid + 1]) {
      right = mid; // Peak is at mid or to the left
    } else {
      left = mid + 1; // Peak is to the right
    }
  }
  return left; // left === right === peak index
}
// Time: O(log n), Space: O(1)
```

```
[1, 3, 5, 4, 2]
      ↑ peak (5 > 3 and 5 > 4)

Binary search: if mid < mid+1, peak must be on the right (ascending)
               if mid > mid+1, peak is at mid or left (descending)
```

---

### Integer Square Root

Find the largest integer x such that x² ≤ n.

```javascript
function mySqrt(n) {
  if (n < 2) return n;

  let left = 1;
  let right = Math.floor(n / 2);
  let result = 1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);

    if (mid * mid === n) return mid;
    if (mid * mid < n) {
      result = mid; // mid is a candidate
      left = mid + 1; // Try larger
    } else {
      right = mid - 1; // Too big
    }
  }
  return result;
}
// Time: O(log n), Space: O(1)

mySqrt(8); // 2 (2² = 4 ≤ 8, 3² = 9 > 8)
mySqrt(16); // 4
```

---

### Search in 2D Matrix

Matrix where each row is sorted and first element of each row > last element of previous row. Treat it as a flattened sorted array.

```javascript
function searchMatrix(matrix, target) {
  const rows = matrix.length;
  const cols = matrix[0].length;
  let left = 0;
  let right = rows * cols - 1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    // Convert 1D index to 2D
    const row = Math.floor(mid / cols);
    const col = mid % cols;
    const midVal = matrix[row][col];

    if (midVal === target) return true;
    if (midVal < target) left = mid + 1;
    else right = mid - 1;
  }
  return false;
}
// Time: O(log(m × n)), Space: O(1)
```

```
Matrix:
[1,  3,  5,  7 ]
[10, 11, 16, 20]
[23, 30, 34, 60]

Treat as: [1, 3, 5, 7, 10, 11, 16, 20, 23, 30, 34, 60]
Index 5 → row = 5/4 = 1, col = 5%4 = 1 → matrix[1][1] = 11
```

---

## Two Pointer Technique

Use two pointers to reduce O(n²) brute force to O(n). Works best on **sorted arrays** or when searching for pairs.

### Pair Sum in Sorted Array

```javascript
function twoSumSorted(arr, target) {
  let left = 0;
  let right = arr.length - 1;

  while (left < right) {
    const sum = arr[left] + arr[right];

    if (sum === target) return [left, right];
    if (sum < target)
      left++; // Need bigger sum
    else right--; // Need smaller sum
  }
  return [-1, -1];
}
// Time: O(n), Space: O(1)
```

### Remove Duplicates (Sorted Array)

```javascript
function removeDuplicates(nums) {
  if (nums.length === 0) return 0;
  let slow = 0;

  for (let fast = 1; fast < nums.length; fast++) {
    if (nums[fast] !== nums[slow]) {
      slow++;
      nums[slow] = nums[fast];
    }
  }
  return slow + 1; // New length
}
// Time: O(n), Space: O(1)
```

### Container With Most Water

```javascript
function maxArea(heights) {
  let left = 0;
  let right = heights.length - 1;
  let maxWater = 0;

  while (left < right) {
    const width = right - left;
    const height = Math.min(heights[left], heights[right]);
    maxWater = Math.max(maxWater, width * height);

    // Move the shorter line inward (only way to potentially find bigger area)
    if (heights[left] < heights[right]) left++;
    else right--;
  }
  return maxWater;
}
// Time: O(n), Space: O(1)
```

---

## Hash Map for O(1) Lookup

When you need to check existence or find complements, a hash map (Map/Set) gives O(1) lookup vs O(n) scanning or O(log n) binary search.

### When to Use Map/Set vs Sorting

| Approach      | Time               | Space  | When to Choose                   |
| ------------- | ------------------ | ------ | -------------------------------- |
| Hash Map      | O(n)               | O(n)   | Need O(1) lookups, single pass   |
| Sorting       | O(n log n)         | O(1)\* | Space constrained, need order    |
| Binary Search | O(log n) per query | O(1)   | Already sorted, multiple queries |

```javascript
// Two Sum — Hash Map approach (O(n) time, O(n) space)
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

// Check for duplicates — Set approach (O(n))
function containsDuplicate(nums) {
  const seen = new Set();
  for (const num of nums) {
    if (seen.has(num)) return true;
    seen.add(num);
  }
  return false;
}

// First non-repeating character
function firstUniqChar(s) {
  const freq = new Map();
  for (const char of s) {
    freq.set(char, (freq.get(char) || 0) + 1);
  }
  for (let i = 0; i < s.length; i++) {
    if (freq.get(s[i]) === 1) return i;
  }
  return -1;
}
```

---

## When to Use Which Search Approach

| Situation                                   | Approach                    | Time            |
| ------------------------------------------- | --------------------------- | --------------- |
| Unsorted array, find one element            | Linear Search               | O(n)            |
| Sorted array, find one element              | Binary Search               | O(log n)        |
| Find pair with target sum (sorted)          | Two Pointers                | O(n)            |
| Find pair with target sum (unsorted)        | Hash Map                    | O(n)            |
| Check existence / count occurrences         | Hash Set / Map              | O(1) per        |
| Multiple searches on same sorted data       | Binary Search               | O(log n) per    |
| Find element in rotated sorted array        | Modified Binary Search      | O(log n)        |
| Find min/max satisfying a condition         | Binary Search on answer     | O(log n × f(n)) |
| Search in infinite/unknown size sorted data | Exponential + Binary Search | O(log n)        |

---

## Binary Search Template

Most binary search problems follow this pattern — finding the **boundary** where a condition changes from false to true.

```javascript
// Generic binary search template
// Find the leftmost position where condition(mid) is true
function binarySearchTemplate(arr, condition) {
  let left = 0;
  let right = arr.length - 1;
  let result = -1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);

    if (condition(arr[mid])) {
      result = mid; // Record candidate
      right = mid - 1; // Look for earlier true
    } else {
      left = mid + 1; // Condition not met yet
    }
  }
  return result;
}

// Example: Find first element >= target
const firstGE = binarySearchTemplate([1, 3, 5, 7, 9, 11], (val) => val >= 7); // Returns index 3 (value 7)
```

---

## Common Mistakes in Binary Search

| Mistake                           | Consequence                    | Fix                                             |
| --------------------------------- | ------------------------------ | ----------------------------------------------- |
| `left < right` vs `left <= right` | Misses single-element case     | Use `<=` for standard search                    |
| `mid = (left + right) / 2`        | Integer overflow (other langs) | `left + Math.floor((right - left) / 2)`         |
| Not handling duplicates           | Returns wrong index            | Use first/last occurrence variants              |
| Wrong boundary update             | Infinite loop                  | Ensure `left` or `right` changes each iteration |
| Forgetting array must be sorted   | Wrong results                  | Sort first or verify precondition               |

---

## Summary

- **Linear Search** — O(n), brute force, works on any array. Use for small/unsorted data.
- **Binary Search** — O(log n), requires sorted array. Halves search space each step.
- Iterative binary search is preferred (O(1) space, no stack overflow risk).
- **Binary search variations** handle rotated arrays, duplicates, peak finding, and answer-space searching.
- **Two Pointers** — O(n) for pair/triplet problems on sorted arrays. Move pointers based on comparison.
- **Hash Map/Set** — O(1) lookup. Use when you need to find complements, check existence, or count frequencies.
- **Decision framework:** Is data sorted? → Binary Search. Need pairs? → Two Pointers (sorted) or Hash Map (unsorted). Just finding one thing? → Linear if unsorted, Binary if sorted.
