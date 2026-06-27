# useCallback Hook

## Why Functions Cause Re-renders

In JavaScript, functions are **objects**. Every time a component re-renders, all functions inside it are **recreated** — with a new memory reference.

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  // This function is RECREATED on every render — new reference each time
  const handleClick = () => {
    console.log("clicked");
  };

  // Even though handleClick does the same thing, it's a NEW object reference
  // If Child is wrapped in React.memo, it still re-renders because props "changed"
  return <Child onClick={handleClick} />;
}
```

**Analogy:** Imagine you give your friend a phone number written on a new sticky note every day — even though the number is the same. Your friend sees a "new" sticky note each time and assumes something changed. `useCallback` is like laminating the note and reusing it — same note, same reference, nothing changed.

---

## The Problem in Detail

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState("");

  // ❌ New function reference on EVERY render
  const handleDelete = (id) => {
    console.log("Deleting", id);
  };

  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={() => setCount((c) => c + 1)}>Count: {count}</button>
      {/* ExpensiveList re-renders every time Parent renders
          because handleDelete is a new reference */}
      <ExpensiveList items={items} onDelete={handleDelete} />
    </div>
  );
}

const ExpensiveList = React.memo(function ExpensiveList({ items, onDelete }) {
  console.log("ExpensiveList rendered!"); // Logs on EVERY parent render
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>
          {item.name}
          <button onClick={() => onDelete(item.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
});
```

`React.memo` compares props with shallow equality. Since `handleDelete` is a new reference each time, `React.memo` sees it as "changed" and re-renders anyway.

---

## useCallback Syntax

```jsx
import { useCallback } from "react";

const memoizedFn = useCallback(() => {
  // function body
}, [dependencies]);
```

- **First argument:** The function you want to memoize.
- **Second argument:** Dependency array — React returns a new function only when dependencies change.
- **Returns:** The same function reference as long as dependencies haven't changed.

---

## useCallback with React.memo

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState("");
  const [items, setItems] = useState([
    { id: 1, name: "React" },
    { id: 2, name: "Vue" },
    { id: 3, name: "Angular" },
  ]);

  // ✅ Same function reference across renders (unless setItems changes — it won't)
  const handleDelete = useCallback((id) => {
    setItems((prev) => prev.filter((item) => item.id !== id));
  }, []); // setItems is stable, so empty deps is safe

  // ✅ Recreates only when text changes
  const handleSearch = useCallback(
    (query) => {
      console.log("Searching for:", query, "in context of:", text);
    },
    [text],
  );

  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={() => setCount((c) => c + 1)}>Count: {count}</button>
      {/* ✅ ExpensiveList does NOT re-render when count or text changes */}
      <ExpensiveList items={items} onDelete={handleDelete} />
    </div>
  );
}

const ExpensiveList = React.memo(function ExpensiveList({ items, onDelete }) {
  console.log("ExpensiveList rendered!");
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>
          {item.name}
          <button onClick={() => onDelete(item.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
});
```

Now `ExpensiveList` only re-renders when `items` actually changes — not on every keystroke in the input.

---

## useCallback vs useMemo

They're closely related — in fact, `useCallback` is syntactic sugar for a specific use of `useMemo`:

```jsx
// These are EQUIVALENT:
const memoizedFn = useCallback((a, b) => a + b, []);
const memoizedFn = useMemo(() => (a, b) => a + b, []);
```

| Feature          | `useCallback`                              | `useMemo`                            |
| ---------------- | ------------------------------------------ | ------------------------------------ |
| What it memoizes | A **function**                             | A **value** (result of computation)  |
| Returns          | The function itself                        | The return value of the function     |
| Use case         | Stable callback references for child props | Cached expensive computation results |
| Syntax sugar?    | Yes — it's `useMemo(() => fn, deps)`       | —                                    |

### When to Use Which

```jsx
// useCallback — memoize the function itself
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);

// useMemo — memoize the RESULT of calling a function
const sortedList = useMemo(() => {
  return [...items].sort((a, b) => a.name.localeCompare(b.name));
}, [items]);
```

---

## When to Use useCallback

### ✅ Use When:

1. **Passing callbacks to memoized children** (`React.memo` components)
2. **Passing callbacks as dependencies to useEffect** in children
3. **Custom hooks that return functions** consumed by optimized components

### ❌ Don't Use When:

1. **The child is NOT memoized** — useCallback without React.memo is pointless
2. **The function is only used locally** — no referential equality concern
3. **The component rarely re-renders** — optimization overhead > benefit
4. **The function has many dependencies that change often** — it recreates anyway

```jsx
// ❌ Pointless — Child is not wrapped in React.memo
function Parent() {
  const handleClick = useCallback(() => {
    console.log("clicked");
  }, []);

  return <Child onClick={handleClick} />; // Child re-renders anyway!
}

function Child({ onClick }) {
  return <button onClick={onClick}>Click</button>;
}
```

```jsx
// ❌ Pointless — function used locally, no child prop concern
function Counter() {
  const [count, setCount] = useState(0);

  // No benefit — this is just used on the button below
  const increment = useCallback(() => {
    setCount((c) => c + 1);
  }, []);

  return <button onClick={increment}>Count: {count}</button>;
}
```

---

## Example: Passing Callbacks to Optimized Children

### Todo App with Optimized Items

```jsx
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [newText, setNewText] = useState("");

  const addTodo = () => {
    setTodos((prev) => [
      ...prev,
      { id: Date.now(), text: newText, done: false },
    ]);
    setNewText("");
  };

  // ✅ Stable references — TodoItem won't re-render unnecessarily
  const toggleTodo = useCallback((id) => {
    setTodos((prev) =>
      prev.map((todo) =>
        todo.id === id ? { ...todo, done: !todo.done } : todo,
      ),
    );
  }, []);

  const deleteTodo = useCallback((id) => {
    setTodos((prev) => prev.filter((todo) => todo.id !== id));
  }, []);

  return (
    <div>
      <input value={newText} onChange={(e) => setNewText(e.target.value)} />
      <button onClick={addTodo}>Add</button>
      {/* Typing in the input re-renders Parent, but NOT individual TodoItems */}
      {todos.map((todo) => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={toggleTodo}
          onDelete={deleteTodo}
        />
      ))}
    </div>
  );
}

