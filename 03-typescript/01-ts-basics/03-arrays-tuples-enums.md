# Arrays, Tuples & Enums in TypeScript

## Array Types

### Basic Array Syntax

```typescript
// Two equivalent syntaxes
let numbers: number[] = [1, 2, 3];
let names: Array<string> = ["Vikas", "Rahul", "Priya"];

// Mixed arrays (union type)
let mixed: (string | number)[] = [1, "two", 3, "four"];

// Array of objects
let users: { name: string; age: number }[] = [
  { name: "Vikas", age: 25 },
  { name: "Rahul", age: 30 },
];
```

### Readonly Arrays

Prevents mutations (push, pop, splice, direct assignment):

```typescript
// readonly modifier
const colors: readonly string[] = ["red", "green", "blue"];
// colors.push("yellow");  // Error: Property 'push' does not exist
// colors[0] = "purple";   // Error: Index signature only permits reading

// ReadonlyArray generic
const scores: ReadonlyArray<number> = [90, 85, 92];

// as const — makes it deeply readonly with literal types
const directions = ["north", "south", "east", "west"] as const;
// Type: readonly ["north", "south", "east", "west"]
```

### Array Methods Retain Types

```typescript
const numbers = [1, 2, 3, 4, 5];

const doubled = numbers.map((n) => n * 2); // number[]
const evens = numbers.filter((n) => n % 2 === 0); // number[]
const first = numbers.find((n) => n > 3); // number | undefined
const sum = numbers.reduce((acc, n) => acc + n, 0); // number
```

---

## Tuple Types

Tuples are **fixed-length arrays where each position has a specific type**. Unlike regular arrays, tuples know exactly how many elements they have and what type each element is.

### Basic Tuples

```typescript
// [type1, type2, ...]
let coordinate: [number, number] = [10, 20];
let entry: [string, number, boolean] = ["Vikas", 25, true];

// Accessing — TypeScript knows the type at each index
coordinate[0]; // number
coordinate[1]; // number
entry[0].toUpperCase(); // OK — knows it's string
entry[1].toFixed(2); // OK — knows it's number
```

### Labeled Tuples (Documentation)

```typescript
type UserTuple = [name: string, age: number, active: boolean];
let user: UserTuple = ["Vikas", 25, true];

// Labels are purely documentation — they don't enforce named access
// You still access by index: user[0], user[1], user[2]

type HTTPResponse = [
  status: number,
  body: string,
  headers: Record<string, string>,
];
```

### Optional Tuple Elements

```typescript
type Point2D = [number, number];
type Point3D = [number, number, number?]; // Third element optional

const p1: Point3D = [1, 2]; // OK
const p2: Point3D = [1, 2, 3]; // OK

type NameAge = [string, number?];
const entry1: NameAge = ["Vikas"]; // OK
const entry2: NameAge = ["Vikas", 25]; // OK
```

### Rest Elements in Tuples

```typescript
// Rest at the end — variable length after fixed positions
type StringNumberRest = [string, number, ...boolean[]];
const a: StringNumberRest = ["hello", 1, true, false, true]; // OK

// Rest at the beginning
type LastIsString = [...number[], string];
const b: LastIsString = [1, 2, 3, "end"]; // OK

// Fixed start, rest middle, fixed end
type Padded = [number, ...string[], number];
const c: Padded = [1, "a", "b", "c", 99]; // OK
```

### Tuple Destructuring

```typescript
type APIResult = [data: unknown, error: string | null];

function fetchUser(): APIResult {
  return [{ name: "Vikas" }, null];
}

const [data, error] = fetchUser(); // Typed correctly
// data: unknown
// error: string | null
```

---

## Numeric Enums

Enums define a set of **named constants**. Numeric enums auto-increment from 0 (or a specified start value).

### Auto-Increment

```typescript
enum Direction {
  Up, // 0
  Down, // 1
  Left, // 2
  Right, // 3
}

let move: Direction = Direction.Up;
console.log(Direction.Right); // 3
console.log(Direction[0]); // "Up" — reverse mapping!
```

### Custom Values

```typescript
enum StatusCode {
  OK = 200,
  Created = 201,
  BadRequest = 400,
  NotFound = 404,
  ServerError = 500,
}

function handleResponse(code: StatusCode) {
  if (code === StatusCode.OK) {
    console.log("Success!");
  }
}
```

### Reverse Mapping

Numeric enums generate a **reverse mapping** — you can get the name from the value:

```typescript
enum Color {
  Red, // 0
  Green, // 1
  Blue, // 2
}

console.log(Color.Red); // 0
console.log(Color[0]); // "Red" — reverse mapping
console.log(Color[1]); // "Green"
```

This works because numeric enums compile to an object with both forward and reverse entries.

