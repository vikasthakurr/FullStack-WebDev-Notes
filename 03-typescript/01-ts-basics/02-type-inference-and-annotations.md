# TypeScript Type Inference & Annotations

## What Is Type Inference?

Type inference is TypeScript's ability to **automatically determine types without explicit annotations**. When you initialize a variable, return a value from a function, or pass arguments, TypeScript analyzes the code and assigns types behind the scenes.

**Analogy:** Type inference is like a detective — it looks at the evidence (your code) and deduces the type. You only need to explicitly state the type when the evidence is ambiguous.

---

## When TypeScript Infers Automatically

### Variable Initialization

```typescript
let name = "Vikas"; // Inferred: string
let age = 25; // Inferred: number
let isActive = true; // Inferred: boolean
let items = [1, 2, 3]; // Inferred: number[]
let mixed = [1, "two"]; // Inferred: (string | number)[]

const PI = 3.14; // Inferred: 3.14 (literal type!)
const greeting = "hello"; // Inferred: "hello" (literal type!)
```

### Return Types

```typescript
// TypeScript infers return type as number
function add(a: number, b: number) {
  return a + b;
}

// Inferred: string
function greet(name: string) {
  return `Hello, ${name}`;
}

// Inferred: { name: string; age: number }
function createUser(name: string, age: number) {
  return { name, age };
}
```

### Contextual Typing

TypeScript infers types based on the context where a value is used:

```typescript
// Event handler — TypeScript knows `e` is MouseEvent
document.addEventListener("click", (e) => {
  console.log(e.clientX); // e inferred as MouseEvent
});

// Array methods — callback parameter types are inferred
const numbers = [1, 2, 3];
numbers.map((n) => n * 2); // n inferred as number

// Object methods
const user = {
  name: "Vikas",
  greet() {
    return this.name.toUpperCase(); // this.name inferred as string
  },
};
```

---

## When to Add Explicit Annotations

### Function Parameters (Always)

TypeScript cannot infer parameter types — they must be annotated:

```typescript
// ❌ Error: Parameter 'name' implicitly has 'any' type
function greet(name) {
  return `Hello, ${name}`;
}

// ✅ Explicit parameter annotation
function greet(name: string): string {
  return `Hello, ${name}`;
}
```

### Complex Object Types

```typescript
// Without annotation — works but unclear what shape is expected
const config = {
  host: "localhost",
  port: 3000,
  debug: true,
};

// With annotation — documents the contract, catches missing fields
interface AppConfig {
  host: string;
  port: number;
  debug: boolean;
  apiKey?: string;
}

const config: AppConfig = {
  host: "localhost",
  port: 3000,
  debug: true,
};
```

### When Inference Is Wrong

```typescript
// Inferred as (string | number)[] — but you want separate handling
let ids = []; // Inferred as any[] — BAD
ids.push(1);
ids.push("abc");

// ✅ Annotate to enforce the correct type
let ids: number[] = [];

// Another case — delayed initialization
let result; // Inferred as any
result = fetchData(); // Still any

let result: User | null = null; // ✅ Explicit
result = await fetchUser();
```

### Return Type Annotations for Public APIs

```typescript
// For internal functions — let inference handle it
function double(n: number) {
  return n * 2; // Return type inferred as number — fine
}

// For exported/public functions — annotate return type
export function parseConfig(raw: string): AppConfig {
  return JSON.parse(raw); // Without annotation, return type would be 'any'
}
```

---

## Type Widening vs Narrowing

### Type Widening

`let` variables get **widened** to general types. `const` variables keep **literal types**.

```typescript
let color = "red"; // Type: string (widened)
const color2 = "red"; // Type: "red" (literal — not widened)

let count = 0; // Type: number (widened)
const count2 = 0; // Type: 0 (literal)

let flag = true; // Type: boolean (widened)
const flag2 = true; // Type: true (literal)
```

**Why?** `let` variables can be reassigned, so TypeScript widens to allow any value of that type. `const` can never change, so the literal type is safe.

### Preventing Widening

```typescript
// as const prevents widening
let direction = "north" as const; // Type: "north" (not string)

const config = {
  mode: "production",
  port: 3000,
} as const;
// Type: { readonly mode: "production"; readonly port: 3000 }
// Without as const: { mode: string; port: number }
```

