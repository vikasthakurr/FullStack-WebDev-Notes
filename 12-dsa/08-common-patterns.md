# Common Patterns for Coding Interviews

## Sliding Window Pattern

Maintain a window (contiguous subarray/substring) that expands or shrinks to find an optimal solution. Avoids recalculating from scratch — O(n) instead of O(n²).

### Fixed-Size Window

Window size is given. Slide it one position at a time.

```
Array: [2, 1, 5, 1, 3, 2]  k = 3

Window 1: [2, 1, 5] → sum = 8
Window 2: [1, 5, 1] → sum = 7  (add 1, remove 2)
Window 3: [5, 1, 3] → sum = 9  (add 3, remove 1)
Window 4: [1, 3, 2] → sum = 6  (add 2, remove 5)

Max sum = 9
```

#### Maximum Sum Subarray of Size k

```javascript
function maxSubarraySum(arr, k) {
  if (arr.length < k) return null;

  let windowSum = 0;
  // Build first window
  for (let i = 0; i < k; i++) {
    windowSum += arr[i];
  }

  let maxSum = windowSum;
  // Slide: add next, remove first of previous window
  for (let i = k; i < arr.length; i++) {
    windowSum += arr[i] - arr[i - k];
    maxSum = Math.max(maxSum, windowSum);
  }
  return maxSum;
}
// Time: O(n), Space: O(1)
```

### Variable-Size Window

Window expands (right pointer) until a condition breaks, then shrinks (left pointer) until valid again.

#### Longest Substring Without Repeating Characters

```javascript
function lengthOfLongestSubstring(s) {
  const charSet = new Set();
  let left = 0;
  let maxLen = 0;

  for (let right = 0; right < s.length; right++) {
    // Shrink window until no duplicate
    while (charSet.has(s[right])) {
      charSet.delete(s[left]);
      left++;
    }
    charSet.add(s[right]);
    maxLen = Math.max(maxLen, right - left + 1);
  }
  return maxLen;
}
// Time: O(n), Space: O(min(n, alphabet_size))

lengthOfLongestSubstring("abcabcbb"); // 3 ("abc")
lengthOfLongestSubstring("pwwkew"); // 3 ("wke")
```

#### Minimum Window Substring

Find the smallest substring of `s` that contains all characters of `t`.

```javascript
function minWindow(s, t) {
  if (t.length > s.length) return "";

  // Count required characters
  const need = new Map();
  for (const char of t) {
    need.set(char, (need.get(char) || 0) + 1);
  }

  let left = 0;
  let matched = 0; // Number of characters fully matched
  let minLen = Infinity;
  let minStart = 0;
  const window = new Map();

  for (let right = 0; right < s.length; right++) {
    // Expand window
    const char = s[right];
    window.set(char, (window.get(char) || 0) + 1);

    // Check if this character is now fully matched
    if (need.has(char) && window.get(char) === need.get(char)) {
      matched++;
    }

    // Shrink window while all characters are matched
    while (matched === need.size) {
      const windowLen = right - left + 1;
      if (windowLen < minLen) {
        minLen = windowLen;
        minStart = left;
      }

      // Remove left character from window
      const leftChar = s[left];
      window.set(leftChar, window.get(leftChar) - 1);
      if (need.has(leftChar) && window.get(leftChar) < need.get(leftChar)) {
        matched--;
      }
      left++;
    }
  }

  return minLen === Infinity ? "" : s.slice(minStart, minStart + minLen);
}
// Time: O(n), Space: O(n)

minWindow("ADOBECODEBANC", "ABC"); // "BANC"
```

---

## Two Pointers Pattern

Use two pointers that move towards each other or in the same direction to process sorted data efficiently.

### Two Sum — Sorted Array

```javascript
function twoSumSorted(numbers, target) {
  let left = 0;
  let right = numbers.length - 1;

  while (left < right) {
    const sum = numbers[left] + numbers[right];
    if (sum === target) return [left, right];
    if (sum < target)
      left++; // Need larger sum
    else right--; // Need smaller sum
  }
  return [-1, -1];
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

    // Move the shorter side — only chance to find a taller line
    if (heights[left] < heights[right]) left++;
    else right--;
  }
  return maxWater;
}
// Time: O(n), Space: O(1)
```

