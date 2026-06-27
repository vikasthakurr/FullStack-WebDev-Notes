# Custom Hooks

## What Are Custom Hooks?

Custom hooks are **JavaScript functions that extract and reuse stateful logic** from components. They let you share behavior (not UI) between components without duplicating code or using complex patterns like render props or higher-order components.

**Analogy:** Think of custom hooks as utility functions that can carry state. Regular utility functions are stateless helpers. Custom hooks are helpers that remember things between renders.

**Key rule:** Custom hooks must start with `use` — this is how React knows to apply the Rules of Hooks to them.

---

## Why Custom Hooks?

| Problem                                       | Custom Hook Solution                     |
| --------------------------------------------- | ---------------------------------------- |
| Same `useState` + `useEffect` in 5 components | Extract into one hook, import everywhere |
| Component is 200 lines of mixed logic         | Split into focused hooks for readability |
| Testing stateful logic tied to UI             | Test the hook independently              |
| Sharing logic without sharing UI              | Hooks return data, not JSX               |

---

## Rules of Hooks (Apply to Custom Hooks Too)

1. **Only call hooks at the top level** — never inside loops, conditions, or nested functions.
2. **Only call hooks from React functions** — components or other custom hooks.
3. **Name must start with `use`** — `useWhatever`. This triggers lint rules and React's hook detection.

---

## `useToggle` — Simple State Toggle

```jsx
import { useState, useCallback } from "react";

function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => setValue((v) => !v), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);

  return { value, toggle, setTrue, setFalse };
}

// Usage
function Navbar() {
  const { value: isMenuOpen, toggle: toggleMenu } = useToggle(false);

  return (
    <nav>
      <button onClick={toggleMenu}>{isMenuOpen ? "Close" : "Menu"}</button>
      {isMenuOpen && <MobileMenu />}
    </nav>
  );
}
```

---

## `useFetch` — Data Fetching

```jsx
import { useState, useEffect } from "react";

function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const abortController = new AbortController();

    async function fetchData() {
      setLoading(true);
      setError(null);

      try {
        const response = await fetch(url, { signal: abortController.signal });
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        const json = await response.json();
        setData(json);
      } catch (err) {
        if (err.name !== "AbortError") {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    }

    fetchData();
    return () => abortController.abort();
  }, [url]);

  return { data, loading, error };
}

// Usage
function UserProfile({ userId }) {
  const { data: user, loading, error } = useFetch(`/api/users/${userId}`);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;
  return <h1>{user.name}</h1>;
}
```

---

## `useLocalStorage` — Sync with localStorage

```jsx
import { useState, useEffect } from "react";

function useLocalStorage(key, initialValue) {
  // Initialize state from localStorage or fallback
  const [value, setValue] = useState(() => {
    try {
      const stored = localStorage.getItem(key);
      return stored ? JSON.parse(stored) : initialValue;
    } catch {
      return initialValue;
    }
  });

  // Sync to localStorage on change
  useEffect(() => {
    try {
      localStorage.setItem(key, JSON.stringify(value));
    } catch (err) {
      console.error("Failed to save to localStorage:", err);
    }
  }, [key, value]);

  return [value, setValue];
}

// Usage
function ThemeToggle() {
  const [theme, setTheme] = useLocalStorage("theme", "light");

  return (
    <button onClick={() => setTheme(theme === "light" ? "dark" : "light")}>
      Current: {theme}
    </button>
  );
}
```

---

## `useDebounce` — Delay Value Updates

```jsx
import { useState, useEffect } from "react";

function useDebounce(value, delay = 500) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// Usage — search input that waits for user to stop typing
function SearchBar() {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebounce(query, 300);
  const { data: results } = useFetch(
    debouncedQuery ? `/api/search?q=${debouncedQuery}` : null,
  );

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      {results?.map((item) => (
        <p key={item.id}>{item.name}</p>
      ))}
    </div>
  );
}
```

---

## `useWindowSize` — Track Viewport Dimensions