### Type Narrowing

Narrowing makes a type more specific within a code block:

```typescript
function process(value: string | number) {
  // value is string | number here

  if (typeof value === "string") {
    // value is narrowed to string
    console.log(value.toUpperCase());
  } else {
    // value is narrowed to number
    console.log(value.toFixed(2));
  }
}
```

---

## `const` vs `let` Inference Differences

```typescript
// --- Primitives ---
const name = "Vikas"; // Type: "Vikas" (literal)
let name2 = "Vikas"; // Type: string (widened)

// --- Objects ---
const user = { name: "Vikas", age: 25 };
// Type: { name: string; age: number } — properties are still widened!
// Because object PROPERTIES can be mutated even with const

const user2 = { name: "Vikas", age: 25 } as const;
// Type: { readonly name: "Vikas"; readonly age: 25 } — fully literal

// --- Arrays ---
const nums = [1, 2, 3]; // Type: number[] (mutable array)
const nums2 = [1, 2, 3] as const; // Type: readonly [1, 2, 3] (tuple)
```

---

## Literal Types

Literal types are exact values used as types:

```typescript
// String literals
type Direction = "north" | "south" | "east" | "west";
let dir: Direction = "north"; // OK
// dir = "up"; // Error: Type '"up"' is not assignable

// Numeric literals
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;
let roll: DiceRoll = 4; // OK
// roll = 7; // Error

// Boolean literal
type True = true;

// Literal types in function parameters
function setAlignment(align: "left" | "center" | "right") {
  // align can only be one of these three strings
}
setAlignment("center"); // OK
// setAlignment("justify"); // Error
```

### Literal Types and Const

```typescript
// Problem: widening breaks literal type expectations
let method = "GET";
// fetch(url, { method }); // Error: string not assignable to "GET" | "POST"...

// Solution 1: const
const method = "GET"; // Type: "GET"

// Solution 2: as const
let method = "GET" as const; // Type: "GET"

// Solution 3: explicit annotation
let method: "GET" | "POST" = "GET";
```

---

## Best Practices

1. **Let inference work for local variables** — do not annotate when the type is obvious from initialization.
2. **Always annotate function parameters** — TypeScript cannot infer these.
3. **Annotate return types on public/exported functions** — protects against accidental `any` leaks and documents intent.
4. **Use `as const` for configuration objects** — prevents widening and ensures literal types.
5. **Annotate when delayed initialization** — variables declared without initializer become `any`.
6. **Prefer `const` over `let`** — gives you literal types and prevents accidental reassignment.
7. **Don't fight inference** — if TypeScript infers correctly, adding annotations is just noise.

---

## Common Mistakes

| Mistake                                               | Why It's Wrong                                    | Fix                                              |
| ----------------------------------------------------- | ------------------------------------------------- | ------------------------------------------------ |
| Annotating everything                                 | Adds noise, makes refactoring harder              | Only annotate when inference is insufficient     |
| Not annotating function parameters                    | Parameters become `any` (with strict mode errors) | Always annotate params                           |
| Using `let` when `const` is appropriate               | Widens type unnecessarily                         | Default to `const`, use `let` only when needed   |
| Ignoring widening with object literals                | Expected literal type but got `string`            | Use `as const` or explicit type annotation       |
| Declaring variables without initializer               | Type defaults to `any`                            | Initialize immediately or annotate type          |
| Not returning types from functions using `JSON.parse` | Returns `any` — defeats type safety               | Annotate return type or validate with type guard |
| Over-relying on `as` type assertions                  | Bypasses type checking instead of fixing          | Use type guards or proper annotations            |

---

## Summary

- TypeScript infers types from variable initialization, return values, and contextual usage — let it work.
- Always annotate function parameters — TypeScript cannot infer them.
- Add explicit annotations for: public APIs, complex objects, delayed initialization, and when inference is wrong.
- `const` preserves literal types (`"hello"`, `42`). `let` widens to general types (`string`, `number`).
- `as const` prevents widening on objects and arrays — creates deeply readonly literal types.
- Type narrowing makes types more specific inside conditional blocks (typeof, instanceof, etc.).
- Rule of thumb: annotate at boundaries (function signatures, exports), let inference handle the rest.
