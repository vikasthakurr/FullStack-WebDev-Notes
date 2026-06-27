# useState Hook

## What Is State in React?

State is data that belongs to a component and can change over time. When state changes, React **re-renders** the component to reflect the new data in the UI.

**Analogy:** Think of a scoreboard at a cricket match. The scoreboard (UI) displays the current score (state). Every time a run is scored, the state changes and the scoreboard updates. You don't rebuild the entire stadium — just the numbers on the board change.

---

## Why We Need State

| Without State (plain variables)             | With State (`useState`)                 |
| ------------------------------------------- | --------------------------------------- |
| Changing a variable does NOT re-render UI   | Calling the setter triggers a re-render |
| UI and data get out of sync                 | UI always reflects current state        |
| No way to "remember" values between renders | State persists across renders           |

```jsx
// ❌ This does NOT work — UI won't update
function Counter() {
  let count = 0;
  return <button onClick={() => count++}>Count: {count}</button>;
}

// ✅ This works — React re-renders on state change
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>Count: {count}</button>;
}
```

---

## useState Syntax

```jsx
import { useState } from "react";

const [value, setValue] = useState(initialValue);
```

- **`value`** — the current state value.
- **`setValue`** — the function to update the state.
- **`initialValue`** — the starting value (only used on first render).

The `useState` call uses **array destructuring**. You can name the pair anything:

```jsx
const [age, setAge] = useState(25);
const [name, setName] = useState("Vikas");
const [isVisible, setIsVisible] = useState(false);
const [items, setItems] = useState([]);
```

---

## Re-rendering on State Change

When you call the setter function, React:

1. Schedules a re-render of the component.
2. Calls the component function again.
3. `useState` returns the **new** value this time.
4. React diffs the Virtual DOM and patches the real DOM.

```jsx
function Toggle() {
  const [isOn, setIsOn] = useState(false);

  console.log("Component rendered! isOn =", isOn);

  return <button onClick={() => setIsOn(!isOn)}>{isOn ? "ON" : "OFF"}</button>;
}
// Every click logs "Component rendered!" with the new value
```

**Important:** State updates are **asynchronous** and **batched**. You don't get the new value immediately after calling the setter within the same function execution.

```jsx
function Example() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount(count + 1);
    console.log(count); // Still 0! (stale value in this closure)
  };

  return <button onClick={handleClick}>Count: {count}</button>;
}
```

---

## Updating State with Previous Value

When the new state depends on the previous state, use the **functional updater** form:

```jsx
// ❌ Risky — uses stale closure value
setCount(count + 1);

// ✅ Safe — always uses the latest state
setCount((prev) => prev + 1);
```

