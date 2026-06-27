# Sorting Algorithms

## Why Sorting Matters

Sorting is fundamental because it:

- **Enables binary search** — O(log n) search only works on sorted data
- **Simplifies problems** — finding duplicates, closest pairs, merging datasets becomes easier
- **Required for many algorithms** — greedy algorithms, merge operations, and many interview problems assume sorted input
- **Optimizes other operations** — grouping, ranking, and data presentation

---

## Bubble Sort

Repeatedly compare adjacent elements and swap if they're in the wrong order. Largest elements "bubble" to the end.

```
Pass 1: [5, 3, 8, 1, 2]
         3, 5 → swap    → [3, 5, 8, 1, 2]
            5, 8 → ok    → [3, 5, 8, 1, 2]
               8, 1 → swap → [3, 5, 1, 8, 2]
                  8, 2 → swap → [3, 5, 1, 2, 8] ← 8 is in place

Pass 2: [3, 5, 1, 2, 8]
         3, 5 → ok       → [3, 5, 1, 2, 8]
            5, 1 → swap  → [3, 1, 5, 2, 8]
               5, 2 → swap → [3, 1, 2, 5, 8] ← 5 is in place

...continues until no swaps needed
```

```javascript
function bubbleSort(arr) {
  const n = arr.length;

  for (let i = 0; i < n - 1; i++) {
    let swapped = false;

    for (let j = 0; j < n - 1 - i; j++) {
      if (arr[j] > arr[j + 1]) {
        [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]]; // Swap
        swapped = true;
      }
    }

    // Optimization: if no swaps occurred, array is sorted
    if (!swapped) break;
  }
  return arr;
}
```

| Case    | Time  | Why                       |
| ------- | ----- | ------------------------- |
| Best    | O(n)  | Already sorted (no swaps) |
| Average | O(n²) | Random order              |
| Worst   | O(n²) | Reverse sorted            |
| Space   | O(1)  | In-place                  |

> Educational only. Never use in production — too slow for any meaningful data size.

---

## Selection Sort

Find the minimum element in the unsorted portion and swap it with the first unsorted position. Repeat.

```
[64, 25, 12, 22, 11]
 Find min (11) → swap with position 0
[11, 25, 12, 22, 64]
     Find min (12) → swap with position 1
[11, 12, 25, 22, 64]
         Find min (22) → swap with position 2
[11, 12, 22, 25, 64]
             Find min (25) → already in position 3
[11, 12, 22, 25, 64] ✓
```

```javascript
function selectionSort(arr) {
  const n = arr.length;

  for (let i = 0; i < n - 1; i++) {
    let minIndex = i;

    // Find minimum in unsorted portion
    for (let j = i + 1; j < n; j++) {
      if (arr[j] < arr[minIndex]) {
        minIndex = j;
      }
    }

    // Swap minimum with first unsorted position
    if (minIndex !== i) {
      [arr[i], arr[minIndex]] = [arr[minIndex], arr[i]];
    }
  }
  return arr;
}
```

| Case    | Time  | Why                          |
| ------- | ----- | ---------------------------- |
| Best    | O(n²) | Always scans entire unsorted |
| Average | O(n²) | Always scans entire unsorted |
| Worst   | O(n²) | Always scans entire unsorted |
| Space   | O(1)  | In-place                     |

> Advantage: minimizes the number of swaps (at most n-1). Useful when write operations are expensive.

---

## Insertion Sort

Build the sorted portion one element at a time. Take each element and insert it into its correct position in the already-sorted portion.

```
[5, 3, 8, 1, 2]
 ↑ sorted | unsorted

Take 3: insert into [5] → [3, 5, 8, 1, 2]
Take 8: insert into [3, 5] → [3, 5, 8, 1, 2] (already in place)
Take 1: insert into [3, 5, 8] → [1, 3, 5, 8, 2]
Take 2: insert into [1, 3, 5, 8] → [1, 2, 3, 5, 8] ✓
```

```javascript
function insertionSort(arr) {
  const n = arr.length;

  for (let i = 1; i < n; i++) {
    const key = arr[i]; // Element to insert
    let j = i - 1;

    // Shift elements greater than key to the right
    while (j >= 0 && arr[j] > key) {
      arr[j + 1] = arr[j];
      j--;
    }
    arr[j + 1] = key; // Insert in correct position
  }
  return arr;
}
```

| Case    | Time  | Why                                    |
| ------- | ----- | -------------------------------------- |
| Best    | O(n)  | Already sorted — inner loop never runs |
| Average | O(n²) | Random order                           |
| Worst   | O(n²) | Reverse sorted                         |
| Space   | O(1)  | In-place                               |

> **Good for:** nearly sorted arrays (adaptive), small arrays (low overhead), and online sorting (can sort as data arrives).

---

## Merge Sort

Divide the array in half, recursively sort each half, then merge the two sorted halves. Classic **divide and conquer**.