### Remove Duplicates From Sorted Array

```javascript
function removeDuplicates(nums) {
  if (nums.length === 0) return 0;
  let slow = 0; // Last unique position

  for (let fast = 1; fast < nums.length; fast++) {
    if (nums[fast] !== nums[slow]) {
      slow++;
      nums[slow] = nums[fast];
    }
  }
  return slow + 1;
}
// Time: O(n), Space: O(1)
// [1,1,2,2,3] → [1,2,3,...] returns 3
```

### Three Sum

Find all triplets that sum to zero. Sort + two pointers for each element.

```javascript
function threeSum(nums) {
  nums.sort((a, b) => a - b);
  const result = [];

  for (let i = 0; i < nums.length - 2; i++) {
    // Skip duplicate first elements
    if (i > 0 && nums[i] === nums[i - 1]) continue;

    let left = i + 1;
    let right = nums.length - 1;

    while (left < right) {
      const sum = nums[i] + nums[left] + nums[right];

      if (sum === 0) {
        result.push([nums[i], nums[left], nums[right]]);
        // Skip duplicates
        while (left < right && nums[left] === nums[left + 1]) left++;
        while (left < right && nums[right] === nums[right - 1]) right--;
        left++;
        right--;
      } else if (sum < 0) {
        left++;
      } else {
        right--;
      }
    }
  }
  return result;
}
// Time: O(n²), Space: O(1) excluding output

threeSum([-1, 0, 1, 2, -1, -4]);
// [[-1, -1, 2], [-1, 0, 1]]
```

---

## Fast & Slow Pointers

Two pointers moving at different speeds. Primarily used in linked list problems.

### Linked List Cycle Detection

```javascript
function hasCycle(head) {
  let slow = head;
  let fast = head;

  while (fast && fast.next) {
    slow = slow.next; // 1 step
    fast = fast.next.next; // 2 steps
    if (slow === fast) return true;
  }
  return false;
}
// Time: O(n), Space: O(1)
```

### Find Middle of Linked List

```javascript
function findMiddle(head) {
  let slow = head;
  let fast = head;

  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
  }
  return slow; // Middle node
}
// Time: O(n), Space: O(1)
```

### Happy Number

A number is "happy" if repeatedly replacing it with the sum of squares of its digits eventually reaches 1. If it loops forever, it's not happy. Use Floyd's cycle detection.

```javascript
function isHappy(n) {
  function getNext(num) {
    let sum = 0;
    while (num > 0) {
      const digit = num % 10;
      sum += digit * digit;
      num = Math.floor(num / 10);
    }
    return sum;
  }

  let slow = n;
  let fast = getNext(n);

  while (fast !== 1 && slow !== fast) {
    slow = getNext(slow); // 1 step
    fast = getNext(getNext(fast)); // 2 steps
  }
  return fast === 1;
}
// Time: O(log n), Space: O(1)

isHappy(19); // true: 19→82→68→100→1
isHappy(2); // false: enters a cycle
```

---

## Frequency Counter / Hash Map Pattern

Use a hash map to count occurrences — transforms O(n²) comparisons into O(n) lookups.

### Two Sum (Unsorted — Hash Map)

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

### Group Anagrams

Group words that are anagrams of each other. Use sorted word as key.

```javascript
function groupAnagrams(strs) {
  const map = new Map();

  for (const str of strs) {
    const key = str.split("").sort().join(""); // "eat" → "aet"
    if (!map.has(key)) map.set(key, []);
    map.get(key).push(str);
  }
  return Array.from(map.values());
}
// Time: O(n × k log k) where k = max word length
// Space: O(n × k)

groupAnagrams(["eat", "tea", "tan", "ate", "nat", "bat"]);
// [["eat","tea","ate"], ["tan","nat"], ["bat"]]
```

### First Non-Repeating Character

```javascript
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
// Time: O(n), Space: O(1) — at most 26 lowercase letters

firstUniqChar("leetcode"); // 0 ("l")
firstUniqChar("aabb"); // -1
```

---

## Recursion & Backtracking

### Core Idea

**Recursion:** Solve a problem by breaking it into smaller subproblems of the same type.

**Backtracking:** Build a solution incrementally, abandon ("backtrack") paths that can't lead to a valid solution.

