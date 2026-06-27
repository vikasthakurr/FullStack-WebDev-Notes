# useOptimistic (React 19)

## What Is Optimistic UI?

Optimistic UI is a pattern where the **interface updates immediately as if the server already confirmed the action**, then reconciles when the actual server response arrives. If the server rejects the action, the UI rolls back to the previous state.

**Analogy:** When you send a message in a chat app, it appears instantly in your conversation (optimistic). If the network fails, a red "failed to send" indicator appears (rollback). You don't wait 2 seconds staring at a spinner for every message.

---

## Why Optimistic Updates?

| Traditional Approach                          | Optimistic Approach                                           |
| --------------------------------------------- | ------------------------------------------------------------- |
| Click → Spinner → Server responds → Update UI | Click → Update UI immediately → Server confirms in background |
| Feels slow (300-2000ms delay)                 | Feels instant                                                 |
| User waits for confirmation                   | User sees result immediately                                  |
| Simple error handling                         | Requires rollback logic                                       |

---

## `useOptimistic` Hook API

```jsx
import { useOptimistic } from "react";

const [optimisticState, addOptimistic] = useOptimistic(state, updateFn);
```

**Parameters:**

- `state` — The actual/current state value (source of truth from server).
- `updateFn(currentState, optimisticValue)` — A pure function that merges the optimistic value into the current state. Called when `addOptimistic` is invoked.

**Returns:**

- `optimisticState` — The state to render. Equals `state` normally, but reflects optimistic updates while an async action is pending.
- `addOptimistic(optimisticValue)` — Call this to trigger an optimistic update. Passes `optimisticValue` to `updateFn`.

**Key behavior:** When the async action completes (the form action or transition finishes), `optimisticState` automatically reverts to the real `state` value. No manual rollback needed.

---

## Practical Example — Like Button

```jsx
import { useOptimistic } from "react";

function LikeButton({ postId, initialLikes, isLiked }) {
  const [optimisticLikes, addOptimisticLike] = useOptimistic(
    { count: initialLikes, liked: isLiked },
    (current, newLiked) => ({
      count: newLiked ? current.count + 1 : current.count - 1,
      liked: newLiked,
    }),
  );

  async function handleLike() {
    const newLiked = !optimisticLikes.liked;
    addOptimisticLike(newLiked); // Instant UI update

    // Server request happens in background
    await fetch(`/api/posts/${postId}/like`, {
      method: newLiked ? "POST" : "DELETE",
    });
    // When this completes, React reconciles with actual server state
  }

  return (
    <button onClick={handleLike}>
      {optimisticLikes.liked ? "❤️" : "🤍"} {optimisticLikes.count}
    </button>
  );
}
```

---

## With Form Actions (Server Actions)

`useOptimistic` works best inside form actions or transitions:

```jsx
import { useOptimistic } from "react";

function TodoList({ todos, addTodoAction }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (currentTodos, newTodoText) => [
      ...currentTodos,
      { id: crypto.randomUUID(), text: newTodoText, pending: true },
    ],
  );

  async function formAction(formData) {
    const text = formData.get("todo");
    addOptimisticTodo(text); // Show immediately with pending indicator
    await addTodoAction(text); // Server action
  }

  return (
    <div>
      <form action={formAction}>
        <input name="todo" placeholder="Add todo..." />
        <button type="submit">Add</button>
      </form>
      <ul>
        {optimisticTodos.map((todo) => (
          <li key={todo.id} style={{ opacity: todo.pending ? 0.5 : 1 }}>
            {todo.text}
            {todo.pending && " (saving...)"}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Add to Cart Example

```jsx
import { useOptimistic, useTransition } from "react";

