# Functions & Higher-Order Functions

## Functions in JavaScript

Functions are reusable blocks of code. In JavaScript, functions are **first-class citizens** — they can be assigned to variables, passed as arguments, and returned from other functions.

---

## Declaring Functions

### Function Declaration (Hoisted)

```javascript
greet("Vikas"); // ✅ Works — declarations are hoisted

function greet(name) {
  return `Hello, ${name}!`;
}
```

### Function Expression (Not Hoisted)

```javascript
// sayHi("Vikas"); // ❌ ReferenceError

const sayHi = function (name) {
  return `Hi, ${name}!`;
};
```

### Arrow Functions (ES6)

```javascript
const add = (a, b) => a + b; // Implicit return (single expression)

const multiply = (a, b) => {
  const result = a * b;
  return result; // Explicit return needed with {}
};

// Single parameter — no parentheses needed
const double = (x) => x * 2;
```

**Key differences of arrow functions:**

- No own `this` — inherits from enclosing scope (lexical `this`).
- No `arguments` object.
- Cannot be used as constructors (`new`).
- Cannot be used as generator functions.

---

## Parameters and Arguments

### Default Parameters

```javascript
function createUser(name, role = "user", isActive = true) {
  return { name, role, isActive };
}

createUser("Vikas"); // { name: "Vikas", role: "user", isActive: true }
createUser("Admin", "admin"); // { name: "Admin", role: "admin", isActive: true }
```

### Rest Parameters

Collect all remaining arguments into an array:

```javascript
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}

sum(1, 2, 3, 4); // 10

// Rest must be last
function logAll(first, ...rest) {
  console.log("First:", first);
  console.log("Rest:", rest);
}
logAll("a", "b", "c"); // First: a, Rest: ["b", "c"]
```

### The `arguments` Object (Legacy)

```javascript
function oldStyle() {
  console.log(arguments); // Array-like (not a real array)
  console.log(arguments[0]); // First argument
}
```

- Not available in arrow functions.
- Prefer rest parameters (`...args`) in modern code.

---

## Callback Functions

A callback is a function passed as an argument to another function, to be called later:

```javascript
function fetchData(url, onSuccess, onError) {
  // Simulate async operation
  setTimeout(() => {
    const success = Math.random() > 0.3;
    if (success) {
      onSuccess({ data: "Some data" });
    } else {
      onError(new Error("Failed to fetch"));
    }
  }, 1000);
}

fetchData(
  "/api/users",
  (result) => console.log("Got:", result),
  (error) => console.error("Error:", error),
);
```

### Common Callback Patterns

```javascript
// Array methods use callbacks
[1, 2, 3].forEach((num) => console.log(num));

// Event listeners
button.addEventListener("click", () => {
  console.log("Clicked!");
});

// setTimeout / setInterval
setTimeout(() => console.log("Delayed"), 1000);
```

---

## Higher-Order Functions

A higher-order function is a function that either:

1. **Takes a function as an argument**, or
2. **Returns a function**

### Taking a Function

```javascript
function repeat(n, action) {
  for (let i = 0; i < n; i++) {
    action(i);
  }
}

repeat(3, console.log); // 0, 1, 2
repeat(3, (i) => console.log(`Step ${i + 1}`)); // Step 1, Step 2, Step 3
```

### Returning a Function

```javascript
function greaterThan(n) {
  return (value) => value > n;
}

const greaterThan10 = greaterThan(10);
greaterThan10(15); // true
greaterThan10(5); // false
```

---

## `map`, `filter`, `reduce`

### `map` — Transform Each Element

```javascript
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((n) => n * 2);
// [2, 4, 6, 8, 10]

const users = [
  { name: "Vikas", age: 25 },
  { name: "Rahul", age: 30 },
];
const names = users.map((user) => user.name);
// ["Vikas", "Rahul"]
```

### `filter` — Keep Elements That Pass a Test

```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8];
const evens = numbers.filter((n) => n % 2 === 0);
// [2, 4, 6, 8]

const adults = users.filter((user) => user.age >= 18);
```

### `reduce` — Accumulate into a Single Value