```
Backtracking template:
1. Choose — pick an option
2. Explore — recurse with that choice
3. Un-choose — undo the choice (backtrack)
```

### Generate All Subsets

```javascript
function subsets(nums) {
  const result = [];

  function backtrack(start, current) {
    result.push([...current]); // Add current subset

    for (let i = start; i < nums.length; i++) {
      current.push(nums[i]); // Choose
      backtrack(i + 1, current); // Explore
      current.pop(); // Un-choose (backtrack)
    }
  }

  backtrack(0, []);
  return result;
}
// Time: O(2^n), Space: O(n) recursion depth

subsets([1, 2, 3]);
// [[], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]]
```

### Generate All Permutations

```javascript
function permutations(nums) {
  const result = [];

  function backtrack(current, remaining) {
    if (remaining.length === 0) {
      result.push([...current]);
      return;
    }

    for (let i = 0; i < remaining.length; i++) {
      current.push(remaining[i]);
      backtrack(current, remaining.slice(0, i).concat(remaining.slice(i + 1)));
      current.pop(); // Backtrack
    }
  }

  backtrack([], nums);
  return result;
}
// Time: O(n! × n), Space: O(n)

permutations([1, 2, 3]);
// [[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

### N-Queens (Concept)

Place N queens on an N×N chessboard so no two queens attack each other. Classic backtracking problem.

```
4-Queens solution:
. Q . .
. . . Q
Q . . .
. . Q .

Strategy:
- Place queens row by row
- For each row, try each column
- Check if placement is valid (no conflicts in column, diagonals)
- If valid, recurse to next row
- If no valid placement exists, backtrack
```

```javascript
function solveNQueens(n) {
  const result = [];
  const board = Array.from({ length: n }, () => Array(n).fill("."));

  function isValid(row, col) {
    // Check column above
    for (let i = 0; i < row; i++) {
      if (board[i][col] === "Q") return false;
    }
    // Check upper-left diagonal
    for (let i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
      if (board[i][j] === "Q") return false;
    }
    // Check upper-right diagonal
    for (let i = row - 1, j = col + 1; i >= 0 && j < n; i--, j++) {
      if (board[i][j] === "Q") return false;
    }
    return true;
  }

  function backtrack(row) {
    if (row === n) {
      result.push(board.map((r) => r.join("")));
      return;
    }

    for (let col = 0; col < n; col++) {
      if (isValid(row, col)) {
        board[row][col] = "Q"; // Choose
        backtrack(row + 1); // Explore
        board[row][col] = "."; // Un-choose
      }
    }
  }

  backtrack(0);
  return result;
}
```

### Word Search in Grid

Given a 2D grid of characters, check if a word exists by following adjacent cells (no reuse).

```javascript
function exist(board, word) {
  const rows = board.length;
  const cols = board[0].length;

  function backtrack(row, col, index) {
    if (index === word.length) return true; // Found complete word

    // Boundary and character check
    if (
      row < 0 ||
      row >= rows ||
      col < 0 ||
      col >= cols ||
      board[row][col] !== word[index]
    )
      return false;

    // Mark as visited
    const temp = board[row][col];
    board[row][col] = "#";

    // Explore 4 directions
    const found =
      backtrack(row + 1, col, index + 1) ||
      backtrack(row - 1, col, index + 1) ||
      backtrack(row, col + 1, index + 1) ||
      backtrack(row, col - 1, index + 1);

    // Un-choose (restore)
    board[row][col] = temp;
    return found;
  }

  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (backtrack(r, c, 0)) return true;
    }
  }
  return false;
}
// Time: O(m × n × 4^L) where L = word length
// Space: O(L) recursion depth
```

---

## Dynamic Programming (DP)

### Core Concept

DP solves problems by breaking them into **overlapping subproblems** and storing results to avoid redundant computation.

Two key properties:

1. **Overlapping subproblems** — same subproblem is solved multiple times
2. **Optimal substructure** — optimal solution is built from optimal solutions to subproblems

### Approaches

| Approach    | Direction | How it works                                    |
| ----------- | --------- | ----------------------------------------------- |
| Memoization | Top-down  | Recursion + cache (store results as computed)   |
| Tabulation  | Bottom-up | Iterative, fill table from smallest subproblems |

---

### Fibonacci — Evolution from Naive to Optimal

**Naive Recursive — O(2^n):**

```javascript
function fib(n) {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2); // Recomputes same values!
}
```

```
fib(5) call tree — massive redundancy:
         fib(5)
        /      \
     fib(4)    fib(3)
     /   \     /   \
  fib(3) fib(2) fib(2) fib(1)
  /  \
