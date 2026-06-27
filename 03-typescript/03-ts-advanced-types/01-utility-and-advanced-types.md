# TypeScript Advanced Types

## Utility Types

TypeScript provides built-in utility types that transform existing types.

### `Partial<T>` — All Properties Optional

```typescript
interface User {
  name: string;
  age: number;
  email: string;
}

type UpdateUser = Partial<User>;
// { name?: string; age?: number; email?: string }

function updateUser(id: string, updates: Partial<User>): void {
  // Can pass any subset of User properties
}

updateUser("123", { name: "New Name" }); // OK — only updating name
```

### `Required<T>` — All Properties Required

```typescript
interface Config {
  host?: string;
  port?: number;
}

type FullConfig = Required<Config>;
// { host: string; port: number } — no optionals
```

### `Pick<T, K>` — Select Specific Properties

```typescript
type UserPreview = Pick<User, "name" | "email">;
// { name: string; email: string }
```

### `Omit<T, K>` — Remove Specific Properties

```typescript
type UserWithoutEmail = Omit<User, "email">;
// { name: string; age: number }
```

### `Record<K, V>` — Object with Specified Keys and Value Type

```typescript
type Roles = "admin" | "user" | "moderator";

const permissions: Record<Roles, string[]> = {
  admin: ["read", "write", "delete"],
  user: ["read"],
  moderator: ["read", "write"],
};

// Dynamic object type
type StringMap = Record<string, string>;
```

### `Readonly<T>` — All Properties Readonly

```typescript
type ImmutableUser = Readonly<User>;
// All properties become readonly — cannot be reassigned
```

### `Exclude<T, U>` and `Extract<T, U>`

```typescript
type AllTypes = string | number | boolean;

type OnlyStrNum = Exclude<AllTypes, boolean>; // string | number
type OnlyBool = Extract<AllTypes, boolean>; // boolean
```

### `NonNullable<T>`

```typescript
type MaybeString = string | null | undefined;
type DefinitelyString = NonNullable<MaybeString>; // string
```

### `ReturnType<T>` and `Parameters<T>`

```typescript
function createUser(name: string, age: number) {
  return { name, age, id: Math.random() };
}

type UserReturn = ReturnType<typeof createUser>;
// { name: string; age: number; id: number }

type UserParams = Parameters<typeof createUser>;
// [string, number]
```

---

## Mapped Types

Create new types by transforming properties of existing types:

```typescript
// Make all properties optional (this is how Partial works internally)
type MyPartial<T> = {
  [K in keyof T]?: T[K];
};

// Make all properties readonly
type MyReadonly<T> = {
  readonly [K in keyof T]: T[K];
};

// Make all properties nullable
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};

interface User {
  name: string;
  age: number;
}

type NullableUser = Nullable<User>;
// { name: string | null; age: number | null }
```

---

## Conditional Types

Types that depend on a condition:

```typescript
type IsString<T> = T extends string ? "yes" : "no";

type A = IsString<string>; // "yes"
type B = IsString<number>; // "no"

// Practical example: unwrap array type
type Unwrap<T> = T extends Array<infer U> ? U : T;

type X = Unwrap<string[]>; // string
type Y = Unwrap<number>; // number
```

### `infer` Keyword

Extract a type from within a pattern:

```typescript
type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type Fn = (x: number) => string;
type Result = GetReturnType<Fn>; // string
```

---

## Template Literal Types

Build types from string patterns:

```typescript
type Color = "red" | "blue" | "green";
type Size = "small" | "medium" | "large";

type ColorSize = `${Color}-${Size}`;
// "red-small" | "red-medium" | "red-large" | "blue-small" | ... (9 combinations)

// Event names
type EventName = `on${Capitalize<"click" | "focus" | "blur">}`;
// "onClick" | "onFocus" | "onBlur"
```

---

## Discriminated Unions

Union types with a common literal property that TypeScript uses for narrowing:

