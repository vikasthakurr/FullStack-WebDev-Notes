# React.memo (Optimization)

## What React.memo Does

`React.memo` is a higher-order component that **skips re-rendering** a component if its props haven't changed. It performs a **shallow comparison** of the previous and current props.

**Analogy:** Think of a security guard at a building entrance. Every time someone arrives, the guard checks their ID badge. If it's the same person with the same badge (same props), the guard says "You're already cleared, go ahead" — no need to go through the full check-in process (render) again.

---

## The Problem: Unnecessary Re-renders

By default, when a parent re-renders, **all children re-render too** — even if their props haven't changed:

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>Count: {count}</button>
      {/* ❌ This re-renders on every count change, even though name never changes */}
      <ExpensiveChild name="Vikas" />
    </div>
  );
}

function ExpensiveChild({ name }) {
  console.log("ExpensiveChild rendered!"); // Logs every time parent renders
  // Imagine heavy computation or complex JSX here
  return <div>Hello, {name}!</div>;
}
```

---

## Wrapping a Component with React.memo

```jsx
import { memo } from "react";

const ExpensiveChild = memo(function ExpensiveChild({ name }) {
  console.log("ExpensiveChild rendered!");
  return <div>Hello, {name}!</div>;
});

// Now it only re-renders when `name` actually changes
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>Count: {count}</button>
      {/* ✅ Skips re-render because name="Vikas" hasn't changed */}
      <ExpensiveChild name="Vikas" />
    </div>
  );
}
```

### How Shallow Comparison Works

React.memo compares each prop with `Object.is()` (similar to `===`):

```jsx
// Primitives — compared by value
"Vikas" === "Vikas"  // true → skip re-render ✅
42 === 42            // true → skip re-render ✅
true === true        // true → skip re-render ✅

// Objects/Arrays/Functions — compared by REFERENCE
{ name: "Vikas" } === { name: "Vikas" }  // false → re-renders ❌
[1, 2, 3] === [1, 2, 3]                  // false → re-renders ❌
(() => {}) === (() => {})                  // false → re-renders ❌
```

This is why you need `useMemo` for objects/arrays and `useCallback` for functions passed to memoized components.

---

## Custom Comparison Function

For more control over when to re-render, provide a custom comparator as the second argument:

```jsx
const UserCard = memo(
  function UserCard({ user, theme }) {
    console.log("UserCard rendered");
    return (
      <div className={`card card-${theme}`}>
        <h2>{user.name}</h2>
        <p>{user.email}</p>
      </div>
    );
  },
  // Custom comparison: returns TRUE to SKIP re-render, FALSE to re-render
  (prevProps, nextProps) => {
    // Only re-render if user data actually changed (ignore theme changes)
    return (
      prevProps.user.name === nextProps.user.name &&
      prevProps.user.email === nextProps.user.email
    );
  },
);
```

### When Custom Comparison Helps

```jsx
// Skip re-render based on specific fields of a large object
const DataRow = memo(
  function DataRow({ row, onEdit, isSelected }) {
    return (
      <tr className={isSelected ? "selected" : ""}>
        <td>{row.name}</td>
        <td>{row.value}</td>
        <td>
          <button onClick={() => onEdit(row.id)}>Edit</button>
        </td>
      </tr>
    );
  },
  (prev, next) => {
    // Only care about these specific changes
    return (
      prev.row.name === next.row.name &&
      prev.row.value === next.row.value &&
      prev.isSelected === next.isSelected &&
      prev.onEdit === next.onEdit
    );
  },
);
```

**Warning:** Custom comparison functions can introduce bugs if you forget to compare a prop that affects output. Use them only when you have a clear reason.

---

## When to Use React.memo

### ✅ Use When:

1. **Component renders the same output for the same props** (pure component)
2. **Renders often with unchanged props** (parent re-renders frequently)
3. **Component is expensive to render** (large JSX, heavy calculations, many children)
4. **Rendering lists of items** (only changed items should re-render)

### ❌ Don't Use When:

1. **Props change on almost every render** — memo comparison runs but never skips
2. **Component is cheap to render** — memo overhead isn't worth it
3. **Component uses context** — context changes bypass memo anyway
4. **You haven't measured a performance problem** — premature optimization

---

## Combined Pattern: React.memo + useCallback + useMemo

The three optimization tools work as a team:

```jsx
import { useState, useCallback, useMemo, memo } from "react";

function ProductPage() {
  const [products, setProducts] = useState(initialProducts);
  const [searchTerm, setSearchTerm] = useState("");
  const [cartCount, setCartCount] = useState(0);

  // useMemo: cache the filtered list
  const filteredProducts = useMemo(() => {
    return products.filter((p) =>
      p.name.toLowerCase().includes(searchTerm.toLowerCase()),
    );
  }, [products, searchTerm]);

  // useCallback: stable function references
  const handleAddToCart = useCallback((productId) => {
    setCartCount((prev) => prev + 1);
    console.log("Added product:", productId);
  }, []);

  const handleRemoveProduct = useCallback((productId) => {
    setProducts((prev) => prev.filter((p) => p.id !== productId));
  }, []);

  return (
    <div>
      <header>
        <input
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
          placeholder="Search products..."
        />
        <span>Cart: {cartCount}</span>
      </header>

      {/* React.memo: ProductList skips re-render if its props haven't changed */}
      <ProductList
        products={filteredProducts}
        onAddToCart={handleAddToCart}
        onRemove={handleRemoveProduct}
      />
    </div>
  );
}

