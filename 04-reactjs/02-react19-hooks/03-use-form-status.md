# useFormStatus (React 19)

## What Is `useFormStatus`?

`useFormStatus` is a hook that lets you **read the submission status of a parent `<form>`** from within a child component. It tells you whether the form is currently submitting, what data was submitted, and which method/action was used.

**Key constraint:** It must be called from a component that is rendered **inside** a `<form>`. It reads the status of the nearest parent form.

**Analogy:** Think of `useFormStatus` as a spy inside the form. The form knows whether it's submitting or idle, and this hook lets any child component peek at that information without props being passed down.

---

## Why `useFormStatus`?

| Problem                                        | `useFormStatus` Solution                      |
| ---------------------------------------------- | --------------------------------------------- |
| Submit button needs to know if form is pending | Reads `pending` directly from parent form     |
| Loading spinner inside form                    | No prop drilling — reads status automatically |
| Disable inputs during submission               | Check `pending` in any child component        |
| Need to access submitted FormData              | `data` property provides it                   |

---

## API

```jsx
import { useFormStatus } from "react-dom";

const { pending, data, method, action } = useFormStatus();
```

**Returns an object with:**

- `pending` — `boolean`. `true` while the parent form's action is executing.
- `data` — `FormData | null`. The form data being submitted. `null` when not submitting.
- `method` — `string`. The HTTP method of the form (`'get'` or `'post'`).
- `action` — `function | string | null`. Reference to the action function passed to the form's `action` prop.

---

## Critical Rule: Must Be Inside a Form Child

`useFormStatus` reads the status of the **nearest parent `<form>`**. It does NOT work if called in the same component that renders the `<form>`.

```jsx
// ❌ WRONG — useFormStatus called in the same component as <form>
function Form() {
  const { pending } = useFormStatus(); // This will NEVER show pending!
  return (
    <form action={someAction}>
      <button disabled={pending}>Submit</button>
    </form>
  );
}

// ✅ CORRECT — useFormStatus called inside a child component
function SubmitButton() {
  const { pending } = useFormStatus(); // Reads from parent <form>
  return (
    <button type="submit" disabled={pending}>
      {pending ? "Submitting..." : "Submit"}
    </button>
  );
}

function Form() {
  return (
    <form action={someAction}>
      <input name="email" />
      <SubmitButton /> {/* Child component — useFormStatus works here */}
    </form>
  );
}
```

---

## Practical Example — Submit Button with Spinner

```jsx
import { useFormStatus } from "react-dom";

function SubmitButton({ label = "Submit" }) {
  const { pending } = useFormStatus();

  return (
    <button type="submit" disabled={pending} className="submit-btn">
      {pending ? (
        <>
          <span className="spinner" aria-hidden="true" />
          Processing...
        </>
      ) : (
        label
      )}
    </button>
  );
}

// Usage in any form
function CheckoutForm() {
  async function processOrder(formData) {
    await fetch("/api/orders", {
      method: "POST",
      body: formData,
    });
  }

  return (
    <form action={processOrder}>
      <input name="address" placeholder="Shipping address" required />
      <input name="card" placeholder="Card number" required />
      <SubmitButton label="Place Order" />
    </form>
  );
}
```

---

## Disabling Inputs While Pending

```jsx
import { useFormStatus } from "react-dom";

function FormFields() {
  const { pending } = useFormStatus();

  return (
    <fieldset disabled={pending} style={{ opacity: pending ? 0.6 : 1 }}>
      <label>
        Name
        <input name="name" required />
      </label>
      <label>
        Email
        <input name="email" type="email" required />
      </label>
      <label>
        Message
        <textarea name="message" required />
      </label>
    </fieldset>
  );
}

function ContactForm({ submitAction }) {
  return (
    <form action={submitAction}>
      <FormFields />
      <SubmitButton label="Send" />
    </form>
  );
}
```

