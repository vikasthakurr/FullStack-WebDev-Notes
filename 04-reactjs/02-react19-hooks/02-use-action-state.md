# useActionState (React 19)

## What Is `useActionState`?

`useActionState` is a hook that **manages the state returned by form actions**. It connects a form submission action to a state value, giving you the result of the last action, a wrapped action to pass to the form, and a pending indicator — all in one hook.

**Analogy:** Think of `useActionState` as a `useReducer` specifically designed for form submissions. Instead of dispatching actions manually, the form triggers the reducer when submitted, and you get the result + pending state automatically.

---

## Why `useActionState`?

| Before React 19                         | With `useActionState`                    |
| --------------------------------------- | ---------------------------------------- |
| Manual `useState` for form result       | State managed by the hook                |
| Manual `useState` for loading           | `isPending` returned automatically       |
| Manual `onSubmit` + `preventDefault`    | Pass action directly to `<form action>`  |
| Manual error state management           | Errors returned as state from the action |
| Complex `useReducer` patterns for forms | One hook handles it all                  |

---

## API

```jsx
import { useActionState } from 'react';

const [state, formAction, isPending] = useActionState(action, initialState, permalink?);
```

**Parameters:**

- `action(previousState, formData)` — The function called when the form submits. Receives the previous state and the `FormData` object. Returns the new state (can be async).
- `initialState` — The initial state value before any action has been dispatched.
- `permalink` (optional) — URL for progressive enhancement with server-side rendering.

**Returns:**

- `state` — The current state (result of the last action call, or `initialState` before first submit).
- `formAction` — A wrapped action to pass to `<form action={formAction}>`.
- `isPending` — `true` while the action is executing.

---

## Basic Example — Contact Form

```jsx
import { useActionState } from "react";

async function submitContact(previousState, formData) {
  const name = formData.get("name");
  const email = formData.get("email");
  const message = formData.get("message");

  // Validation
  if (!name || !email || !message) {
    return { success: false, error: "All fields are required" };
  }

  // Server submission
  const response = await fetch("/api/contact", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ name, email, message }),
  });

  if (!response.ok) {
    return { success: false, error: "Failed to send message. Try again." };
  }

  return { success: true, error: null };
}

function ContactForm() {
  const [state, formAction, isPending] = useActionState(submitContact, {
    success: false,
    error: null,
  });

  return (
    <form action={formAction}>
      <input name="name" placeholder="Name" required />
      <input name="email" type="email" placeholder="Email" required />
      <textarea name="message" placeholder="Message" required />

      <button type="submit" disabled={isPending}>
        {isPending ? "Sending..." : "Send Message"}
      </button>

      {state.error && <p className="error">{state.error}</p>}
      {state.success && <p className="success">Message sent!</p>}
    </form>
  );
}
```

---

## With Server Actions (Next.js / React Server Components)

```jsx
// actions.js (Server Action)
"use server";

export async function createUser(previousState, formData) {
  const username = formData.get("username");
  const email = formData.get("email");

  // Server-side validation
  const existingUser = await db.user.findByEmail(email);
  if (existingUser) {
    return { error: "Email already registered", success: false };
  }

  await db.user.create({ username, email });
  return { error: null, success: true };
}
```

```jsx
// SignupForm.jsx (Client Component)
"use client";

import { useActionState } from "react";
import { createUser } from "./actions";

function SignupForm() {
  const [state, formAction, isPending] = useActionState(createUser, {
    error: null,
    success: false,
  });

  return (
    <form action={formAction}>
      <input name="username" placeholder="Username" />
      <input name="email" type="email" placeholder="Email" />

      <button type="submit" disabled={isPending}>
        {isPending ? "Creating account..." : "Sign Up"}
      </button>

      {state.error && <p style={{ color: "red" }}>{state.error}</p>}
      {state.success && <p style={{ color: "green" }}>Account created!</p>}
    </form>
  );
}
```

---

## Error Handling Patterns

