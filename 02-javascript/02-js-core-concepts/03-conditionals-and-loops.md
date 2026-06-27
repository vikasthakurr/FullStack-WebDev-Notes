# Conditionals and Loops

## Conditional Statements

### `if / else if / else`

```javascript
let score = 85;

if (score >= 90) {
  console.log("A grade");
} else if (score >= 80) {
  console.log("B grade");
} else if (score >= 70) {
  console.log("C grade");
} else {
  console.log("Needs improvement");
}
// Output: B grade
```

### Ternary Operator

A concise one-line conditional for simple cases:

```javascript
let age = 20;
let status = age >= 18 ? "adult" : "minor";
// "adult"

// Nested ternary (avoid — hard to read)
let category = age < 13 ? "child" : age < 18 ? "teen" : "adult";
```

### `switch`

Use when comparing a single value against multiple options:

```javascript
let day = "Monday";

switch (day) {
  case "Monday":
  case "Tuesday":
  case "Wednesday":
  case "Thursday":
  case "Friday":
    console.log("Weekday");
    break;
  case "Saturday":
  case "Sunday":
    console.log("Weekend");
    break;
  default:
    console.log("Invalid day");
}
```

**Important:** Without `break`, execution "falls through" to the next case.

```javascript
// Switch uses strict equality (===)
switch (1) {
  case "1": // Does NOT match — different types
    break;
  case 1: // Matches
    console.log("Found it");
    break;
}
```

### Logical Operators in Conditions

```javascript
// AND (&&) — both must be true
if (age >= 18 && hasLicense) {
  console.log("Can drive");
}

// OR (||) — at least one must be true
if (isAdmin || isModerator) {
  console.log("Has access");
}

// NOT (!) — inverts boolean
if (!isLoggedIn) {
  console.log("Please log in");
}

// Nullish coalescing (??) — fallback for null/undefined
let username = inputName ?? "Anonymous";

// Optional chaining (?.) — safe property access
let city = user?.address?.city; // undefined if any is null/undefined (no error)
```

### Short-Circuit Evaluation

```javascript
// && returns first falsy OR last value
let result = "hello" && 42 && true; // true (all truthy, returns last)
let result2 = "hello" && 0 && true; // 0 (first falsy)

// || returns first truthy OR last value
let name = "" || "Default"; // "Default" (first truthy)
let port = config.port || 3000;

// ?? returns first non-null/undefined
let value = 0 ?? 42; // 0 (0 is not null/undefined)
let value2 = null ?? 42; // 42
```

---

## Loops

### `for` Loop

Classic counting loop:

```javascript
for (let i = 0; i < 5; i++) {
  console.log(i); // 0, 1, 2, 3, 4
}

// Reverse
for (let i = 10; i > 0; i--) {
  console.log(i); // 10, 9, 8, ..., 1
}

// Step by 2
for (let i = 0; i <= 10; i += 2) {
  console.log(i); // 0, 2, 4, 6, 8, 10
}
```

### `while` Loop

Runs while condition is true — use when iteration count is unknown:

```javascript
let attempts = 0;

while (attempts < 3) {
  console.log(`Attempt ${attempts + 1}`);
  attempts++;
}
```

### `do...while` Loop

Guarantees at least one execution:

```javascript
let input;
do {
  input = prompt("Enter 'yes' to continue:");
} while (input !== "yes");
```

### `for...of` Loop

Iterates over **values** of iterable objects (arrays, strings, Maps, Sets):

```javascript
let fruits = ["apple", "banana", "cherry"];

for (let fruit of fruits) {
  console.log(fruit); // apple, banana, cherry
}

// Works with strings
for (let char of "hello") {
  console.log(char); // h, e, l, l, o
}
```

### `for...in` Loop

Iterates over **keys** (property names) of an object:

```javascript
let user = { name: "Vikas", age: 25, city: "Delhi" };

for (let key in user) {
  console.log(`${key}: ${user[key]}`);
}
// name: Vikas
// age: 25
// city: Delhi
```

**Warning:** `for...in` also iterates over inherited properties. Use `Object.keys()` or `hasOwnProperty()` if that is a concern.

### `forEach` (Array Method)

```javascript
let numbers = [1, 2, 3, 4, 5];

numbers.forEach((num, index) => {
  console.log(`Index ${index}: ${num}`);
});
```

- Cannot use `break` or `continue` inside `forEach`.
- Does not return a value (unlike `map`).

---

## Loop Control

### `break` — Exit the Loop Entirely

```javascript
for (let i = 0; i < 100; i++) {
  if (i === 5) break; // Stop at 5
  console.log(i); // 0, 1, 2, 3, 4
}
```

### `continue` — Skip Current Iteration

```javascript
for (let i = 0; i < 10; i++) {
  if (i % 2 === 0) continue; // Skip even numbers
  console.log(i); // 1, 3, 5, 7, 9
}
```

---

## Choosing the Right Loop

| Scenario                            | Best Loop          |
| ----------------------------------- | ------------------ |
| Known number of iterations          | `for`              |
| Unknown iterations, condition-based | `while`            |
| Need at least one execution         | `do...while`       |
| Iterating array/string values       | `for...of`         |
| Iterating object keys               | `for...in`         |
| Array with side effects (no return) | `forEach`          |
| Transform array into new array      | `map` (not a loop) |
| Filter items from array             | `filter`           |

---

## Common Mistakes

| Mistake                           | Why It Is Wrong                                            | Fix                                       |
| --------------------------------- | ---------------------------------------------------------- | ----------------------------------------- |
| Infinite loop (no exit condition) | Browser/Node.js freezes                                    | Ensure condition eventually becomes false |
| `for...in` on arrays              | Iterates indices as strings, includes inherited properties | Use `for...of` for arrays                 |
| Missing `break` in switch         | Falls through to next case                                 | Add `break` after each case               |
| `==` in conditions                | Coercion causes unexpected matches                         | Use `===`                                 |
| Modifying array while iterating   | Skips items or causes infinite loop                        | Iterate a copy or use `filter`            |

---

## Summary

- Use `if/else` for branching logic, `switch` for comparing one value against many options.
- Use `===` (strict equality) in all comparisons.
- `for` is for counted loops, `while` for condition-based loops, `for...of` for iterables, `for...in` for object keys.
- `break` exits the loop; `continue` skips to the next iteration.
- Short-circuit evaluation (`&&`, `||`, `??`) is a powerful alternative to simple `if` statements.
- Prefer modern array methods (`map`, `filter`, `reduce`) over loops when transforming data.
