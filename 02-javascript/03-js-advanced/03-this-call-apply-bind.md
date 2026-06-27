# `this`, `call`, `apply` & `bind`

## What Is `this`?

`this` is a special keyword that refers to the **context** in which a function is executing. Its value is determined at **call time**, not at definition time (except for arrow functions).

**Analogy:** `this` is like the word "I" in a conversation. Who "I" refers to depends on who is speaking, not on the sentence itself.

---

## Rules for `this`

### Rule 1: Global Context

```javascript
// In browser
console.log(this); // window object

// In Node.js (module)
console.log(this); // {} (module.exports)

// In strict mode
("use strict");
function strictFn() {
  console.log(this); // undefined
}
```

### Rule 2: Object Method — `this` = the object

```javascript
const user = {
  name: "Vikas",
  greet() {
    console.log(this.name); // "Vikas" — this refers to user
  },
};

user.greet(); // "Vikas"
```

### Rule 3: Regular Function — `this` = global (or undefined in strict mode)

```javascript
function showThis() {
  console.log(this); // window (non-strict) or undefined (strict)
}

showThis();
```

### Rule 4: Arrow Functions — `this` = inherited from enclosing scope (lexical)

```javascript
const user = {
  name: "Vikas",
  greet: () => {
    console.log(this.name); // undefined! Arrow inherits outer this (global/module)
  },
  delayedGreet() {
    setTimeout(() => {
      console.log(this.name); // "Vikas" — arrow inherits from delayedGreet's this
    }, 1000);
  },
};
```

### Rule 5: Constructor (`new`) — `this` = the new object

```javascript
function Person(name) {
  this.name = name; // this = newly created object
}

const p = new Person("Vikas");
console.log(p.name); // "Vikas"
```

### Rule 6: Event Handlers — `this` = the element that received the event

```javascript
button.addEventListener("click", function () {
  console.log(this); // the button element
});

button.addEventListener("click", () => {
  console.log(this); // window/undefined — arrow does NOT get element as this
});
```

---

## Summary of `this` Rules

| Call Style              | `this` Value                    |
| ----------------------- | ------------------------------- |
| `obj.method()`          | `obj`                           |
| `func()`                | `window` / `undefined` (strict) |
| `new Func()`            | New object                      |
| Arrow function          | Inherited from enclosing scope  |
| `call` / `apply`        | Explicitly set                  |
| Event handler (regular) | The DOM element                 |

---

## The Problem: Losing `this`

```javascript
const user = {
  name: "Vikas",
  greet() {
    console.log(`Hello, ${this.name}`);
  },
};

user.greet(); // "Hello, Vikas" ✅

const greetFn = user.greet;
greetFn(); // "Hello, undefined" ❌ — this is now window/undefined

// Same problem with callbacks
setTimeout(user.greet, 1000); // "Hello, undefined" ❌
```

When a method is extracted from its object, it loses its `this` context. This is where `call`, `apply`, and `bind` come in.

---

## `call()`

Calls a function with a specified `this` value and arguments passed **individually**.

```javascript
function greet(greeting, punctuation) {
  console.log(`${greeting}, ${this.name}${punctuation}`);
}

const user = { name: "Vikas" };

greet.call(user, "Hello", "!"); // "Hello, Vikas!"
greet.call({ name: "Rahul" }, "Hi", "."); // "Hi, Rahul."
```

**Syntax:** `func.call(thisArg, arg1, arg2, ...)`

---

## `apply()`

Identical to `call`, but arguments are passed as an **array**.

```javascript
greet.apply(user, ["Hello", "!"]); // "Hello, Vikas!"

// Useful when you have arguments in an array
const args = ["Hey", "!!!"];
greet.apply(user, args); // "Hey, Vikas!!!"
```

**Syntax:** `func.apply(thisArg, [arg1, arg2, ...])`

### `call` vs `apply`

| Method  | Arguments       | Mnemonic              |
| ------- | --------------- | --------------------- |
| `call`  | Comma-separated | **C**all = **C**ommas |
| `apply` | Array           | **A**pply = **A**rray |

> In modern JavaScript, you can use `call` with spread: `func.call(obj, ...args)` — making `apply` less necessary.

---

## `bind()`

Returns a **new function** with `this` permanently set. Does not call the function immediately.

