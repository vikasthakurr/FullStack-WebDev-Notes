# React 19 New Hooks

React 19 introduces new hooks designed for working with **forms**, **actions**, and **optimistic UI updates**. These hooks simplify patterns that previously required significant boilerplate with `useState` and `useEffect`.

---

## React 19 Actions Concept

In React 19, **Actions** are functions that handle async transitions — typically form submissions. Instead of manually managing pending/error/success states, you pass an async function to a `<form action={...}>` or use the new hooks.

```jsx
// Before React 19 — manual state management
function OldForm() {
  const [isPending, setIsPending] = useState(false);
  const [error, setError] = useState(null);

  async function handleSubmit(e) {
    e.preventDefault();
    setIsPending(true);
    setError(null);
    try {
      await submitData(new FormData(e.target));
    } catch (err) {
      setError(err.message);
    } finally {
      setIsPending(false);
    }
  }

  return <form onSubmit={handleSubmit}>...</form>;
}

// React 19 — actions handle this automatically
function NewForm() {
  async function createPost(formData) {
    "use server";
    await db.posts.create({ title: formData.get("title") });
  }

  return (
    <form action={createPost}>
      <input name="title" />
      <button type="submit">Create</button>
    </form>
  );
}
```

Actions automatically:

- Track **pending** state during the async operation.
- Handle **errors** thrown during execution.
- Support **optimistic updates** (show expected result before confirmation).
- Prevent **duplicate submissions** while pending.

---

## `useOptimistic`

Updates the UI **immediately** with an expected value before the server confirms. If the operation fails, React automatically reverts to the previous state.

### Why Use It?

Without optimistic updates, users wait for the server before seeing changes — this feels sluggish. With `useOptimistic`, the UI responds instantly.

### Signature

```javascript
const [optimisticState, addOptimistic] = useOptimistic(state, updateFn);
```

- **`state`** — the actual current state (source of truth).
- **`updateFn(currentState, optimisticValue)`** — pure function that merges the optimistic value into current state.
- **`optimisticState`** — the state to render (includes pending optimistic values).
- **`addOptimistic(value)`** — call this to trigger an optimistic update.

### Full Example: Todo List with Optimistic Adds

```jsx
import { useOptimistic, useState, useRef } from "react";

async function addTodoToServer(title) {
  // Simulate network delay
  await new Promise((resolve) => setTimeout(resolve, 1500));

  // Simulate occasional failure
  if (Math.random() < 0.2) {
    throw new Error("Server error: failed to save todo");
  }

  return { id: Date.now(), title, completed: false };
}

function TodoApp() {
  const [todos, setTodos] = useState([
    { id: 1, title: "Learn React 19", completed: false },
    { id: 2, title: "Build a project", completed: false },
  ]);

  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (currentTodos, newTitle) => [
      ...currentTodos,
      {
        id: crypto.randomUUID(),
        title: newTitle,
        completed: false,
        pending: true, // Flag to show loading indicator
      },
    ],
  );

  const formRef = useRef(null);

  async function formAction(formData) {
    const title = formData.get("title");
    formRef.current.reset();

    // Immediately show the new todo (optimistic)
    addOptimisticTodo(title);

    try {
      // Actually save to server
      const savedTodo = await addTodoToServer(title);
      // Update real state — optimistic version auto-replaced
      setTodos((prev) => [...prev, savedTodo]);
    } catch (error) {
      // On failure, optimistic state reverts automatically
      console.error(error.message);
    }
  }

  return (
    <div>
      <h1>My Todos</h1>

      <ul>
        {optimisticTodos.map((todo) => (
          <li key={todo.id} style={{ opacity: todo.pending ? 0.6 : 1 }}>
            {todo.title}
            {todo.pending && <span> (saving...)</span>}
          </li>
        ))}
      </ul>

      <form action={formAction} ref={formRef}>
        <input name="title" placeholder="Add a todo..." required />
        <button type="submit">Add</button>
      </form>
    </div>
  );
}
```

### Like Button Example

```jsx
import { useOptimistic } from "react";

function LikeButton({ postId, initialLikes, isLiked }) {
  const [optimisticLike, toggleOptimisticLike] = useOptimistic(
    { count: initialLikes, liked: isLiked },
    (current, _) => ({
      count: current.liked ? current.count - 1 : current.count + 1,
      liked: !current.liked,
    }),
  );

  async function handleLike() {
    toggleOptimisticLike(null); // Trigger optimistic update

    try {
      await fetch(`/api/posts/${postId}/like`, { method: "POST" });
    } catch (error) {
      // Reverts automatically on error
      console.error("Failed to update like");
    }
  }

  return (
    <button onClick={handleLike}>
      {optimisticLike.liked ? "❤️" : "🤍"} {optimisticLike.count}
    </button>
  );
}
```

---

## `useActionState`

Manages the **full lifecycle** of a form action — tracks pending state, captures the result or error, and provides a wrapped action to pass to `<form>`.

