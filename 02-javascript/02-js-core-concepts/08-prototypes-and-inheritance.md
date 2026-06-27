# Prototypes & Inheritance in JavaScript

## The Prototype Chain

Every object in JavaScript has a hidden internal link to another object called its **prototype**. When you access a property on an object, JavaScript first looks at the object itself. If not found, it walks up the prototype chain — checking each prototype in turn — until it reaches `null`.

**Analogy:** Think of the prototype chain like a family lineage. If you don't know how to do something (property not found), you ask your parent. If they don't know either, they ask their parent. This continues up the family tree until you either find someone who knows, or reach the end of the lineage (`null`).

```javascript
const person = {
  name: "Vikas",
  greet() {
    return `Hi, I'm ${this.name}`;
  },
};
const developer = Object.create(person);
developer.language = "JavaScript";

console.log(developer.language); // "JavaScript" — own property
console.log(developer.name); // "Vikas" — found on prototype (person)
console.log(developer.greet()); // "Hi, I'm Vikas" — method from prototype
console.log(developer.toString()); // "[object Object]" — from Object.prototype
```

### How the Lookup Works

```
developer.name
  1. Check developer itself → no "name" property
  2. Check developer's prototype (person) → found! Return "Vikas"

developer.toString()
  1. Check developer → not found
  2. Check person → not found
  3. Check Object.prototype → found! Return "[object Object]"

developer.nonExistent
  1. Check developer → not found
  2. Check person → not found
  3. Check Object.prototype → not found
  4. Reach null → return undefined
```

---

## `__proto__` vs `prototype` Property

These two are commonly confused. They serve different purposes:

### `__proto__` (Every Object Has This)

`__proto__` is the actual link to an object's prototype. Every object has it. It points to the object from which this instance inherits.

```javascript
const arr = [1, 2, 3];
console.log(arr.__proto__ === Array.prototype); // true
console.log(arr.__proto__.__proto__ === Object.prototype); // true
console.log(arr.__proto__.__proto__.__proto__ === null); // true
```

> **Note:** `__proto__` is deprecated as a direct accessor. Use `Object.getPrototypeOf(obj)` (read) and `Object.setPrototypeOf(obj, proto)` (write) instead.

### `prototype` (Only Functions Have This)

`prototype` is a property that exists only on **functions** (specifically, constructor functions and classes). It defines what will become the `__proto__` of instances created with `new`.

```javascript
function Dog(name) {
  this.name = name;
}
Dog.prototype.bark = function () {
  return `${this.name} says Woof!`;
};

const rex = new Dog("Rex");

// rex.__proto__ === Dog.prototype
console.log(Object.getPrototypeOf(rex) === Dog.prototype); // true
console.log(rex.bark()); // "Rex says Woof!"

// Dog.prototype is an object that instances inherit from
// Dog.__proto__ is Function.prototype (Dog itself is a function)
```

### Visual Comparison

```
┌─────────────────────────────────────────────────────┐
│                   Dog (function)                      │
│  - prototype: { bark: fn, constructor: Dog }         │
│  - __proto__: Function.prototype                     │
└─────────────────────────────────────────────────────┘
                         │
            new Dog("Rex") creates:
                         ▼
┌─────────────────────────────────────────────────────┐
│                   rex (instance)                      │
│  - name: "Rex"                                       │
│  - __proto__: Dog.prototype ─────────┐               │
└─────────────────────────────────────────────────────┘
                                       │
                                       ▼
                         ┌─────────────────────────┐
                         │   Dog.prototype          │
                         │  - bark: function        │
                         │  - constructor: Dog      │
                         │  - __proto__: Object.prototype
                         └─────────────────────────┘
