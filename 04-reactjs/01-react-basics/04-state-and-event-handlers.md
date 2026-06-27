# State and Event Handlers

## What Is State in React?

State is data that belongs to a component and can **change over time**. When state changes, React re-renders the component so the UI always reflects the current data.

**Analogy:** Imagine a light switch. The switch position (on/off) is the state. Flipping the switch is an event. The room (UI) updates based on the new state.

State in React is managed via the `useState` hook (and other hooks for complex scenarios). For a deep dive into `useState`, see [05-useState-hook.md](./05-useState-hook.md).

Key points:

- State is **local** to a component instance.
- Updating state triggers a **re-render**.
- State persists across renders (unlike plain variables).

---

## Event Handlers in React

Event handlers are functions that run in response to user interactions — clicks, key presses, form submissions, etc.

---

### Passing Handler Functions (Reference, Not Calling)

In React, you pass a **reference** to the function, not the result of calling it:

```jsx
function App() {
  const handleClick = () => {
    console.log("Button clicked!");
  };

  // ✅ Correct — passing a reference
  return <button onClick={handleClick}>Click Me</button>;

  // ❌ Wrong — this CALLS the function immediately on render
  return <button onClick={handleClick()}>Click Me</button>;
}
```

**Why?** `handleClick()` executes instantly during render and passes the **return value** (likely `undefined`) as the handler. `handleClick` (no parentheses) passes the function itself, so React can call it later when the event fires.

---

### SyntheticEvent Wrapper

React wraps the browser's native event in a **SyntheticEvent**. This provides a consistent, cross-browser interface.

```jsx
function App() {
  const handleClick = (event) => {
    console.log(event); // SyntheticEvent object
    console.log(event.nativeEvent); // The underlying browser event
    console.log(event.type); // "click"
    console.log(event.target); // The DOM element that was clicked
  };

  return <button onClick={handleClick}>Click</button>;
}
```

**Key characteristics:**

- Same API across all browsers (no more IE quirks).
- Pooled for performance (in React 16 and earlier; React 17+ no longer pools).
- Access the native event via `event.nativeEvent` if needed.

---

### Common Events

| Event          | Fires When               | Typical Element      |
| -------------- | ------------------------ | -------------------- |
| `onClick`      | Element is clicked       | Buttons, divs, links |
| `onChange`     | Input value changes      | Inputs, selects      |
| `onSubmit`     | Form is submitted        | Forms                |
| `onKeyDown`    | A key is pressed down    | Inputs, document     |
| `onKeyUp`      | A key is released        | Inputs, document     |
| `onFocus`      | Element gains focus      | Inputs               |
| `onBlur`       | Element loses focus      | Inputs               |
| `onMouseEnter` | Mouse enters the element | Any                  |
| `onMouseLeave` | Mouse leaves the element | Any                  |

```jsx
function FormExample() {
  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        console.log("submitted");
      }}
    >
      <input
        onChange={(e) => console.log("Value:", e.target.value)}
        onFocus={() => console.log("Input focused")}
        onBlur={() => console.log("Input blurred")}
        onKeyDown={(e) => console.log("Key:", e.key)}
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

### Passing Arguments to Event Handlers

Often you need to pass additional data (like an item's ID) to a handler. Use an **arrow function wrapper**:

```jsx
function TodoList() {
  const todos = [
    { id: 1, text: "Learn React" },
    { id: 2, text: "Build App" },
  ];

  const handleDelete = (id) => {
    console.log("Deleting todo:", id);
  };

  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>
          {todo.text}
          {/* ✅ Arrow function wraps the call with the argument */}
          <button onClick={() => handleDelete(todo.id)}>Delete</button>

          {/* ❌ Wrong — calls immediately during render */}
          {/* <button onClick={handleDelete(todo.id)}>Delete</button> */}
        </li>
      ))}
    </ul>
  );
}
```

If you also need the event object:

```jsx
<button onClick={(e) => handleDelete(todo.id, e)}>Delete</button>
```

---

### Inline vs Named Handlers

**Inline handlers** — defined directly in JSX:

```jsx
<button onClick={() => setCount(count + 1)}>+1</button>
```

**Named handlers** — defined as separate functions:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  const increment = () => {
    setCount((prev) => prev + 1);
  };

  return <button onClick={increment}>Count: {count}</button>;
}
```

**When to use which:**

| Inline                         | Named                                   |
| ------------------------------ | --------------------------------------- |
| Simple one-liners              | Complex logic (multiple lines)          |
| No need to reuse               | Reused in multiple places               |
| Quick prototyping              | Easier to debug (named in stack traces) |
| Can clutter JSX if logic grows | Keeps JSX clean and readable            |

**Rule of thumb:** If the handler is more than one simple expression, extract it into a named function above the return.

---

### Preventing Default Behavior

Some elements have default browser behavior (forms submit and reload the page, links navigate). Use `e.preventDefault()` to stop it:

```jsx
function LoginForm() {
  const [email, setEmail] = useState("");

  const handleSubmit = (e) => {
    e.preventDefault(); // ⛔ Stops page reload
    console.log("Submitting:", email);
    // ...API call
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

Similarly for links:

```jsx
<a
  href="/old-page"
  onClick={(e) => {
    e.preventDefault();
    navigate("/new-page"); // Use React Router instead
  }}
>
  Go somewhere
