# Union & Intersection Types, Type Aliases vs Interfaces

## Union Types (`|`)

A union type means a value can be **one of several types**. Use the pipe (`|`) operator.

```typescript
let id: string | number;
id = "abc"; // OK
id = 123; // OK
// id = true; // Error: boolean not in the union

type Status = "idle" | "loading" | "success" | "error";
type InputValue = string | number | null;
```

### Narrowing Unions

You must narrow union types before using type-specific methods:

```typescript
function formatValue(value: string | number): string {
  // ❌ value.toUpperCase() — Error: number doesn't have toUpperCase

  if (typeof value === "string") {
    return value.toUpperCase(); // Narrowed to string
  }
  return value.toFixed(2); // Narrowed to number
}

function processInput(input: string | string[]) {
  if (Array.isArray(input)) {
    return input.join(", "); // Narrowed to string[]
  }
  return input.trim(); // Narrowed to string
}
```

---

## Discriminated Unions

Discriminated unions use a **common literal property** (the "discriminant") to distinguish between variants:

```typescript
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "rectangle"; width: number; height: number }
  | { kind: "triangle"; base: number; height: number };
```

function getArea(shape: Shape): number {
switch (shape.kind) {
case "circle":
return Math.PI _ shape.radius \*\* 2;
case "rectangle":
return shape.width _ shape.height;
case "triangle":
return 0.5 _ shape.base _ shape.height;
}
}

// Real-world example — API response
type ApiResponse<T> =
| { status: "success"; data: T }
| { status: "error"; message: string }
| { status: "loading" };

function handleResponse(response: ApiResponse<User>) {
switch (response.status) {
case "success":
console.log(response.data.name); // data is available
break;
case "error":
console.error(response.message); // message is available
break;
case "loading":
console.log("Loading...");
break;
}
}

````

---

## Intersection Types (`&`)

An intersection type combines **all properties** from multiple types. The resulting type must satisfy ALL combined types.

```typescript
type HasName = { name: string };
type HasAge = { age: number };
type HasEmail = { email: string };

type Person = HasName & HasAge;
// Must have BOTH name AND age

const user: Person = {
  name: "Vikas",
  age: 25,
}; // OK — has both

type Employee = HasName & HasAge & HasEmail & {
  department: string;
};

const emp: Employee = {
  name: "Vikas",
  age: 25,
  email: "vikas@company.com",
  department: "Engineering",
};
````

### Intersection vs Union

```typescript
// Union: value is ONE of the types
type StringOrNumber = string | number;
// Can be string OR number, not both

// Intersection: value has ALL properties from all types
type Named = { name: string };
type Aged = { age: number };
type NamedAndAged = Named & Aged;
// Must have name AND age
```

---

## Type Aliases (`type` keyword)

Type aliases create a **name for any type expression** — primitives, unions, objects, functions, tuples:

```typescript
// Object shape
type User = {
  name: string;
  age: number;
  email?: string;
};

// Union
type ID = string | number;
type Status = "active" | "inactive" | "banned";

// Function
type Formatter = (value: string) => string;

// Tuple
type Coordinate = [number, number];

// Generic
type ApiResponse<T> = {
  data: T;
  timestamp: number;
  success: boolean;
};

// Computed/conditional types
type Nullable<T> = T | null;
type ReadonlyUser = Readonly<User>;
```

---

## Interfaces (`interface` keyword)

Interfaces define **object shapes and class contracts**:

```typescript
interface User {
  name: string;
  age: number;
  email?: string; // Optional
}

interface Config {
  readonly apiKey: string; // Cannot be changed
  host: string;
  port: number;
}
```

### Extending Interfaces

```typescript
interface Animal {
  name: string;
  sound(): string;
}

interface Dog extends Animal {
  breed: string;
  fetch(): void;
}

// Multiple inheritance
interface Pet extends Animal {
  owner: string;
}

interface ServiceDog extends Dog, Pet {
  task: string;
}

const buddy: ServiceDog = {
  name: "Buddy",
  breed: "Labrador",
  owner: "Vikas",
  task: "Guide dog",
  sound() {
    return "Woof!";
  },
  fetch() {
    console.log("Fetching!");
  },
};
```

### Declaration Merging

Interfaces with the same name are **automatically merged** — type aliases cannot do this:

```typescript
interface Window {
  myCustomProperty: string;
}

interface Window {
  anotherProperty: number;
}