// React.memo on the list
const ProductList = memo(function ProductList({
  products,
  onAddToCart,
  onRemove,
}) {
  console.log("ProductList rendered");
  return (
    <div className="grid">
      {products.map((product) => (
        <ProductCard
          key={product.id}
          product={product}
          onAddToCart={onAddToCart}
          onRemove={onRemove}
        />
      ))}
    </div>
  );
});

// React.memo on individual items
const ProductCard = memo(function ProductCard({
  product,
  onAddToCart,
  onRemove,
}) {
  console.log(`ProductCard rendered: ${product.name}`);
  return (
    <div className="card">
      <h3>{product.name}</h3>
      <p>₹{product.price}</p>
      <button onClick={() => onAddToCart(product.id)}>Add to Cart</button>
      <button onClick={() => onRemove(product.id)}>Remove</button>
    </div>
  );
});
```

### How It All Fits Together

```
Parent re-renders (e.g., cartCount changes)
│
├── filteredProducts → useMemo → same reference (products/searchTerm unchanged)
├── handleAddToCart → useCallback → same reference (no deps changed)
├── handleRemoveProduct → useCallback → same reference (no deps changed)
│
└── ProductList (React.memo)
    ├── Compares props: products ✅ same, onAddToCart ✅ same, onRemove ✅ same
    └── SKIPS RE-RENDER ✅
```

---

## When NOT to Use React.memo

```jsx
// ❌ Props change every render anyway — memo comparison is wasted work
function Parent() {
  const data = fetchData(); // New object every render
  return <Child data={data} />;
}
const Child = memo(function Child({ data }) {
  return <div>{data.value}</div>;
});

// ❌ Component is extremely simple — rendering is cheaper than comparison
const SimpleLabel = memo(function SimpleLabel({ text }) {
  return <span>{text}</span>;
});

// ❌ Component reads from context — context changes bypass memo
const ThemedButton = memo(function ThemedButton({ onClick }) {
  const { theme } = useContext(ThemeContext); // Bypasses memo!
  return (
    <button className={theme} onClick={onClick}>
      Click
    </button>
  );
});
```

---

## Performance Profiling

Before adding `React.memo`, verify there's an actual problem:

```jsx
// Quick way to see if a component re-renders unnecessarily
function ExpensiveComponent(props) {
  console.log("ExpensiveComponent rendered at", performance.now());
  // ... component code
}

// Better: Use React DevTools Profiler
// 1. Open React DevTools → Profiler tab
// 2. Click Record, interact with the app, click Stop
// 3. Look for components that render often with "unchanged" props
// 4. Those are candidates for React.memo
```

### Measuring Impact

```jsx
// Wrap with memo and compare before/after
const OptimizedList = memo(function OptimizedList({ items, onSelect }) {
  console.time("OptimizedList render");
  const result = (
    <ul>
      {items.map((item) => (
        <li key={item.id} onClick={() => onSelect(item.id)}>
          {item.name}
        </li>
      ))}
    </ul>
  );
  console.timeEnd("OptimizedList render");
  return result;
});
```

---

## Best Practices

1. **Profile first** — don't wrap everything in memo. Find the bottleneck, then optimize.
2. **Memo the leaf components** in lists — individual items benefit most from skipping re-renders.
3. **Combine with useCallback and useMemo** — memo is useless if object/function props change every render.
4. **Keep memoized components pure** — same props should always produce the same output.
5. **Avoid custom comparison unless necessary** — default shallow compare works for most cases.
6. **Don't memo components that consume context** — context changes bypass memo and re-render anyway.
7. **Consider component splitting** instead of memo — extract the changing part into its own component.

---

## Common Mistakes

| Mistake                                                | Why It's Wrong                                                       |
| ------------------------------------------------------ | -------------------------------------------------------------------- |
| Using memo without stabilizing object/function props   | Props are new references every render — memo comparison always fails |
| Wrapping every single component in memo                | Adds memory overhead and comparison cost for no benefit              |
| Custom comparison that forgets to compare a prop       | Component shows stale data — subtle bugs                             |
| Expecting memo to work with context consumers          | Context changes bypass memo entirely                                 |
| Using memo on components that always receive new props | Comparison runs every render but never skips — pure overhead         |
| Not testing that memo actually helps                   | Assumption without measurement — might even make things slower       |

---

## Summary

- `React.memo` wraps a component to **skip re-renders** when props haven't changed (shallow comparison).
- It compares props by **reference** — primitives work naturally, but objects/functions need `useMemo`/`useCallback`.
- Use for **expensive components** that render often with stable props, especially **list items**.
- Provide a **custom comparison** only when you need to ignore specific props or compare deeply.
- The **full optimization trio**: `React.memo` (skip render) + `useCallback` (stable functions) + `useMemo` (stable values).
- **Don't overuse** — measure first with React DevTools Profiler. Optimization without measurement is guessing.
