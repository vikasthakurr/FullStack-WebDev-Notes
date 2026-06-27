# Props (Passing Data)

## What Are Props?

Props (short for "properties") are the mechanism for passing data from a **parent** component to a **child** component. They are the inputs to a component — just like arguments to a function.

**Analogy:** Think of a component as a vending machine. Props are the buttons you press — you provide inputs (selection, money) and the machine gives you an output (the drink). The machine doesn't decide what you want; you tell it through props.

---

## Why Props Matter

| Without Props                       | With Props                                  |
| ----------------------------------- | ------------------------------------------- |
| Every component is hardcoded        | Components are reusable with different data |
| Duplication everywhere              | One component, many uses                    |
| No communication between components | Parent controls child behavior              |
| Static, inflexible UI               | Dynamic, data-driven UI                     |

---

## Passing Props Parent → Child

Props are passed as attributes on JSX elements and received as a single object parameter:

```jsx
// Parent component
function App() {
  return (
    <div>
      <UserCard name="Vikas" age={25} isActive={true} />
      <UserCard name="Priya" age={22} isActive={false} />
      <UserCard name="Rahul" age={28} isActive={true} />
    </div>
  );
}

// Child component — receives all props as one object
function UserCard(props) {
  return (
    <div className={`card ${props.isActive ? "active" : ""}`}>
      <h2>{props.name}</h2>
      <p>Age: {props.age}</p>
      <span>{props.isActive ? "🟢 Online" : "🔴 Offline"}</span>
    </div>
  );
}
```

### What Can Be Passed as Props?

Anything JavaScript can hold:

```jsx
<Component
  text="hello" // String
  count={42} // Number
  isActive={true} // Boolean
  items={["a", "b", "c"]} // Array
  user={{ name: "Vikas" }} // Object
  onClick={() => alert("hi")} // Function
  icon={<Icon />} // JSX element
/>
```

---

## Destructuring Props

Instead of writing `props.name`, `props.age` everywhere, destructure in the parameter:

```jsx
// ❌ Verbose — repeating "props." everywhere
function UserCard(props) {
  return (
    <h2>
      {props.name}, {props.age}
    </h2>
  );
}

// ✅ Clean — destructure in the parameter
function UserCard({ name, age, isActive }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      {isActive && <span>🟢 Active</span>}
    </div>
  );
}
```

### Destructuring with Rename

```jsx
function UserCard({ name: userName, age: userAge }) {
  return (
    <p>
      {userName} is {userAge} years old
    </p>
  );
}
```

---

## Default Props

Provide fallback values when a prop is not passed:

```jsx
// Method 1: Default in destructuring (recommended)
function Button({ text = "Click Me", variant = "primary", size = "md" }) {
  return <button className={`btn btn-${variant} btn-${size}`}>{text}</button>;
}

// Usage
<Button />                          // Uses all defaults
<Button text="Submit" />            // Overrides text, keeps other defaults
<Button variant="danger" size="lg" text="Delete" />
```

```jsx
// Method 2: Default with || or ?? (inside component)
function Greeting({ name }) {
  const displayName = name ?? "Stranger";
  return <h1>Hello, {displayName}!</h1>;
}
```

---

## The `children` Prop

`children` is a special prop that contains anything placed **between** the opening and closing tags of a component:

```jsx
// A reusable Card wrapper
function Card({ title, children }) {
  return (
    <div className="card">
      <h2 className="card-title">{title}</h2>
      <div className="card-body">{children}</div>
    </div>
  );
}

// Usage — anything between <Card> and </Card> becomes children
function App() {
  return (
    <Card title="Profile">
      <img src="avatar.jpg" alt="User avatar" />
      <p>Welcome back, Vikas!</p>
      <button>Edit Profile</button>
    </Card>
  );
}
```

### Composition Pattern with Children

```jsx
function Modal({ isOpen, onClose, children }) {
  if (!isOpen) return null;

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        <button className="modal-close" onClick={onClose}>
          ×
        </button>
        {children}
      </div>
    </div>
  );
}

// Flexible — any content can go inside
<Modal isOpen={showModal} onClose={() => setShowModal(false)}>
  <h2>Confirm Delete</h2>
  <p>Are you sure you want to delete this item?</p>
  <button onClick={handleDelete}>Yes, Delete</button>
</Modal>;
```

### Layout Pattern

```jsx
function PageLayout({ header, sidebar, children }) {
  return (
    <div className="layout">
      <header>{header}</header>
      <aside>{sidebar}</aside>
      <main>{children}</main>
    </div>
  );
}

<PageLayout header={<Navbar />} sidebar={<SideMenu />}>
  <h1>Page Content</h1>
  <p>This goes into the main area</p>
</PageLayout>;
```

---

## Props Are Read-Only (One-Way Data Flow)

A child component **cannot modify** its props. Data flows in one direction: parent → child.

```jsx
// ❌ NEVER do this — props are immutable
function UserCard({ name }) {
  name = "Hacked"; // This violates React's contract
  return <h2>{name}</h2>;
}

// ✅ If a child needs to "change" something, the parent passes a callback
function Parent() {
  const [name, setName] = useState("Vikas");

  return <ChildEditor name={name} onNameChange={setName} />;
}

function ChildEditor({ name, onNameChange }) {
  return (
    <input
      value={name}
      onChange={(e) => onNameChange(e.target.value)} // Tells parent to update
    />
  );
}
```