```javascript
const numbers = [1, 2, 3, 4, 5];

// Sum
const sum = numbers.reduce((accumulator, current) => accumulator + current, 0);
// 15

// Max value
const max = numbers.reduce((max, n) => (n > max ? n : max), -Infinity);
// 5

// Group by
const people = [
  { name: "Vikas", dept: "eng" },
  { name: "Rahul", dept: "eng" },
  { name: "Priya", dept: "design" },
];

const byDept = people.reduce((groups, person) => {
  const dept = person.dept;
  groups[dept] = groups[dept] || [];
  groups[dept].push(person);
  return groups;
}, {});
// { eng: [{...}, {...}], design: [{...}] }
```

### Chaining Methods

```javascript
const transactions = [
  { amount: 100, type: "credit" },
  { amount: 50, type: "debit" },
  { amount: 200, type: "credit" },
  { amount: 75, type: "debit" },
];

const totalCredits = transactions
  .filter((t) => t.type === "credit")
  .map((t) => t.amount)
  .reduce((sum, amount) => sum + amount, 0);
// 300
```

---

## Currying

Currying transforms a function with multiple arguments into a sequence of functions each taking one argument:

```javascript
// Regular function
function add(a, b, c) {
  return a + b + c;
}
add(1, 2, 3); // 6

// Curried version
function curriedAdd(a) {
  return function (b) {
    return function (c) {
      return a + b + c;
    };
  };
}
curriedAdd(1)(2)(3); // 6

// Arrow function style
const curriedAdd2 = (a) => (b) => (c) => a + b + c;
```

### Practical Currying

```javascript
// Logger with preset level
const log = (level) => (message) => console.log(`[${level}] ${message}`);

const info = log("INFO");
const error = log("ERROR");

info("Server started"); // [INFO] Server started
error("Connection lost"); // [ERROR] Connection lost

// API endpoint builder
const api = (baseUrl) => (endpoint) => `${baseUrl}${endpoint}`;
const githubApi = api("https://api.github.com");

githubApi("/users"); // "https://api.github.com/users"
githubApi("/repos"); // "https://api.github.com/repos"
```

---

## IIFE (Immediately Invoked Function Expression)

A function that runs immediately after being defined:

```javascript
(function () {
  let secret = "hidden";
  console.log("IIFE ran!");
  // secret is not accessible outside
})();

// Arrow function IIFE
(() => {
  console.log("Arrow IIFE");
})();

// With parameters
((name) => {
  console.log(`Hello, ${name}!`);
})("Vikas");
```

**Use cases:**

- Avoid polluting the global scope (pre-ES6 modules).
- Create a private scope for variables.
- Initialize code that should run once.

---

## Best Practices

1. **Use arrow functions for callbacks** — concise and lexically bind `this`.
2. **Use descriptive function names** — `calculateTax` over `calc`.
3. **Keep functions small** — one function, one job (Single Responsibility).
4. **Prefer `map`/`filter`/`reduce` over loops** for data transformation — more declarative and readable.
5. **Use default parameters** instead of checking for `undefined` inside the function.
6. **Avoid deeply nested callbacks** — use Promises or async/await instead (callback hell).

---

## Common Mistakes

| Mistake                                  | Why It Is Wrong                              | Fix                                     |
| ---------------------------------------- | -------------------------------------------- | --------------------------------------- |
| Forgetting `return` in `map`/`reduce`    | Returns `undefined` for each element         | Always return or use implicit return    |
| Mutating the original array in `map`     | `map` should create new data, not modify old | Return new objects                      |
| Using `forEach` expecting a return value | `forEach` always returns `undefined`         | Use `map` or `reduce` for results       |
| Arrow function with `this` in methods    | Arrow inherits `this` from enclosing scope   | Use regular function for object methods |
| Not providing initial value to `reduce`  | First element becomes accumulator            | Always pass the initial value           |

---

## Summary

- Functions are first-class in JavaScript — they can be passed, returned, and stored like any value.
- Arrow functions are concise and lexically bind `this` — use them for callbacks.
- Higher-order functions take or return functions — they enable powerful abstractions.
- `map` transforms, `filter` selects, `reduce` accumulates — learn to chain them fluently.
- Currying creates specialized functions from general ones by partially applying arguments.
- IIFE creates isolated scopes — useful for one-time initialization and avoiding global pollution.