```

### Key Distinction

| Term         | Lives On       | Purpose                                             |
| ------------ | -------------- | --------------------------------------------------- |
| `__proto__`  | Every object   | Points to the object's prototype (inheritance link) |
| `.prototype` | Functions only | Template for instances created with `new`           |

---

## Object.create() and Prototypal Inheritance

`Object.create(proto)` creates a new object with `proto` as its prototype. This is the purest form of prototypal inheritance — no constructors, no `new`, just objects inheriting from objects.

```javascript
// Base object (the "parent")
const animal = {
  alive: true,
  eat() {
    return `${this.name} is eating.`;
  },
  sleep() {
    return `${this.name} is sleeping.`;
  },
};

// Create an object that inherits from animal
const dog = Object.create(animal);
dog.name = "Rex";
dog.bark = function () {
  return `${this.name} says Woof!`;
};

console.log(dog.eat()); // "Rex is eating." — inherited from animal
console.log(dog.bark()); // "Rex says Woof!" — own method
console.log(dog.alive); // true — inherited

// Create another level
const puppy = Object.create(dog);
puppy.name = "Buddy";
puppy.play = function () {
  return `${this.name} is playing!`;
};

console.log(puppy.bark()); // "Buddy says Woof!" — from dog
console.log(puppy.eat()); // "Buddy is eating." — from animal
console.log(puppy.play()); // "Buddy is playing!" — own method
```

### Object.create with Property Descriptors

```javascript
const base = { type: "base" };

const child = Object.create(base, {
  name: {
    value: "Child Object",
    writable: true,
    enumerable: true,
    configurable: true,
  },
  id: {
    value: 42,
    writable: false, // read-only
  },
});

console.log(child.name); // "Child Object"
console.log(child.type); // "base" — inherited
child.id = 99; // silently fails (non-writable)
console.log(child.id); // 42
```

### Object.create(null) — No Prototype

Creates a truly empty object with no inherited properties at all (not even `toString` or `hasOwnProperty`):

```javascript
const dict = Object.create(null);
dict.hello = "world";

console.log(dict.toString); // undefined — no prototype chain!
console.log("hello" in dict); // true

// Useful for: pure dictionaries, avoiding prototype pollution
```

---

## Constructor Functions and the `new` Keyword

Before ES6 classes, constructor functions were the standard way to create objects with shared methods.

### How `new` Works

When you call a function with `new`, JavaScript does 4 things:

1. Creates a new empty object `{}`.
2. Sets the new object's `__proto__` to the constructor's `.prototype`.
3. Calls the constructor with `this` bound to the new object.
4. Returns the new object (unless the constructor explicitly returns a different object).

```javascript
function Person(name, age) {
  // Step 3: `this` is the new empty object
  this.name = name;
  this.age = age;
  // Step 4: implicitly returns `this`
}

// Shared methods go on the prototype (not duplicated per instance)
Person.prototype.greet = function () {
  return `Hi, I'm ${this.name}, age ${this.age}`;
};

Person.prototype.birthday = function () {
  this.age++;
  return `${this.name} is now ${this.age}`;
};

const vikas = new Person("Vikas", 25);
const rahul = new Person("Rahul", 30);

vikas.greet(); // "Hi, I'm Vikas, age 25"
rahul.greet(); // "Hi, I'm Rahul, age 30"

// Methods are shared — same reference
vikas.greet === rahul.greet; // true (both point to Person.prototype.greet)
```

### Simulating `new` Manually

```javascript
function simulateNew(Constructor, ...args) {
  // 1. Create empty object with Constructor.prototype as prototype
  const obj = Object.create(Constructor.prototype);
  // 2. Call constructor with new object as `this`
  const result = Constructor.apply(obj, args);
  // 3. If constructor returns an object, use that; otherwise use obj
  return result instanceof Object ? result : obj;
}

const person = simulateNew(Person, "Vikas", 25);
person.greet(); // "Hi, I'm Vikas, age 25"
```

### Why Put Methods on `.prototype` Instead of Inside the Constructor?

```javascript
// ❌ Bad: each instance gets its own copy of the function
function BadPerson(name) {
  this.name = name;
  this.greet = function () {
    return `Hi, ${this.name}`;
  };
}
// 1000 instances = 1000 function copies in memory