```javascript
const user = { name: "Vikas" };

function greet(greeting) {
  console.log(`${greeting}, ${this.name}`);
}

const boundGreet = greet.bind(user);
boundGreet("Hello"); // "Hello, Vikas"
boundGreet("Hi"); // "Hi, Vikas"

// this is permanently bound — cannot be overridden
boundGreet.call({ name: "Rahul" }, "Hey"); // "Hey, Vikas" (still Vikas!)
```

### Fixing the Lost `this` Problem

```javascript
const user = {
  name: "Vikas",
  greet() {
    console.log(`Hello, ${this.name}`);
  },
};

// Fix: bind before passing as callback
setTimeout(user.greet.bind(user), 1000); // "Hello, Vikas" ✅

// Fix: bind for event handlers
button.addEventListener("click", user.greet.bind(user));
```

### Partial Application with `bind`

`bind` can also pre-fill arguments:

```javascript
function multiply(a, b) {
  return a * b;
}

const double = multiply.bind(null, 2); // Pre-fills first argument
double(5); // 10
double(10); // 20

const triple = multiply.bind(null, 3);
triple(5); // 15
```

---

## Debounce and Throttle

### Debounce

Delays execution until the user **stops** triggering the event for a specified time. Useful for search inputs, window resize.

```javascript
function debounce(fn, delay) {
  let timeoutId;

  return function (...args) {
    clearTimeout(timeoutId); // Cancel previous timer
    timeoutId = setTimeout(() => {
      fn.apply(this, args); // Preserve this context
    }, delay);
  };
}

// Usage
const searchInput = document.querySelector("#search");
const handleSearch = debounce((e) => {
  console.log("Searching for:", e.target.value);
  // API call here
}, 300);

searchInput.addEventListener("input", handleSearch);
```

**How it works:** Every keystroke resets the timer. The function only fires 300ms after the user stops typing.

### Throttle

Ensures a function runs **at most once** per specified time interval. Useful for scroll events, button clicks.

```javascript
function throttle(fn, limit) {
  let inThrottle = false;

  return function (...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(() => {
        inThrottle = false;
      }, limit);
    }
  };
}

// Usage
const handleScroll = throttle(() => {
  console.log("Scroll position:", window.scrollY);
}, 200);

window.addEventListener("scroll", handleScroll);
```

**How it works:** The first call executes immediately. Subsequent calls within the time limit are ignored. After the limit passes, the next call goes through.

### Debounce vs Throttle

| Feature      | Debounce                      | Throttle                             |
| ------------ | ----------------------------- | ------------------------------------ |
| Fires        | After activity stops          | At regular intervals during activity |
| Use case     | Search input, form validation | Scroll, resize, rate-limiting        |
| First event  | Delayed                       | Immediate                            |
| During burst | Only last event fires         | Events fire at interval              |

---

## Best Practices

1. **Use arrow functions** in callbacks when you want to preserve the outer `this`.
2. **Use `bind`** when passing methods as callbacks: `setTimeout(obj.method.bind(obj), 100)`.
3. **Use debounce** for search/autocomplete inputs, window resize handlers.
4. **Use throttle** for scroll events, mouse move tracking, rate-limited button clicks.
5. **Avoid relying on `this` in standalone functions** — it is confusing. Use arrow functions or explicit parameters.

---

## Common Mistakes

| Mistake                                 | Why It Is Wrong                                | Fix                                             |
| --------------------------------------- | ---------------------------------------------- | ----------------------------------------------- |
| Arrow function as object method         | `this` will not refer to the object            | Use regular function for methods                |
| Forgetting to `bind` in event handlers  | `this` becomes the DOM element, not the object | Use `.bind(obj)` or arrow function              |
| Using `call`/`apply` on arrow functions | Arrow `this` cannot be overridden              | Use regular function if you need dynamic `this` |
| Not clearing debounce on unmount        | Timer fires after component is removed         | Clear timeout on cleanup                        |

---

## Summary

- `this` is determined by **how** a function is called, not where it is defined (except arrows).
- `call(thisArg, ...args)` — invoke with explicit `this`, arguments as list.
- `apply(thisArg, [args])` — invoke with explicit `this`, arguments as array.
- `bind(thisArg)` — returns a new function with `this` permanently bound (does not invoke).
- Arrow functions inherit `this` lexically — they cannot be rebound.
- **Debounce** delays until inactivity; **throttle** limits to once per interval.
- These are essential for performance optimization and UX in event-heavy applications.