---

## String Enums

String enums have **no auto-increment** — every member must be explicitly assigned:

```typescript
enum Theme {
  Light = "LIGHT",
  Dark = "DARK",
  System = "SYSTEM",
}

let currentTheme: Theme = Theme.Dark;
// currentTheme = "DARK"; // Error — must use enum value

enum LogLevel {
  Debug = "DEBUG",
  Info = "INFO",
  Warn = "WARN",
  Error = "ERROR",
}
```

**Note:** String enums do NOT have reverse mapping (unlike numeric enums).

---

## Const Enums (Inlined at Compile Time)

`const enum` members are **replaced with their literal values** at compile time — no runtime object is generated:

```typescript
const enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}

let d = Direction.Up;
// Compiles to: let d = "UP";
// No Direction object exists at runtime — more efficient
```

**Trade-off:** Const enums cannot be iterated or used with reverse mapping. They're purely a compile-time optimization.

---

## Enum vs Union Type

| Feature          | Enum                        | Union Type               |
| ---------------- | --------------------------- | ------------------------ |
| Runtime object   | Yes (except `const enum`)   | No — purely compile-time |
| Tree-shaking     | No (runtime code generated) | Yes — zero runtime cost  |
| Reverse mapping  | Yes (numeric only)          | No                       |
| Iteration        | Yes (Object.values)         | No                       |
| DevTools display | Shows enum name             | Shows literal value      |
| Type safety      | Nominal (distinct type)     | Structural (just values) |
| Intellisense     | Shows member names          | Shows literal values     |

### When to Use Enum

```typescript
// Use enum when you need runtime access or iteration
enum Permission {
  Read = "READ",
  Write = "WRITE",
  Admin = "ADMIN",
}

// Can iterate over values at runtime
Object.values(Permission); // ["READ", "WRITE", "ADMIN"]
```

### When to Use Union Type

```typescript
// Use union when you just need type safety — lighter
type Direction = "north" | "south" | "east" | "west";
type Status = "idle" | "loading" | "success" | "error";

// Zero runtime overhead — just types
function move(dir: Direction) {
  console.log(`Moving ${dir}`);
}
```

**Rule of thumb:** Prefer union types for most cases. Use enums only when you need runtime iteration, reverse mapping, or a distinct nominal type.

---

## Best Practices

1. **Prefer `number[]` syntax** over `Array<number>` — shorter and more idiomatic.
2. **Use `readonly` arrays** for data that should not be mutated (configs, constants).
3. **Use tuples for fixed-structure returns** — like `[value, setter]` from hooks or `[error, data]` patterns.
4. **Label tuple elements** — `[name: string, age: number]` is self-documenting.
5. **Prefer union types over enums** — zero runtime cost, better tree-shaking.
6. **Use `const enum` if you must use enums** — avoids generating a runtime object.
7. **Always use string enums** if you use enums — they're more debuggable than numbers and don't have reverse mapping confusion.
8. **Use `as const`** on arrays when you need tuple behavior from a plain array literal.

---

## Common Mistakes

| Mistake                                    | Why It's Wrong                                     | Fix                                           |
| ------------------------------------------ | -------------------------------------------------- | --------------------------------------------- |
| Mutating readonly arrays                   | TypeScript errors, runtime bugs if bypassed        | Use spread/map to create new arrays           |
| Assuming tuples prevent extra elements     | `push()` still works on tuples (TypeScript hole)   | Use `readonly` tuples to prevent mutations    |
| Using numeric enums for string-like values | Harder to debug (seeing `0` instead of `"active"`) | Use string enums or union types               |
| Iterating over const enums                 | They don't exist at runtime                        | Use regular enums if runtime access is needed |
| Forgetting `as const` for literal tuples   | Array gets widened to `(string \| number)[]`       | Add `as const` for precise tuple inference    |
| Enum values not matching expectations      | Auto-increment may not give expected numbers       | Assign explicit values to avoid surprises     |
| Using enums when union types suffice       | Unnecessary runtime code and complexity            | Default to union types for type-only needs    |

---

## Summary

- **Arrays**: `number[]` or `Array<T>`. Use `readonly` for immutable arrays. `as const` creates readonly tuples with literal types.
- **Tuples**: Fixed-length arrays with specific types at each index. Support labels, optional elements, and rest elements.
- **Numeric enums**: Auto-increment from 0, support reverse mapping (`Color[0]` → `"Red"`).
- **String enums**: Every member explicitly assigned, no reverse mapping, more debuggable.
- **Const enums**: Inlined at compile time — no runtime object, better performance, but no iteration.
- **Enum vs Union**: Prefer union types (`"a" | "b" | "c"`) for zero-cost type safety. Use enums only when you need runtime access or iteration.