```
[38, 27, 43, 3, 9, 82, 10]

Divide:
[38, 27, 43, 3]     [9, 82, 10]
[38, 27] [43, 3]    [9, 82] [10]
[38][27] [43][3]    [9][82] [10]

Merge (sorted):
[27, 38] [3, 43]    [9, 82] [10]
[3, 27, 38, 43]     [9, 10, 82]
[3, 9, 10, 27, 38, 43, 82] ✓
```

```javascript
function mergeSort(arr) {
  // Base case: single element is already sorted
  if (arr.length <= 1) return arr;

  // Divide
  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));

  // Conquer (merge)
  return merge(left, right);
}

function merge(left, right) {
  const result = [];
  let i = 0,
    j = 0;

  while (i < left.length && j < right.length) {
    if (left[i] <= right[j]) {
      result.push(left[i]);
      i++;
    } else {
      result.push(right[j]);
      j++;
    }
  }

  // Append remaining elements
  return result.concat(left.slice(i)).concat(right.slice(j));
}
```

| Case    | Time       | Why                               |
| ------- | ---------- | --------------------------------- |
| Best    | O(n log n) | Always divides and merges         |
| Average | O(n log n) | Always divides and merges         |
| Worst   | O(n log n) | Always divides and merges         |
| Space   | O(n)       | Needs auxiliary array for merging |

> **Stable sort** — equal elements maintain their relative order. Guaranteed O(n log n) regardless of input. The go-to when stability or worst-case guarantees matter.

---

## Quick Sort

Choose a **pivot**, partition the array so elements smaller than pivot go left and larger go right, then recursively sort each partition.

```
[8, 3, 1, 7, 0, 10, 2]  pivot = 7

Partition around 7:
Left (< 7): [3, 1, 0, 2]    pivot: [7]    Right (> 7): [8, 10]

Recursively sort each:
[0, 1, 2, 3] + [7] + [8, 10]
= [0, 1, 2, 3, 7, 8, 10] ✓
```

```javascript
function quickSort(arr, low = 0, high = arr.length - 1) {
  if (low < high) {
    const pivotIndex = partition(arr, low, high);
    quickSort(arr, low, pivotIndex - 1); // Sort left
    quickSort(arr, pivotIndex + 1, high); // Sort right
  }
  return arr;
}

function partition(arr, low, high) {
  const pivot = arr[high]; // Choose last element as pivot
  let i = low - 1; // Index of smaller element boundary

  for (let j = low; j < high; j++) {
    if (arr[j] < pivot) {
      i++;
      [arr[i], arr[j]] = [arr[j], arr[i]]; // Swap to left section
    }
  }

  // Place pivot in correct position
  [arr[i + 1], arr[high]] = [arr[high], arr[i + 1]];
  return i + 1; // Pivot's final index
}
```

| Case    | Time       | Why                                        |
| ------- | ---------- | ------------------------------------------ |
| Best    | O(n log n) | Pivot splits evenly                        |
| Average | O(n log n) | Random pivot typically splits well         |
| Worst   | O(n²)      | Already sorted + always pick min/max pivot |
| Space   | O(log n)   | Recursion stack (in-place partitioning)    |

> **Not stable** — equal elements may change relative order. Faster than merge sort in practice due to cache locality and smaller constants. Worst case avoided with randomized pivot.

**Randomized pivot to avoid worst case:**

```javascript
function partition(arr, low, high) {
  // Randomize pivot to avoid O(n²) on sorted input
  const randomIndex = low + Math.floor(Math.random() * (high - low + 1));
  [arr[randomIndex], arr[high]] = [arr[high], arr[randomIndex]];

  const pivot = arr[high];
  let i = low - 1;

  for (let j = low; j < high; j++) {
    if (arr[j] < pivot) {
      i++;
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
  }
  [arr[i + 1], arr[high]] = [arr[high], arr[i + 1]];
  return i + 1;
}
```

---

## Counting Sort

A non-comparison sort for integers within a known range. Counts occurrences of each value, then reconstructs the sorted array.

```
Input: [4, 2, 2, 8, 3, 3, 1]
Range: 1 to 8

Count array (index = value):
Index: 0  1  2  3  4  5  6  7  8
Count: 0  1  2  2  1  0  0  0  1

Reconstruct: [1, 2, 2, 3, 3, 4, 8] ✓
```

```javascript
function countingSort(arr) {
  if (arr.length === 0) return arr;

  const max = Math.max(...arr);
  const min = Math.min(...arr);
  const range = max - min + 1;
  const count = new Array(range).fill(0);
  const output = new Array(arr.length);

  // Count occurrences
  for (const num of arr) {
    count[num - min]++;
  }

  // Cumulative count (for stable sort)
  for (let i = 1; i < range; i++) {
    count[i] += count[i - 1];
  }

  // Build output array (traverse input in reverse for stability)
  for (let i = arr.length - 1; i >= 0; i--) {
    const pos = count[arr[i] - min] - 1;
    output[pos] = arr[i];
    count[arr[i] - min]--;
  }

  return output;
}
```

