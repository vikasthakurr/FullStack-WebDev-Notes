# Redux (with Redux Toolkit)

## What Is Redux?

Redux is a **predictable state container** for JavaScript applications. It provides a centralized store that holds your entire application's state in one place, making state changes predictable and traceable.

**Analogy:** Think of Redux like a bank. You (component) can't just reach into the vault and change your balance. You must fill out a form (dispatch an action) that goes to the teller (reducer), who processes it according to strict rules and updates your account (store). Everything is logged, auditable, and predictable.

---

## Why Redux?

| Problem Without Redux                       | How Redux Solves It                          |
| ------------------------------------------- | -------------------------------------------- |
| Prop drilling through many component layers | Any component can access the store directly  |
| State scattered across dozens of components | Single source of truth in one store          |
| Hard to track what caused a state change    | Actions describe every change explicitly     |
| Difficult to reproduce bugs                 | Time-travel debugging — step through actions |
| Complex shared state logic                  | Reducers are pure functions — easy to test   |

**When Redux makes sense:**

- Large applications with lots of shared state.
- State that needs to be accessed by many unrelated components.
- Complex state update logic.
- Team needs predictable, traceable state changes.

**When you DON'T need Redux:**

- Small apps with simple state.
- State that's local to one or two components.
- Server state (use React Query/TanStack Query instead).

---

## Core Concepts

### Store

The single JavaScript object that holds the entire state tree of your application.

```js
// Conceptual shape of a store's state
{
  user: { name: "Vikas", isLoggedIn: true },
  todos: [{ id: 1, text: "Learn Redux", done: false }],
  theme: "dark"
}
```

### Actions

Plain objects that describe **what happened**. Every action must have a `type` field.

```js
// Action examples
{ type: "todos/added", payload: { id: 2, text: "Build App", done: false } }
{ type: "todos/toggled", payload: { id: 1 } }
{ type: "user/loggedOut" }
```

### Reducers

Pure functions that take the current state + an action and return the **new state**. They describe **how** state changes in response to actions.

```js
// Reducer example (without Redux Toolkit)
function todosReducer(state = [], action) {
  switch (action.type) {
    case "todos/added":
      return [...state, action.payload];
    case "todos/toggled":
      return state.map((todo) =>
        todo.id === action.payload.id ? { ...todo, done: !todo.done } : todo,
      );
    default:
      return state;
  }
}
```

### Dispatch

The method to send actions to the store. It's the **only way** to trigger state changes.

```js
store.dispatch({
  type: "todos/added",
  payload: { id: 1, text: "Learn", done: false },
});
```

### The Redux Flow

```
UI Event → dispatch(action) → Reducer(state, action) → New State → UI Re-renders
```

---

## Redux Toolkit (Modern Redux)

**Redux Toolkit (RTK)** is the official, recommended way to write Redux logic. It eliminates boilerplate and includes best practices by default.

```bash
npm install @reduxjs/toolkit react-redux
```

RTK provides:

- `configureStore` — sets up the store with good defaults (DevTools, middleware).
- `createSlice` — generates action creators and reducers together.
- `createAsyncThunk` — handles async logic with pending/fulfilled/rejected states.

---

## Setting Up Redux Toolkit in React

### 1. Create a Slice

A **slice** is a collection of reducer logic and actions for a single feature:

```jsx
// src/features/counter/counterSlice.js
import { createSlice } from "@reduxjs/toolkit";

const counterSlice = createSlice({
  name: "counter",
  initialState: {
    value: 0,
  },
  reducers: {
    increment: (state) => {
      state.value += 1; // RTK uses Immer — "mutating" syntax is safe here!
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    },
    reset: (state) => {
      state.value = 0;
    },
  },
});

// Export the auto-generated action creators
export const { increment, decrement, incrementByAmount, reset } =
  counterSlice.actions;

// Export the reducer to add to the store
export default counterSlice.reducer;
```

