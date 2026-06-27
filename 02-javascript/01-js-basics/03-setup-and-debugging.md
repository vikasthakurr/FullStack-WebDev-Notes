# Editor Setup & Debugging Basics

## Installing Node.js & npm

### What Is Node.js?

Node.js is a JavaScript runtime built on Chrome's V8 engine. It allows you to run JavaScript outside the browser — on your machine, servers, and anywhere else.

### What Is npm?

npm (Node Package Manager) comes bundled with Node.js. It manages third-party libraries (packages) and provides a command-line interface for running scripts.

### Installation

1. Go to [https://nodejs.org](https://nodejs.org)
2. Download the **LTS** (Long Term Support) version — more stable than "Current."
3. Run the installer (includes both `node` and `npm`).

### Verify Installation

```bash
node --version
# v20.x.x (or similar)

npm --version
# 10.x.x (or similar)
```

### nvm (Node Version Manager) — Recommended

Lets you switch between Node.js versions easily:

```bash
# Install nvm (see nvm GitHub repo for latest instructions)
# Then:
nvm install 20
nvm use 20
nvm list
```

---

## Using the Browser Console

The browser console is the fastest way to experiment with JavaScript.

### How to Open It

| Browser       | Shortcut (Windows/Linux)                              | Shortcut (Mac) |
| ------------- | ----------------------------------------------------- | -------------- |
| Chrome / Edge | `F12` or `Ctrl+Shift+J`                               | `Cmd+Option+J` |
| Firefox       | `F12` or `Ctrl+Shift+K`                               | `Cmd+Option+K` |
| Safari        | Enable via Preferences → Advanced → Show Develop menu | `Cmd+Option+C` |

### What You Can Do

```javascript
// Execute any JavaScript
let x = 42;
console.log(x * 2); // 84

// Access the current page's DOM
document.title; // Returns page title
document.querySelectorAll("a").length; // Count all links

// Multi-line code (Shift+Enter for new line)
function greet(name) {
  return `Hello, ${name}!`;
}
greet("Vikas"); // "Hello, Vikas!"
```

### Console Methods

```javascript
console.log("Basic output");
console.warn("Warning message"); // Yellow
console.error("Error message"); // Red
console.table([{ a: 1 }, { a: 2 }]); // Formatted table
console.time("timer"); // Start timer
// ... code ...
console.timeEnd("timer"); // End timer — shows duration
console.group("Group"); // Collapsible group
console.log("Inside group");
console.groupEnd();
console.clear(); // Clear the console
```

---

## Editor Setup (VS Code)

### Essential Extensions for JavaScript

| Extension                      | Purpose                                     |
| ------------------------------ | ------------------------------------------- |
| ESLint                         | Linting — catches errors and enforces style |
| Prettier                       | Auto-formatting on save                     |
| JavaScript (ES6) code snippets | Quick code templates                        |
| Path Intellisense              | Autocomplete for file paths                 |
| Error Lens                     | Shows errors inline (not just underlines)   |

### Recommended Settings (settings.json)

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.tabSize": 2,
  "editor.wordWrap": "on",
  "javascript.updateImportsOnFileMove.enabled": "always",
  "emmet.includeLanguages": {
    "javascript": "javascriptreact"
  }
}
```

---

## Running JavaScript

### Method 1: Browser (Script Tag)

```html
<!-- index.html -->
<script src="app.js"></script>
```

Open `index.html` in a browser. Use Live Server extension for auto-reload.

### Method 2: Node.js (Terminal)

```bash
node app.js
```

### Method 3: Node.js REPL

```bash
node
# Starts interactive mode (like browser console)
> 2 + 2
4
> .exit
```

### Method 4: VS Code — Run and Debug

1. Open a `.js` file.
2. Press `F5` or click "Run and Debug" in the sidebar.
3. Choose "Node.js" as the environment.
4. Code runs with full debugging support.

---

## Debugging Basics

### `console.log` Debugging (Quick & Dirty)

```javascript
function calculateTotal(items) {
  console.log("Items received:", items); // Check input

  let total = 0;
  for (let item of items) {
    console.log("Processing:", item.name, item.price); // Check each iteration
    total += item.price;
  }

  console.log("Total:", total); // Check output
  return total;
}
```

### The `debugger` Statement

```javascript
function buggyFunction(x) {
  debugger; // Execution pauses here when DevTools is open
  let result = x * 2;
  return result;
}
```

When DevTools is open and the engine hits `debugger`, it pauses execution and you can:

- Inspect all variables in scope.
- Step through code line by line.
- Watch expressions.
- View the call stack.

### VS Code Breakpoints

1. Click the red dot in the gutter (left of line numbers) to set a breakpoint.
2. Press `F5` to start debugging.
3. Execution pauses at the breakpoint.
4. Use the debug toolbar:
   - **Continue (F5)** — run until next breakpoint.
   - **Step Over (F10)** — execute current line, move to next.
   - **Step Into (F11)** — jump into the function being called.
   - **Step Out (Shift+F11)** — finish current function, return to caller.

### Chrome DevTools Debugger

1. Open DevTools → Sources tab.
2. Find your file in the file tree.
3. Click a line number to set a breakpoint.
4. Refresh the page — execution pauses at your breakpoint.
5. Use the same step controls as VS Code.

---

## Common Debugging Techniques

### Conditional Breakpoints

Right-click a breakpoint → "Edit Breakpoint" → add a condition:

```
item.price > 100
```

The breakpoint only triggers when the condition is true.

### Watch Expressions

Add variables or expressions to the Watch panel to monitor their values as you step through code.

### Network Tab

For API debugging — see every HTTP request, its status, headers, and response body.

### `typeof` for Type Issues

```javascript
function add(a, b) {
  console.log(typeof a, typeof b); // Quick type check
  return a + b;
}

add("5", 3); // "string" "number" → "53" (string concatenation, not addition!)
```

---

## Best Practices

1. **Remove `console.log` before committing** — use a linter rule to catch them.
2. **Learn your debugger** — breakpoints are faster than adding/removing log statements.
3. **Use descriptive log messages** — `console.log("here")` is useless; `console.log("User data after fetch:", userData)` tells you what and where.
4. **Install ESLint early** — it catches bugs before you even run the code.
5. **Use `"use strict"` in scripts** — catches common mistakes like undeclared variables.

---

## Summary

- Install Node.js LTS from nodejs.org — gives you `node` and `npm`.
- Use the browser console for quick experiments, Node.js REPL for server-side testing.
- VS Code with ESLint + Prettier is the standard JavaScript development setup.
- Learn the debugger (`F5` in VS Code, `debugger` statement, Chrome DevTools Sources) — it is vastly more efficient than `console.log` for complex issues.
- The `console` object has many useful methods beyond `.log`: `.table`, `.time`, `.group`, `.error`.