Using `<fieldset disabled={pending}>` disables all inputs inside it during submission — no need to track each input individually.

---

## Accessing Submitted Data

```jsx
import { useFormStatus } from "react-dom";

function FormDebugger() {
  const { pending, data, method, action } = useFormStatus();

  if (!pending) return null;

  return (
    <div className="debug-panel">
      <p>Submitting via: {method.toUpperCase()}</p>
      <p>Fields being sent:</p>
      <ul>
        {data &&
          [...data.entries()].map(([key, value]) => (
            <li key={key}>
              {key}: {String(value)}
            </li>
          ))}
      </ul>
    </div>
  );
}
```

---

## Full Form Example with `useActionState` + `useFormStatus`

These two hooks complement each other — `useActionState` manages the result, `useFormStatus` handles the pending UI.

```jsx
import { useActionState } from "react";
import { useFormStatus } from "react-dom";

// Child component — reads form pending status
function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? "Creating..." : "Create Account"}
    </button>
  );
}

// Child component — disables fields while submitting
function FormInputs() {
  const { pending } = useFormStatus();
  return (
    <fieldset disabled={pending}>
      <input name="username" placeholder="Username" required />
      <input name="email" type="email" placeholder="Email" required />
      <input name="password" type="password" placeholder="Password" required />
    </fieldset>
  );
}

// Parent component — manages action state
function SignupForm() {
  const [state, formAction] = useActionState(
    async (prev, formData) => {
      const response = await fetch("/api/signup", {
        method: "POST",
        body: formData,
      });

      if (!response.ok) {
        const error = await response.json();
        return { error: error.message, success: false };
      }

      return { error: null, success: true };
    },
    { error: null, success: false },
  );

  return (
    <form action={formAction}>
      <FormInputs />
      <SubmitButton />
      {state.error && <p className="error">{state.error}</p>}
      {state.success && <p className="success">Account created!</p>}
    </form>
  );
}
```

---

## Best Practices

1. **Always use in a child component** — never in the same component that renders `<form>`.
2. **Create reusable submit buttons** — `<SubmitButton />` components work in any form.
3. **Use `<fieldset disabled>`** — disable all form inputs at once during submission.
4. **Provide visual feedback** — spinners, reduced opacity, or text changes for pending state.
5. **Keep accessible** — add `aria-busy="true"` to the form or disable buttons properly.
6. **Combine with `useActionState`** — use `useFormStatus` for UI feedback and `useActionState` for result state.

---

## Common Mistakes

| Mistake                                      | Why It's Wrong                                   | Fix                                         |
| -------------------------------------------- | ------------------------------------------------ | ------------------------------------------- |
| Calling in the same component as `<form>`    | Hook reads the PARENT form, not a sibling        | Extract into a child component              |
| Not extracting a submit button component     | Can't access form status without being a child   | Create `<SubmitButton />` component         |
| Using outside any `<form>`                   | Returns `{ pending: false }` always              | Ensure component is rendered inside a form  |
| Relying on `data` when not pending           | `data` is `null` when form is idle               | Only access `data` when `pending` is `true` |
| Not disabling inputs during submission       | Users can modify fields while form is submitting | Wrap in `<fieldset disabled={pending}>`     |
| Forgetting it's from `react-dom` not `react` | Wrong import path causes errors                  | Import from `'react-dom'`                   |

---

## Summary

- `useFormStatus()` reads the submission status of the nearest parent `<form>` — returns `{ pending, data, method, action }`.
- **Must be called in a child component** of the form, not in the same component that renders `<form>`.
- Primary use: showing loading spinners, disabling buttons/inputs, and providing visual feedback during form submission.
- Import from `'react-dom'`, not `'react'`.
- Combine with `useActionState` — one manages the action result, the other provides real-time pending status to child components.
- Create reusable components like `<SubmitButton />` and `<FormFields />` that internally use `useFormStatus` for any form.
