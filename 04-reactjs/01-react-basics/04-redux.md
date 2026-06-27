# Redux (Global State Management)

## What Is Redux?

Redux is a **predictable state container** for JavaScript applications. It provides a centralized store for all application state, with strict rules about how that state can be updated — making state changes predictable, traceable, and debuggable.

**Analogy:** Imagine a bank. You can't just walk into the vault and take money (direct mutation). You must fill out a withdrawal form (dispatch an action), give it to the teller (reducer), and the teller updates your account balance (state) following strict rules.

---

## Why Redux?

| Problem                                     | Redux Solution                            |
| ------------------------------------------- | ----------------------------------------- |
| Prop drilling through many component layers | Any component reads from global store     |
| State scattered across many components      | Single source of truth in one store       |
| Unpredictable state changes                 | Only reducers can update state            |
| Hard to debug what changed and when         | Time-travel debugging with Redux DevTools |
| Multiple components need the same data      | Shared state without lifting up           |

---

## Core Concepts

### The Three Principles

1. **Single source of truth** — The entire app state lives in one store object.
2. **State is read-only** — The only way to change state is to dispatch an action.
3. **Changes are made with pure functions** — Reducers take previous state + action and return new state.

### Data Flow

```
User clicks button
       ↓
dispatch(action)
       ↓
Reducer receives (state, action)
       ↓
Returns new state
       ↓
Store updates
       ↓
Subscribed components re-render
```

---

## Redux Toolkit (Modern Redux)

Redux Toolkit (RTK) is the official, recommended way to write Redux. It eliminates boilerplate and includes best practices by default.

```bash
npm install @reduxjs/toolkit react-redux
```

---

## Setting Up the Store

```jsx
// store.js
import { configureStore } from "@reduxjs/toolkit";
import counterReducer from "./features/counterSlice";
import todosReducer from "./features/todosSlice";

const store = configureStore({
  reducer: {
    counter: counterReducer,
    todos: todosReducer,
  },
});

export default store;
```

### Provider Setup

```jsx
// main.jsx
import { Provider } from "react-redux";
import store from "./store";
import App from "./App";

createRoot(document.getElementById("root")).render(
  <Provider store={store}>
    <App />
  </Provider>,
);
```

---

## Creating a Slice (`createSlice`)

A slice combines the reducer logic and action creators for a feature in one place.

```jsx
// features/counterSlice.js
import { createSlice } from "@reduxjs/toolkit";

const counterSlice = createSlice({
  name: "counter",
  initialState: { value: 0 },
  reducers: {
    increment(state) {
      state.value += 1; // RTK uses Immer — "mutating" syntax is safe
    },
    decrement(state) {
      state.value -= 1;
    },
    incrementByAmount(state, action) {
      state.value += action.payload;
    },
    reset(state) {
      state.value = 0;
    },
  },
});

// Export action creators
export const { increment, decrement, incrementByAmount, reset } =
  counterSlice.actions;

// Export reducer
export default counterSlice.reducer;
```

**Note:** Redux Toolkit uses **Immer** internally. You can write "mutating" code like `state.value += 1` — Immer converts it to an immutable update behind the scenes.

---

## Using Redux in Components

```jsx
import { useSelector, useDispatch } from "react-redux";
import {
  increment,
  decrement,
  incrementByAmount,
  reset,
} from "./features/counterSlice";

function Counter() {
  // Read state from store
  const count = useSelector((state) => state.counter.value);

  // Get dispatch function
  const dispatch = useDispatch();

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => dispatch(increment())}>+1</button>
      <button onClick={() => dispatch(decrement())}>-1</button>
      <button onClick={() => dispatch(incrementByAmount(5))}>+5</button>
      <button onClick={() => dispatch(reset())}>Reset</button>
    </div>
  );
}
```

---

## Async Logic with `createAsyncThunk`

For API calls and async operations, use `createAsyncThunk`:

```jsx
// features/todosSlice.js
import { createSlice, createAsyncThunk } from "@reduxjs/toolkit";

// Define async thunk
export const fetchTodos = createAsyncThunk("todos/fetchTodos", async () => {
  const response = await fetch("/api/todos");
  return response.json();
});

export const addTodo = createAsyncThunk("todos/addTodo", async (text) => {
  const response = await fetch("/api/todos", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ text }),
  });
  return response.json();
});

const todosSlice = createSlice({
  name: "todos",
  initialState: {
    items: [],
    loading: false,
    error: null,
  },
  reducers: {
    clearTodos(state) {
      state.items = [];
    },
  },
  // Handle async thunk lifecycle
  extraReducers: (builder) => {
    builder
      .addCase(fetchTodos.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchTodos.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      .addCase(fetchTodos.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      })
      .addCase(addTodo.fulfilled, (state, action) => {
        state.items.push(action.payload);
      });
  },
});

export const { clearTodos } = todosSlice.actions;
export default todosSlice.reducer;
```

### Using Async Thunks in Components