fib(2) fib(1)

fib(2) computed 3 times! fib(3) computed 2 times!
```

**Memoized (Top-Down) — O(n):**

```javascript
function fibMemo(n, memo = {}) {
  if (n <= 1) return n;
  if (memo[n] !== undefined) return memo[n];

  memo[n] = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
  return memo[n];
}
// Time: O(n), Space: O(n)
```

**Tabulation (Bottom-Up) — O(n):**

```javascript
function fibTab(n) {
  if (n <= 1) return n;
  const dp = [0, 1];

  for (let i = 2; i <= n; i++) {
    dp[i] = dp[i - 1] + dp[i - 2];
  }
  return dp[n];
}
// Time: O(n), Space: O(n)
```

**Space-Optimized — O(n) time, O(1) space:**

```javascript
function fibOptimized(n) {
  if (n <= 1) return n;
  let prev2 = 0,
    prev1 = 1;

  for (let i = 2; i <= n; i++) {
    const current = prev1 + prev2;
    prev2 = prev1;
    prev1 = current;
  }
  return prev1;
}
```

---

### Climbing Stairs

You can climb 1 or 2 steps. How many distinct ways to reach step n?

```
n=1: [1]                         → 1 way
n=2: [1,1], [2]                  → 2 ways
n=3: [1,1,1], [1,2], [2,1]      → 3 ways
n=4: [1,1,1,1], [1,1,2], [1,2,1], [2,1,1], [2,2] → 5 ways

Pattern: dp[i] = dp[i-1] + dp[i-2] — same as Fibonacci!
```

```javascript
function climbStairs(n) {
  if (n <= 2) return n;
  let prev2 = 1,
    prev1 = 2;

  for (let i = 3; i <= n; i++) {
    const current = prev1 + prev2;
    prev2 = prev1;
    prev1 = current;
  }
  return prev1;
}
// Time: O(n), Space: O(1)
```

---

### Coin Change

Find minimum number of coins to make a given amount. Coins can be used unlimited times.

```
Coins: [1, 5, 10]  Amount: 12

dp[0] = 0 (base case)
dp[1] = 1  (1)
dp[5] = 1  (5)
dp[10] = 1 (10)
dp[11] = 2 (10+1)
dp[12] = 3 (10+1+1) — but wait, is there better?
        Actually: dp[12] = dp[12-10] + 1 = dp[2] + 1 = 3
        Or: dp[12-5] + 1 = dp[7] + 1 = 3... let's check dp[7] = dp[7-5]+1 = 3? No: dp[2]+1=3
        Best: 3 coins (10+1+1)

Coins: [1, 3, 4]  Amount: 6
Greedy would pick 4+1+1 = 3 coins. But optimal is 3+3 = 2 coins!
DP handles this correctly.
```

```javascript
function coinChange(coins, amount) {
  // dp[i] = minimum coins needed to make amount i
  const dp = new Array(amount + 1).fill(Infinity);
  dp[0] = 0; // Base case: 0 coins to make amount 0

  for (let i = 1; i <= amount; i++) {
    for (const coin of coins) {
      if (coin <= i && dp[i - coin] !== Infinity) {
        dp[i] = Math.min(dp[i], dp[i - coin] + 1);
      }
    }
  }
  return dp[amount] === Infinity ? -1 : dp[amount];
}
// Time: O(amount × coins.length), Space: O(amount)

