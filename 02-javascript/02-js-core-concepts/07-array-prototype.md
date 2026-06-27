# Array.prototype — Methods, Patterns & Performance

## What Is Array.prototype?

`Array.prototype` is the object from which all JavaScript arrays inherit their methods. When you create an array, it does not carry its own copies of `map`, `filter`, `push`, etc. — it inherits them through the **prototype chain** from `Array.prototype`.

**Analogy:** Think of `Array.prototype` as a shared toolbox in a workshop. Every workbench (array instance) can reach into the toolbox to use any tool (method). The tools are not duplicated on every workbench — they live in one shared place.

---

## The Prototype Chain for Arrays

```javascript
const arr = [1, 2, 3];

// arr itself has no 'map' property — it inherits it
console.log(arr.hasOwnProperty("map")); // false
console.log(Array.prototype.hasOwnProperty("map")); // true

// The chain
Object.getPrototypeOf(arr) === Array.prototype; // true
Object.getPrototypeOf(Array.prototype) === Object.prototype; // true
Object.getPrototypeOf(Object.prototype) === null; // true
```

**Chain:** `array instance → Array.prototype → Object.prototype → null`

---

## Mutating Methods — push, pop, shift, unshift, splice

These methods **modify the original array** in place.

### push / pop (End of Array)

```javascript
const fruits = ["apple", "banana"];

fruits.push("cherry"); // Returns new length: 3
console.log(fruits); // ["apple", "banana", "cherry"]

fruits.push("date", "elderberry"); // Push multiple
console.log(fruits); // ["apple", "banana", "cherry", "date", "elderberry"]

const removed = fruits.pop(); // Returns removed element
console.log(removed); // "elderberry"
console.log(fruits); // ["apple", "banana", "cherry", "date"]
```

### shift / unshift (Start of Array)

```javascript
const nums = [10, 20, 30];

nums.unshift(0); // Returns new length: 4
console.log(nums); // [0, 10, 20, 30]

nums.unshift(-2, -1); // Unshift multiple
console.log(nums); // [-2, -1, 0, 10, 20, 30]

const first = nums.shift(); // Returns removed element
console.log(first); // -2
console.log(nums); // [-1, 0, 10, 20, 30]
```

> **Performance note:** `shift` and `unshift` are O(n) because every element must be re-indexed. `push` and `pop` are O(1).

### splice (The Swiss Army Knife)

`splice(startIndex, deleteCount, ...itemsToInsert)` — removes and/or inserts elements at any position.

```javascript
const colors = ["red", "green", "blue", "yellow", "purple"];

// Remove 2 elements starting at index 1
const removed = colors.splice(1, 2);
console.log(removed); // ["green", "blue"]
console.log(colors); // ["red", "yellow", "purple"]

// Insert without removing (deleteCount = 0)
colors.splice(1, 0, "orange", "pink");
console.log(colors); // ["red", "orange", "pink", "yellow", "purple"]

// Replace: remove 1 element at index 2, insert "cyan"
colors.splice(2, 1, "cyan");
console.log(colors); // ["red", "orange", "cyan", "yellow", "purple"]

// Negative index: counts from end
colors.splice(-1, 1); // Remove last element
console.log(colors); // ["red", "orange", "cyan", "yellow"]
```

### slice (Non-Mutating Copy)

`slice(start, end)` — returns a **shallow copy** of a portion. Does NOT modify the original.

```javascript
const arr = [10, 20, 30, 40, 50];

arr.slice(1, 3); // [20, 30] — from index 1, up to (not including) 3
arr.slice(2); // [30, 40, 50] — from index 2 to end
arr.slice(-2); // [40, 50] — last 2 elements
arr.slice(); // [10, 20, 30, 40, 50] — full shallow copy

console.log(arr); // [10, 20, 30, 40, 50] — unchanged!
```

**Splice vs Slice:**

| Feature     | `splice`                  | `slice`                    |
| ----------- | ------------------------- | -------------------------- |
| Mutates?    | Yes                       | No                         |
| Returns     | Array of removed elements | New array (copied portion) |
| Can insert? | Yes                       | No                         |
| Use case    | Modify array in place     | Extract a section safely   |

---

## Iteration Methods — map, filter, reduce, find, findIndex, some, every, includes

These methods iterate over the array and return a new value without mutating the original.

### map — Transform Each Element

Returns a **new array** with the result of calling the callback on each element.