### One-Way Data Flow Diagram

```
┌─────────────────────────────┐
│         Parent              │
│   state: { name: "Vikas" } │
│                             │
│   ┌─── props (data down) ──┼──→ Child receives name="Vikas"
│   │                         │
│   ←── callback (events up) ─┼──← Child calls onNameChange("Priya")
│                             │
│   state updates → re-render │
└─────────────────────────────┘
```

---

## Props Drilling Problem

Props drilling happens when you pass props through multiple intermediate components that **don't use the data themselves** — they just pass it along.

```jsx
// The problem: App → Layout → Sidebar → UserInfo → Avatar
// Every component in the chain must accept and forward the prop

function App() {
  const [user, setUser] = useState({ name: "Vikas", avatar: "url" });

  return <Layout user={user} />; // Layout doesn't need user
}

function Layout({ user }) {
  return (
    <div>
      <Header />
      <Sidebar user={user} /> {/* Sidebar doesn't need user either */}
    </div>
  );
}

function Sidebar({ user }) {
  return (
    <nav>
      <Menu />
      <UserInfo user={user} /> {/* Just passing it along... */}
    </nav>
  );
}

function UserInfo({ user }) {
  return (
    <div>
      <Avatar src={user.avatar} /> {/* Finally used here! */}
      <span>{user.name}</span>
    </div>
  );
}
```

### Why It's a Problem

- **Maintenance nightmare** — renaming a prop means changing every component in the chain.
- **Unnecessary coupling** — intermediate components know about data they don't use.
- **Performance** — intermediate components may re-render unnecessarily.
- **Readability** — harder to trace where data comes from.

---

## Solutions to Props Drilling

### 1. Component Composition (No library needed)

Move the child component up and pass it as children or a prop:

```jsx
function App() {
  const [user, setUser] = useState({ name: "Vikas", avatar: "url" });

  // Build the component at the top level where data lives
  return (
    <Layout
      sidebar={
        <Sidebar>
          <UserInfo user={user} />
        </Sidebar>
      }
    />
  );
}

function Layout({ sidebar }) {
  return (
    <div>
      <Header />
      {sidebar}
    </div>
  );
}
```

### 2. Context API (Built into React)

Share state across the tree without passing through every level:

```jsx
const UserContext = createContext();

function App() {
  const [user, setUser] = useState({ name: "Vikas", avatar: "url" });

  return (
    <UserContext.Provider value={user}>
      <Layout />
    </UserContext.Provider>
  );
}

// Any deeply nested component can access user directly
function Avatar() {
  const user = useContext(UserContext);
  return <img src={user.avatar} alt={user.name} />;
}
```

### 3. State Management Libraries (Redux, Zustand, Jotai)

For large apps with complex state interactions:

```jsx
// Zustand example — global store accessible anywhere
import { create } from "zustand";

const useUserStore = create((set) => ({
  user: { name: "Vikas", avatar: "url" },
  setUser: (user) => set({ user }),
}));

// Any component can read/write without props
function Avatar() {
  const user = useUserStore((state) => state.user);
  return <img src={user.avatar} alt={user.name} />;
}
```

### When to Use Which Solution

| Solution        | When to Use                                    |
| --------------- | ---------------------------------------------- |
| Composition     | 1–2 levels of drilling; restructuring helps    |
| Context API     | Theme, auth, locale — read by many components  |
| Redux / Zustand | Complex state with many actions across the app |

---

## Best Practices

1. **Destructure props** in the function parameter for cleaner code.
2. **Use `children`** for wrapper/layout components — makes them flexible and composable.
3. **Keep props minimal** — pass only what the child needs, not entire objects when a single field suffices.
4. **Name callback props with `on` prefix** — `onClick`, `onSubmit`, `onChange`, `onNameChange`.
5. **Use composition before Context** — often restructuring components solves drilling without adding complexity.
6. **Document expected props** with TypeScript or PropTypes for better DX and error messages.
7. **Avoid spreading all props blindly** (`{...props}`) — it hides what data flows where and makes debugging harder.

---

## Common Mistakes

| Mistake                                                            | Why It's Wrong                                                             |
| ------------------------------------------------------------------ | -------------------------------------------------------------------------- |
| Mutating props inside a child component                            | Breaks one-way data flow — unpredictable bugs                              |
| Passing unnecessary props through intermediate components          | Creates coupling and maintenance burden (props drilling)                   |
| Forgetting that missing boolean props are `undefined`, not `false` | `<Comp active />` means `active={true}`, but omitting it gives `undefined` |
| Using index as key when rendering prop arrays                      | Causes bugs when items reorder — use unique IDs                            |
| Not providing default values for optional props                    | Component crashes with `Cannot read property of undefined`                 |
| Over-drilling when Context or composition would be simpler         | Adds complexity to every component in the chain                            |

---

## Summary

- **Props** are inputs from parent to child — like function arguments.
- **Destructure** props in the parameter list for clean, readable components.
- Use **`children`** for flexible wrapper components (modals, cards, layouts).
- Props are **read-only** — data flows one way (parent → child). Children communicate back via **callback functions**.
- **Props drilling** is passing data through many levels — solve it with composition, Context API, or state management libraries.
- **Default props** prevent undefined errors and make components safer to use.
- Think of props as the component's **public API** — keep it clear and minimal.
