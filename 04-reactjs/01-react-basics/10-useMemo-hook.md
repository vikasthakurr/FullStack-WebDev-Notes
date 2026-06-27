# useMemo Hook

## What Is Memoization?

Memoization is a performance optimization technique that **caches the result** of an expensive computation and returns the cached result when the same inputs occur again, instead of recalculating.

**Analogy:** Imagine you're asked "What's 847 × 293?" You calculate it once: 248,171. If someone asks the same question again 5 minutes later, you don't redo the multiplication — you just recall the answer. That's memoization. You only recalculate when the numbers change.

---

## Why useMemo?

In React, component functions re-run on every render. Without memoization:

```jsx
function ProductList({ products, filterText }) {
  // ❌ This runs on EVERY render — even when only unrelated state changes
  const filtered = products.filter((p) =>
    p.name.toLowerCase().includes(filterText.toLowerCase()),
  );

  return (
    <ul>
      {filtered.map((p) => (
        <li key={p.id}>{p.name}</li>
      ))}
    </ul>
  );
}
```

If the parent re-renders for any reason (typing in a different input, toggling a modal), this expensive filter runs again even though `products` and `filterText` haven't changed.

---

## useMemo Syntax

```jsx
import { useMemo } from "react";

const memoizedValue = useMemo(() => {
  return expensiveCalculation(a, b);
}, [a, b]); // Only recalculates when a or b change
```

- **First argument:** A function that returns the computed value.
- **Second argument:** Dependency array — React recalculates only when dependencies change.
- **Returns:** The cached (memoized) value.

---

## When to Use useMemo

### 1. Expensive Calculations

```jsx
function ProductList({ products, filterText, sortBy }) {
  // ✅ Only recalculates when products, filterText, or sortBy change
  const filteredAndSorted = useMemo(() => {
    console.log("Filtering and sorting..."); // See when it runs

    const filtered = products.filter((p) =>
      p.name.toLowerCase().includes(filterText.toLowerCase()),
    );

    return filtered.sort((a, b) => {
      if (sortBy === "price") return a.price - b.price;
      if (sortBy === "name") return a.name.localeCompare(b.name);
      return 0;
    });
  }, [products, filterText, sortBy]);

  return (
    <ul>
      {filteredAndSorted.map((product) => (
        <li key={product.id}>
          {product.name} — ₹{product.price}
        </li>
      ))}
    </ul>
  );
}
```

### 2. Referential Equality for Objects/Arrays

When passing objects or arrays to child components wrapped in `React.memo`, you need stable references:

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const [query, setQuery] = useState("");

  // ❌ New object on every render — causes child to re-render
  const options = { threshold: 10, limit: 50 };

  // ✅ Same object reference unless dependencies change
  const options = useMemo(
    () => ({
      threshold: 10,
      limit: 50,
    }),
    [],
  ); // Empty deps = created once, never changes

  // ✅ Only recalculates when query changes
  const searchConfig = useMemo(
    () => ({
      query: query.trim().toLowerCase(),
      caseSensitive: false,
    }),
    [query],
  );

  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>Count: {count}</button>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <ExpensiveChild options={options} config={searchConfig} />
    </div>
  );
}

const ExpensiveChild = React.memo(function ExpensiveChild({ options, config }) {
  console.log("ExpensiveChild rendered");
  return <div>...</div>;
});
```

### 3. Expensive Derived Data

```jsx
function Dashboard({ transactions }) {
  const stats = useMemo(() => {
    const total = transactions.reduce((sum, t) => sum + t.amount, 0);
    const average = total / transactions.length;
    const max = Math.max(...transactions.map((t) => t.amount));
    const min = Math.min(...transactions.map((t) => t.amount));
    const categories = [...new Set(transactions.map((t) => t.category))];

    return { total, average, max, min, categories };
  }, [transactions]);

  return (
    <div>
      <p>Total: ₹{stats.total}</p>
      <p>Average: ₹{stats.average.toFixed(2)}</p>
      <p>
        Range: ₹{stats.min} – ₹{stats.max}
      </p>
      <p>Categories: {stats.categories.join(", ")}</p>
    </div>
  );
}
```

---

## When NOT to Use useMemo (Premature Optimization)

`useMemo` itself has a cost — React must store the previous value, compare dependencies, and manage the cache. Don't use it for:

```jsx
// ❌ Simple calculations — useMemo overhead > calculation cost
const doubled = useMemo(() => count * 2, [count]);
// ✅ Just compute it
const doubled = count * 2;

// ❌ Simple string concatenation
const fullName = useMemo(() => `${first} ${last}`, [first, last]);
// ✅ Just compute it
const fullName = `${first} ${last}`;

// ❌ When the component rarely re-renders anyway
const formatted = useMemo(() => formatDate(date), [date]);
// ✅ If the parent only renders once or twice, just compute it
const formatted = formatDate(date);
```

### Decision Guide

```
Is this calculation actually slow (>1ms)?
├── NO → Don't use useMemo
└── YES
    └── Does this component re-render frequently with unchanged deps?
        ├── NO → Probably don't need useMemo
        └── YES → ✅ Use useMemo

Are you passing an object/array to a React.memo child?
├── NO → Don't need useMemo for referential equality
└── YES
    └── Does the child actually benefit from skipping re-renders?
        ├── NO → Don't bother
        └── YES → ✅ Use useMemo
