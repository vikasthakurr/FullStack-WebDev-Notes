# Introduction to TypeScript

## What Is TypeScript?

TypeScript is a **typed superset of JavaScript** developed by Microsoft. It adds optional static types, interfaces, and compile-time checks to JavaScript. Every valid JavaScript file is valid TypeScript, but TypeScript adds a layer of safety on top.

**Analogy:** JavaScript is like driving without a GPS — you can get anywhere but might take wrong turns and only discover them when you arrive at the wrong place. TypeScript is the GPS — it warns you about wrong turns before you make them.

---

## Why TypeScript?

| Problem in JavaScript              | TypeScript Solution              |
| ---------------------------------- | -------------------------------- |
| Typos in property names            | Caught at compile time           |
| Wrong argument types passed        | Type errors shown in editor      |
| Unclear function contracts         | Explicit parameter/return types  |
| Refactoring breaks things silently | Compiler finds all affected code |
| No autocomplete for object shapes  | Full IntelliSense based on types |

---

## Installing TypeScript

```bash
# Install globally
npm install -g typescript

# Or as a dev dependency (recommended)
npm install --save-dev typescript

# Check version
tsc --version
```

### Compiling TypeScript

```bash
# Compile a single file
tsc app.ts
# Produces app.js

# Watch mode (recompiles on save)
tsc --watch

# Initialize tsconfig.json
tsc --init
```

---

## `tsconfig.json` Overview

```json
{
  "compilerOptions": {
    "target": "ES2020", // Output JS version
    "module": "ESNext", // Module system
    "strict": true, // Enable all strict checks
    "outDir": "./dist", // Output directory
    "rootDir": "./src", // Source directory
    "esModuleInterop": true, // Compatibility with CommonJS
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,
    "declaration": true // Generate .d.ts files
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

Key options:

- `strict: true` — enables `strictNullChecks`, `noImplicitAny`, and other checks. Always use this.
- `target` — what JavaScript version to output (ES5 for old browsers, ES2020+ for modern).
- `module` — module format (`ESNext` for modern, `CommonJS` for Node.js).

---

## Type Annotations

### Basic Types

```typescript
let username: string = "Vikas";
let age: number = 25;
let isActive: boolean = true;
let nothing: null = null;
let notDefined: undefined = undefined;
```

### Special Types

```typescript
// any — disables type checking (avoid)
let flexible: any = "hello";
flexible = 42; // No error — defeats the purpose of TypeScript

// unknown — safer alternative to any
let input: unknown = "hello";
// input.toUpperCase(); // Error — must narrow first
if (typeof input === "string") {
  input.toUpperCase(); // OK — narrowed to string
}

// void — function returns nothing
function logMessage(msg: string): void {
  console.log(msg);
}

// never — function never returns (throws or infinite loop)
function throwError(msg: string): never {
  throw new Error(msg);
}
```

---

## Type Inference

TypeScript often infers types without explicit annotations:

```typescript
let name = "Vikas"; // Inferred as string
let count = 42; // Inferred as number
let items = [1, 2, 3]; // Inferred as number[]

// No need to annotate when inference is clear
const user = {
  name: "Vikas", // string
  age: 25, // number
  isActive: true, // boolean
};
```

**Rule:** Let TypeScript infer when the type is obvious. Annotate when it is not clear or for function parameters.

---

## Arrays & Tuples

### Arrays

```typescript
let numbers: number[] = [1, 2, 3];
let names: string[] = ["Vikas", "Rahul"];
let mixed: (string | number)[] = [1, "two", 3];

// Alternative syntax
let scores: Array<number> = [90, 85, 92];
```

### Tuples

Fixed-length arrays with specific types at each position:

```typescript
let coordinate: [number, number] = [10, 20];
let entry: [string, number] = ["Vikas", 25];

// Accessing
coordinate[0]; // number
entry[1]; // number