```javascript
const prices = [10, 20, 30, 40];

const withTax = prices.map((price) => price * 1.18);
// [11.8, 23.6, 35.4, 47.2]

// Access index and full array
const indexed = prices.map((price, index, array) => {
  return `Item ${index + 1}: ₹${price} of ${array.length} items`;
});

// Practical: extract fields from objects
const users = [
  { name: "Vikas", age: 25 },
  { name: "Rahul", age: 30 },
  { name: "Priya", age: 22 },
];
const names = users.map((user) => user.name);
// ["Vikas", "Rahul", "Priya"]
```

### filter — Keep Elements That Pass a Test

Returns a **new array** with only elements where the callback returns `true`.

```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

const evens = numbers.filter((n) => n % 2 === 0);
// [2, 4, 6, 8, 10]

const adults = users.filter((user) => user.age >= 25);
// [{ name: "Vikas", age: 25 }, { name: "Rahul", age: 30 }]

// Remove falsy values
const mixed = [0, "hello", "", null, 42, undefined, "world"];
const truthy = mixed.filter(Boolean);
// ["hello", 42, "world"]
```

### reduce — Accumulate into a Single Value

The most powerful iteration method. Reduces an array to a single value (number, string, object, array — anything).

```javascript
const numbers = [1, 2, 3, 4, 5];

// Sum
const sum = numbers.reduce((accumulator, current) => accumulator + current, 0);
// 15

// Max value
const max = numbers.reduce((max, n) => (n > max ? n : max), -Infinity);
// 5

// Count occurrences
const fruits = ["apple", "banana", "apple", "cherry", "banana", "apple"];
const counts = fruits.reduce((acc, fruit) => {
  acc[fruit] = (acc[fruit] || 0) + 1;
  return acc;
}, {});
// { apple: 3, banana: 2, cherry: 1 }

// Group by property
const people = [
  { name: "Vikas", dept: "engineering" },
  { name: "Rahul", dept: "engineering" },
  { name: "Priya", dept: "design" },
  { name: "Sneha", dept: "design" },
];

const byDept = people.reduce((groups, person) => {
  const key = person.dept;
  groups[key] = groups[key] || [];
  groups[key].push(person);
  return groups;
}, {});
// { engineering: [{...}, {...}], design: [{...}, {...}] }

// Flatten (manual alternative to .flat())
const nested = [[1, 2], [3, 4], [5]];
const flat = nested.reduce((acc, arr) => acc.concat(arr), []);
// [1, 2, 3, 4, 5]
```

> **Always provide an initial value** (the second argument to `reduce`). Without it, the first element becomes the initial accumulator, which causes bugs with empty arrays and unexpected types.

### find / findIndex — Locate a Single Element

`find` returns the **first element** that satisfies the condition (or `undefined`).
`findIndex` returns the **index** of the first match (or `-1`).

```javascript
const users = [
  { id: 1, name: "Vikas", role: "admin" },
  { id: 2, name: "Rahul", role: "user" },
  { id: 3, name: "Priya", role: "user" },
];

const admin = users.find((user) => user.role === "admin");
// { id: 1, name: "Vikas", role: "admin" }

const notFound = users.find((user) => user.role === "superadmin");
// undefined

const adminIndex = users.findIndex((user) => user.role === "admin");
// 0

const missingIndex = users.findIndex((user) => user.id === 99);
// -1
```

**When to use `find` vs `filter`:**

- `find` — you want the **first** match (stops iterating after finding it).
- `filter` — you want **all** matches.

### some / every — Boolean Tests

`some` returns `true` if **at least one** element passes the test.
`every` returns `true` only if **all** elements pass the test.

```javascript
const ages = [16, 18, 21, 25, 30];

ages.some((age) => age >= 21); // true — at least one is 21+
ages.every((age) => age >= 18); // false — 16 fails

const scores = [85, 90, 78, 92, 88];
scores.every((s) => s >= 70); // true — all pass
scores.some((s) => s === 100); // false — none is 100

// Practical: check if array has duplicates
const hasDuplicates = (arr) =>
  arr.some((item, index) => arr.indexOf(item) !== index);
hasDuplicates([1, 2, 3, 4]); // false
hasDuplicates([1, 2, 2, 4]); // true
```

> `some` short-circuits on the first `true`. `every` short-circuits on the first `false`. They do not always iterate the full array.

### includes — Check for Existence

Returns `true` if the array contains the value (uses `===` comparison, but handles `NaN`).