> **Note:** This was previously called `useFormState` in the React canary releases, then briefly `useAction`. The stable name in React 19 is `useActionState`.

### Signature

```javascript
const [state, formAction, isPending] = useActionState(actionFn, initialState);
```

- **`actionFn(previousState, formData)`** — async function called on form submission. Receives previous state and FormData.
- **`initialState`** — the initial state before any submission.
- **`state`** — current state (updated after action completes).
- **`formAction`** — the wrapped action to pass to `<form action={...}>`.
- **`isPending`** — boolean, `true` while the action is executing.

### Full Example: User Registration Form

```jsx
import { useActionState } from "react";

async function registerUser(previousState, formData) {
  const email = formData.get("email");
  const password = formData.get("password");
  const name = formData.get("name");

  // Validation
  if (password.length < 8) {
    return { error: "Password must be at least 8 characters", success: false };
  }

  try {
    const response = await fetch("/api/register", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email, password, name }),
    });

    if (!response.ok) {
      const data = await response.json();
      return { error: data.message, success: false };
    }

    return { error: null, success: true, message: "Account created!" };
  } catch (err) {
    return { error: "Network error. Please try again.", success: false };
  }
}

function RegistrationForm() {
  const [state, formAction, isPending] = useActionState(registerUser, {
    error: null,
    success: false,
  });

  return (
    <form action={formAction}>
      <h2>Create Account</h2>

      {state.error && <p className="error">{state.error}</p>}
      {state.success && <p className="success">{state.message}</p>}

      <label>
        Name
        <input name="name" type="text" required disabled={isPending} />
      </label>

      <label>
        Email
        <input name="email" type="email" required disabled={isPending} />
      </label>

      <label>
        Password
        <input name="password" type="password" required disabled={isPending} />
      </label>

      <button type="submit" disabled={isPending}>
        {isPending ? "Creating account..." : "Register"}
      </button>
    </form>
  );
}
```

### Key Points

- The action function receives the **previous state** as the first argument — useful for tracking attempts or accumulating errors.
- Whatever you **return** from the action becomes the new `state`.
- `isPending` replaces the need for a separate `useState` loading flag.
- Works with both client-side and server-side actions.

---

## `useFormStatus`

Reads the **status of a parent `<form>`** from within a child component. This is useful for building reusable submit buttons that automatically show loading states.

### Signature

```javascript
const { pending, data, method, action } = useFormStatus();
```

- **`pending`** — `true` if the parent form's action is executing.
- **`data`** — the `FormData` object being submitted (or `null`).
- **`method`** — the HTTP method (`"get"` or `"post"`).
- **`action`** — reference to the action function passed to the parent form.

### Important Rule

`useFormStatus` must be called from a component that is **rendered inside a `<form>`**. It reads the status of the nearest parent form — it does not work if called in the same component that renders the `<form>`.

### Example: Reusable Submit Button

```jsx
import { useFormStatus } from "react-dom";

function SubmitButton({ children = "Submit", loadingText = "Submitting..." }) {
  const { pending } = useFormStatus();

  return (
    <button type="submit" disabled={pending}>
      {pending ? loadingText : children}
    </button>
  );
}

// Usage in a form
function ContactForm() {
  async function sendMessage(formData) {
    await fetch("/api/contact", {
      method: "POST",
      body: formData,
    });
  }

  return (
    <form action={sendMessage}>
      <input name="email" type="email" placeholder="Your email" required />
      <textarea name="message" placeholder="Your message" required />
      <SubmitButton loadingText="Sending...">Send Message</SubmitButton>
    </form>
  );
}
```

### Example: Form with Field Disabling

```jsx
import { useFormStatus } from "react-dom";

function FormFields() {
  const { pending } = useFormStatus();

  return (
    <fieldset disabled={pending}>
      <label>
        Title
        <input name="title" type="text" required />
      </label>
      <label>
        Content
        <textarea name="content" required />
      </label>
    </fieldset>
  );
}

function FormProgress() {
  const { pending } = useFormStatus();

  return pending ? (
    <div className="progress-bar" aria-label="Submitting form..." />
  ) : null;
}

function CreatePostForm({ action }) {
  return (
    <form action={action}>
      <FormProgress />
      <FormFields />
      <SubmitButton>Publish Post</SubmitButton>
    </form>
  );
}
```

---

## Server Components & These Hooks

React 19's new hooks are designed to work seamlessly with the **Server Components** architecture:

- **Server Components** render on the server and send HTML to the client — they can directly access databases, file systems, and APIs without exposing secrets.
- **Server Actions** (`"use server"` directive) are async functions that run on the server but can be called from the client — ideal as form actions.
- **`useActionState`** can wrap a server action, automatically handling the round-trip.
- **`useOptimistic`** shows immediate feedback while a server action processes.
- **`useFormStatus`** reflects pending state during server action execution.