// ✅ Good: all instances share one function on the prototype
function GoodPerson(name) {
  this.name = name;
}
GoodPerson.prototype.greet = function () {
  return `Hi, ${this.name}`;
};
// 1000 instances = 1 shared function
```

---

## Constructor-Based Inheritance (Pre-ES6)

```javascript
// Parent constructor
function Animal(name, sound) {
  this.name = name;
  this.sound = sound;
}
Animal.prototype.speak = function () {
  return `${this.name} says ${this.sound}`;
};
Animal.prototype.sleep = function () {
  return `${this.name} is sleeping...`;
};

// Child constructor
function Dog(name, breed) {
  // Call parent constructor (like super())
  Animal.call(this, name, "Woof");
  this.breed = breed;
}

// Set up inheritance chain
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog; // Fix the constructor reference

// Add child-specific methods
Dog.prototype.fetch = function () {
  return `${this.name} fetches the ball!`;
};

// Override parent method
Dog.prototype.speak = function () {
  return `${this.name} barks: ${this.sound}! ${this.sound}!`;
};

const rex = new Dog("Rex", "German Shepherd");
rex.speak(); // "Rex barks: Woof! Woof!" — overridden
rex.sleep(); // "Rex is sleeping..." — inherited from Animal
rex.fetch(); // "Rex fetches the ball!" — own method

rex instanceof Dog; // true
rex instanceof Animal; // true
```

---

## ES6 Classes (Syntactic Sugar Over Prototypes)

ES6 classes provide cleaner syntax but work exactly the same way under the hood — they still use prototype-based inheritance.

```javascript
class Animal {
  constructor(name, sound) {
    this.name = name;
    this.sound = sound;
  }

  speak() {
    return `${this.name} says ${this.sound}`;
  }

  sleep() {
    return `${this.name} is sleeping...`;
  }
}

// Under the hood, this is equivalent to:
// function Animal(name, sound) { this.name = name; this.sound = sound; }
// Animal.prototype.speak = function() { ... };
// Animal.prototype.sleep = function() { ... };
```

### Proving Classes Are Syntactic Sugar

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }
  greet() {
    return `Hello from ${this.name}`;
  }
}

// It's still prototype-based:
console.log(typeof Person); // "function"
console.log(Person.prototype.greet); // [Function: greet]

const vikas = new Person("Vikas");
console.log(Object.getPrototypeOf(vikas) === Person.prototype); // true
```

### Class Features

```javascript
class User {
  // Public field (on the instance, not prototype)
  role = "user";

  // Private field (truly private — not accessible outside)
  #password;

  // Static property (on the class itself, not instances)
  static count = 0;

  constructor(name, password) {
    this.name = name;
    this.#password = password;
    User.count++;
  }

  // Instance method (on prototype)
  greet() {
    return `Hi, I'm ${this.name} (${this.role})`;
  }

  // Private method
  #hashPassword() {
    return `hashed_${this.#password}`;
  }

  // Getter
  get displayName() {
    return `@${this.name.toLowerCase()}`;
  }

  // Setter
  set displayName(value) {
    this.name = value.replace("@", "");
  }

  // Static method (called on the class, not instances)
  static getCount() {
    return `Total users: ${User.count}`;
  }
}

const user = new User("Vikas", "secret123");
user.greet(); // "Hi, I'm Vikas (user)"
user.displayName; // "@vikas"
user.displayName = "@NewName";
user.name; // "NewName"
User.getCount(); // "Total users: 1"

// user.#password;      // SyntaxError — truly private!
```

---

## `extends` and `super`

`extends` sets up the prototype chain between two classes. `super` calls the parent class's constructor or methods.

```javascript
class Animal {
  constructor(name, legs) {
    this.name = name;
    this.legs = legs;
  }

  describe() {
    return `${this.name} has ${this.legs} legs`;
  }

  move() {
    return `${this.name} is moving`;
  }
}