```javascript
const fruits = ["apple", "banana", "cherry"];

fruits.includes("banana"); // true
fruits.includes("grape"); // false

// Handles NaN (unlike indexOf)
[1, 2, NaN].includes(NaN); // true
[1, 2, NaN].indexOf(NaN); // -1 ← fails!

// Optional second argument: start searching from index
fruits.includes("apple", 1); // false — starts searching at index 1
```

**`includes` vs `indexOf`:**

| Feature       | `includes`      | `indexOf`         |
| ------------- | --------------- | ----------------- |
| Returns       | `boolean`       | `number` (index)  |
| Handles `NaN` | Yes             | No                |
| Readability   | More intuitive  | Requires `!== -1` |
| Use case      | Existence check | Need the position |

---

## Transformation Methods — flat, flatMap, Array.from, Array.of

### flat — Flatten Nested Arrays

```javascript
const nested = [1, [2, 3], [4, [5, 6]]];

nested.flat(); // [1, 2, 3, 4, [5, 6]] — flattens 1 level (default)
nested.flat(2); // [1, 2, 3, 4, 5, 6] — flattens 2 levels
nested.flat(Infinity); // [1, 2, 3, 4, 5, 6] — flattens all levels

// Practical: merge arrays of results
const responses = [[user1, user2], [user3], [user4, user5]];
const allUsers = responses.flat(); // [user1, user2, user3, user4, user5]

// Also removes empty slots
const sparse = [1, , 3, , 5];
sparse.flat(); // [1, 3, 5]
```

### flatMap — map + flat(1) in One Step

Maps each element, then flattens the result by one level. More efficient than calling `.map().flat()` separately.

```javascript
const sentences = ["Hello world", "How are you"];

// split each sentence into words, then flatten
const words = sentences.flatMap((s) => s.split(" "));
// ["Hello", "world", "How", "are", "you"]

// Without flatMap: sentences.map(s => s.split(" ")).flat()
// Same result, but two passes over the data

// Practical: expand items
const cart = [
  { item: "shirt", qty: 2 },
  { item: "pants", qty: 1 },
];
const expanded = cart.flatMap((entry) => Array(entry.qty).fill(entry.item));
// ["shirt", "shirt", "pants"]

// Filter and transform in one pass (return empty array to "remove")
const nums = [1, 2, 3, 4, 5, 6];
const doubledEvens = nums.flatMap((n) => (n % 2 === 0 ? [n * 2] : []));
// [4, 8, 12] — filters odds AND doubles evens in one step
```

### Array.from — Create Arrays from Iterables/Array-likes

```javascript
// From a string
Array.from("hello"); // ["h", "e", "l", "l", "o"]

// From a NodeList
const divs = document.querySelectorAll("div");
const divArray = Array.from(divs); // Now a real array with map, filter, etc.

// From a Set
Array.from(new Set([1, 2, 2, 3])); // [1, 2, 3]

// With a mapping function (second argument)
Array.from({ length: 5 }, (_, i) => i * 2);
// [0, 2, 4, 6, 8]

// Generate a range
const range = (start, end) =>
  Array.from({ length: end - start }, (_, i) => start + i);
range(1, 6); // [1, 2, 3, 4, 5]
```

### Array.of — Create Array from Arguments

Solves the `new Array()` ambiguity:

```javascript
// The problem with new Array():
new Array(3); // [empty × 3] — creates 3 empty slots (not [3])
new Array(1, 2, 3); // [1, 2, 3] — inconsistent behavior

// Array.of is always predictable:
Array.of(3); // [3]
Array.of(1, 2, 3); // [1, 2, 3]
Array.of(undefined); // [undefined]
```

---

## Sorting — sort (with Comparator) and reverse

### sort — Default Behavior (Dangerous!)

By default, `sort()` converts elements to strings and sorts lexicographically:

```javascript
const nums = [10, 9, 80, 2, 33, 1];
nums.sort();
console.log(nums); // [1, 10, 2, 33, 80, 9] ← WRONG for numbers!

// "10" < "2" because "1" < "2" in Unicode
// "80" < "9" because "8" < "9" in Unicode
```

### sort — With Comparator Function

The comparator takes two elements `a` and `b` and returns:

- **Negative** → `a` comes first
- **Zero** → order unchanged
- **Positive** → `b` comes first