| Case | Time     | Space    | Condition           |
| ---- | -------- | -------- | ------------------- |
| All  | O(n + k) | O(n + k) | k = range of values |

> Only efficient when k (range) is not significantly larger than n. Useless for large range or floating point numbers.

---

## Comparison Table

| Algorithm      | Best       | Average    | Worst      | Space    | Stable | In-Place |
| -------------- | ---------- | ---------- | ---------- | -------- | ------ | -------- |
| Bubble Sort    | O(n)       | O(n²)      | O(n²)      | O(1)     | Yes    | Yes      |
| Selection Sort | O(n²)      | O(n²)      | O(n²)      | O(1)     | No     | Yes      |
| Insertion Sort | O(n)       | O(n²)      | O(n²)      | O(1)     | Yes    | Yes      |
| Merge Sort     | O(n log n) | O(n log n) | O(n log n) | O(n)     | Yes    | No       |
| Quick Sort     | O(n log n) | O(n log n) | O(n²)      | O(log n) | No     | Yes      |
| Counting Sort  | O(n + k)   | O(n + k)   | O(n + k)   | O(n + k) | Yes    | No       |

**Stability** means equal elements maintain their original relative order. Important when sorting objects by one key while preserving order from a previous sort.

---

## JavaScript's Built-in .sort()

JavaScript uses **TimSort** (hybrid of merge sort + insertion sort) — O(n log n) average and worst, stable.

```javascript
// Default: converts to strings and sorts lexicographically
[10, 9, 80, 2].sort();
// [10, 2, 80, 9] ← WRONG for numbers!

// Always provide a comparator for numbers
[10, 9, 80, 2].sort((a, b) => a - b);
// [2, 9, 10, 80] ✓

// Descending
[10, 9, 80, 2].sort((a, b) => b - a);
// [80, 10, 9, 2]

// Sort objects
const users = [
  { name: "Vikas", age: 25 },
  { name: "Priya", age: 22 },
  { name: "Rahul", age: 30 },
];
users.sort((a, b) => a.age - b.age);
// [Priya(22), Vikas(25), Rahul(30)]

// Sort strings (case-insensitive)
["banana", "Apple", "cherry"].sort((a, b) =>
  a.toLowerCase().localeCompare(b.toLowerCase()),
);
// ["Apple", "banana", "cherry"]
```

> **Important:** `.sort()` mutates the array. Use `[...arr].sort()` or `.toSorted()` (ES2023) for non-mutating sort.

---

## When to Use Which Sort

| Situation                        | Best Choice        | Why                                                   |
| -------------------------------- | ------------------ | ----------------------------------------------------- |
| General purpose                  | Built-in `.sort()` | TimSort, optimized, stable, O(n log n)                |
| Nearly sorted data               | Insertion Sort     | O(n) best case, adaptive                              |
| Guaranteed O(n log n) needed     | Merge Sort         | No worst case degradation                             |
| Average case performance matters | Quick Sort         | Fastest in practice, good cache locality              |
| Integers in small range          | Counting Sort      | O(n + k) — beats comparison sorts                     |
| Stability required               | Merge Sort         | Stable + guaranteed O(n log n)                        |
| Memory constrained               | Quick Sort         | O(log n) space vs Merge Sort's O(n)                   |
| Small arrays (n < 20)            | Insertion Sort     | Low overhead, often faster than O(n log n) algorithms |
| Linked lists                     | Merge Sort         | No random access needed, O(1) extra space             |

---

## Interview Tips

1. **Almost never implement sorting from scratch** in an interview problem. Use the built-in `.sort()` and focus on the problem logic.

2. **Know when to sort first:** If sorting the input makes the problem easier (e.g., two pointers, binary search, removing duplicates), sort it — O(n log n) is usually acceptable.

3. **Be ready to explain** Merge Sort and Quick Sort at a conceptual level — interviewers love asking how they work.

4. **Understand stability:** When asked "sort by X then by Y," you need a stable sort or a multi-key comparator.

5. **Quick Sort's weakness:** Already sorted data + bad pivot choice = O(n²). Randomized pivot fixes this in practice.

6. **Merge Sort's strength:** Guaranteed O(n log n) and stable. Ideal for linked lists (no extra space for merging).

---

## Summary

- **O(n²) sorts** (Bubble, Selection, Insertion) are educational — understand them, rarely use them.
- **Insertion Sort** is the exception — excellent for nearly sorted data and small arrays.
- **Merge Sort** — divide and conquer, O(n log n) guaranteed, stable, but O(n) extra space.
- **Quick Sort** — fastest in practice, O(n log n) average, in-place, but O(n²) worst case without random pivot.
- **Counting Sort** — O(n + k) for integers in limited range, not comparison-based.
- **JavaScript's `.sort()`** uses TimSort — always provide a comparator for numbers.
- In interviews: use built-in sort + solve the actual problem. Know Merge Sort and Quick Sort conceptually.
