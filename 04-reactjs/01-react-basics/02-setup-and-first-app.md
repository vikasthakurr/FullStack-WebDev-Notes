# Setting Up React with Vite & First App

## Why Vite?

Vite (French for "fast") is the modern build tool for React projects. It replaced Create React App (CRA) as the recommended setup.

| Feature           | Vite                       | CRA (deprecated)           |
| ----------------- | -------------------------- | -------------------------- |
| Dev server start  | ~200ms (instant)           | ~10–30 seconds             |
| Hot Module Reload | Instant (ESM-based)        | Slow (full bundle rebuild) |
| Build tool        | Rollup (optimized)         | Webpack (heavy)            |
| Config            | Minimal (`vite.config.js`) | Hidden (ejecting needed)   |
| Maintained        | Actively developed         | No longer recommended      |

---

## Creating a New React Project

```bash
# JavaScript template
npm create vite@latest my-app -- --template react

# TypeScript template
npm create vite@latest my-app -- --template react-ts

# Navigate and install
cd my-app
npm install

# Start dev server
npm run dev
```

Dev server runs at `http://localhost:5173` by default.

---

## Project Structure

```
my-app/
├── public/                 # Static assets (served as-is)
│   └── vite.svg
├── src/
│   ├── App.jsx            # Root component
│   ├── App.css            # Component styles
│   ├── main.jsx           # Entry point — renders App into DOM
│   ├── index.css          # Global styles
│   └── assets/            # Images, fonts (processed by Vite)
├── index.html             # Single HTML file (entry for Vite)
├── package.json           # Dependencies and scripts
├── vite.config.js         # Vite configuration
└── .eslintrc.cjs          # Linting rules
```

---

## Understanding the Entry Point

### `index.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>My React App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

### `src/main.jsx`

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App.jsx";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
);
```

- `ReactDOM.createRoot()` — creates the React root on the `#root` div.
- `<React.StrictMode>` — highlights potential problems during development (double-invokes effects).
- `<App />` — your root component.

---

## Your First Component

### `src/App.jsx`

```jsx
import "./App.css";

function App() {
  return (
    <div className="App">
      <h1>Hello, React!</h1>
      <p>This is my first React app with Vite.</p>
    </div>
  );
}

export default App;
```

---

## Components

Components are the building blocks of React — reusable, self-contained pieces of UI.

### Function Components (Modern Standard)

```jsx
// Named function
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>
}

// Arrow function
const Greeting = ({ name }) => <h1>Hello, {name}!</h1>

// Usage
<Greeting name="Vikas" />
```

### JSX Rules

```jsx
function Demo() {
  const isLoggedIn = true;
  const items = ["React", "Vue", "Angular"];

  return (
    // 1. Single root element (or Fragment <>...</>)
    <div>
      {/* 2. JS expressions in curly braces */}
      <h1>{isLoggedIn ? "Welcome!" : "Log in"}</h1>

      {/* 3. className not class */}
      <div className="container">
        {/* 4. All tags must close */}
        <img src="logo.png" alt="Logo" />
        <br />

        {/* 5. Rendering lists with key */}
        <ul>
          {items.map((item, i) => (
            <li key={i}>{item}</li>
          ))}
        </ul>

        {/* 6. Conditional rendering */}
        {isLoggedIn && <button>Logout</button>}
      </div>
    </div>
  );
}
```

---

## Event Handlers

```jsx
function Button() {
  // Event handler function
  const handleClick = (e) => {
    console.log("Clicked!", e.target);
  };

  const handleInput = (e) => {
    console.log("Typed:", e.target.value);
  };

  return (
    <div>
      {/* Pass function reference, don't call it */}
      <button onClick={handleClick}>Click Me</button>

      {/* Inline handler */}
      <button onClick={() => alert("Hi!")}>Alert</button>

      {/* Input events */}
      <input onChange={handleInput} placeholder="Type..." />
    </div>
  );
}
```

**Common mistake:** `onClick={handleClick()}` ← this CALLS it immediately. Use `onClick={handleClick}`.

---

## Conditional Rendering

```jsx
function Dashboard({ user, notifications }) {
  // 1. If/else (outside JSX)
  if (!user) {
    return <LoginPage />;
  }

  return (
    <div>
      {/* 2. Ternary */}
      <h1>{user.isAdmin ? "Admin Dashboard" : "User Dashboard"}</h1>

      {/* 3. Logical AND (short-circuit) */}
      {notifications.length > 0 && (
        <span className="badge">{notifications.length}</span>
      )}

      {/* 4. Nullish — render nothing */}
      {user.avatar && <img src={user.avatar} alt="Avatar" />}
    </div>
  );
}
```

---

## Available Scripts

```json
{
  "scripts": {
    "dev": "vite", // Start dev server
    "build": "vite build", // Production build → dist/
    "preview": "vite preview" // Preview production build locally
  }
}
```

---

## Summary

- Use **Vite** to create React projects — fast dev server, instant HMR, optimized builds.
- `main.jsx` is the entry point — it renders `<App />` into the `#root` DOM element.
- **Components** are functions that return JSX — they are reusable and composable.
- **JSX** uses `className`, requires closing tags, and evaluates JS inside `{}`.
- **Event handlers** are passed as references (`onClick={fn}`) — never call them inline.
- Use **ternary**, **&&**, or **early returns** for conditional rendering.