```javascript
const nums = [10, 9, 80, 2, 33, 1];

// Ascending (a - b)
nums.sort((a, b) => a - b);
console.log(nums); // [1, 2, 9, 10, 33, 80] ✅

// Descending (b - a)
nums.sort((a, b) => b - a);
console.log(nums); // [80, 33, 10, 9, 2, 1]

// Sort strings (case-insensitive)
const names = ["banana", "Apple", "cherry", "avocado"];
names.sort((a, b) => a.toLowerCase().localeCompare(b.toLowerCase()));
// ["Apple", "avocado", "banana", "cherry"]

// Sort objects by property
const users = [
  { name: "Vikas", age: 25 },
  { name: "Rahul", age: 30 },
  { name: "Priya", age: 22 },
];

users.sort((a, b) => a.age - b.age);
// [Priya(22), Vikas(25), Rahul(30)]

// Multi-level sort: by department, then by name
const employees = [
  { name: "Vikas", dept: "eng" },
  { name: "Ankit", dept: "eng" },
  { name: "Priya", dept: "design" },
  { name: "Zara", dept: "design" },
];

employees.sort((a, b) => {
  if (a.dept !== b.dept) return a.dept.localeCompare(b.dept);
  return a.name.localeCompare(b.name);
});
// design: Priya, Zara | eng: Ankit, Vikas
```

### reverse — Reverses In Place

```javascript
const arr = [1, 2, 3, 4, 5];
arr.reverse();
console.log(arr); // [5, 4, 3, 2, 1] — original is mutated!
```

### Non-Mutating Alternatives (ES2023+)

```javascript
const original = [3, 1, 4, 1, 5];

// toSorted — returns a new sorted array
const sorted = original.toSorted((a, b) => a - b);
console.log(sorted); // [1, 1, 3, 4, 5]
console.log(original); // [3, 1, 4, 1, 5] — unchanged!

// toReversed — returns a new reversed array
const reversed = original.toReversed();
console.log(reversed); // [5, 1, 4, 1, 3]
console.log(original); // [3, 1, 4, 1, 5] — unchanged!

// Pre-ES2023 workaround: spread then sort
const safeSorted = [...original].sort((a, b) => a - b);
```

> **Important:** `sort()` mutates the array AND returns the same reference. `const sorted = arr.sort()` does NOT protect the original — `sorted` and `arr` point to the same array.

---

## Creating Custom Methods on Array.prototype

You **can** add methods to `Array.prototype` so all arrays inherit them. This is educational — understand it, but avoid it in production.

### Example: Adding a `.last()` Method

```javascript
Array.prototype.last = function () {
  return this[this.length - 1];
};

[10, 20, 30].last(); // 30
["a", "b", "c"].last(); // "c"
[].last(); // undefined
```

### Example: Adding a `.sum()` Method

```javascript
Array.prototype.sum = function () {
  return this.reduce((total, n) => total + n, 0);
};

[1, 2, 3, 4, 5].sum(); // 15
```

### Example: Adding a `.unique()` Method

```javascript
Array.prototype.unique = function () {
  return [...new Set(this)];
};

[1, 2, 2, 3, 3, 3].unique(); // [1, 2, 3]
```

### Why You Should NOT Do This in Production

1. **Name collisions** — a future JavaScript spec may add a method with the same name (this happened with `Array.prototype.at`).
2. **Pollutes all arrays** — every array in your app (including from libraries) gets your method.
3. **Breaks `for...in`** — custom prototype properties are enumerable by default.
4. **Hard to debug** — tracing where a prototype method was defined across a large codebase is difficult.

### Safer Alternative: `Object.defineProperty`

If you must extend the prototype, make it non-enumerable:

```javascript
Object.defineProperty(Array.prototype, "last", {
  value: function () {
    return this[this.length - 1];
  },
  enumerable: false,
  writable: true,
  configurable: true,
});

// Won't show up in for...in
for (let key in [1, 2, 3]) {
  console.log(key); // "0", "1", "2" — no "last"
}
```

### Better Alternative: Utility Functions or Subclassing

```javascript
// Utility function approach
const last = (arr) => arr[arr.length - 1];
last([1, 2, 3]); // 3

// Subclass approach
class SuperArray extends Array {
  last() {
    return this[this.length - 1];
  }
  sum() {
    return this.reduce((t, n) => t + n, 0);
  }
}

const arr = SuperArray.from([1, 2, 3, 4, 5]);
arr.last(); // 5
arr.sum(); // 15
arr.filter((n) => n > 2); // SuperArray [3, 4, 5] — preserves type!
```

---

## Method Chaining Patterns

Method chaining works because most non-mutating array methods return a new array, which you can immediately call another method on.