```jsx
import { useState, useEffect } from "react";

function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };

    window.addEventListener("resize", handleResize);
    return () => window.removeEventListener("resize", handleResize);
  }, []);

  return size;
}

// Usage
function ResponsiveLayout() {
  const { width } = useWindowSize();

  return (
    <div>
      {width > 768 ? <DesktopNav /> : <MobileNav />}
      <p>Current width: {width}px</p>
    </div>
  );
}
```

---

## `useOnClickOutside` — Detect Clicks Outside an Element

```jsx
import { useEffect } from "react";

function useOnClickOutside(ref, handler) {
  useEffect(() => {
    const listener = (event) => {
      if (!ref.current || ref.current.contains(event.target)) return;
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

// Usage — close dropdown when clicking outside
function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef(null);

  useOnClickOutside(dropdownRef, () => setIsOpen(false));

  return (
    <div ref={dropdownRef}>
      <button onClick={() => setIsOpen(!isOpen)}>Options</button>
      {isOpen && (
        <ul className="dropdown-menu">
          <li>Edit</li>
          <li>Delete</li>
        </ul>
      )}
    </div>
  );
}
```

---

## When to Extract a Custom Hook

Extract when you see:

- **Duplicated `useState` + `useEffect` combos** across components.
- **A component doing too many things** — split logic into named hooks for clarity.
- **Logic you want to test independently** — hooks can be unit tested without rendering.
- **A reusable behavior pattern** — anything 2+ components share.

Do NOT extract when:

- The logic is only used once and is simple.
- You are just wrapping a single `useState` for no benefit.
- The hook would need too many parameters (sign of wrong abstraction).

---

## Best Practices

1. **Start with `use` prefix** — `useAuth`, `useFetch`, `useCart`. Required for hook rules to apply.
2. **Return objects for multiple values** — `{ data, loading, error }` is clearer than array destructuring for 3+ values.
3. **Return arrays for simple pairs** — `[value, setValue]` mirrors `useState` convention.
4. **Include cleanup** — always return cleanup functions in `useEffect` within hooks.
5. **Accept configuration via parameters** — make hooks flexible: `useDebounce(value, delay)`.
6. **Keep hooks focused** — one hook, one responsibility. Compose multiple hooks for complex logic.
7. **Memoize callbacks** — use `useCallback` for returned functions to prevent unnecessary re-renders in consumers.
8. **Handle edge cases** — null URLs, SSR environments (`window` not available), unmounting during async operations.

---

## Common Mistakes

| Mistake                               | Why It's Wrong                                            | Fix                                          |
| ------------------------------------- | --------------------------------------------------------- | -------------------------------------------- |
| Not starting name with `use`          | React can't enforce hook rules, linter won't catch errors | Always prefix with `use`                     |
| Calling hooks conditionally           | Violates Rules of Hooks, causes bugs                      | Call at top level, use conditions inside     |
| Missing cleanup in effects            | Memory leaks, stale subscriptions                         | Return cleanup function from `useEffect`     |
| Not aborting fetches on unmount       | State updates on unmounted component                      | Use `AbortController` in fetch hooks         |
| Returning too many values as an array | Array position is hard to remember beyond 2 items         | Return an object for 3+ values               |
| Over-abstracting simple logic         | More indirection without benefit                          | Only extract when logic is reused or complex |
| Not memoizing returned callbacks      | Causes re-renders in consuming components                 | Wrap with `useCallback`                      |
| Using `window` without SSR check      | Breaks server-side rendering                              | Guard with `typeof window !== 'undefined'`   |

---

## Summary

- Custom hooks are functions prefixed with `use` that extract reusable stateful logic from components.
- They follow the same Rules of Hooks — only call at the top level, only from React functions.
- Common patterns: `useFetch` (data fetching), `useLocalStorage` (persistence), `useDebounce` (rate limiting), `useToggle` (boolean state), `useWindowSize` (viewport tracking).
- Return objects for complex data `{ data, loading, error }` and arrays for simple pairs `[value, setValue]`.
- Always include cleanup logic and handle edge cases (unmounting, SSR, abort signals).
- Extract a hook when logic is duplicated, complex, or independently testable — not just for the sake of abstraction.