coinChange([1, 5, 10], 12); // 3 (10+1+1)
coinChange([1, 3, 4], 6); // 2 (3+3)
coinChange([2], 3); // -1 (impossible)
```

---

## Greedy Algorithms

### Core Idea

Make the locally optimal choice at each step, hoping it leads to a globally optimal solution. Works when the problem has **greedy-choice property** (local optimal → global optimal).

> **Greedy vs DP:** Greedy makes one choice and never reconsiders. DP considers all choices and picks the best. Greedy is faster but doesn't always work.

### Activity Selection

Given activities with start and end times, find the maximum number of non-overlapping activities.

**Greedy strategy:** Always pick the activity that finishes earliest (leaves most room for future activities).

```javascript
function activitySelection(activities) {
  // Sort by end time
  activities.sort((a, b) => a.end - b.end);

  const selected = [activities[0]];
  let lastEnd = activities[0].end;

  for (let i = 1; i < activities.length; i++) {
    if (activities[i].start >= lastEnd) {
      selected.push(activities[i]);
      lastEnd = activities[i].end;
    }
  }
  return selected;
}
// Time: O(n log n) for sorting, Space: O(n)

activitySelection([
  { start: 1, end: 3 },
  { start: 2, end: 5 },
  { start: 4, end: 7 },
  { start: 1, end: 8 },
  { start: 6, end: 9 },
]);
// [{1,3}, {4,7}, {6,9}] — 3 activities (non-overlapping, max possible is 3 not this exact set)
```

### Fractional Knapsack

Given items with weights and values, maximize value in a knapsack of fixed capacity. You can take fractions of items.

**Greedy strategy:** Sort by value-per-weight ratio (descending). Take as much of the highest ratio item as possible.

```javascript
function fractionalKnapsack(capacity, items) {
  // Sort by value/weight ratio (descending)
  items.sort((a, b) => b.value / b.weight - a.value / a.weight);

  let totalValue = 0;
  let remainingCapacity = capacity;

  for (const item of items) {
    if (item.weight <= remainingCapacity) {
      // Take the whole item
      totalValue += item.value;
      remainingCapacity -= item.weight;
    } else {
      // Take a fraction
      const fraction = remainingCapacity / item.weight;
      totalValue += item.value * fraction;
      break; // Knapsack is full
    }
  }
  return totalValue;
}
// Time: O(n log n), Space: O(1)

fractionalKnapsack(50, [
  { weight: 10, value: 60 }, // ratio: 6
  { weight: 20, value: 100 }, // ratio: 5
  { weight: 30, value: 120 }, // ratio: 4
]);
// Take all of item 1 (60) + all of item 2 (100) + 20/30 of item 3 (80) = 240
```

> **Note:** Fractional Knapsack works with greedy. The **0/1 Knapsack** (can't take fractions) requires DP.

---

## Pattern Recognition Cheat Sheet

| If you see...                                | Think...                     |
| -------------------------------------------- | ---------------------------- |
| Contiguous subarray/substring                | Sliding Window               |
| Sorted array + find pair                     | Two Pointers                 |
| Linked list cycle/middle                     | Fast & Slow Pointers         |
| Count occurrences / find duplicates          | Hash Map / Frequency Counter |
| "Find all combinations/permutations"         | Backtracking                 |
| "Minimum/maximum of something"               | DP or Greedy                 |
| "How many ways to..."                        | DP                           |
| Overlapping subproblems                      | DP (memoization/tabulation)  |
| "Make locally optimal choice"                | Greedy                       |
| Tree/graph exploration                       | BFS or DFS                   |
| "Shortest path" (unweighted)                 | BFS                          |
| "Find connected components" / "detect cycle" | DFS                          |
| Prefix matching / autocomplete               | Trie                         |
| "Top K" / "Kth largest"                      | Heap / Quick Select          |
| "Intervals" (merge, overlap)                 | Sort by start + sweep        |

---

## Summary

- **Sliding Window** — O(n) for contiguous subarray/substring problems. Fixed size (add/remove) or variable (expand/shrink).
- **Two Pointers** — O(n) for sorted array pair/triplet problems. Opposite direction or same direction (fast/slow).
- **Fast & Slow Pointers** — cycle detection, finding midpoints. O(1) space.
- **Frequency Counter** — hash map to count/group in O(n). Replaces O(n²) nested loops.
- **Backtracking** — generate all valid combinations by choosing, exploring, and un-choosing. Time usually exponential but pruning helps.
- **Dynamic Programming** — break into overlapping subproblems. Use memoization (top-down) or tabulation (bottom-up). Key: identify state and transitions.
- **Greedy** — make locally optimal choices. Faster than DP but only works when local optimal → global optimal.
- Recognizing which pattern to apply is half the battle in interviews. Practice mapping problem characteristics to patterns.
