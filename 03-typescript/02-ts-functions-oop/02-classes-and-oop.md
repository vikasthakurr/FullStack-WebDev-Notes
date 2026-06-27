# Classes & OOP in TypeScript

## What Are Classes in TypeScript?

TypeScript enhances JavaScript classes with **type annotations, access modifiers, abstract classes, and interface implementation**. You get the same class syntax as JavaScript, plus compile-time safety for your object-oriented designs.

**Analogy:** JavaScript classes are like building a house with no building codes — anything goes. TypeScript classes add blueprints and inspections — you define what's public, what's private, and what's required before you start building.

---

## Basic Class Syntax with Types

```typescript
class User {
  name: string;
  email: string;
  age: number;

  constructor(name: string, email: string, age: number) {
    this.name = name;
    this.email = email;
    this.age = age;
  }

  greet(): string {
    return `Hi, I'm ${this.name}`;
  }

  isAdult(): boolean {
    return this.age >= 18;
  }
}

const user = new User("Vikas", "vikas@example.com", 25);
console.log(user.greet()); // "Hi, I'm Vikas"
```

---