### Basic Chaining

```javascript
const transactions = [
  { amount: 100, type: "credit", category: "salary" },
  { amount: 50, type: "debit", category: "food" },
  { amount: 200, type: "credit", category: "freelance" },
  { amount: 75, type: "debit", category: "transport" },
  { amount: 30, type: "debit", category: "food" },
];

// Get total spending on food
const foodSpending = transactions
  .filter((t) => t.type === "debit")
  .filter((t) => t.category === "food")
  .map((t) => t.amount)
  .reduce((sum, amount) => sum + amount, 0);
// 80
```

### Real-World Chaining Examples

```javascript
// Process API data: filter active users, sort by name, get display strings
const users = [
  { name: "Vikas", active: true, score: 85 },
  { name: "Rahul", active: false, score: 72 },
  { name: "Priya", active: true, score: 91 },
  { name: "Ankit", active: true, score: 68 },
];

const leaderboard = users
  .filter((u) => u.active)
  .sort((a, b) => b.score - a.score)
  .map((u, i) => `${i + 1}. ${u.name} (${u.score})`);
// ["1. Priya (91)", "2. Vikas (85)", "3. Ankit (68)"]

// Pipeline: clean, validate, transform
const rawEmails = [
  "  Vikas@Email.COM ",
  "invalid",
  "rahul@test.com",
  "",
  "  PRIYA@dev.IO  ",
];

const validEmails = rawEmails
  .map((e) => e.trim().toLowerCase())
  .filter((e) => e.includes("@") && e.includes("."))
  .filter((e) => e.length > 0);
// ["vikas@email.com", "rahul@test.com", "priya@dev.io"]
```

### Chaining with Intermediate Debugging

```javascript
// Use a tap utility for debugging mid-chain
const tap = (label) => (value) => {
  console.log(`[${label}]`, value);
  return value;
};

// Unfortunately you can't directly tap in a chain without a custom method.
// Common pattern: break the chain for debugging
const step1 = transactions.filter((t) => t.type === "debit");
console.log("After filter:", step1);
const step2 = step1.map((t) => t.amount);
console.log("After map:", step2);
const result = step2.reduce((sum, a) => sum + a, 0);
```

---

## Performance Considerations

### Method Complexity

| Method                 | Time Complexity | Notes                                       |
| ---------------------- | --------------- | ------------------------------------------- |
| `push` / `pop`         | O(1)            | Operates on end — no re-indexing            |
| `shift` / `unshift`    | O(n)            | Must re-index all elements                  |
| `splice`               | O(n)            | Worst case: shifting elements after removal |
| `map` / `filter`       | O(n)            | Single pass through the array               |
| `reduce`               | O(n)            | Single pass                                 |
| `find` / `findIndex`   | O(n)            | Stops at first match (best case O(1))       |
| `some` / `every`       | O(n)            | Short-circuits (best case O(1))             |
| `includes` / `indexOf` | O(n)            | Linear search                               |
| `sort`                 | O(n log n)      | Typically TimSort in modern engines         |
| `concat`               | O(n + m)        | Creates new array with all elements         |
| `flat`                 | O(n)            | Per level of depth                          |
| `slice`                | O(k)            | k = number of elements in the slice         |

### Chaining Performance — Multiple Passes

Each chained method creates a new intermediate array and iterates through it:

```javascript
// This creates 3 intermediate arrays and does 3 passes:
const result = hugeArray
  .filter((x) => x > 10) // Pass 1: new array
  .map((x) => x * 2) // Pass 2: new array
  .filter((x) => x < 100); // Pass 3: new array

// Single-pass alternative with reduce (1 pass, 1 new array):
const result2 = hugeArray.reduce((acc, x) => {
  if (x > 10) {
    const doubled = x * 2;
    if (doubled < 100) acc.push(doubled);
  }
  return acc;
}, []);
```

> **Readability vs Performance:** For most applications, chaining is perfectly fine. Only optimize to single-pass when working with very large arrays (100k+ elements) where you've measured a bottleneck.

### Avoid Unnecessary Work

```javascript
// ❌ Bad: sort entire array then take first element
const youngest = users.sort((a, b) => a.age - b.age)[0];
// O(n log n) + mutates the original array!

// ✅ Good: single pass with reduce
const youngest2 = users.reduce((min, user) =>
  user.age < min.age ? user : min,
);
// O(n), no mutation

// ❌ Bad: checking existence with filter
if (users.filter((u) => u.active).length > 0) {
  /* ... */
}
// Creates entire filtered array just to check length

// ✅ Good: short-circuit with some
if (users.some((u) => u.active)) {
  /* ... */
}
// Stops at first match
```

