# useRef Hook

## What Is useRef?

`useRef` is a hook that creates a **mutable reference** that persists across renders without causing re-renders when changed. It returns an object with a single `.current` property.

**Analogy:** Think of `useRef` as a sticky note you keep beside your desk. You can write on it, erase it, update it anytime — but updating your sticky note doesn't cause everyone in the office to stop and look (no re-render). It's your private, persistent scratch pad.

---

## Why useRef?

| Problem                                                                   | useRef Solution                                       |
| ------------------------------------------------------------------------- | ----------------------------------------------------- |
| Need to access a DOM element directly (focus, scroll, measure)            | Attach ref to element, access via `ref.current`       |
| Need a value that persists across renders but shouldn't trigger re-render | Store in `ref.current` — survives re-renders silently |
| Need to store previous state/props values                                 | Refs retain their value between renders               |
| Need to hold timer IDs, WebSocket instances, etc.                         | Ref keeps the reference alive without re-render       |

---

## useRef Syntax

```jsx
import { useRef } from "react";

const myRef = useRef(initialValue);

// Access or update:
myRef.current; // Read the value
myRef.current = newValue; // Write the value (no re-render!)
```

- `useRef(initialValue)` — creates `{ current: initialValue }`.
- Updating `.current` does **NOT** trigger a re-render.
- The same object reference persists for the entire component lifetime.

---

## Accessing DOM Elements

The most common use case — get a direct reference to a DOM node:

```jsx
import { useRef } from "react";

function TextInput() {
  const inputRef = useRef(null);

  const focusInput = () => {
    inputRef.current.focus(); // Direct DOM manipulation
  };

  const clearInput = () => {
    inputRef.current.value = "";
    inputRef.current.focus();
  };

  return (
    <div>
      <input ref={inputRef} type="text" placeholder="Type here..." />
      <button onClick={focusInput}>Focus Input</button>
      <button onClick={clearInput}>Clear & Focus</button>
    </div>
  );
}
```

### Scrolling to an Element

```jsx
function ScrollToSection() {
  const sectionRef = useRef(null);

  const scrollDown = () => {
    sectionRef.current.scrollIntoView({ behavior: "smooth" });
  };

  return (
    <div>
      <button onClick={scrollDown}>Scroll to Section</button>
      {/* ... lots of content ... */}
      <section ref={sectionRef}>
        <h2>Target Section</h2>
      </section>
    </div>
  );
}
```

### Measuring Element Size

```jsx
function MeasuredBox() {
  const boxRef = useRef(null);
  const [dimensions, setDimensions] = useState({ width: 0, height: 0 });

  const measure = () => {
    const rect = boxRef.current.getBoundingClientRect();
    setDimensions({ width: rect.width, height: rect.height });
  };

  return (
    <div>
      <div ref={boxRef} style={{ padding: "2rem", border: "1px solid" }}>
        Measure me!
      </div>
      <button onClick={measure}>Get Dimensions</button>
      <p>
        Width: {dimensions.width}px, Height: {dimensions.height}px
      </p>
    </div>
  );
}
```

### Video/Audio Controls

```jsx
function VideoPlayer({ src }) {
  const videoRef = useRef(null);

  return (
    <div>
      <video ref={videoRef} src={src} />
      <button onClick={() => videoRef.current.play()}>Play</button>
      <button onClick={() => videoRef.current.pause()}>Pause</button>
      <button onClick={() => (videoRef.current.currentTime = 0)}>Reset</button>
    </div>
  );
}
```

---

## Persisting Values Across Renders Without Re-render

Unlike state, updating a ref does **not** re-render the component. This makes it perfect for values you need to track silently.

### Counting Renders

```jsx
function RenderCounter() {
  const renderCount = useRef(0);

  renderCount.current += 1; // Increments every render, but doesn't CAUSE a render

  return <p>This component has rendered {renderCount.current} times</p>;
}
```

### Storing Timer IDs

```jsx
function Stopwatch() {
  const [time, setTime] = useState(0);
  const intervalRef = useRef(null); // Stores the interval ID

  const start = () => {
    if (intervalRef.current) return; // Already running
    intervalRef.current = setInterval(() => {
      setTime((prev) => prev + 1);
    }, 1000);
  };

  const stop = () => {
    clearInterval(intervalRef.current);
    intervalRef.current = null;
  };

  const reset = () => {
    stop();
    setTime(0);
  };

  // Cleanup on unmount
  useEffect(() => {
    return () => clearInterval(intervalRef.current);
  }, []);

  return (
    <div>
      <p>Time: {time}s</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

### Tracking Previous Value

```jsx
function usePrevious(value) {
  const prevRef = useRef();

  useEffect(() => {
    prevRef.current = value; // Update AFTER render
  }, [value]);

  return prevRef.current; // Returns the OLD value during this render
}

// Usage
function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCount}</p>
      <button onClick={() => setCount((prev) => prev + 1)}>Increment</button>
    </div>
  );
}
```

---

## useRef vs useState

| Feature                                | `useRef`                    | `useState`             |
| -------------------------------------- | --------------------------- | ---------------------- |
| Triggers re-render on update           | ❌ No                       | ✅ Yes                 |
| Persists across renders                | ✅ Yes                      | ✅ Yes                 |
| Synchronous access to latest value     | ✅ Yes (`ref.current`)      | ❌ No (async, batched) |
| Use for UI display                     | ❌ No (won't update screen) | ✅ Yes                 |
| Use for DOM access                     | ✅ Yes                      | ❌ No                  |
| Use for mutable values (timers, flags) | ✅ Yes                      | ❌ Overkill            |

### Decision Guide

```
Do you need the UI to update when this value changes?
├── YES → use useState
└── NO
    ├── Do you need to access a DOM element?
    │   └── YES → use useRef
    └── Do you need to persist a value between renders silently?
        └── YES → use useRef
