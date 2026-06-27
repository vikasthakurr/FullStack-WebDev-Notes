# Big-O Notation & Complexity Analysis

## What is Big-O?

Big-O notation describes the **upper bound** of an algorithm's growth rate — how the runtime or memory usage scales as the input size increases. It answers the question: "If I double the input, how much more work does this algorithm do?"

**Analogy:** Imagine counting people in a room. If you count one by one, doubling the room size doubles your work (O(n)). If you just check whether the room is empty or not, it takes the same time regardless of room size (O(1)).

Big-O is not about exact time in milliseconds — it's about the **rate of growth** relative to input size `n`.

---

## Time Complexity vs Space Complexity

| Aspect           | Time Complexity                         | Space Complexity                             |
| ---------------- | --------------------------------------- | -------------------------------------------- |
| What it measures | Number of operations as input grows     | Amount of memory used as input grows         |
| Why it matters   | Determines how fast your algorithm runs | Determines how much RAM your algorithm needs |
| Trade-off        | Often you can trade space for time      | Using more memory can speed things up        |

```javascript
// Time: O(n), Space: O(n)
function collectDoubles(arr) {
  const result = []; // Extra space grows with input
  for (let i = 0; i < arr.length; i++) {
    result.push(arr[i] * 2);
  }
  return result;
}

// Time: O(n), Space: O(1)
function sumArray(arr) {
  let total = 0; // Fixed extra space regardless of input
  for (let i = 0; i < arr.length; i++) {
    total += arr[i];
  }
  return total;
}
```

---

## Common Complexities

```
Performance (Best to Worst):
───────────────────────────────────────────────────────
O(1)        │ Constant    │ ██
O(log n)    │ Logarithmic │ ████
O(n)        │ Linear      │ ████████
O(n log n)  │ Linearithmic│ ████████████
O(n²)       │ Quadratic   │ ████████████████████
O(2ⁿ)       │ Exponential │ ████████████████████████████████████████
───────────────────────────────────────────────────────
```

| Complexity | Name         | n=10  | n=100     | n=1000        | Example                      |
| ---------- | ------------ | ----- | --------- | ------------- | ---------------------------- |
| O(1)       | Constant     | 1     | 1         | 1             | Array access by index        |
| O(log n)   | Logarithmic  | 3     | 7         | 10            | Binary search                |
| O(n)       | Linear       | 10    | 100       | 1,000         | Loop through array           |
| O(n log n) | Linearithmic | 33    | 664       | 9,966         | Merge sort, quick sort (avg) |
| O(n²)      | Quadratic    | 100   | 10,000    | 1,000,000     | Nested loops, bubble sort    |
| O(2ⁿ)      | Exponential  | 1,024 | 1.26×10³⁰ | ∞ (too large) | Recursive Fibonacci (naive)  |

---

## How to Analyze Loops

### Single Loop — O(n)

```javascript
function printAll(arr) {
  for (let i = 0; i < arr.length; i++) {
    // Runs n times
    console.log(arr[i]); // O(1) work per iteration
  }
}
// Total: O(n)
```

### Nested Loops — O(n²)

```javascript
function printPairs(arr) {
  for (let i = 0; i < arr.length; i++) {
    // Outer: n times
    for (let j = 0; j < arr.length; j++) {
      // Inner: n times for each outer
      console.log(arr[i], arr[j]);
    }
  }
}
// Total: O(n × n) = O(n²)
```

### Logarithmic Loop — O(log n)

```javascript
function logLoop(n) {
  let i = 1;
  while (i < n) {
    // How many times does this run?
    console.log(i);
    i = i * 2; // i doubles each time: 1, 2, 4, 8, 16, ...
  }
}
// Runs log₂(n) times → O(log n)
```

### Two Separate Loops — O(n + m)

```javascript
function processTwo(arr1, arr2) {
  for (let i = 0; i < arr1.length; i++) {
    // O(n) where n = arr1.length
    console.log(arr1[i]);
  }
  for (let j = 0; j < arr2.length; j++) {
    // O(m) where m = arr2.length
    console.log(arr2[j]);
  }
}
// Total: O(n + m) — different inputs, don't combine
```

---

## Best Case, Worst Case, Average Case

```javascript
function linearSearch(arr, target) {
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === target) return i;
  }
  return -1;
}
```

| Case         | Scenario                          | Complexity    |
| ------------ | --------------------------------- | ------------- |
| Best Case    | Target is the first element       | O(1)          |
| Worst Case   | Target is last or not present     | O(n)          |
| Average Case | Target is somewhere in the middle | O(n/2) → O(n) |

**In interviews, always analyze the worst case unless stated otherwise.** Big-O represents the worst-case upper bound by default.

---

## Amortized Analysis

Some operations are expensive occasionally but cheap most of the time. **Amortized analysis** averages the cost over a sequence of operations.

```javascript
// Dynamic array (JavaScript's push)
const arr = [];
arr.push(1); // O(1) — just add to end
arr.push(2); // O(1)
arr.push(3); // O(1)
arr.push(4); // O(1)
arr.push(5); // O(n) — array is full, must resize (copy all elements to new larger array)
```

