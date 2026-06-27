# Custom Hooks

## What Are Custom Hooks?

A custom hook is a JavaScript function that **starts with `use`** and can call other React hooks inside it. Custom hooks let you extract reusable stateful logic from components without changing the component hierarchy.

**Analogy:** Think of custom hooks like kitchen appliances. A blender (useBlender) encapsulates the complex mechanics of blending — you just put stuff in and get a smoothie out. You don't need to understand the motor; you just use the interface. Similarly, `useFetch` encapsulates the complexity of fetching data — your component just gets `{ data, loading, error }`.

---

## Rules of Custom Hooks

1. **Must start with `use`** — this tells React (and linters) it's a hook that follows the Rules of Hooks.
2. **Can call other hooks** — `useState`, `useEffect`, `useRef`, other custom hooks, etc.
3. **Follow the Rules of Hooks** — can't be called conditionally, inside loops, or in nested functions.
4. **Each call gets its own state** — two components using the same custom hook don't share state.

```jsx
// ✅ Valid custom hook — starts with "use", calls hooks inside
function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  const increment = () => setCount((prev) => prev + 1);
  const decrement = () => setCount((prev) => prev - 1);
  const reset = () => setCount(initialValue);
  return { count, increment, decrement, reset };
}

// ❌ Not a hook — doesn't start with "use"
function getCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue); // ERROR: Hooks in non-hook function
  // ...
}
```

---

## Why Create Custom Hooks?

