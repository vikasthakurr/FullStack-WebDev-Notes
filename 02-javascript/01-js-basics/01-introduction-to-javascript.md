# Introduction to JavaScript

## What Is JavaScript?

JavaScript is a high-level, interpreted, dynamically-typed programming language primarily used to add interactivity and behavior to web pages. Unlike HTML (structure) and CSS (presentation), JavaScript handles **logic** — making decisions, responding to user actions, manipulating data, and communicating with servers.

**Analogy:** If HTML is the skeleton of a building and CSS is the paint and decor, JavaScript is the electricity, plumbing, and elevator systems — everything that makes the building functional and interactive.

---

## Why JavaScript Matters

- It is the **only programming language** that runs natively in web browsers.
- It powers both **frontend** (React, Vue, Angular) and **backend** (Node.js, Deno, Bun).
- It is the most widely used programming language in the world (Stack Overflow surveys, GitHub statistics).
- It handles everything from simple form validation to complex real-time applications, games, and AI.

---

## What JavaScript Can Do

### In the Browser

- Manipulate the DOM (change content, styles, structure dynamically).
- Handle user events (clicks, keypresses, scrolls, form submissions).
- Make HTTP requests (fetch data from APIs without reloading the page).
- Store data locally (localStorage, sessionStorage, IndexedDB).
- Create animations and visual effects.
- Access device APIs (camera, geolocation, notifications).

### On the Server (Node.js)

- Build web servers and REST APIs.
- Read/write files on the filesystem.
- Connect to databases (MongoDB, PostgreSQL, MySQL).
- Handle real-time communication (WebSockets, Server-Sent Events).
- Run scripts, automation, and CLI tools.

---

## A Brief History

| Year  | Milestone                                                                 |
| ----- | ------------------------------------------------------------------------- |
| 1995  | Brendan Eich creates JavaScript in 10 days at Netscape                    |
| 1997  | ECMAScript 1 — first standardized version                                 |
| 2009  | ES5 — `strict mode`, JSON, array methods (`forEach`, `map`)               |
| 2009  | Node.js released — JavaScript moves to the server                         |
| 2015  | ES6 (ES2015) — `let`/`const`, arrow functions, classes, promises, modules |
| 2016+ | Annual releases — async/await (2017), optional chaining (2020), etc.      |

ES6 was the biggest leap. Modern JavaScript is significantly different from pre-2015 code.

---

## JavaScript vs Other Languages

| Feature     | JavaScript                   | Python           | Java                 |
| ----------- | ---------------------------- | ---------------- | -------------------- |
| Typing      | Dynamic                      | Dynamic          | Static               |
| Compilation | JIT (Just-In-Time)           | Interpreted      | Compiled to bytecode |
| Paradigm    | Multi (OOP, Functional)      | Multi            | OOP                  |
| Runs in     | Browser + Server             | Server + Desktop | Server + Desktop     |
| Concurrency | Single-threaded + Event Loop | Multi-threaded   | Multi-threaded       |

---

## Your First JavaScript

### In the Browser Console

Open any browser → Right-click → Inspect → Console tab:

```javascript
console.log("Hello, World!");
// Output: Hello, World!

let name = "Vikas";
console.log(`Welcome, ${name}!`);
// Output: Welcome, Vikas!
```

### In an HTML File

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>JS Demo</title>
  </head>
  <body>
    <h1 id="greeting">Hello</h1>

    <!-- External script (preferred) -->
    <script src="app.js"></script>

    <!-- Inline script -->
    <script>
      document.getElementById("greeting").textContent =
        "Hello from JavaScript!";
    </script>
  </body>
</html>
```

### With Node.js

```bash
# Create a file
echo "console.log('Hello from Node.js!');" > hello.js

# Run it
node hello.js
# Output: Hello from Node.js!
```

---

## JavaScript Is NOT Java

Despite the similar name, JavaScript and Java are completely different languages. The name was a marketing decision by Netscape in 1995 to capitalize on Java's popularity. They share basic C-style syntax (`{}`, `;`, `if/else`) but differ in everything else.

---

## Summary

- JavaScript is the language of the web — it adds behavior and interactivity.
- It runs in browsers (frontend) and servers (Node.js backend).
- It is dynamically typed, single-threaded (with an event loop for concurrency), and supports both OOP and functional programming.
- ES6 (2015) modernized the language significantly — always write modern JavaScript.
- It is the most versatile language in web development: one language for frontend, backend, mobile (React Native), and desktop (Electron).