// Both declarations merge into one interface:
// interface Window {
//   myCustomProperty: string;
//   anotherProperty: number;
// }
```

This is useful for extending third-party types (like `Window`, `Express.Request`, etc.).

---

## When to Use `type` vs `interface`

| Feature                       | `type`                      | `interface`              |
| ----------------------------- | --------------------------- | ------------------------ |
| Object shapes                 | ✅                          | ✅                       |
| Union types                   | ✅                          | ❌                       |
| Intersection types            | ✅                          | Use `extends`            |
| Primitives & tuples           | ✅                          | ❌                       |
| Declaration merging           | ❌                          | ✅                       |
| `extends` keyword             | ❌ (use `&` instead)        | ✅                       |
| `implements` in classes       | ✅                          | ✅                       |
| Computed/mapped types         | ✅                          | ❌                       |
| Performance (large codebases) | Slightly slower for complex | Slightly faster (cached) |

**Guidelines:**

- Use `interface` for object shapes, especially when they'll be extended or implemented by classes.
- Use `type` for unions, intersections, tuples, function types, and complex type expressions.
- Be consistent within a project — pick one default and use the other only when needed.

---

## Optional Properties

```typescript
interface UserProfile {
  name: string; // Required
  email: string; // Required
  bio?: string; // Optional — may be undefined
  avatar?: string; // Optional
  age?: number; // Optional
}

const user: UserProfile = {
  name: "Vikas",
  email: "vikas@example.com",
  // bio, avatar, age can be omitted
};

// Accessing optional properties
console.log(user.bio?.toUpperCase()); // Optional chaining — safe
```

---

## Readonly Modifier

```typescript
interface Config {
  readonly apiKey: string;
  readonly baseUrl: string;
  timeout: number; // Mutable
}

const config: Config = {
  apiKey: "secret",
  baseUrl: "https://api.example.com",
  timeout: 5000,
};

// config.apiKey = "new"; // Error: Cannot assign to 'apiKey' — readonly
config.timeout = 10000; // OK — not readonly

// Readonly utility type — makes ALL properties readonly
type FrozenConfig = Readonly<Config>;
```

---

## Index Signatures

Index signatures allow objects with **dynamic keys** that all share the same value type:

```typescript
interface StringMap {
  [key: string]: string;
}

const colors: StringMap = {
  red: "#ff0000",
  green: "#00ff00",
  blue: "#0000ff",
  // Can add any string key with string value
};

// Combining fixed and dynamic properties
interface UserCache {
  total: number; // Fixed property
  [userId: string]: number | User; // Dynamic keys (must include 'total's type)
}

// Record utility — cleaner syntax for index signatures
type Scores = Record<string, number>;
const gameScores: Scores = {
  player1: 100,
  player2: 85,
};
```

---

## Best Practices

1. **Default to `interface` for objects** — they give better error messages and support `extends`.
2. **Use `type` for everything else** — unions, tuples, function types, mapped types.
3. **Prefer discriminated unions** over broad unions — they enable exhaustive checking with `switch`.
4. **Use `Readonly<T>` for immutable data** — prevents accidental mutations.
5. **Use optional properties sparingly** — too many optionals make objects hard to work with.
6. **Prefer `Record<K, V>` over index signatures** — more readable for simple key-value mappings.
7. **Use intersection for composition** — combine small types into larger ones rather than duplicating.
8. **Document interfaces with JSDoc** — especially for public APIs and shared types.

---

## Common Mistakes

| Mistake                                        | Why It's Wrong                                | Fix                                         |
| ---------------------------------------------- | --------------------------------------------- | ------------------------------------------- |
| Using `interface` for union types              | Interfaces can't represent unions             | Use `type` for unions                       |
| Not narrowing before accessing union members   | TypeScript can't know which type it is        | Use `typeof`, `in`, or discriminant checks  |
| Forgetting optional chaining on optional props | Potential `undefined` access error            | Use `?.` or null checks                     |
| Overusing `any` in index signatures            | Defeats type safety                           | Use specific value types or union types     |
| Extending interfaces with conflicting types    | Creates `never` types or errors               | Ensure extended properties are compatible   |
| Declaration merging unintentionally            | Same interface name in different files merges | Use unique names or `type` instead          |
| Using intersection when union is needed        | `string & number` is `never`                  | Think: "AND" (intersection) vs "OR" (union) |

---

## Summary

- **Union types** (`|`): A value can be one of several types. Narrow with `typeof`, `in`, or discriminant properties.
- **Discriminated unions**: Use a common literal property (`kind`, `type`, `status`) to distinguish variants — enables exhaustive `switch`.
- **Intersection types** (`&`): Combine multiple types — the result must have ALL properties from all types.
- **Type aliases**: Name any type expression — objects, unions, tuples, functions, generics, computed types.
- **Interfaces**: Define object shapes with support for `extends`, declaration merging, and class `implements`.
- **Optional (`?`)**: Properties that may be `undefined`. **Readonly**: Properties that cannot be reassigned.
- **Index signatures**: Dynamic keys with shared value types. Prefer `Record<K, V>` for simple cases.
- **Choose `interface` for objects** that will be extended. **Choose `type` for everything else.**