```

---

## useMemo vs useCallback vs React.memo

| Tool          | What It Memoizes                                     | Purpose                                           |
| ------------- | ---------------------------------------------------- | ------------------------------------------------- |
| `useMemo`     | A **computed value** (number, string, object, array) | Skip expensive recalculations                     |
| `useCallback` | A **function reference**                             | Maintain stable function identity for child props |
| `React.memo`  | A **component's render output**                      | Skip re-rendering when props haven't changed      |

### How They Work Together

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([1, 2, 3, 4, 5]);

  // useMemo: expensive computation cached
  const total = useMemo(() => {
    return items.reduce((sum, item) => sum + item, 0);
  }, [items]);

  // useCallback: stable function reference
  const handleDelete = useCallback((id) => {
    setItems((prev) => prev.filter((item) => item !== id));
  }, []);

  return (
    <div>
      <p>Total: {total}</p>
      <button onClick={() => setCount((c) => c + 1)}>Count: {count}</button>
      {/* React.memo child won't re-render when count changes */}
      <ItemList items={items} onDelete={handleDelete} />
    </div>
  );
}

// React.memo: skip render if props unchanged
const ItemList = React.memo(function ItemList({ items, onDelete }) {
  console.log("ItemList rendered");
  return (
    <ul>
      {items.map((item) => (
        <li key={item}>
          {item} <button onClick={() => onDelete(item)}>×</button>
        </li>
      ))}
    </ul>
  );
});
```

---

## Example: Filtering and Sorting Large Lists

```jsx
function EmployeeDirectory({ employees }) {
  const [search, setSearch] = useState("");
  const [sortField, setSortField] = useState("name");
  const [sortOrder, setSortOrder] = useState("asc");
  const [selectedDept, setSelectedDept] = useState("all");

  // Expensive: filter + sort 10,000 employees
  const displayedEmployees = useMemo(() => {
    let result = employees;

    // Filter by department
    if (selectedDept !== "all") {
      result = result.filter((emp) => emp.department === selectedDept);
    }

    // Filter by search
    if (search.trim()) {
      const query = search.toLowerCase();
      result = result.filter(
        (emp) =>
          emp.name.toLowerCase().includes(query) ||
          emp.email.toLowerCase().includes(query),
      );
    }

    // Sort
    result = [...result].sort((a, b) => {
      const valA = a[sortField];
      const valB = b[sortField];
      const comparison =
        typeof valA === "string" ? valA.localeCompare(valB) : valA - valB;
      return sortOrder === "asc" ? comparison : -comparison;
    });

    return result;
  }, [employees, search, sortField, sortOrder, selectedDept]);

  // Departments list (computed once from employees)
  const departments = useMemo(() => {
    return [...new Set(employees.map((emp) => emp.department))].sort();
  }, [employees]);

  return (
    <div>
      <input
        value={search}
        onChange={(e) => setSearch(e.target.value)}
        placeholder="Search employees..."
      />
      <select
        value={selectedDept}
        onChange={(e) => setSelectedDept(e.target.value)}
      >
        <option value="all">All Departments</option>
        {departments.map((dept) => (
          <option key={dept} value={dept}>
            {dept}
          </option>
        ))}
      </select>
      <button
        onClick={() => setSortOrder((o) => (o === "asc" ? "desc" : "asc"))}
      >
        Sort: {sortOrder}
      </button>

      <p>
        Showing {displayedEmployees.length} of {employees.length}
      </p>
      <ul>
        {displayedEmployees.map((emp) => (
          <li key={emp.id}>
            {emp.name} — {emp.department}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Best Practices

1. **Profile before optimizing** — use React DevTools Profiler to confirm an actual performance problem before adding `useMemo`.
2. **Only memoize expensive computations** — simple arithmetic, string concatenation, and basic lookups don't need it.
3. **Include all dependencies** — missing dependencies mean stale cached values.
4. **Use for referential equality** when passing objects/arrays to `React.memo` children.
5. **Don't replace state with useMemo** — if you need the value to persist and trigger renders, use `useState`.
6. **Keep the computation pure** — no side effects inside `useMemo` (no API calls, no DOM manipulation).
7. **Consider restructuring** — sometimes splitting a component is better than memoizing within one.

---

## Common Mistakes

| Mistake                                           | Why It's Wrong                                               |
| ------------------------------------------------- | ------------------------------------------------------------ |
| Memoizing trivial computations (`count * 2`)      | useMemo overhead exceeds the computation cost                |
| Missing dependencies                              | Returns stale cached value — bugs that are hard to trace     |
| Using useMemo for side effects                    | useMemo should be pure — use useEffect for side effects      |
| Assuming useMemo guarantees caching               | React may discard cached values (e.g., offscreen components) |
| Wrapping every variable in useMemo "just in case" | Adds complexity and memory cost without measurable benefit   |
| Not pairing with React.memo on children           | useMemo alone doesn't prevent child re-renders               |

---

## Summary

- **Memoization** caches expensive computation results and returns them when inputs haven't changed.
- `useMemo(() => computation, [deps])` — recalculates only when dependencies change.
- Use for **expensive calculations** (filtering/sorting large arrays) and **referential equality** (passing objects to memoized children).
- **Don't overuse** — profile first. Simple computations don't need memoization.
- Works together with `useCallback` (functions) and `React.memo` (components) as a complete optimization strategy.
- `useMemo` is a **performance hint**, not a guarantee — React may choose to recalculate.