```jsx
import { useEffect } from "react";
import { useSelector, useDispatch } from "react-redux";
import { fetchTodos, addTodo } from "./features/todosSlice";

function TodoList() {
  const { items, loading, error } = useSelector((state) => state.todos);
  const dispatch = useDispatch();

  useEffect(() => {
    dispatch(fetchTodos());
  }, [dispatch]);

  const handleAdd = () => {
    dispatch(addTodo("New task"));
  };

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;

  return (
    <div>
      <button onClick={handleAdd}>Add Todo</button>
      <ul>
        {items.map((todo) => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## TypeScript with Redux Toolkit

```typescript
// store.ts
import { configureStore } from "@reduxjs/toolkit";
import counterReducer from "./features/counterSlice";

const store = configureStore({
  reducer: { counter: counterReducer },
});

// Infer types from store
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

export default store;
```

```typescript
// hooks.ts — typed versions of useSelector and useDispatch
import { useSelector, useDispatch } from "react-redux";
import type { RootState, AppDispatch } from "./store";

export const useAppSelector = useSelector.withTypes<RootState>();
export const useAppDispatch = useDispatch.withTypes<AppDispatch>();
```

```typescript
// Usage in component
const count = useAppSelector((state) => state.counter.value); // Fully typed
const dispatch = useAppDispatch();
```

---

## Redux vs Context API

| Criteria       | Redux                     | Context API               |
| -------------- | ------------------------- | ------------------------- |
| Best for       | Large, complex state      | Small, infrequent updates |
| Performance    | Optimized selectors       | Re-renders all consumers  |
| DevTools       | Time-travel debugging     | None built-in             |
| Middleware     | Thunks, sagas, listeners  | Not supported             |
| Boilerplate    | Moderate (RTK reduces it) | Minimal                   |
| Learning curve | Higher                    | Lower                     |
| Async logic    | Built-in with thunks      | Manual implementation     |

**Use Redux when:** You have complex state logic, many components need the same data, state updates are frequent, or you need middleware/DevTools.

**Use Context when:** Passing theme, locale, or auth status — values that change infrequently and are consumed broadly.

---

## Redux DevTools

Install the [Redux DevTools browser extension](https://github.com/reduxjs/redux-devtools). Redux Toolkit enables DevTools automatically.

Features:

- **Action log** — see every dispatched action with its payload.
- **State diff** — see exactly what changed in state after each action.
- **Time-travel** — jump to any previous state.
- **Action replay** — replay actions to reproduce bugs.
- **Export/import** — share state snapshots for debugging.

---

## Best Practices

1. **Use Redux Toolkit** — never write Redux from scratch with manual action types and switch statements.
2. **Normalize complex state** — use flat structures with IDs instead of deeply nested objects.
3. **Keep slices focused** — one slice per feature/domain (`userSlice`, `cartSlice`, `todosSlice`).
4. **Use selectors** — create reusable selector functions for derived data.
5. **Don't put everything in Redux** — form input state, UI toggles, and ephemeral state belong in component state.
6. **Use typed hooks** — create `useAppSelector` and `useAppDispatch` wrappers for TypeScript projects.
7. **Handle all thunk states** — always handle `pending`, `fulfilled`, and `rejected` in `extraReducers`.
8. **Use RTK Query for data fetching** — it handles caching, deduplication, and loading states automatically.

---

## Common Mistakes

| Mistake                                           | Why It's Wrong                    | Fix                                                   |
| ------------------------------------------------- | --------------------------------- | ----------------------------------------------------- |
| Mutating state without Immer (plain Redux)        | State becomes unpredictable       | Use Redux Toolkit (Immer built-in)                    |
| Putting all state in Redux                        | Overkill for local UI state       | Only global/shared state belongs in Redux             |
| Not handling loading/error states                 | UI doesn't reflect async status   | Always handle pending/fulfilled/rejected              |
| Calling `useSelector` with new object each render | Causes unnecessary re-renders     | Select primitives or use `shallowEqual`               |
| Dispatching in render (no useEffect)              | Infinite re-render loop           | Dispatch in event handlers or useEffect               |
| Skipping Redux DevTools in development            | Missing the best debugging tool   | RTK enables it automatically — just install extension |
| Not using `createAsyncThunk` for API calls        | Manual async logic is error-prone | Use thunks for consistent loading/error handling      |

---

## Summary

- Redux is a predictable state container — one store, actions describe what happened, reducers determine how state changes.
- **Redux Toolkit** is the modern standard — use `configureStore`, `createSlice`, and `createAsyncThunk`.
- Components read state with `useSelector` and dispatch actions with `useDispatch`.
- Async logic uses `createAsyncThunk` with `extraReducers` handling pending/fulfilled/rejected lifecycle.
- Redux excels for complex, shared state. Use Context API for simple, infrequent updates like theme or locale.
- Redux DevTools provide time-travel debugging — install the browser extension for full visibility.
- Always use Redux Toolkit — never write manual action types, switch-case reducers, or spread-based immutable updates.