// Labeled tuples (documentation)
type UserTuple = [name: string, age: number, active: boolean];
let user: UserTuple = ["Vikas", 25, true];
```

---

## Enums

### Numeric Enums

```typescript
enum Direction {
  Up, // 0
  Down, // 1
  Left, // 2
  Right, // 3
}

let move: Direction = Direction.Up;
console.log(Direction.Right); // 3
```

### String Enums

```typescript
enum Status {
  Active = "ACTIVE",
  Inactive = "INACTIVE",
  Pending = "PENDING",
}

let userStatus: Status = Status.Active;
// userStatus = "ACTIVE"; // Error — must use enum value
```

### `const enum` (Inlined at Compile Time)

```typescript
const enum Color {
  Red = "RED",
  Green = "GREEN",
  Blue = "BLUE",
}
// More efficient — no runtime object generated
```

---

## Union & Intersection Types

### Union (`|`) — Either Type

```typescript
let id: string | number;
id = "abc"; // OK
id = 123; // OK
// id = true; // Error

function format(value: string | number): string {
  if (typeof value === "string") {
    return value.toUpperCase();
  }
  return value.toFixed(2);
}
```

### Intersection (`&`) — Both Types Combined

```typescript
type HasName = { name: string };
type HasAge = { age: number };

type Person = HasName & HasAge;

const user: Person = {
  name: "Vikas",
  age: 25,
}; // Must have BOTH name AND age
```

---

## Type Aliases vs Interfaces

### Type Alias

```typescript
type User = {
  name: string;
  age: number;
  email?: string; // Optional
};

type ID = string | number; // Can alias primitives and unions
```

### Interface

```typescript
interface User {
  name: string;
  age: number;
  email?: string;
}

// Interfaces can be extended
interface Admin extends User {
  role: "admin";
  permissions: string[];
}
```

### When to Use Which?

| Feature                  | Type Alias | Interface |
| ------------------------ | ---------- | --------- |
| Object shapes            | ✅         | ✅        |
| Union types              | ✅         | ❌        |
| Intersection             | ✅         | Extend    |
| Declaration merging      | ❌         | ✅        |
| `extends` / `implements` | ❌         | ✅        |
| Primitives & tuples      | ✅         | ❌        |

**Rule of thumb:** Use `interface` for object shapes and class contracts. Use `type` for unions, intersections, and complex type expressions.

---

## Optional & Readonly

```typescript
interface Config {
  host: string;
  port?: number; // Optional — may be undefined
  readonly apiKey: string; // Cannot be changed after creation
}

const config: Config = {
  host: "localhost",
  apiKey: "secret123",
};

// config.apiKey = "new"; // Error — readonly
config.port = 3000; // OK — optional but assignable
```

### `const` Assertion

```typescript
const colors = ["red", "green", "blue"] as const;
// Type: readonly ["red", "green", "blue"]
// colors.push("yellow"); // Error — readonly

const config = {
  port: 3000,
  host: "localhost",
} as const;
// All properties are readonly, values are literal types
```

---

## Best Practices

1. **Enable `strict: true`** in tsconfig — it catches the most bugs.
2. **Avoid `any`** — use `unknown` and narrow with type guards.
3. **Let inference work** — do not annotate when TypeScript already knows the type.
4. **Use interfaces for objects** that will be extended or implemented by classes.
5. **Use type aliases** for unions, computed types, and function signatures.
6. **Use `readonly`** for data that should not be mutated after creation.

---

## Summary

- TypeScript adds static types to JavaScript — catching bugs at compile time instead of runtime.
- Install with `npm install typescript`; compile with `tsc`.
- `tsconfig.json` configures the compiler — always enable `strict: true`.
- Basic types: `string`, `number`, `boolean`, `null`, `undefined`, `void`, `never`, `any`, `unknown`.
- Arrays (`number[]`), tuples (`[string, number]`), and enums provide structured data types.
- Union (`|`) and intersection (`&`) combine types; interfaces and type aliases define shapes.
- TypeScript inference is powerful — annotate only when needed for clarity or function boundaries.