### Why This Matters

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  const incrementThree = () => {
    // ❌ All three read the same stale `count` — result: count + 1
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
    // Result: 1 (not 3!)

    // ✅ Each uses the latest state — result: count + 3
    setCount((prev) => prev + 1);
    setCount((prev) => prev + 1);
    setCount((prev) => prev + 1);
    // Result: 3
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={incrementThree}>+3</button>
    </div>
  );
}
```

**Rule of thumb:** If the new value depends on the old value, use the functional form `prev => newValue`.

---

## Multiple State Variables

Each piece of independent data gets its own `useState`:

```jsx
function UserForm() {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [age, setAge] = useState(0);
  const [isSubscribed, setIsSubscribed] = useState(false);

  return (
    <form>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <input
        type="number"
        value={age}
        onChange={(e) => setAge(Number(e.target.value))}
      />
      <label>
        <input
          type="checkbox"
          checked={isSubscribed}
          onChange={(e) => setIsSubscribed(e.target.checked)}
        />
        Subscribe to newsletter
      </label>
    </form>
  );
}
```

**When to group vs separate:**

- **Separate** — when values change independently.
- **Grouped (object)** — when values always change together or represent one entity.

---

## State with Objects and Arrays (Immutable Updates)

React uses **reference comparison** to detect changes. You must create a **new** object/array — never mutate the existing one.

### Objects

```jsx
function Profile() {
  const [user, setUser] = useState({
    name: "Vikas",
    age: 25,
    city: "Delhi",
  });

  // ❌ WRONG — mutating state directly
  const handleBirthday = () => {
    user.age += 1;
    setUser(user); // Same reference! React won't re-render
  };

  // ✅ CORRECT — create a new object with spread
  const handleBirthday = () => {
    setUser((prev) => ({
      ...prev,
      age: prev.age + 1,
    }));
  };

  // ✅ Update multiple fields
  const handleMove = () => {
    setUser((prev) => ({
      ...prev,
      city: "Mumbai",
      age: prev.age, // unchanged fields carry over from spread
    }));
  };

  return (
    <div>
      <p>
        {user.name}, {user.age}, {user.city}
      </p>
      <button onClick={handleBirthday}>Birthday</button>
    </div>
  );
}
```

### Arrays

```jsx
function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: "Learn React", done: false },
    { id: 2, text: "Build Project", done: false },
  ]);

  // Add item
  const addTodo = (text) => {
    setTodos((prev) => [...prev, { id: Date.now(), text, done: false }]);
  };

  // Remove item
  const removeTodo = (id) => {
    setTodos((prev) => prev.filter((todo) => todo.id !== id));
  };

  // Update item
  const toggleTodo = (id) => {
    setTodos((prev) =>
      prev.map((todo) =>
        todo.id === id ? { ...todo, done: !todo.done } : todo,
      ),
    );
  };

  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>
          <span style={{ textDecoration: todo.done ? "line-through" : "none" }}>
            {todo.text}
          </span>
          <button onClick={() => toggleTodo(todo.id)}>Toggle</button>
          <button onClick={() => removeTodo(todo.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

### Immutable Update Cheat Sheet

| Operation         | Immutable Pattern                                                 |
| ----------------- | ----------------------------------------------------------------- |
| Add to array      | `[...prev, newItem]`                                              |
| Remove from array | `prev.filter(item => item.id !== id)`                             |
| Update in array   | `prev.map(item => item.id === id ? {...item, ...changes} : item)` |
| Update object     | `{ ...prev, key: newValue }`                                      |
| Nested object     | `{ ...prev, nested: { ...prev.nested, key: val } }`               |

---

## Lazy Initialization

If the initial state requires an expensive computation, pass a **function** to `useState`:

```jsx
// ❌ Expensive function runs on EVERY render
const [data, setData] = useState(expensiveCalculation());

// ✅ Expensive function runs only on the FIRST render
const [data, setData] = useState(() => expensiveCalculation());
```

### Real-world Examples

```jsx
// Reading from localStorage
const [theme, setTheme] = useState(() => {
  return localStorage.getItem("theme") || "light";
});

// Parsing a large JSON
const [config, setConfig] = useState(() => {
  return JSON.parse(largeJsonString);
});

// Computing initial value from props
function FilteredList({ items }) {
  const [filtered, setFiltered] = useState(() => {
    return items.filter((item) => item.isActive);
  });
  // ...
}
```

---

## State Is Local to Each Component Instance

Each instance of a component has its **own** isolated state:

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount((prev) => prev + 1)}>Count: {count}</button>
  );
}

function App() {
  return (
    <div>
      <Counter /> {/* Has its own count: 0 */}
      <Counter /> {/* Has its own count: 0 */}
      <Counter /> {/* Has its own count: 0 */}
    </div>
  );
}
// Clicking one counter does NOT affect the others
```

If you need shared state between components, **lift state up** to a common parent:

```jsx
function App() {
  const [sharedCount, setSharedCount] = useState(0);

  return (
    <div>
      <Display count={sharedCount} />
      <Controls onIncrement={() => setSharedCount((prev) => prev + 1)} />
    </div>
  );
}
```

---

## Best Practices

1. **Use functional updates** when new state depends on previous state — avoids stale closure bugs.
2. **Keep state minimal** — derive computed values during render instead of storing them as separate state.
3. **Never mutate state directly** — always create new objects/arrays with spread or array methods.
4. **Colocate state** — keep state as close as possible to where it's used.
5. **Use descriptive names** — `[isModalOpen, setIsModalOpen]` is clearer than `[open, setOpen]`.
6. **Lazy initialize** expensive computations — pass a function to `useState`.
7. **Split unrelated state** into separate `useState` calls for clarity and independent updates.

---

## Common Mistakes

| Mistake                                                    | Why It's Wrong                                                                            |
| ---------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| Mutating state directly (`state.push(item)`)               | React doesn't detect the change — no re-render happens                                    |
| Reading state right after setting it                       | State updates are async — the old value is still in the closure                           |
| Using `setCount(count + 1)` multiple times                 | All reads from the same stale closure value — use `prev => prev + 1`                      |
| Storing derived data in state                              | Creates sync issues — compute it during render instead                                    |
| Forgetting that state resets on unmount                    | Component state is destroyed when removed from the tree                                   |
| Putting `useState` inside conditions/loops                 | Hooks must be called in the same order every render — breaks Rules of Hooks               |
| Not using lazy initialization for expensive initial values | The expensive function runs on every render even though its result is ignored after first |

---

## Summary

- **State** is component-owned data that triggers re-renders when updated.
- `useState` returns `[value, setter]` — call the setter to update and re-render.
- Use **functional updates** (`prev => prev + 1`) when new state depends on old state.
- Never **mutate** objects/arrays — create new ones with spread syntax.
- Use **lazy initialization** (`useState(() => ...)`) for expensive initial computations.
- Each component **instance** has isolated state — to share, lift state up.
- State updates are **asynchronous** and **batched** — don't rely on reading state immediately after setting it.
