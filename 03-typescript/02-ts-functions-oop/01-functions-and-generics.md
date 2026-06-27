# TypeScript Functions, Classes & Generics

## Function Types

### Parameter & Return Types

```typescript
function add(a: number, b: number): number {
  return a + b;
}

// Arrow function
const multiply = (a: number, b: number): number => a * b;

// Void return
function logMessage(msg: string): void {
  console.log(msg);
}
```

### Optional & Default Parameters

```typescript
function createUser(name: string, age?: number, role: string = "user"): object {
  return { name, age, role };
}

createUser("Vikas"); // OK
createUser("Vikas", 25); // OK
createUser("Vikas", 25, "admin"); // OK
```

### Rest Parameters

```typescript
function sum(...numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3, 4); // 10
```

### Function Overloading

Provide multiple type signatures for a single function:

```typescript
function format(value: string): string;
function format(value: number): string;
function format(value: string | number): string {
  if (typeof value === "string") {
    return value.toUpperCase();
  }
  return value.toFixed(2);
}

format("hello"); // Return type: string
format(3.14159); // Return type: string
```

### Function Type Expressions

```typescript
type MathOperation = (a: number, b: number) => number;

const add: MathOperation = (a, b) => a + b;
const subtract: MathOperation = (a, b) => a - b;

// As parameter type
function calculate(a: number, b: number, operation: MathOperation): number {
  return operation(a, b);
}
```

---

## Classes in TypeScript

### Basic Class

```typescript
class User {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }

  greet(): string {
    return `Hi, I'm ${this.name}`;
  }
}

const user = new User("Vikas", 25);
```

### Shorthand Constructor

```typescript
class User {
  constructor(
    public name: string,
    public age: number,
    private email: string,
  ) {}
  // Properties are declared AND assigned automatically
}
```

### Access Modifiers

```typescript
class BankAccount {
  public owner: string; // Accessible everywhere (default)
  private balance: number; // Accessible only inside this class
  protected accountType: string; // Accessible in this class and subclasses

  constructor(owner: string, initialBalance: number) {
    this.owner = owner;
    this.balance = initialBalance;
    this.accountType = "savings";
  }

  public deposit(amount: number): void {
    this.balance += amount;
  }

  public getBalance(): number {
    return this.balance;
  }
}

const account = new BankAccount("Vikas", 1000);
account.owner; // ✅ public
// account.balance;  // ❌ Error — private
account.getBalance(); // ✅ access through method
```

### Readonly

```typescript
class Config {
  readonly dbHost: string;

  constructor(host: string) {
    this.dbHost = host; // Can assign in constructor
  }

  // changeHost() { this.dbHost = "new"; } // ❌ Error — readonly
}
```

### Inheritance

```typescript
class Animal {
  constructor(public name: string) {}

  move(distance: number): void {
    console.log(`${this.name} moved ${distance}m`);
  }
}

class Dog extends Animal {
  bark(): void {
    console.log("Woof!");
  }

  // Override parent method
  move(distance: number): void {
    console.log("Running...");
    super.move(distance); // Call parent's move
  }
}

const dog = new Dog("Rex");
dog.bark(); // "Woof!"
dog.move(10); // "Running..." then "Rex moved 10m"
```

### Abstract Classes

Cannot be instantiated directly — must be extended:

```typescript
abstract class Shape {
  abstract area(): number; // Must be implemented by subclass
  abstract perimeter(): number;

  describe(): string {
    return `Area: ${this.area()}, Perimeter: ${this.perimeter()}`;
  }
}

class Circle extends Shape {
  constructor(private radius: number) {
    super();
  }

  area(): number {
    return Math.PI * this.radius ** 2;
  }

  perimeter(): number {
    return 2 * Math.PI * this.radius;
  }
}

// const s = new Shape(); // ❌ Error — abstract
const c = new Circle(5);
c.describe(); // "Area: 78.54, Perimeter: 31.42"
```

### Interfaces with Classes

```typescript
interface Serializable {
  serialize(): string;
}

