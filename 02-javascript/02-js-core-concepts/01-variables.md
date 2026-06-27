# Variables in JavaScript

## What Are Variables?

Variables are named containers that store data values. They give you a way to label, store, retrieve, and manipulate data throughout your program.

**Analogy:** Variables are labeled boxes in a warehouse. `let age = 25` means there is a box labeled "age" with the number 25 inside it. You can open the box (read the value), replace what is inside (reassign), or throw the box away (let it be garbage collected).

---

## Three Ways to Declare Variables

### `var` (Legacy — Avoid)

```javascript
var name = "Vikas";
var name = "Rahul"; // No error — redeclaration allowed
name = "Amit"; // Reassignment allowed
```

- **Function-scoped** — not block-scoped.
- Can be redeclared in the same scope.
- Hoisted with initial value `undefined`.
- Attaches to the `window` object in browsers.

### `let` (Use for Mutable Values)

```javascript
let age = 25;
// let age = 30; // SyntaxError — cannot redeclare
age = 30; // Reassignment allowed
```

- **Block-scoped** — only exists within `{}`.
- Cannot be redeclared in the same scope.
- Hoisted but in Temporal Dead Zone (TDZ) — accessing before declaration throws `ReferenceError`.
- Does not attach to `window`.

### `const` (Use for Immutable Bindings)

```javascript
const PI = 3.14159;
// PI = 3.14; // TypeError — cannot reassign
// const PI = 3; // SyntaxError — cannot redeclare
```

- **Block-scoped** — same as `let`.
- Cannot be reassigned or redeclared.
- Must be initialized at declaration.
- **The binding is constant, not the value:**

```javascript
const user = { name: "Vikas" };
user.name = "Rahul"; // ✅ Allowed — mutating the object, not reassigning the variable
// user = {};        // ❌ TypeError — cannot reassign the binding
```

---

## Comparison Table

| Feature         | `var`           | `let`      | `const`   |
| --------------- | --------------- | ---------- | --------- |
| Scope           | Function        | Block      | Block     |
| Hoisting        | Yes (undefined) | Yes (TDZ)  | Yes (TDZ) |
| Redeclaration   | ✅ Allowed      | ❌ Error   | ❌ Error  |
| Reassignment    | ✅ Allowed      | ✅ Allowed | ❌ Error  |
| Must initialize | No              | No         | Yes       |
| Global object   | Yes (`window`)  | No         | No        |

---

## Scope

### Block Scope (`let` and `const`)

```javascript
if (true) {
  let x = 10;
  const y = 20;
  var z = 30;
}

// console.log(x); // ReferenceError — block scoped
// console.log(y); // ReferenceError — block scoped
console.log(z); // 30 — var ignores block scope
```

### Function Scope (`var`)

```javascript
function demo() {
  var secret = "hidden";
}
// console.log(secret); // ReferenceError — function scoped
```

---

## Hoisting and Temporal Dead Zone

### `var` Hoisting

```javascript
console.log(a); // undefined (not ReferenceError!)
var a = 5;
console.log(a); // 5

// Engine sees it as:
// var a;           ← hoisted to top with value undefined
// console.log(a); ← undefined
// a = 5;          ← assignment stays in place
// console.log(a); ← 5
```

### `let` / `const` Temporal Dead Zone

```javascript
// console.log(b); // ReferenceError: Cannot access 'b' before initialization
let b = 10;
console.log(b); // 10
```

The variable exists (hoisted) but cannot be accessed until the declaration line. The zone between scope entry and declaration is the **Temporal Dead Zone (TDZ)**.

---

## Naming Conventions

```javascript
// camelCase for variables and functions
let firstName = "Vikas";
let isLoggedIn = true;
let calculateTotal = () => {};

// UPPER_SNAKE_CASE for true constants
const API_BASE_URL = "https://api.example.com";
const MAX_RETRIES = 3;

// PascalCase for classes and constructors
class UserAccount {}
```

### Rules

- Must start with a letter, `_`, or `$`.
- Cannot use reserved words (`let`, `const`, `class`, `if`, `return`, etc.).
- Case-sensitive (`Name` and `name` are different variables).

---

## Best Practices

1. **Default to `const`** — only use `let` if you need to reassign.
2. **Never use `var`** — it has confusing scope behavior and is a legacy feature.
3. **Declare variables close to their first use** — not at the top of the function.
4. **Use meaningful names** — `userAge` over `x`, `isLoading` over `flag`.
5. **One declaration per line** — easier to read and diff in version control.

```javascript
// Good
const maxItems = 100;
let currentCount = 0;

// Avoid
var x = 1,
  y = 2,
  z = 3;
```

---

## Common Mistakes

| Mistake                                    | Why It Is Wrong                                    | Fix                             |
| ------------------------------------------ | -------------------------------------------------- | ------------------------------- |
| Using `var` in modern code                 | Function-scoped, hoisted, redeclarable — confusing | Use `const` or `let`            |
| `const` on values that change              | Will throw TypeError on reassignment               | Use `let` for mutable values    |
| Forgetting `const` objects are mutable     | The object contents can still change               | Use `Object.freeze()` if needed |
| Not initializing `const`                   | `const x;` is a SyntaxError                        | Always assign a value           |
| Accessing `let`/`const` before declaration | TDZ causes ReferenceError                          | Declare before use              |

---

## Summary

- Use `const` by default, `let` when reassignment is needed, never `var`.
- `let` and `const` are block-scoped; `var` is function-scoped.
- Hoisting exists for all three, but `let`/`const` have a Temporal Dead Zone that prevents access before declaration.
- `const` makes the **binding** immutable, not the value — objects and arrays declared with `const` can still be mutated.
- Meaningful variable names and consistent naming conventions make code self-documenting.