```jsx
async function loginAction(previousState, formData) {
  const email = formData.get("email");
  const password = formData.get("password");

  // Field-level errors
  const errors = {};
  if (!email) errors.email = "Email is required";
  if (!password) errors.password = "Password is required";
  if (password && password.length < 8) {
    errors.password = "Password must be at least 8 characters";
  }

  if (Object.keys(errors).length > 0) {
    return { errors, success: false };
  }

  try {
    const response = await fetch("/api/login", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email, password }),
    });

    if (response.status === 401) {
      return { errors: { form: "Invalid credentials" }, success: false };
    }

    if (!response.ok) {
      return { errors: { form: "Something went wrong" }, success: false };
    }

    return { errors: {}, success: true };
  } catch {
    return {
      errors: { form: "Network error. Please try again." },
      success: false,
    };
  }
}

function LoginForm() {
  const [state, formAction, isPending] = useActionState(loginAction, {
    errors: {},
    success: false,
  });

  return (
    <form action={formAction}>
      <div>
        <input name="email" type="email" placeholder="Email" />
        {state.errors.email && (
          <span className="error">{state.errors.email}</span>
        )}
      </div>

      <div>
        <input name="password" type="password" placeholder="Password" />
        {state.errors.password && (
          <span className="error">{state.errors.password}</span>
        )}
      </div>

      {state.errors.form && <p className="error">{state.errors.form}</p>}

      <button type="submit" disabled={isPending}>
        {isPending ? "Logging in..." : "Log In"}
      </button>
    </form>
  );
}
```

---

## Replacing `useReducer` for Form Submissions

### Before (useReducer + manual submit)

```jsx
// ❌ Verbose pattern
function FormOld() {
  const [state, dispatch] = useReducer(formReducer, initialState);
  const [isPending, setIsPending] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();
    setIsPending(true);
    try {
      const result = await submitToServer(new FormData(e.target));
      dispatch({ type: "SUCCESS", payload: result });
    } catch (err) {
      dispatch({ type: "ERROR", payload: err.message });
    } finally {
      setIsPending(false);
    }
  }

  return <form onSubmit={handleSubmit}>...</form>;
}
```

### After (useActionState)

```jsx
// ✅ Clean and integrated
function FormNew() {
  const [state, formAction, isPending] = useActionState(
    async (prev, formData) => {
      try {
        const result = await submitToServer(formData);
        return { data: result, error: null };
      } catch (err) {
        return { data: null, error: err.message };
      }
    },
    { data: null, error: null },
  );

  return <form action={formAction}>...</form>;
}
```

---

## Best Practices

1. **Return structured state** — always return objects with clear fields: `{ data, error, success }`.
2. **Handle all outcomes** — validation errors, server errors, network errors, and success.
3. **Use field-level errors** — return `{ errors: { email: '...', password: '...' } }` for precise feedback.
4. **Disable submit while pending** — use `isPending` to prevent double submissions.
5. **Keep actions pure** — actions should validate, call APIs, and return state. No DOM manipulation.
6. **Use with Server Actions** — `useActionState` integrates naturally with React Server Components.
7. **Progressive enhancement** — use the `permalink` parameter for forms that should work without JavaScript.

---

## Common Mistakes

| Mistake                              | Why It's Wrong                                  | Fix                                             |
| ------------------------------------ | ----------------------------------------------- | ----------------------------------------------- |
| Using `onSubmit` instead of `action` | Bypasses React 19's form integration            | Pass `formAction` to `<form action={...}>`      |
| Not handling pending state           | Users can double-submit                         | Disable button with `isPending`                 |
| Throwing errors inside the action    | Crashes the form instead of showing errors      | Return error state, never throw                 |
| Forgetting `previousState` parameter | Function signature is wrong, state won't work   | Always accept `(previousState, formData)`       |
| Calling `e.preventDefault()`         | Not needed with `action` prop — form handles it | Remove `preventDefault` when using form actions |
| Storing sensitive data in state      | State persists and may be exposed               | Only store display-relevant results             |
| Not validating on server side        | Client validation can be bypassed               | Always validate in the action function too      |

---

## Summary

- `useActionState(action, initialState)` manages form submission state — the result, the wrapped action, and pending status.
- The action function receives `(previousState, formData)` and returns new state.
- `isPending` is automatically `true` while the action executes — no manual loading state needed.
- Integrates with Server Actions for full-stack form handling in React Server Components.
- Returns field-level and form-level errors as structured state for precise error display.
- Replaces verbose `useReducer` + `useState(isPending)` + `handleSubmit` patterns for form submissions.