const TodoItem = React.memo(function TodoItem({ todo, onToggle, onDelete }) {
  console.log(`Rendering: ${todo.text}`);
  return (
    <div>
      <span
        style={{ textDecoration: todo.done ? "line-through" : "none" }}
        onClick={() => onToggle(todo.id)}
      >
        {todo.text}
      </span>
      <button onClick={() => onDelete(todo.id)}>×</button>
    </div>
  );
});
```

### Callbacks as useEffect Dependencies

```jsx
function SearchResults({ query, onResultsLoaded }) {
  // If parent doesn't memoize onResultsLoaded, this effect
  // runs on every parent render!
  useEffect(() => {
    fetchResults(query).then((results) => {
      onResultsLoaded(results);
    });
  }, [query, onResultsLoaded]); // onResultsLoaded is a dependency

  return <div>...</div>;
}

// Parent must use useCallback to prevent infinite effect loops
function Parent() {
  const [results, setResults] = useState([]);

  // ✅ Stable reference — child's useEffect won't re-run unnecessarily
  const handleResults = useCallback((data) => {
    setResults(data);
  }, []);

  return <SearchResults query="react" onResultsLoaded={handleResults} />;
}
```

---

## The Complete Optimization Pattern

```jsx
import { useState, useCallback, useMemo, memo } from "react";

function App() {
  const [filter, setFilter] = useState("");
  const [items, setItems] = useState(generateLargeList());
  const [selectedId, setSelectedId] = useState(null);

  // useMemo: expensive filtering
  const filteredItems = useMemo(() => {
    return items.filter((item) =>
      item.name.toLowerCase().includes(filter.toLowerCase()),
    );
  }, [items, filter]);

  // useCallback: stable callbacks for memoized children
  const handleSelect = useCallback((id) => {
    setSelectedId(id);
  }, []);

  const handleDelete = useCallback((id) => {
    setItems((prev) => prev.filter((item) => item.id !== id));
  }, []);

  return (
    <div>
      <input
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="Filter..."
      />
      <ItemList
        items={filteredItems}
        selectedId={selectedId}
        onSelect={handleSelect}
        onDelete={handleDelete}
      />
    </div>
  );
}

// React.memo: skip render if props unchanged
const ItemList = memo(function ItemList({
  items,
  selectedId,
  onSelect,
  onDelete,
}) {
  return (
    <ul>
      {items.map((item) => (
        <ItemRow
          key={item.id}
          item={item}
          isSelected={item.id === selectedId}
          onSelect={onSelect}
          onDelete={onDelete}
        />
      ))}
    </ul>
  );
});

const ItemRow = memo(function ItemRow({
  item,
  isSelected,
  onSelect,
  onDelete,
}) {
  return (
    <li className={isSelected ? "selected" : ""}>
      <span onClick={() => onSelect(item.id)}>{item.name}</span>
      <button onClick={() => onDelete(item.id)}>×</button>
    </li>
  );
});
```

---

## Best Practices

1. **Always pair with React.memo** — `useCallback` without a memoized consumer is wasted effort.
2. **Use functional state updates** inside callbacks — avoids adding state as a dependency.
3. **Keep dependency arrays honest** — include all values used inside the callback.
4. **Profile first** — don't add useCallback everywhere "just in case." Measure with React DevTools.
5. **Prefer useCallback for callbacks passed as props** — especially to list items that render many times.
6. **Don't wrap every function** — only those that cause measurable re-render problems.

---

## Common Mistakes

| Mistake                                                                | Why It's Wrong                                             |
| ---------------------------------------------------------------------- | ---------------------------------------------------------- |
| Using useCallback without React.memo on the child                      | The child re-renders regardless — useCallback does nothing |
| Adding too many dependencies that change often                         | Function recreates anyway — no benefit                     |
| Wrapping every single function in useCallback                          | Adds memory and complexity with no performance gain        |
| Using useCallback for inline event handlers with no child prop concern | Premature optimization — just use the inline function      |
| Forgetting that useCallback memoizes the function, not the result      | Use useMemo for cached computation results                 |
| Not including needed dependencies                                      | Stale closure — function captures outdated values          |

---

## Summary

- Functions in React components are **recreated** on every render with new references.
- `useCallback(fn, [deps])` returns the **same function reference** as long as dependencies haven't changed.
- Its main purpose: **prevent unnecessary re-renders** of `React.memo` children that receive callbacks as props.
- **useCallback = function identity.** useMemo = value caching. They solve different (but related) problems.
- Only use when passing callbacks to **memoized children** or as **effect dependencies** — otherwise it's overhead with no benefit.
- The full optimization pattern: `React.memo` (child) + `useCallback` (functions) + `useMemo` (values).