**Note on Immer:** Inside `createSlice` reducers, you can write "mutating" code like `state.value += 1`. Redux Toolkit uses Immer under the hood, which tracks your mutations and produces a new immutable state. You're not actually mutating — it just looks like it.

### 2. Configure the Store

```jsx
// src/app/store.js
import { configureStore } from "@reduxjs/toolkit";
import counterReducer from "../features/counter/counterSlice";

export const store = configureStore({
  reducer: {
    counter: counterReducer,
    // Add more slice reducers here as your app grows
    // todos: todosReducer,
    // user: userReducer,
  },
});
```

### 3. Provide the Store to React

```jsx
// src/main.jsx (or index.js)
import React from "react";
import ReactDOM from "react-dom/client";
import { Provider } from "react-redux";
import { store } from "./app/store";
import App from "./App";

ReactDOM.createRoot(document.getElementById("root")).render(
  <Provider store={store}>
    <App />
  </Provider>,
);
```

### 4. Use in Components

```jsx
// src/features/counter/Counter.jsx
import { useSelector, useDispatch } from "react-redux";
import { increment, decrement, incrementByAmount, reset } from "./counterSlice";

function Counter() {
  const count = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();

  return (
    <div>
      <h2>Count: {count}</h2>
      <button onClick={() => dispatch(increment())}>+1</button>
      <button onClick={() => dispatch(decrement())}>-1</button>
      <button onClick={() => dispatch(incrementByAmount(5))}>+5</button>
      <button onClick={() => dispatch(reset())}>Reset</button>
    </div>
  );
}

export default Counter;
```

---

## useSelector and useDispatch

### useSelector

Reads data from the Redux store. The component re-renders when the selected value changes.

```jsx
import { useSelector } from "react-redux";

// Select a single value
const count = useSelector((state) => state.counter.value);

// Select derived/computed data
const completedTodos = useSelector((state) =>
  state.todos.items.filter((todo) => todo.done),
);

// Select multiple values (creates new object every render — be careful)
// ❌ This causes unnecessary re-renders
const { name, email } = useSelector((state) => ({
  name: state.user.name,
  email: state.user.email,
}));

// ✅ Better — call useSelector separately for each value
const name = useSelector((state) => state.user.name);
const email = useSelector((state) => state.user.email);
```

### useDispatch

Returns the store's `dispatch` function to send actions:

```jsx
import { useDispatch } from "react-redux";
import { increment } from "./counterSlice";

function IncrementButton() {
  const dispatch = useDispatch();

  return <button onClick={() => dispatch(increment())}>+1</button>;
}
```

---

## Real-World Example: Todo Slice

```jsx
// src/features/todos/todosSlice.js
import { createSlice } from "@reduxjs/toolkit";

const todosSlice = createSlice({
  name: "todos",
  initialState: {
    items: [],
    filter: "all", // "all" | "active" | "completed"
  },
  reducers: {
    addTodo: (state, action) => {
      state.items.push({
        id: Date.now(),
        text: action.payload,
        done: false,
      });
    },
    toggleTodo: (state, action) => {
      const todo = state.items.find((t) => t.id === action.payload);
      if (todo) {
        todo.done = !todo.done;
      }
    },
    removeTodo: (state, action) => {
      state.items = state.items.filter((t) => t.id !== action.payload);
    },
    setFilter: (state, action) => {
      state.filter = action.payload;
    },
    clearCompleted: (state) => {
      state.items = state.items.filter((t) => !t.done);
    },
  },
});

export const { addTodo, toggleTodo, removeTodo, setFilter, clearCompleted } =
  todosSlice.actions;

export default todosSlice.reducer;
```

Using it in a component:

