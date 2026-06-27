# Objects in JavaScript

## What Are Objects?

Objects are collections of key-value pairs (properties). They are the fundamental data structure in JavaScript for grouping related data and behavior together.

**Analogy:** An object is like a person's ID card — it holds multiple related pieces of information (name, age, address, photo) under a single entity.

---

## Creating Objects

### Object Literal (Most Common)

```javascript
const user = {
  name: "Vikas",
  age: 25,
  isActive: true,
  address: {
    city: "Delhi",
    country: "India",
  },
  greet() {
    return `Hi, I'm ${this.name}`;
  },
};
```

### `new Object()` (Rarely Used)

```javascript
const obj = new Object();
obj.name = "Vikas";
```

### `Object.create()` (For Prototype Inheritance)

```javascript
const proto = {
  greet() {
    return "Hello";
  },
};
const child = Object.create(proto);
child.greet(); // "Hello" — inherited from proto
```

---

## Accessing Properties

```javascript
const user = { name: "Vikas", "full-name": "Vikas Kumar" };

// Dot notation (preferred)
user.name; // "Vikas"

// Bracket notation (required for special characters or dynamic keys)
user["full-name"]; // "Vikas Kumar"
let key = "name";
user[key]; // "Vikas"
```

---

## Modifying Objects

```javascript
const user = { name: "Vikas", age: 25 };

// Add property
user.email = "vikas@example.com";

// Update property
user.age = 26;

// Delete property
delete user.email;

// Check if property exists
"name" in user; // true
user.hasOwnProperty("age"); // true
```

---

## Object Methods

### `Object.keys()`, `Object.values()`, `Object.entries()`

```javascript
const car = { brand: "Toyota", model: "Camry", year: 2023 };

Object.keys(car); // ["brand", "model", "year"]
Object.values(car); // ["Toyota", "Camry", 2023]
Object.entries(car); // [["brand","Toyota"], ["model","Camry"], ["year",2023]]
```

### `Object.assign()` (Shallow Copy/Merge)

```javascript
const defaults = { theme: "light", lang: "en" };
const userPrefs = { theme: "dark" };

const settings = Object.assign({}, defaults, userPrefs);
// { theme: "dark", lang: "en" }
```

### `Object.freeze()` (Immutable)

```javascript
const config = Object.freeze({ port: 3000, host: "localhost" });
config.port = 8080; // Silently fails (or throws in strict mode)
console.log(config.port); // 3000 — unchanged
```

- Prevents adding, removing, or modifying properties.
- **Shallow freeze only** — nested objects are still mutable.

### `Object.seal()` (No Add/Delete, Can Modify)

```javascript
const user = Object.seal({ name: "Vikas", age: 25 });
user.age = 26; // ✅ Allowed — existing property modified
user.email = "v@x.com"; // ❌ Silently fails — cannot add
delete user.name; // ❌ Silently fails — cannot delete
```

---

## Rest & Spread Operators

### Spread (`...`) — Expand an Object

```javascript
const base = { a: 1, b: 2 };
const extended = { ...base, c: 3, b: 10 };
// { a: 1, b: 10, c: 3 } — later properties override earlier

// Shallow copy
const copy = { ...base };
copy.a = 99;
console.log(base.a); // 1 — original unchanged
```

### Rest (`...`) — Collect Remaining Properties

```javascript
const { name, ...rest } = { name: "Vikas", age: 25, city: "Delhi" };
// name → "Vikas"
// rest → { age: 25, city: "Delhi" }
```

---

## Destructuring

```javascript
const user = { name: "Vikas", age: 25, city: "Delhi" };

// Basic
const { name, age } = user;

// Rename
const { name: userName } = user; // userName = "Vikas"

// Default values
const { country = "India" } = user; // country = "India" (not in user)

// Nested
const response = { data: { users: [{ id: 1 }] } };
const {
  data: { users },
} = response;
```

---

## Shallow vs Deep Copy

### Shallow Copy (One Level Deep)

```javascript
const original = { name: "Vikas", address: { city: "Delhi" } };

// Methods for shallow copy
const copy1 = { ...original };
const copy2 = Object.assign({}, original);

copy1.name = "Rahul"; // ✅ Does not affect original
copy1.address.city = "Mumbai"; // ❌ AFFECTS original (shared reference)
```

### Deep Copy

```javascript
// Method 1: structuredClone (modern — recommended)
const deepCopy = structuredClone(original);
deepCopy.address.city = "Mumbai"; // ✅ Does NOT affect original

// Method 2: JSON trick (has limitations)
const deepCopy2 = JSON.parse(JSON.stringify(original));
// ❌ Loses: functions, undefined, Infinity, NaN, Dates (become strings), circular refs
```

---

## Computed Property Names

```javascript
const field = "email";
const value = "vikas@example.com";

const user = {
  [field]: value, // { email: "vikas@example.com" }
  [`${field}Verified`]: true, // { emailVerified: true }
};
```

---

## Optional Chaining (`?.`)

Safely access deeply nested properties:

```javascript
const user = { name: "Vikas", address: null };

// Without optional chaining
// user.address.city; // TypeError: Cannot read property 'city' of null

// With optional chaining
user.address?.city; // undefined (no error)
user.getProfile?.(); // undefined (method does not exist, no error)
user.friends?.[0]; // undefined (safe array access)
```

---

## Iterating Over Objects

```javascript
const scores = { math: 90, science: 85, english: 88 };

// for...in
for (let subject in scores) {
  console.log(`${subject}: ${scores[subject]}`);
}

// Object.entries + for...of
for (let [subject, score] of Object.entries(scores)) {
  console.log(`${subject}: ${score}`);
}

// Object.entries + forEach
Object.entries(scores).forEach(([subject, score]) => {
  console.log(`${subject}: ${score}`);
});
```

---

## Best Practices

1. **Use `const` for objects** — prevents reassignment while allowing property modification.
2. **Use shorthand syntax** when variable name matches property name: `{ name, age }`.
3. **Use `Object.freeze()` for config** that should never change.
4. **Use optional chaining** for accessing nested data from APIs.
5. **Use `structuredClone()`** for deep copies — it handles most types correctly.
6. **Destructure in function parameters** for cleaner APIs.

```javascript
// Instead of:
function createUser(options) {
  const name = options.name;
  const age = options.age;
}

// Do:
function createUser({ name, age, role = "user" }) {
  // name, age, and role are directly available
}
```

---

## Common Mistakes

| Mistake                              | Why It Is Wrong                              | Fix                                   |
| ------------------------------------ | -------------------------------------------- | ------------------------------------- |
| Mutating objects passed as arguments | Changes the original object (reference type) | Spread copy: `{ ...obj }`             |
| Using `==` to compare objects        | Compares references, not contents            | Use deep comparison or JSON.stringify |
| Forgetting shallow copy limitations  | Nested objects still share references        | Use `structuredClone()`               |
| Deleting in a loop                   | Can skip items or cause unexpected behavior  | Build a new object without those keys |

---

## Summary

- Objects store data as key-value pairs and are created with `{}` literals.
- Access properties with dot notation or brackets (brackets for dynamic keys).
- Spread (`...`) copies/merges objects; rest (`...`) collects remaining properties.
- `Object.freeze()` makes an object immutable; `Object.seal()` prevents add/delete but allows modification.
- Shallow copies (`...`, `Object.assign()`) only work one level deep — use `structuredClone()` for deep copies.
- Optional chaining (`?.`) prevents errors when accessing deeply nested properties.
- Objects are reference types — passing them to functions or assigning to variables creates shared references, not copies.