### Large Array Tips

1. **Use `for` loops for hot paths** — method calls have overhead (function creation, stack frames).
2. **Prefer `Set` over `includes` for repeated lookups** — `Set.has()` is O(1) vs `includes` O(n).
3. **Use `TypedArrays`** for numeric data (e.g., `Float64Array`) — contiguous memory, no boxing.
4. **Avoid `delete arr[i]`** — creates sparse arrays that engines cannot optimize. Use `splice` or `filter`.
5. **Pre-allocate when possible** — `new Array(size)` then fill, instead of repeated `push`.

```javascript
// ❌ Slow for repeated lookups
const allowedIds = [1, 2, 3, 4, 5 /* ...thousands */];
data.filter((item) => allowedIds.includes(item.id)); // O(n * m)

// ✅ Fast with Set
const allowedSet = new Set(allowedIds);
data.filter((item) => allowedSet.has(item.id)); // O(n)
```

---

## Best Practices

1. **Prefer non-mutating methods** (`map`, `filter`, `slice`, `toSorted`) over mutating ones (`splice`, `sort`, `reverse`) — predictable code with fewer bugs.
2. **Always provide an initial value to `reduce`** — prevents errors on empty arrays and makes the accumulator type explicit.
3. **Use `find` when you need one result, `filter` when you need many** — `find` stops early.
4. **Use `some`/`every` instead of `filter().length`** — they short-circuit and don't allocate.
5. **Do not extend `Array.prototype` in production** — use utility functions or subclassing.
6. **Spread before sorting** if you need the original unchanged — `[...arr].sort(comparator)`.
7. **Use `Array.isArray()` for type checking** — more reliable than `instanceof` across realms.
8. **Use `flatMap` instead of `.map().flat()`** — single pass, more efficient.

---

## Common Mistakes

| Mistake                                       | Why It Is Wrong                                              | Fix                                                  |
| --------------------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------- |
| Sorting numbers without a comparator          | Default sort is lexicographic: `[10, 9, 80]` → `[10, 80, 9]` | Always pass `(a, b) => a - b` for numbers            |
| Forgetting `return` in `map`/`reduce`         | Returns `undefined` for each element                         | Use implicit return or explicit `return`             |
| Using `map` when you don't need the result    | `map` creates a new array — wasteful if unused               | Use `forEach` for side effects                       |
| Assuming `sort()` returns a new array         | `sort()` mutates in place AND returns the same reference     | Use `[...arr].sort()` or `.toSorted()` (ES2023)      |
| Adding enumerable prototype methods           | Breaks `for...in`, pollutes all arrays globally              | Use `Object.defineProperty` with `enumerable: false` |
| Not providing initial value to `reduce`       | Fails on empty arrays, unexpected accumulator type           | Always pass the second argument                      |
| Using `indexOf` to find objects               | Objects are compared by reference, not value                 | Use `find` or `findIndex` with a predicate           |
| Chaining after a mutating method              | `push` returns length (number), not the array                | Use `concat` or spread for immutable append + chain  |
| Using `includes` for repeated lookups in loop | O(n) per check → O(n²) total                                 | Convert to `Set` first for O(1) lookups              |
| Using arrow functions for prototype methods   | Arrow functions don't have their own `this`                  | Use `function` keyword for prototype methods         |

---

## Summary

- `Array.prototype` is the shared object from which all arrays inherit methods via the prototype chain.
- **Mutating methods** (`push`, `pop`, `shift`, `unshift`, `splice`, `sort`, `reverse`) change the original array in place.
- **Iteration methods** (`map`, `filter`, `reduce`, `find`, `findIndex`, `some`, `every`, `includes`) return new values without mutation.
- **Transformation methods** (`flat`, `flatMap`, `Array.from`, `Array.of`) create arrays from other structures.
- **Sorting** requires a comparator function for numbers — default sort is lexicographic and almost always wrong for numbers.
- You can extend `Array.prototype` with custom methods, but should not in production — use utility functions or subclass `Array` instead.
- **Method chaining** is powerful and readable — each non-mutating method returns a new array you can call the next method on.
- **Performance:** chaining creates intermediate arrays (multiple passes). For most apps this is fine — optimize with `reduce` or loops only when profiling shows a bottleneck.