Even though resizing is O(n), it happens so rarely that the **amortized cost of push is O(1)**. The expensive operation is "spread out" over many cheap operations.

---

## Space Complexity

### Auxiliary Space vs Input Space

- **Input Space:** Memory taken by the input itself (usually not counted).
- **Auxiliary Space:** Extra memory your algorithm uses beyond the input.

```javascript
// Auxiliary Space: O(1) — only a few variables regardless of input size
function findMax(arr) {
  let max = arr[0];
  for (let i = 1; i < arr.length; i++) {
    if (arr[i] > max) max = arr[i];
  }
  return max;
}

// Auxiliary Space: O(n) — creating a new array of size n
function reverseArray(arr) {
  const reversed = [];
  for (let i = arr.length - 1; i >= 0; i--) {
    reversed.push(arr[i]);
  }
  return reversed;
}

// Auxiliary Space: O(n) — recursive call stack depth
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1); // Each call adds a frame to the stack
}
```

---

## Rules for Simplifying Big-O

### Rule 1: Drop Constants

```javascript
// O(2n) → O(n)
function example(arr) {
  for (let i = 0; i < arr.length; i++) {
    console.log(arr[i]);
  } // n
  for (let i = 0; i < arr.length; i++) {
    console.log(arr[i]);
  } // n
}
// Total: O(2n) → simplified to O(n)
```

### Rule 2: Drop Non-Dominant Terms

```javascript
// O(n² + n) → O(n²)
function example(arr) {
  for (let i = 0; i < arr.length; i++) {
    // O(n)
    for (let j = 0; j < arr.length; j++) {
      console.log(arr[i], arr[j]); // O(n²) total
    }
  }
  for (let k = 0; k < arr.length; k++) {
    // O(n)
    console.log(arr[k]);
  }
}
// Total: O(n² + n) → O(n²) — n² dominates when n is large
```

### Rule 3: Different Inputs Get Different Variables

```javascript
function processBoth(arr1, arr2) {
  for (let i = 0; i < arr1.length; i++) {
    for (let j = 0; j < arr2.length; j++) {
      console.log(arr1[i], arr2[j]);
    }
  }
}
// Total: O(n × m) — NOT O(n²) because inputs are different
```

---

## Step-by-Step Analysis Examples

### Example 1: Finding Duplicates

```javascript
function hasDuplicates(arr) {
  for (let i = 0; i < arr.length; i++) {
    // n iterations
    for (let j = i + 1; j < arr.length; j++) {
      // n-1, n-2, ... 1 iterations
      if (arr[i] === arr[j]) return true; // O(1) comparison
    }
  }
  return false;
}
```

**Analysis:**

- Outer loop: n times
- Inner loop: (n-1) + (n-2) + ... + 1 = n(n-1)/2
- Total: O(n²/2) → **O(n²)**
- Space: O(1) — no extra data structures

### Example 2: Optimized with Hash Set

```javascript
function hasDuplicates(arr) {
  const seen = new Set(); // Space: grows up to n
  for (let i = 0; i < arr.length; i++) {
    // n iterations
    if (seen.has(arr[i])) return true; // O(1) lookup
    seen.add(arr[i]); // O(1) insert
  }
  return false;
}
```

**Analysis:**

- Loop: n iterations, each with O(1) work
- Time: **O(n)**
- Space: **O(n)** — Set stores up to n elements
- Trade-off: Used extra space to reduce time from O(n²) to O(n)

### Example 3: Recursive Function

```javascript
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}
```

**Analysis:**

- Each call branches into 2 calls
- Tree height is n
- Time: **O(2ⁿ)** — exponential
- Space: **O(n)** — max call stack depth is n

---

## Common Interview Questions

1. **"What is the time complexity of this code?"** — Walk through the loops, identify the dominant term.
2. **"Can you optimize this?"** — Often involves trading space for time (hash maps, sets).
3. **"What is the space complexity?"** — Count extra variables, arrays, recursive call stack depth.
4. **"What's the difference between O(n) and O(n log n)?"** — O(n log n) does slightly more work per element (log n factor), common in efficient sorting.
5. **"Is O(n²) always bad?"** — Not for small inputs (n < 50), but for large datasets it becomes impractical.

---

## Summary

- Big-O measures **how an algorithm scales** — not exact speed, but growth rate.
- Always analyze **worst case** unless asked otherwise.
- **Drop constants** (O(2n) → O(n)) and **drop non-dominant terms** (O(n² + n) → O(n²)).
- Different inputs use different variables: O(n + m), not O(2n).
- **Space complexity** counts the extra memory your algorithm uses (auxiliary space).
- Common trade-off: use more space (hash maps) to reduce time complexity.
- Amortized analysis explains why operations like `array.push()` are O(1) on average despite occasional O(n) resizes.
- Knowing Big-O helps you choose the right data structure and algorithm for each problem.