```jsx
// src/features/todos/TodoList.jsx
import { useSelector, useDispatch } from "react-redux";
import { toggleTodo, removeTodo } from "./todosSlice";

function TodoList() {
  const { items, filter } = useSelector((state) => state.todos);
  const dispatch = useDispatch();

  const filteredTodos = items.filter((todo) => {
    if (filter === "active") return !todo.done;
    if (filter === "completed") return todo.done;
    return true; // "all"
  });

  return (
    <ul>
      {filteredTodos.map((todo) => (
        <li key={todo.id}>
          <span
            onClick={() => dispatch(toggleTodo(todo.id))}
            style={{
              textDecoration: todo.done ? "line-through" : "none",
              cursor: "pointer",
            }}
          >
            {todo.text}
          </span>
          <button onClick={() => dispatch(removeTodo(todo.id))}>×</button>
        </li>
      ))}
    </ul>
  );
}

export default TodoList;
```

---

## Async Operations with createAsyncThunk

Real apps need to fetch data from APIs. `createAsyncThunk` generates actions for the three stages of an async request: **pending**, **fulfilled**, and **rejected**.

```jsx
// src/features/posts/postsSlice.js
import { createSlice, createAsyncThunk } from "@reduxjs/toolkit";

// Define the async thunk
export const fetchPosts = createAsyncThunk(
  "posts/fetchPosts", // action type prefix
  async (_, { rejectWithValue }) => {
    try {
      const response = await fetch(
        "https://jsonplaceholder.typicode.com/posts",
      );
      if (!response.ok) throw new Error("Failed to fetch");
      return await response.json(); // This becomes action.payload in fulfilled
    } catch (error) {
      return rejectWithValue(error.message); // This becomes action.payload in rejected
    }
  },
);

const postsSlice = createSlice({
  name: "posts",
  initialState: {
    items: [],
    loading: false,
    error: null,
  },
  reducers: {
    // Sync reducers here if needed
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchPosts.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchPosts.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      .addCase(fetchPosts.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  },
});

export default postsSlice.reducer;
```

Using it in a component:

```jsx
// src/features/posts/PostsList.jsx
import { useEffect } from "react";
import { useSelector, useDispatch } from "react-redux";
import { fetchPosts } from "./postsSlice";

function PostsList() {
  const { items: posts, loading, error } = useSelector((state) => state.posts);
  const dispatch = useDispatch();

  useEffect(() => {
    dispatch(fetchPosts());
  }, [dispatch]);

  if (loading) return <p>Loading posts...</p>;
  if (error) return <p>Error: {error}</p>;

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

export default PostsList;
```

### Passing Arguments to Thunks

```jsx
// Thunk that accepts an argument
export const fetchPostById = createAsyncThunk(
  "posts/fetchById",
  async (postId) => {
    const response = await fetch(
      `https://jsonplaceholder.typicode.com/posts/${postId}`,
    );
    return await response.json();
  },
);