</a>
```

---

### Form Handling Pattern (Controlled Components)

In a **controlled component**, React state is the "single source of truth" for the input value. The input displays state, and `onChange` updates state.

```jsx
function SignupForm() {
  const [formData, setFormData] = useState({
    username: "",
    email: "",
    password: "",
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData((prev) => ({
      ...prev,
      [name]: value, // Computed property name — updates the right field
    }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log("Form submitted:", formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="username"
        value={formData.username}
        onChange={handleChange}
        placeholder="Username"
      />
      <input
        name="email"
        type="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <input
        name="password"
        type="password"
        value={formData.password}
        onChange={handleChange}
        placeholder="Password"
      />
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

**Why controlled?**

- You can validate on every keystroke.
- You can conditionally disable the submit button.
- You can format/transform input as the user types.
- Form data is always in sync with state.

---

## Conditional Rendering

React lets you render different UI based on conditions. There are several patterns:

---

### if/else Outside JSX

Use regular `if/else` **before** the return statement:

```jsx
function Greeting({ isLoggedIn }) {
  if (isLoggedIn) {
    return <h1>Welcome back!</h1>;
  } else {
    return <h1>Please sign in.</h1>;
  }
}
```

Or assign to a variable:

```jsx
function Greeting({ isLoggedIn }) {
  let content;

  if (isLoggedIn) {
    content = <h1>Welcome back!</h1>;
  } else {
    content = <h1>Please sign in.</h1>;
  }

  return <div>{content}</div>;
}
```

---

### Ternary Operator

Best for choosing **between two** things inline:

```jsx
function UserStatus({ isOnline }) {
  return (
    <div>
      <span className={isOnline ? "status-green" : "status-gray"}>
        {isOnline ? "Online" : "Offline"}
      </span>
    </div>
  );
}
```

**Avoid nesting** ternaries — they become unreadable fast:

```jsx
// ❌ Hard to read
{
  status === "loading" ? (
    <Spinner />
  ) : status === "error" ? (
    <Error />
  ) : (
    <Data />
  );
}

// ✅ Better — use if/else or early returns
```

---

### Logical AND (&&)

Render something **only when** a condition is true (nothing otherwise):

```jsx
function Notifications({ count }) {
  return (
    <div>
      <h1>Dashboard</h1>
      {count > 0 && <span className="badge">{count} new notifications</span>}
    </div>
  );
}
```

**⚠️ Gotcha with falsy values:**

```jsx
// ❌ Bug — renders "0" on screen when count is 0
{
  count && <span>{count} items</span>;
}

// ✅ Fix — explicitly check for a boolean condition
{
  count > 0 && <span>{count} items</span>;
}
```

This happens because `0` is falsy but React renders the number `0` as text. Always ensure the left side of `&&` is a true boolean expression.

---

### Early Return Pattern

Return early for edge cases, then render the main UI:

```jsx
function UserProfile({ user, isLoading, error }) {
  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage message={error} />;
  if (!user) return <p>No user found.</p>;

  // Main render — only reached if no edge case matched
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

This keeps your code flat and avoids deeply nested ternaries.

---

### Rendering null

Return `null` to render nothing:

```jsx
function WarningBanner({ show, message }) {
  if (!show) {
    return null; // Component produces no output
  }

  return <div className="warning">{message}</div>;
}
```

Returning `null` still means the component **exists** in the tree (lifecycle methods/effects run), it just doesn't produce any DOM output.

---

## Best Practices

1. **Pass function references** — write `onClick={handleClick}`, not `onClick={handleClick()}`.
2. **Use named handlers** for complex logic — keeps JSX clean and stack traces readable.
3. **Always `e.preventDefault()`** in form `onSubmit` — unless you intentionally want a page reload.
4. **Prefer controlled components** — React state as the single source of truth for form inputs.
5. **Use early returns** for loading/error states — avoids deeply nested conditionals.
6. **Beware of `&&` with numbers** — `0 && <Component />` renders `0`. Use `> 0` or `!!value`.
7. **Extract complex conditional logic** into helper functions or sub-components for readability.
8. **Keep event handlers close** to the JSX that uses them — colocate for maintainability.

---

## Common Mistakes

| Mistake                                        | Why It's Wrong                                                                |
| ---------------------------------------------- | ----------------------------------------------------------------------------- |
| `onClick={handleClick()}`                      | Calls the function immediately on render instead of on click                  |
| Forgetting `e.preventDefault()` in form submit | Page reloads and you lose all React state                                     |
| `{count && <Badge />}` when count can be 0     | Renders the number `0` instead of nothing                                     |
| Nested ternaries 3+ levels deep                | Unreadable — use if/else or early returns instead                             |
| Mutating state in event handlers               | React won't detect the change — always use the setter with a new value        |
| Not passing arguments correctly                | `onClick={handleDelete(id)}` calls immediately — use `() => handleDelete(id)` |
| Using index as the only logic for conditionals | Can lead to off-by-one bugs — prefer meaningful boolean conditions            |

---

## Summary

- **State** is component data that triggers re-renders on change — see [05-useState-hook.md](./05-useState-hook.md) for the full guide.
- Pass event handlers as **references** (`handleClick`), not calls (`handleClick()`).
- React wraps native events in **SyntheticEvent** for cross-browser consistency.
- Use **arrow wrappers** to pass arguments: `onClick={() => handler(id)}`.
- Prefer **named handlers** for anything beyond a one-liner.
- Use `e.preventDefault()` to stop default browser behavior in forms and links.
- **Controlled components** keep form state in React for full control over input behavior.
- **Conditional rendering** techniques: if/else, ternary, `&&`, early return, and returning `null`.
- Watch out for the `&&` gotcha with falsy numbers — always use boolean expressions.