| Reason                          | Explanation                                                                       |
| ------------------------------- | --------------------------------------------------------------------------------- |
| **DRY (Don't Repeat Yourself)** | Extract logic used in multiple components into one place                          |
| **Separation of concerns**      | Keep components focused on UI — move data/logic to hooks                          |
| **Testability**                 | Test the hook in isolation without rendering components                           |
| **Readability**                 | Components read like a list of capabilities: `useFetch`, `useAuth`, etc.          |
| **Composability**               | Custom hooks can call other custom hooks — build complex logic from simple pieces |

---

## Example: useToggle

A simple boolean toggle — useful for modals, dropdowns, accordions:

```jsx
import { useState, useCallback } from "react";

function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => setValue((prev) => !prev), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);

  return { value, toggle, setTrue, setFalse };
}

// Usage
function Accordion() {
  const { value: isOpen, toggle } = useToggle(false);

  return (
    <div>
      <button onClick={toggle}>{isOpen ? "Close" : "Open"}</button>
      {isOpen && <p>Accordion content here...</p>}
    </div>
  );
}
```

---

## Example: useLocalStorage

Persist state to `localStorage` so it survives page reloads:

```jsx
import { useState, useEffect } from "react";

function useLocalStorage(key, initialValue) {
  // Lazy initialization — read from localStorage on first render
  const [value, setValue] = useState(() => {
    try {
      const stored = localStorage.getItem(key);
      return stored !== null ? JSON.parse(stored) : initialValue;
    } catch (error) {
      console.error("Error reading localStorage:", error);
      return initialValue;
    }
  });

  // Sync to localStorage whenever value changes
  useEffect(() => {
    try {
      localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error("Error writing to localStorage:", error);
    }
  }, [key, value]);

  return [value, setValue];
}

// Usage — same API as useState, but persisted!
function ThemeSwitcher() {
  const [theme, setTheme] = useLocalStorage("theme", "light");

  return (
    <button onClick={() => setTheme(theme === "light" ? "dark" : "light")}>
      Current: {theme}
    </button>
  );
}
```

---

## Example: useFetch

Data fetching with loading, error, and data states:

```jsx
import { useState, useEffect } from "react";

function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController(); // For cleanup

    const fetchData = async () => {
      setLoading(true);
      setError(null);

      try {
        const response = await fetch(url, { signal: controller.signal });
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const json = await response.json();
        setData(json);
      } catch (err) {
        if (err.name !== "AbortError") {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    };

    fetchData();

    // Cleanup — abort if component unmounts or url changes
    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}

// Usage
function UserList() {
  const { data: users, loading, error } = useFetch("/api/users");

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

---

## Example: useDebounce

Delay updating a value until the user stops changing it — perfect for search inputs:

```jsx
import { useState, useEffect } from "react";

function useDebounce(value, delay = 500) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    // Cleanup: if value changes again before delay, reset the timer
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// Usage
function SearchBar() {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebounce(query, 300);

  // This effect only runs when the user stops typing for 300ms
  useEffect(() => {
    if (debouncedQuery) {
      console.log("Searching for:", debouncedQuery);
      // fetch(`/api/search?q=${debouncedQuery}`)...
    }
  }, [debouncedQuery]);

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

---

## Example: useWindowSize

Track window dimensions for responsive logic in JS:

```jsx
import { useState, useEffect } from "react";

function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    };

    window.addEventListener("resize", handleResize);
    return () => window.removeEventListener("resize", handleResize);
  }, []);

  return size;
}

// Usage
function ResponsiveNav() {
  const { width } = useWindowSize();

  if (width < 768) {
    return <MobileMenu />;
  }

  return <DesktopNav />;
}
```

---

## Example: useOnClickOutside

Detect clicks outside an element — great for closing modals and dropdowns:

```jsx
import { useEffect, useRef } from "react";

function useOnClickOutside(ref, handler) {
  useEffect(() => {
    const listener = (event) => {
      // Do nothing if clicking inside the ref's element
      if (!ref.current || ref.current.contains(event.target)) {
        return;
      }
      handler(event);
    };

    document.addEventListener("mousedown", listener);
    document.addEventListener("touchstart", listener);

    return () => {
      document.removeEventListener("mousedown", listener);
      document.removeEventListener("touchstart", listener);
    };
  }, [ref, handler]);
}

// Usage
function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef(null);

  useOnClickOutside(dropdownRef, () => setIsOpen(false));

  return (
    <div ref={dropdownRef}>
      <button onClick={() => setIsOpen(!isOpen)}>Menu</button>
      {isOpen && (
        <ul className="dropdown-menu">
          <li>Option 1</li>
          <li>Option 2</li>
          <li>Option 3</li>
        </ul>
      )}
    </div>
  );
}
```

---

## Returning Values: Array vs Object Pattern

Custom hooks can return values in two common patterns:

### Array Pattern `[value, setter]`

```jsx
function useToggle(initial = false) {
  const [value, setValue] = useState(initial);
  const toggle = () => setValue((prev) => !prev);
  return [value, toggle]; // Array return
}

// Caller can rename freely
const [isOpen, toggleOpen] = useToggle(false);
const [isDark, toggleDark] = useToggle(true);
```

**Best for:** Simple hooks with 1–2 return values where callers will want to rename.

### Object Pattern `{ value, setter, ... }`

```jsx
function useFetch(url) {
  // ...
  return { data, loading, error, refetch }; // Object return
}

// Caller uses destructuring (can rename with `:`)
const { data, loading, error } = useFetch("/api/users");
const { data: posts, loading: postsLoading } = useFetch("/api/posts");
```

**Best for:** Hooks with 3+ return values where named access improves clarity.

---

## Custom Hooks vs Utility Functions

| Custom Hook                                     | Utility Function                           |
| ----------------------------------------------- | ------------------------------------------ |
| Starts with `use`                               | Any name (`formatDate`, `capitalize`)      |
| Can use `useState`, `useEffect`, `useRef`, etc. | Cannot call any hooks                      |
| Tied to React's render cycle                    | Pure computation, no side effects          |
| Returns reactive state that triggers re-renders | Returns static values                      |
| Example: `useFetch`, `useDebounce`              | Example: `formatCurrency`, `validateEmail` |

```jsx
// ✅ Utility function — no hooks needed, pure transformation
function formatCurrency(amount, currency = "INR") {
  return new Intl.NumberFormat("en-IN", { style: "currency", currency }).format(
    amount,
  );
}

// ✅ Custom hook — needs state and effects
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const goOnline = () => setIsOnline(true);
    const goOffline = () => setIsOnline(false);

    window.addEventListener("online", goOnline);
    window.addEventListener("offline", goOffline);

    return () => {
      window.removeEventListener("online", goOnline);
      window.removeEventListener("offline", goOffline);
    };
  }, []);

  return isOnline;
}
```

**Rule of thumb:** If the logic needs `useState`, `useEffect`, or any other hook — make it a custom hook. If it's just data transformation — make it a utility function.

---

## When to Extract a Custom Hook

Ask yourself these questions:

1. **Am I duplicating stateful logic** in multiple components? → Extract it.
2. **Is my component doing too much?** (fetching + transforming + animating) → Split responsibilities into hooks.
3. **Would this logic benefit from isolated testing?** → Extract and test the hook alone.
4. **Is the logic conceptually a "single thing"?** (e.g., "managing a form", "tracking mouse position") → It belongs in its own hook.

**Don't over-extract.** If logic is used in only one component and is simple, keep it inline. Extract when there's real reuse or the component becomes hard to follow.

---

## Best Practices

1. **Always prefix with `use`** — React's linter uses this to enforce Rules of Hooks.
2. **Keep hooks focused** — one responsibility per hook (SRP). `useFetch` fetches, `useLocalStorage` persists.
3. **Accept configuration as parameters** — make hooks flexible (`useDebounce(value, delay)`).
4. **Handle cleanup** — if your hook sets up listeners or timers, return a cleanup function in `useEffect`.
5. **Use `useCallback`/`useMemo` for returned functions** — prevents unnecessary re-renders in consuming components.
6. **Document the return type** — especially for hooks with object returns. Consider TypeScript for explicit contracts.
7. **Test hooks independently** — use `@testing-library/react-hooks` or `renderHook` from `@testing-library/react`.
8. **Don't prematurely abstract** — extract only when you see duplication or complexity, not "just in case."

---

## Common Mistakes

| Mistake                                           | Why It's Wrong                                                                 |
| ------------------------------------------------- | ------------------------------------------------------------------------------ |
| Not starting name with `use`                      | React can't enforce Rules of Hooks — linter won't catch errors                 |
| Forgetting cleanup in useEffect                   | Memory leaks — event listeners and timers pile up                              |
| Sharing state between hook consumers              | Each component calling the hook gets its own isolated state                    |
| Returning unstable references                     | Objects/functions recreated every render → infinite re-render loops downstream |
| Putting too much logic in one hook                | Becomes hard to test and reuse — split into smaller focused hooks              |
| Using a custom hook when a utility function works | Adds unnecessary React coupling — pure logic doesn't need hooks                |
| Not handling race conditions in async hooks       | Stale data if the component unmounts or deps change mid-fetch                  |

---

## Summary

- **Custom hooks** are functions starting with `use` that encapsulate reusable stateful logic.
- They **can call other hooks** (useState, useEffect, etc.) — utility functions cannot.
- Each component using a custom hook gets **its own isolated state**.
- Common patterns: `useLocalStorage`, `useFetch`, `useDebounce`, `useWindowSize`, `useOnClickOutside`, `useToggle`.
- Return **arrays** for simple hooks (easy renaming) and **objects** for complex hooks (named access).
- Extract a hook when you see **duplicated stateful logic**, a component doing too much, or logic that needs independent testing.
- Don't over-extract — only create custom hooks when there's a real benefit in reuse, separation, or clarity.