```jsx
// Server Action (runs on the server)
"use server";

async function createPost(prevState, formData) {
  const title = formData.get("title");
  await db.posts.insert({ title, authorId: getCurrentUser().id });
  revalidatePath("/posts");
  return { success: true };
}

// Client Component (uses server action)
("use client");

import { useActionState } from "react";
import { createPost } from "./actions";

function NewPostForm() {
  const [state, formAction, isPending] = useActionState(createPost, {});

  return (
    <form action={formAction}>
      <input name="title" />
      <SubmitButton>Create Post</SubmitButton>
    </form>
  );
}
```

---

## Comparison Table: When to Use Each Hook

| Hook             | Purpose               | Use When...                                                                                               | Returns                             |
| ---------------- | --------------------- | --------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| `useOptimistic`  | Instant UI feedback   | You want to show a change immediately before server confirms                                              | `[optimisticState, addOptimistic]`  |
| `useActionState` | Full action lifecycle | You need pending, error, and result state for a form action                                               | `[state, formAction, isPending]`    |
| `useFormStatus`  | Parent form status    | You are building a child component (button, fields) inside a form that needs to react to submission state | `{ pending, data, method, action }` |

### Decision Guide

```
Need to show result before server responds?
  → useOptimistic

Need to manage form submission state (pending + result + error)?
  → useActionState

Building a reusable button/field that needs parent form's pending state?
  → useFormStatus

Need all three?
  → Combine them! useActionState for the form, useOptimistic for instant
    feedback, useFormStatus for child component awareness.
```

### Combined Example

```jsx
import { useOptimistic, useActionState } from "react";
import { useFormStatus } from "react-dom";

function SubmitBtn() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? "Adding..." : "Add Comment"}
    </button>
  );
}

function CommentForm({ comments, setComments }) {
  const [optimisticComments, addOptimisticComment] = useOptimistic(
    comments,
    (current, newComment) => [...current, { ...newComment, pending: true }],
  );

  async function postComment(prevState, formData) {
    const text = formData.get("text");
    addOptimisticComment({ id: Date.now(), text });

    try {
      const res = await fetch("/api/comments", {
        method: "POST",
        body: JSON.stringify({ text }),
        headers: { "Content-Type": "application/json" },
      });
      const saved = await res.json();
      setComments((prev) => [...prev, saved]);
      return { error: null };
    } catch {
      return { error: "Failed to post comment" };
    }
  }

  const [state, formAction] = useActionState(postComment, { error: null });

  return (
    <div>
      <ul>
        {optimisticComments.map((c) => (
          <li key={c.id} style={{ opacity: c.pending ? 0.5 : 1 }}>
            {c.text}
          </li>
        ))}
      </ul>

      {state.error && <p className="error">{state.error}</p>}

      <form action={formAction}>
        <input name="text" required placeholder="Write a comment..." />
        <SubmitBtn />
      </form>
    </div>
  );
}
```

---

## Best Practices

1. **Use `useActionState` as your default** for form handling — it replaces manual `useState` + `useEffect` patterns for loading/error/success.
2. **Extract submit buttons into components** — `useFormStatus` only works inside child components of a `<form>`, not in the same component rendering the form.
3. **Optimistic updates are not free** — only use `useOptimistic` when the action is very likely to succeed (>95% success rate). For risky operations, show a loading state instead.
4. **Return structured state from actions** — use objects like `{ error, success, data }` so your UI can react to different outcomes.
5. **Combine hooks when needed** — `useActionState` for form lifecycle + `useOptimistic` for instant feedback + `useFormStatus` for child components is a powerful pattern.
6. **Validate on both client and server** — client-side validation improves UX, but server-side validation is the security boundary.
7. **Keep actions focused** — one action per form. If a form does multiple things, split it into multiple forms or use conditional logic inside the action.

---

## Common Mistakes

| Mistake                                                 | Why It Fails                                              | Fix                                                                     |
| ------------------------------------------------------- | --------------------------------------------------------- | ----------------------------------------------------------------------- |
| Using `useFormStatus` in the same component as `<form>` | It reads the nearest **parent** form, not the current one | Extract into a child component                                          |
| Forgetting to return state from `useActionState` action | State becomes `undefined`, breaking the UI                | Always return an object from the action function                        |
| Using `useOptimistic` without error handling            | UI stays in optimistic state forever on failure           | Wrap in try/catch; state reverts automatically on error                 |
| Not disabling inputs during pending                     | Users can submit multiple times                           | Use `isPending` from `useActionState` or `pending` from `useFormStatus` |

---

## Summary

- **Actions** are React 19's pattern for handling async operations (especially form submissions) with automatic pending/error state management.
- **`useOptimistic`** shows changes instantly — the UI updates before the server responds, and reverts automatically on failure.
- **`useActionState`** replaces manual `useState` boilerplate for forms — gives you `[state, wrappedAction, isPending]` in one call.
- **`useFormStatus`** lets child components (buttons, field sets) react to their parent form's submission state.
- These hooks work together and are designed to integrate with **Server Components** and **Server Actions** in frameworks like Next.js.
- The combination of all three hooks makes building responsive, error-handled forms dramatically simpler than previous React patterns.