interface Loggable {
  log(): void;
}

class User implements Serializable, Loggable {
  constructor(
    public name: string,
    public age: number,
  ) {}

  serialize(): string {
    return JSON.stringify({ name: this.name, age: this.age });
  }

  log(): void {
    console.log(`User: ${this.name}`);
  }
}
```

---

## Generics

Generics allow you to write reusable code that works with **any type** while maintaining type safety.

### The Problem Without Generics

```typescript
function identity(value: any): any {
  return value;
}
// Type information is lost — returns any, no autocomplete
```

### Generic Function

```typescript
function identity<T>(value: T): T {
  return value;
}

const str = identity("hello"); // Type: string
const num = identity(42); // Type: number
// Full autocomplete and type safety preserved
```

`T` is a **type parameter** — a placeholder that gets filled when the function is called.

### Generic with Constraints

```typescript
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(item: T): void {
  console.log(item.length);
}

logLength("hello"); // ✅ string has .length
logLength([1, 2, 3]); // ✅ array has .length
// logLength(42);      // ❌ number has no .length
```

### Multiple Type Parameters

```typescript
function pair<K, V>(key: K, value: V): [K, V] {
  return [key, value];
}

const entry = pair("name", "Vikas"); // [string, string]
const item = pair(1, true); // [number, boolean]
```

### Generic Interfaces

```typescript
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

const userResponse: ApiResponse<{ name: string; age: number }> = {
  data: { name: "Vikas", age: 25 },
  status: 200,
  message: "Success",
};

const listResponse: ApiResponse<string[]> = {
  data: ["item1", "item2"],
  status: 200,
  message: "Success",
};
```

### Generic Classes

```typescript
class DataStore<T> {
  private items: T[] = [];

  add(item: T): void {
    this.items.push(item);
  }

  get(index: number): T {
    return this.items[index];
  }

  getAll(): T[] {
    return [...this.items];
  }
}

const numberStore = new DataStore<number>();
numberStore.add(1);
numberStore.add(2);

const stringStore = new DataStore<string>();
stringStore.add("hello");
```

### Generic Utility Example

```typescript
function merge<T extends object, U extends object>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

const result = merge({ name: "Vikas" }, { age: 25 });
// Type: { name: string } & { age: number }
result.name; // string
result.age; // number
```

---

## Decorators (Introduction)

Decorators are special annotations that modify class declarations and members. They are experimental and require `"experimentalDecorators": true` in tsconfig.

```typescript
// Class decorator
function Logger(constructor: Function) {
  console.log(`Creating instance of: ${constructor.name}`);
}

@Logger
class Person {
  constructor(public name: string) {}
}
// Logs: "Creating instance of: Person" when class is defined

// Method decorator
function LogMethod(target: any, key: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${key} with:`, args);
    return original.apply(this, args);
  };
}

class Calculator {
  @LogMethod
  add(a: number, b: number): number {
    return a + b;
  }
}
```

---

## Best Practices

1. **Use generics** when a function/class needs to work with multiple types but maintain safety.
2. **Constrain generics** with `extends` — do not make them too broad.
3. **Use access modifiers** — default to `private`, expose only what is needed.
4. **Prefer composition over inheritance** — use interfaces and composition patterns.
5. **Use the shorthand constructor** — reduces boilerplate significantly.
6. **Avoid overusing decorators** — they add magic that is hard to trace.

---

## Summary

- TypeScript functions support typed parameters, return types, overloading, and rest parameters.
- Classes have access modifiers (`public`, `private`, `protected`), `readonly`, abstract classes, and interface implementation.
- Generics (`<T>`) enable reusable, type-safe code that works with any type.
- Constrain generics with `extends` to ensure the type has the required shape.
- Combine generics with interfaces for powerful patterns like `ApiResponse<T>` and `DataStore<T>`.
- Decorators modify classes and methods declaratively — use with caution as they are still evolving.