class Dog extends Animal {
  #tricks = [];

  constructor(name, breed) {
    // MUST call super() before using `this` in a derived class
    super(name, 4); // Calls Animal's constructor
    this.breed = breed;
  }

  // Override parent method
  move() {
    return `${this.name} is running on ${this.legs} legs!`;
  }

  // Call parent method with super
  describe() {
    const base = super.describe(); // "Rex has 4 legs"
    return `${base} and is a ${this.breed}`;
  }

  // Own method
  learn(trick) {
    this.#tricks.push(trick);
    return `${this.name} learned ${trick}!`;
  }

  get tricks() {
    return [...this.#tricks];
  }
}
```

class GuideDog extends Dog {
constructor(name, breed, owner) {
super(name, breed); // Calls Dog's constructor (which calls Animal's)
this.owner = owner;
}

guide() {
return `${this.name} is guiding ${this.owner}`;
}
}

const rex = new Dog("Rex", "German Shepherd");
rex.describe(); // "Rex has 4 legs and is a German Shepherd"
rex.move(); // "Rex is running on 4 legs!"
rex.learn("sit"); // "Rex learned sit!"

const guide = new GuideDog("Buddy", "Labrador", "Vikas");
guide.describe(); // "Buddy has 4 legs and is a Labrador"
guide.guide(); // "Buddy is guiding Vikas"
guide.move(); // "Buddy is running on 4 legs!" — inherited from Dog

````

### Rules for `super`

1. **Must call `super()` in derived constructor** before accessing `this`.
2. `super()` calls the parent's `constructor`.
3. `super.method()` calls a parent's method from within an overriding method.
4. If you don't define a constructor in the child, a default one is created that calls `super(...args)`.

```javascript
class Child extends Parent {
  // If you write NO constructor, this is implicitly created:
  // constructor(...args) { super(...args); }
}
````

### Static Inheritance

Static methods are also inherited through `extends`:

```javascript
class Shape {
  static create(type) {
    switch (type) {
      case "circle":
        return new Circle();
      case "square":
        return new Square();
    }
  }

  static description = "I am a shape";
}

class Circle extends Shape {
  area(radius) {
    return Math.PI * radius ** 2;
  }
}

// Static methods/properties are inherited
Circle.description; // "I am a shape"
Circle.create; // [Function: create] — inherited static method
```

---

## The `instanceof` Operator

`instanceof` checks if an object has a constructor's `prototype` anywhere in its prototype chain.

```javascript
class Animal {}
class Dog extends Animal {}
class Cat extends Animal {}

const rex = new Dog();

rex instanceof Dog; // true
rex instanceof Animal; // true — Animal.prototype is in rex's chain
rex instanceof Cat; // false
rex instanceof Object; // true — everything inherits from Object

// How instanceof works internally:
// rex instanceof Dog
//   → Is Dog.prototype in rex's prototype chain?
//   → rex.__proto__ === Dog.prototype? YES → true
```

### `instanceof` with Constructor Functions

```javascript
function Vehicle(type) {
  this.type = type;
}
function Car(brand) {
  Vehicle.call(this, "car");
  this.brand = brand;
}
Car.prototype = Object.create(Vehicle.prototype);
Car.prototype.constructor = Car;

const tesla = new Car("Tesla");
tesla instanceof Car; // true
tesla instanceof Vehicle; // true
```

### Limitations of `instanceof`

```javascript
// 1. Fails across realms (iframes, different windows)
// An array created in an iframe is NOT instanceof Array in the parent window

// 2. Can be fooled by manual prototype manipulation
const fake = {};
Object.setPrototypeOf(fake, Dog.prototype);
fake instanceof Dog; // true — but it's not really a Dog!

// 3. Does not work with primitives
"hello" instanceof String; // false — it's a primitive, not a String object
42 instanceof Number; // false

// Better alternatives:
Array.isArray([]); // true — works across realms
typeof "hello" === "string"; // true — works for primitives
```

### `Symbol.hasInstance` — Customize instanceof

```javascript
class EvenNumber {
  static [Symbol.hasInstance](value) {
    return typeof value === "number" && value % 2 === 0;
  }
}

4 instanceof EvenNumber; // true
7 instanceof EvenNumber; // false
"8" instanceof EvenNumber; // false — not a number
```

---

## Prototype Pollution Awareness

Prototype pollution is a security vulnerability where an attacker modifies `Object.prototype` (or another shared prototype), affecting all objects in the application.

### How It Happens

```javascript
// Vulnerable function: recursively merges objects
function merge(target, source) {
  for (let key in source) {
    if (typeof source[key] === "object" && source[key] !== null) {
      target[key] = target[key] || {};
      merge(target[key], source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

// Attacker sends malicious JSON:
const malicious = JSON.parse('{"__proto__": {"isAdmin": true}}');

const userConfig = {};
merge(userConfig, malicious);

// Now EVERY object in the app has isAdmin = true
const normalUser = {};
console.log(normalUser.isAdmin); // true ← POLLUTED!
```

### Real-World Impact

```javascript
// After pollution: Object.prototype.isAdmin = true

function checkAccess(user) {
  if (user.isAdmin) {
    // Grant admin access — attacker bypasses auth!
    return "admin";
  }
  return "user";
}

const attacker = { name: "hacker" };
checkAccess(attacker); // "admin" ← because isAdmin is inherited!
```

### Prevention Strategies

```javascript
// 1. Use Object.create(null) for dictionaries (no prototype)
const safeDict = Object.create(null);
safeDict.isAdmin; // undefined — no prototype to pollute

// 2. Check hasOwnProperty before accessing
function safeCheck(user) {
  if (user.hasOwnProperty("isAdmin") && user.isAdmin) {
    return "admin";
  }
  return "user";
}

// 3. Freeze the prototype (prevents modifications)
Object.freeze(Object.prototype); // Extreme — may break libraries

// 4. Sanitize keys in merge functions
function safeMerge(target, source) {
  for (let key of Object.keys(source)) {
    // Object.keys = own properties only
    if (key === "__proto__" || key === "constructor" || key === "prototype") {
      continue; // Skip dangerous keys
    }
    if (typeof source[key] === "object" && source[key] !== null) {
      target[key] = target[key] || {};
      safeMerge(target[key], source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

// 5. Use Map instead of plain objects for dynamic keys
const userPermissions = new Map();
userPermissions.set("isAdmin", false); // No prototype chain involved
```

### Known Vulnerable Patterns

| Pattern                         | Risk                                   | Safe Alternative                         |
| ------------------------------- | -------------------------------------- | ---------------------------------------- |
| Recursive object merge          | `__proto__` key modifies prototype     | Filter dangerous keys, use `Object.keys` |
| `obj[userInput] = value`        | User controls key, can set `__proto__` | Validate key, use `Map`                  |
| Deep clone via JSON parse/merge | Malicious JSON with `__proto__`        | Use `structuredClone()` or safe libs     |
| Query string to object parsing  | `?__proto__[isAdmin]=true`             | Use `URLSearchParams`, sanitize keys     |

---

## Putting It All Together — Class vs Prototype Pattern

### ES6 Class Pattern (Recommended)

```javascript
class EventEmitter {
  #listeners = new Map();

  on(event, callback) {
    if (!this.#listeners.has(event)) {
      this.#listeners.set(event, []);
    }
    this.#listeners.get(event).push(callback);
    return this;
  }

  emit(event, ...args) {
    const callbacks = this.#listeners.get(event) || [];
    callbacks.forEach((cb) => cb(...args));
    return this;
  }

  off(event, callback) {
    const callbacks = this.#listeners.get(event) || [];
    this.#listeners.set(
      event,
      callbacks.filter((cb) => cb !== callback),
    );
    return this;
  }
}

class Logger extends EventEmitter {
  #level;

  constructor(level = "info") {
    super();
    this.#level = level;
  }

  log(message) {
    const entry = { level: this.#level, message, timestamp: Date.now() };
    this.emit("log", entry);
    console.log(`[${this.#level.toUpperCase()}] ${message}`);
  }
}

const logger = new Logger("debug");
logger.on("log", (entry) => {
  /* send to monitoring */
});
logger.log("Server started"); // [DEBUG] Server started
```

---

## Best Practices

1. **Use ES6 classes** for new code — cleaner syntax, same prototype behavior underneath.
2. **Prefer composition over deep inheritance** — avoid chains deeper than 2-3 levels; use mixins or modules to share behavior.
3. **Always call `super()` first** in derived constructors before accessing `this`.
4. **Put shared methods on the prototype** (automatic with class syntax) — not inside the constructor.
5. **Use `Object.getPrototypeOf()` instead of `__proto__`** — `__proto__` is deprecated and non-standard in strict mode.
6. **Use `Object.create(null)` for pure key-value maps** — avoids prototype pollution and unwanted inherited properties.
7. **Use `hasOwnProperty` or `Object.hasOwn()`** (ES2022) when checking for properties that may be inherited.
8. **Never modify `Object.prototype`** in production — it affects every object in your application.
9. **Validate/sanitize keys** in any function that merges user-controlled data into objects.
10. **Use `#private` fields** in classes for encapsulation — they are truly inaccessible from outside.

---

## Common Mistakes

| Mistake                                            | Why It Is Wrong                                              | Fix                                                        |
| -------------------------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------------- |
| Forgetting `super()` in derived constructor        | `this` is uninitialized — throws ReferenceError              | Always call `super()` before accessing `this`              |
| Using arrow functions for prototype methods        | Arrow functions don't have their own `this`                  | Use `function` keyword or class method syntax              |
| Modifying `Object.prototype`                       | Affects every object in the application                      | Never do this; use utility functions or subclassing        |
| Confusing `__proto__` and `.prototype`             | `__proto__` is on instances; `.prototype` is on constructors | Remember: `instance.__proto__ === Constructor.prototype`   |
| Deep inheritance chains (5+ levels)                | Fragile, hard to debug, tightly coupled                      | Prefer composition, mixins, or flat hierarchies            |
| Forgetting `Constructor` fix after `Object.create` | `Dog.prototype.constructor` points to wrong constructor      | Add `Dog.prototype.constructor = Dog` after reassignment   |
| Using `instanceof` across realms                   | Different `Array` constructors in different iframes          | Use `Array.isArray()` or duck typing                       |
| Ignoring prototype pollution in merges             | Attacker can inject properties into all objects              | Sanitize keys, use `Object.create(null)`, use `Map`        |
| Calling class without `new`                        | Classes throw TypeError if called without `new`              | Always use `new ClassName()`                               |
| Overriding without calling `super.method()`        | Loses parent behavior entirely                               | Call `super.method()` when you want to extend, not replace |

---

## Summary

- Every object in JavaScript has a **prototype chain** — a linked list of objects it inherits from, ending at `null`.
- **`__proto__`** is the actual inheritance link on every object; **`.prototype`** is a property on constructor functions that defines what instances inherit.
- **`Object.create(proto)`** creates an object with a specified prototype — the purest form of prototypal inheritance.
- **Constructor functions + `new`** were the pre-ES6 way to create objects with shared methods on their prototype.
- **ES6 classes** are syntactic sugar over constructor functions and prototypes — same mechanics, cleaner syntax.
- **`extends`** sets up prototype chain between classes; **`super`** calls parent constructor/methods.
- **`instanceof`** walks the prototype chain to check if a constructor's prototype exists in an object's lineage.
- **Prototype pollution** is a real security risk — always sanitize user-controlled keys and prefer `Object.create(null)` or `Map` for dynamic dictionaries.
- JavaScript's inheritance model is **prototype-based** (objects inherit from objects), not class-based — classes are just a familiar interface over this system.