// Dispatching with an argument
dispatch(fetchPostById(42));
```

---

## Redux vs Context API

| Feature            | Redux (RTK)                                                       | Context API                                         |
| ------------------ | ----------------------------------------------------------------- | --------------------------------------------------- |
| **Purpose**        | Global state management                                           | Avoiding prop drilling                              |
| **Performance**    | Optimized — components only re-render when selected state changes | All consumers re-render on any context value change |
| **DevTools**       | Redux DevTools with time-travel debugging                         | No built-in dev tooling                             |
| **Middleware**     | Built-in (thunk, listener, etc.)                                  | No middleware support                               |
| **Boilerplate**    | Moderate (but RTK reduces it significantly)                       | Minimal                                             |
| **Async handling** | createAsyncThunk, RTK Query                                       | Manual with useEffect/useReducer                    |
| **Best for**       | Complex, frequently updated shared state                          | Simple shared values (theme, locale, auth status)   |
| **Learning curve** | Steeper                                                           | Minimal — just React built-ins                      |

**Use Context when:**

- The data changes infrequently (theme, language, auth status).
- Only a few components need the data.
- You want zero additional dependencies.

**Use Redux when:**

- Many components share and update the same state frequently.
- You need middleware for async/complex logic.
- You want time-travel debugging and action history.
- The state update logic is complex (many actions, many conditions).

---

## Redux DevTools

Redux DevTools is a browser extension that provides powerful debugging capabilities:

1. **Install the extension** — Available for Chrome and Firefox.
2. **Automatically enabled** — `configureStore` from RTK enables it by default in development.

### What DevTools Show You

- **Action log** — every dispatched action with type and payload.
- **State diff** — what changed in state after each action.
- **Full state tree** — inspect the entire store at any point.
- **Time-travel** — step backward/forward through actions to reproduce bugs.
- **Action replay** — re-dispatch past actions.
- **Export/Import** — share state snapshots with teammates.

```jsx
// DevTools are enabled automatically with configureStore
// No extra config needed!
export const store = configureStore({
  reducer: {
    counter: counterReducer,
    todos: todosReducer,
  },
  // DevTools enabled by default in development
  // Set devTools: false in production builds
});
```

**Tip:** Name your slices meaningfully (`counter`, `todos`, `auth`) — these show up as prefixes in DevTools making it easy to trace which feature dispatched what.

---

## Best Practices

1. **Use Redux Toolkit** — never write Redux "by hand" anymore. RTK is the official recommendation.
2. **Organize by feature** — group slice, components, and selectors together (`features/todos/`).
3. **Keep reducers pure** — no API calls, no random values, no Date.now() inside reducers (put side effects in thunks).
4. **Normalize complex state** — for relational data, use flat structures with IDs instead of nested objects.
5. **Use `createSelector` for derived data** — memoize expensive computations from state.
6. **Don't put everything in Redux** — form input state, UI-only state (modal open), and server cache belong elsewhere.
7. **Name actions meaningfully** — describe what happened (`"posts/fetchFailed"`), not what to do (`"SET_LOADING"`).
8. **Use `extraReducers` for async** — keep slice logic clean with the builder pattern.
9. **Disable DevTools in production** — set `devTools: process.env.NODE_ENV !== 'production'`.
10. **Consider RTK Query** for server state — it handles caching, invalidation, and loading states automatically.

---

## Common Mistakes

| Mistake                                                 | Why It's Wrong                                                                    |
| ------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Mutating state outside of `createSlice` reducers        | Only Immer-wrapped reducers (inside `createSlice`) can use mutating syntax safely |
| Putting API calls directly in reducers                  | Reducers must be pure — side effects go in thunks or middleware                   |
| Storing server-cached data in Redux                     | Use React Query/TanStack Query instead — it handles caching and invalidation      |
| Selecting entire state object (`state => state`)        | Causes re-renders on every single state change — select only what you need        |
| Creating a new object in `useSelector`                  | New reference every render → infinite re-renders. Select primitives separately    |
| Not using the `builder` pattern in `extraReducers`      | The object notation is deprecated — always use `builder.addCase()`                |
| Putting form field state in Redux                       | Overkill and slow — use local state or a form library (React Hook Form)           |
| Forgetting to add the slice reducer to `configureStore` | The slice exists but never affects state — a silent bug                           |

---

## Summary

- **Redux** is a predictable, centralized state management library — one store, explicit actions, pure reducers.
- **Redux Toolkit (RTK)** is the modern, official way to use Redux — eliminates boilerplate with `createSlice` and `configureStore`.
- A **slice** bundles related state, reducers, and action creators in one file.
- **Immer** is built into RTK — write "mutating" code in reducers and it safely produces immutable updates.
- Use **`useSelector`** to read from the store and **`useDispatch`** to send actions.
- Handle async operations with **`createAsyncThunk`** — it gives you pending/fulfilled/rejected lifecycle.
- **Redux DevTools** provide time-travel debugging, action history, and state inspection.
- **Use Redux** for complex, frequently updated global state. **Use Context** for simple, rarely-changing values like theme or locale.
- Keep Redux focused on **app-wide shared state** — local UI state and server cache belong elsewhere.