```typescript
interface Circle {
  kind: "circle"; // Discriminant
  radius: number;
}

interface Rectangle {
  kind: "rectangle"; // Discriminant
  width: number;
  height: number;
}

interface Triangle {
  kind: "triangle"; // Discriminant
  base: number;
  height: number;
}

type Shape = Circle | Rectangle | Triangle;

function area(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2; // TypeScript knows it's Circle
    case "rectangle":
      return shape.width * shape.height; // TypeScript knows it's Rectangle
    case "triangle":
      return 0.5 * shape.base * shape.height;
    default:
      const _exhaustive: never = shape; // Compile error if a case is missed
      return _exhaustive;
  }
}
```

---

## Type Guards & Narrowing

### `typeof` Guard

```typescript
function format(value: string | number): string {
  if (typeof value === "string") {
    return value.toUpperCase(); // TypeScript knows: string
  }
  return value.toFixed(2); // TypeScript knows: number
}
```

### `instanceof` Guard

```typescript
class ApiError extends Error {
  constructor(
    public statusCode: number,
    message: string,
  ) {
    super(message);
  }
}

function handleError(error: Error | ApiError) {
  if (error instanceof ApiError) {
    console.log(error.statusCode); // TypeScript knows: ApiError
  } else {
    console.log(error.message); // TypeScript knows: Error
  }
}
```

### `in` Operator Guard

```typescript
interface Fish {
  swim(): void;
}
interface Bird {
  fly(): void;
}

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    animal.swim(); // TypeScript knows: Fish
  } else {
    animal.fly(); // TypeScript knows: Bird
  }
}
```

### Custom Type Guard (Type Predicate)

```typescript
interface User {
  name: string;
  age: number;
}
interface Admin extends User {
  role: "admin";
  permissions: string[];
}

function isAdmin(user: User | Admin): user is Admin {
  return "role" in user && (user as Admin).role === "admin";
}

function greet(user: User | Admin) {
  if (isAdmin(user)) {
    console.log(`Admin ${user.name} with permissions: ${user.permissions}`);
  } else {
    console.log(`User ${user.name}`);
  }
}
```

---

## TypeScript with DOM

```typescript
// querySelector returns Element | null
const button = document.querySelector("#submit") as HTMLButtonElement;
// or with type guard:
const input = document.querySelector("#email");
if (input instanceof HTMLInputElement) {
  input.value = "hello@example.com"; // TypeScript knows properties
}

// Event handler typing
button.addEventListener("click", (event: MouseEvent) => {
  const target = event.target as HTMLButtonElement;
  console.log(target.textContent);
});

// Generic querySelector
const form = document.querySelector<HTMLFormElement>("#myForm");
form?.addEventListener("submit", (e: SubmitEvent) => {
  e.preventDefault();
});
```

---

## TypeScript in React + Vite

```typescript
// Component with typed props
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: "primary" | "secondary";
  disabled?: boolean;
}

const Button: React.FC<ButtonProps> = ({ label, onClick, variant = "primary", disabled }) => {
  return (
    <button className={variant} onClick={onClick} disabled={disabled}>
      {label}
    </button>
  );
};

// Typed useState
const [count, setCount] = useState<number>(0);
const [user, setUser] = useState<User | null>(null);

// Typed event handlers
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  console.log(e.target.value);
};
```

---

## Best Practices

1. **Use discriminated unions** for complex state — exhaustive switch catches missing cases.
2. **Use utility types** instead of rewriting types manually.
3. **Narrow with type guards** rather than casting with `as`.
4. **Use `never` for exhaustiveness checking** — catches unhandled cases at compile time.
5. **Prefer `unknown` over `any`** — forces you to narrow before using.
6. **Use `as const`** for literal types and readonly data.

---

## Summary

- Utility types (`Partial`, `Pick`, `Omit`, `Record`, `Readonly`) transform types without rewriting.
- Mapped types create new types by iterating over keys of existing types.
- Conditional types (`T extends U ? X : Y`) enable type-level logic.
- Discriminated unions with a literal `kind`/`type` field enable safe narrowing in switch statements.
- Type guards (`typeof`, `instanceof`, `in`, custom predicates) narrow types within control flow.
- Template literal types build string-based types from combinations.
- TypeScript integrates with DOM and React through specific type annotations and generics.