```

---

## Common Use Cases

### 1. Focus Management

```jsx
function LoginForm() {
  const emailRef = useRef(null);
  const passwordRef = useRef(null);

  // Auto-focus email on mount
  useEffect(() => {
    emailRef.current.focus();
  }, []);

  const handleEmailKeyDown = (e) => {
    if (e.key === "Enter") {
      passwordRef.current.focus(); // Move to password on Enter
    }
  };

  return (
    <form>
      <input ref={emailRef} type="email" onKeyDown={handleEmailKeyDown} />
      <input ref={passwordRef} type="password" />
      <button type="submit">Login</button>
    </form>
  );
}
```

### 2. Storing Previous Values

```jsx
function PriceDisplay({ price }) {
  const prevPrice = useRef(price);

  useEffect(() => {
    prevPrice.current = price;
  }, [price]);

  const direction =
    price > prevPrice.current ? "📈" : price < prevPrice.current ? "📉" : "➡️";

  return (
    <p>
      {direction} ${price} (was ${prevPrice.current})
    </p>
  );
}
```

### 3. Debounced Callbacks with Latest Value

```jsx
function SearchInput({ onSearch }) {
  const [query, setQuery] = useState("");
  const timeoutRef = useRef(null);

  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);

    // Clear previous timeout
    clearTimeout(timeoutRef.current);

    // Set new timeout
    timeoutRef.current = setTimeout(() => {
      onSearch(value);
    }, 300);
  };

  return (
    <input value={query} onChange={handleChange} placeholder="Search..." />
  );
}
```

### 4. Uncontrolled Form with Ref

```jsx
function UncontrolledForm() {
  const nameRef = useRef(null);
  const emailRef = useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    const formData = {
      name: nameRef.current.value,
      email: emailRef.current.value,
    };
    console.log("Submitting:", formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input ref={nameRef} defaultValue="" placeholder="Name" />
      <input ref={emailRef} defaultValue="" placeholder="Email" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

## forwardRef — Passing Refs to Child Components

By default, you **cannot** pass a `ref` to a function component. `forwardRef` solves this:

```jsx
import { forwardRef, useRef } from "react";

// Child component wrapped with forwardRef
const CustomInput = forwardRef(function CustomInput({ label, ...props }, ref) {
  return (
    <label>
      {label}
      <input ref={ref} {...props} />
    </label>
  );
});

// Parent can now ref the child's DOM element
function Form() {
  const inputRef = useRef(null);

  const handleClick = () => {
    inputRef.current.focus(); // Accesses the <input> inside CustomInput
  };

  return (
    <div>
      <CustomInput ref={inputRef} label="Username" placeholder="Enter name" />
      <button onClick={handleClick}>Focus Input</button>
    </div>
  );
}
```

### useImperativeHandle — Exposing Limited API

Control what the parent can access through the ref:

```jsx
import { forwardRef, useRef, useImperativeHandle } from "react";

const FancyInput = forwardRef(function FancyInput(props, ref) {
  const inputRef = useRef(null);

  // Only expose specific methods — not the entire DOM node
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
    clear: () => {
      inputRef.current.value = "";
    },
    getValue: () => inputRef.current.value,
  }));

  return <input ref={inputRef} {...props} />;
});

// Parent usage
function Form() {
  const fancyRef = useRef(null);

  return (
    <div>
      <FancyInput ref={fancyRef} />
      <button onClick={() => fancyRef.current.focus()}>Focus</button>
      <button onClick={() => fancyRef.current.clear()}>Clear</button>
      <button onClick={() => alert(fancyRef.current.getValue())}>
        Get Value
      </button>
    </div>
  );
}
```

---

## Best Practices

1. **Use refs for DOM access and imperative operations** — focus, scroll, animate, measure.
2. **Don't use refs to replace state** when the value should be reflected in UI — the screen won't update.
3. **Use refs for "instance variables"** — timer IDs, previous values, flags that don't affect rendering.
4. **Prefer controlled components** (state) over uncontrolled (refs) for form inputs in most cases.
5. **Use `forwardRef`** when building reusable input/form components that parents may need to focus or control.
6. **Use `useImperativeHandle`** to limit what parents can do with the ref — encapsulation.
7. **Don't read/write refs during render** — do it in event handlers or effects. Reading during render makes behavior unpredictable.

---

## Common Mistakes

| Mistake                                                        | Why It's Wrong                                                        |
| -------------------------------------------------------------- | --------------------------------------------------------------------- |
| Expecting UI to update when changing `ref.current`             | Ref changes don't trigger re-renders — use state for displayed values |
| Passing `ref` as a regular prop to a function component        | Won't work — use `forwardRef`                                         |
| Reading `ref.current` during the initial render (before mount) | It's `null` until after the first render                              |
| Using refs as a workaround for stale closures                  | Overcomplicates code — fix the root cause (proper deps)               |
| Storing state-like data in refs to "avoid re-renders"          | Premature optimization — the UI becomes stale and buggy               |
| Not cleaning up timers/intervals stored in refs                | Memory leaks — always clean up in useEffect return                    |

---

## Summary

- `useRef` returns `{ current: value }` — a mutable container that **persists across renders** without triggering re-renders.
- Primary use: **accessing DOM elements** (`ref={myRef}`, then `myRef.current.focus()`).
- Secondary use: **storing mutable values** that shouldn't cause re-renders (timer IDs, previous values, flags).
- **useRef vs useState:** Use state when the UI needs to reflect the change; use ref when it's internal bookkeeping.
- Use **`forwardRef`** to pass refs to child components and **`useImperativeHandle`** to expose a limited API.
- Never rely on `ref.current` for rendering output — the screen won't update.