function AddToCartButton({ product, cartItems }) {
  const [isPending, startTransition] = useTransition();
  const [optimisticCart, addOptimisticItem] = useOptimistic(
    cartItems,
    (currentCart, newItem) => [...currentCart, { ...newItem, pending: true }],
  );

  function handleAddToCart() {
    startTransition(async () => {
      addOptimisticItem(product); // Immediately show in cart

      const response = await fetch("/api/cart", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ productId: product.id }),
      });

      if (!response.ok) {
        // If server rejects, optimisticCart automatically reverts
        // when the transition ends
        throw new Error("Failed to add to cart");
      }
    });
  }

  return (
    <div>
      <button onClick={handleAddToCart} disabled={isPending}>
        {isPending ? "Adding..." : "Add to Cart"}
      </button>
      <p>Cart items: {optimisticCart.length}</p>
    </div>
  );
}
```

---

## Rollback Behavior

The rollback is automatic:

1. You call `addOptimistic(value)` — the `updateFn` produces optimistic state.
2. `optimisticState` reflects the optimistic value during the pending async action.
3. When the action **completes** (success or failure), `optimisticState` reverts to the actual `state` prop.
4. If the server succeeded, the parent re-renders with updated `state` — the optimistic and real states match.
5. If the server failed, `state` hasn't changed — the UI automatically shows the original state (rollback).

**You don't write rollback logic.** The hook handles it by always resetting to the `state` prop when the action finishes.

---

## Comparison with Manual Optimistic Patterns

| Aspect             | `useOptimistic`                     | Manual Pattern (pre-React 19)             |
| ------------------ | ----------------------------------- | ----------------------------------------- |
| Rollback logic     | Automatic                           | Manual state revert on error              |
| Integration        | Works with form actions/transitions | Requires custom try/catch/finally         |
| State management   | Derived from real state             | Separate optimistic + real state tracking |
| Pending indicators | Natural with `pending` flag         | Manual boolean state management           |
| Complexity         | Low — one hook call                 | High — multiple state variables           |
| Race conditions    | Handled by React                    | Must handle manually                      |

### Before React 19 (Manual Pattern)

```jsx
// ❌ More complex, error-prone
function LikeButton({ postId, initialLikes }) {
  const [likes, setLikes] = useState(initialLikes);
  const [isLiked, setIsLiked] = useState(false);
  const previousRef = useRef(null);

  async function handleLike() {
    // Save previous state for rollback
    previousRef.current = { likes, isLiked };

    // Optimistic update
    setLikes((l) => l + 1);
    setIsLiked(true);

    try {
      await fetch(`/api/posts/${postId}/like`, { method: "POST" });
    } catch {
      // Manual rollback
      setLikes(previousRef.current.likes);
      setIsLiked(previousRef.current.isLiked);
    }
  }

  return (
    <button onClick={handleLike}>
      {isLiked ? "❤️" : "🤍"} {likes}
    </button>
  );
}
```

---

## Best Practices

1. **Use inside transitions or form actions** — `useOptimistic` is designed to work with React's async action system.
2. **Mark optimistic items visually** — show reduced opacity, a spinner, or "(saving...)" so users know the update is unconfirmed.
3. **Keep the update function pure** — it should only compute new state from inputs, no side effects.
4. **Pass server state as the first argument** — this is the source of truth that optimistic state resets to.
5. **Don't use for critical operations** — avoid optimistic updates for payments, deletes of important data, or irreversible actions where the user needs confirmation.
6. **Pair with error handling** — even though rollback is automatic, show the user a toast/notification when the server rejects.

---

## Common Mistakes

| Mistake                                     | Why It's Wrong                                   | Fix                                                      |
| ------------------------------------------- | ------------------------------------------------ | -------------------------------------------------------- |
| Using outside a transition/form action      | Optimistic state won't revert properly           | Wrap async work in `startTransition` or use form actions |
| Not showing pending state to users          | Users can't tell what's confirmed vs. optimistic | Add visual indicator (opacity, text label)               |
| Mutating state in update function           | Breaks React's immutability contract             | Return new object/array from updateFn                    |
| Using for destructive actions               | User sees success then rollback — confusing      | Use spinners for delete/payment operations               |
| Ignoring server errors                      | User doesn't know the action failed              | Show error toast even though state rolls back            |
| Managing separate optimistic state manually | Unnecessary complexity in React 19               | Use `useOptimistic` instead                              |

---

## Summary

- `useOptimistic` provides instant UI feedback by showing an optimistic state while an async action is pending.
- API: `const [optimisticState, addOptimistic] = useOptimistic(realState, updateFn)`.
- Rollback is automatic — when the action completes, `optimisticState` resets to the real `state` prop.
- Works best inside form actions or `startTransition` for proper lifecycle management.
- Always mark optimistic items visually and handle errors with user-facing notifications.
- Replaces manual optimistic patterns that required try/catch rollback logic.
