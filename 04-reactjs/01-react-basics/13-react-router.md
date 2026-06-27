# React Router DOM

## What Is Client-Side Routing?

In traditional websites, clicking a link sends a request to the server, which returns an entirely new HTML page. **Client-side routing** intercepts navigation and updates the UI by swapping components — without a full page reload.

**Analogy:** Traditional routing is like changing channels on an old TV — each channel switch goes through a brief static/blank screen (page reload). Client-side routing is like a streaming app that instantly switches between shows without any loading screen — only the content area changes.

---

## Why React Router?

| Traditional (Server-Side) Routing    | Client-Side Routing (React Router)                |
| ------------------------------------ | ------------------------------------------------- |
| Full page reload on every link click | Only the changed component re-renders             |
| Loses JavaScript state on navigation | State persists across route changes               |
| White flash between pages            | Instant, smooth transitions                       |
| Server handles every route           | Client handles navigation, server serves one HTML |

---

## Installation and Setup

```bash
npm install react-router-dom
```

### BrowserRouter Setup

Wrap your app in `BrowserRouter` — it provides the routing context:

```jsx
// main.jsx (entry point)
import { BrowserRouter } from "react-router-dom";
import App from "./App";

createRoot(document.getElementById("root")).render(
  <BrowserRouter>
    <App />
  </BrowserRouter>,
);
```

---

## Routes and Route Components

`Routes` is the container. `Route` defines a path → component mapping:

```jsx
import { Routes, Route } from "react-router-dom";

function App() {
  return (
    <div>
      <Navbar />
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
        <Route path="/products" element={<Products />} />
      </Routes>
      <Footer />
    </div>
  );
}
```

- `path` — the URL pattern to match.
- `element` — the JSX to render when the path matches.
- `<Navbar />` and `<Footer />` stay on screen — only the `<Routes>` area changes.

---

## Link vs Anchor Tag

### Why `<Link>` Instead of `<a>`?

```jsx
// ❌ <a> causes a full page reload — destroys React state
<a href="/about">About</a>;

// ✅ <Link> uses client-side navigation — no reload, state preserved
import { Link } from "react-router-dom";
<Link to="/about">About</Link>;
```

| Feature         | `<a href>`                 | `<Link to>`             |
| --------------- | -------------------------- | ----------------------- |
| Page reload     | Yes — full reload          | No — client-side swap   |
| State preserved | No — lost on reload        | Yes — maintained        |
| Performance     | Slow — re-downloads JS/CSS | Fast — instant swap     |
| Browser history | Managed by browser         | Managed by React Router |

### Basic Navigation

```jsx
import { Link } from "react-router-dom";

function Navbar() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
      <Link to="/products">Products</Link>
      <Link to="/contact">Contact</Link>
    </nav>
  );
}
```

---

## useNavigate for Programmatic Navigation

Navigate from code (after form submit, after login, conditionally):

```jsx
import { useNavigate } from "react-router-dom";

function LoginForm() {
  const navigate = useNavigate();

  const handleSubmit = async (e) => {
    e.preventDefault();
    const success = await loginUser(email, password);

    if (success) {
      navigate("/dashboard"); // Go to dashboard
    } else {
      navigate("/login?error=true"); // Stay on login with error
    }
  };

  return <form onSubmit={handleSubmit}>...</form>;
}
```

### navigate Options

```jsx
const navigate = useNavigate();

navigate("/products"); // Push new entry to history
navigate("/products", { replace: true }); // Replace current entry (no back)
navigate(-1); // Go back (like browser back button)
navigate(-2); // Go back 2 pages
navigate(1); // Go forward
```

### Passing State with Navigation

```jsx
// Send state with navigation
navigate("/order-confirmation", {
  state: { orderId: "12345", total: 499 },
});

// Receive state in the target component
import { useLocation } from "react-router-dom";

function OrderConfirmation() {
  const location = useLocation();
  const { orderId, total } = location.state || {};

  return (
    <div>
      <h1>Order Confirmed!</h1>
      <p>Order ID: {orderId}</p>
      <p>Total: ₹{total}</p>
    </div>
  );
}
```

---

## Route Parameters with useParams

Dynamic segments in the URL:

```jsx
import { Routes, Route } from "react-router-dom";

function App() {
  return (
    <Routes>
      <Route path="/users/:userId" element={<UserProfile />} />
      <Route
        path="/products/:category/:productId"
        element={<ProductDetail />}
      />
    </Routes>
  );
}
```

```jsx
import { useParams } from "react-router-dom";

function UserProfile() {
  const { userId } = useParams(); // Extracts from URL

  // URL: /users/42 → userId = "42"
  return <h1>User ID: {userId}</h1>;
}

function ProductDetail() {
  const { category, productId } = useParams();

  // URL: /products/electronics/101 → category="electronics", productId="101"
  return (
    <div>
      <p>Category: {category}</p>
      <p>Product: {productId}</p>
    </div>
  );
}
```

### Fetching Data Based on Route Params

```jsx
function UserProfile() {
  const { userId } = useParams();
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then((res) => res.json())
      .then((data) => setUser(data));
  }, [userId]); // Re-fetch when URL param changes

  if (!user) return <p>Loading...</p>;
  return <h1>{user.name}</h1>;
}
```

---

## Nested Routes and Outlet

For layouts with sub-navigation (dashboard with tabs, settings pages):

```jsx
import { Routes, Route, Outlet, Link } from "react-router-dom";

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />

      {/* Parent route with nested children */}
      <Route path="/dashboard" element={<DashboardLayout />}>
        <Route index element={<DashboardHome />} /> {/* /dashboard */}
        <Route path="analytics" element={<Analytics />} />{" "}
        {/* /dashboard/analytics */}
        <Route path="settings" element={<Settings />} />{" "}
        {/* /dashboard/settings */}
        <Route path="users/:id" element={<UserDetail />} />{" "}
        {/* /dashboard/users/42 */}
      </Route>
    </Routes>
  );
}
```

```jsx
function DashboardLayout() {
  return (
    <div className="dashboard">
      <aside>
        <nav>
          <Link to="/dashboard">Overview</Link>
          <Link to="/dashboard/analytics">Analytics</Link>
          <Link to="/dashboard/settings">Settings</Link>
        </nav>
      </aside>

      <main>
        {/* Outlet renders the matched child route */}
        <Outlet />
      </main>
    </div>
  );
}
```

### How Outlet Works

```
URL: /dashboard/analytics

Renders:
<DashboardLayout>          ← parent route element
  <aside>...</aside>       ← always visible
  <main>
    <Analytics />          ← <Outlet /> renders the matched child
  </main>
</DashboardLayout>
```

### Index Route

The `index` route renders when the parent path matches exactly (no sub-path):

```jsx
<Route path="/dashboard" element={<DashboardLayout />}>
  <Route index element={<DashboardHome />} /> {/* Matches /dashboard exactly */}
  <Route path="stats" element={<Stats />} /> {/* Matches /dashboard/stats */}
</Route>
```

---

## Query Parameters with useSearchParams

For filters, pagination, search — data in the URL after `?`:

```jsx
import { useSearchParams } from "react-router-dom";

function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();

  // Read query params: /products?category=shoes&sort=price
  const category = searchParams.get("category") || "all";
  const sort = searchParams.get("sort") || "name";
  const page = Number(searchParams.get("page")) || 1;

  // Update query params
  const changeCategory = (newCategory) => {
    setSearchParams({ category: newCategory, sort, page: 1 });
  };

  const nextPage = () => {
    setSearchParams({ category, sort, page: page + 1 });
  };

  return (
    <div>
      <select value={category} onChange={(e) => changeCategory(e.target.value)}>
        <option value="all">All</option>
        <option value="shoes">Shoes</option>
        <option value="clothing">Clothing</option>
      </select>

      <p>
        Category: {category}, Sort: {sort}, Page: {page}
      </p>
      <button onClick={nextPage}>Next Page</button>
    </div>
  );
}
```

**Benefit:** Users can share/bookmark URLs with filters applied — `/products?category=shoes&page=2`.

---

## 404 / Catch-All Route

Handle unmatched URLs with a wildcard `*` route:

```jsx
function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="/products" element={<Products />} />

      {/* Catch-all — matches any URL not matched above */}
      <Route path="*" element={<NotFound />} />
    </Routes>
  );
}

function NotFound() {
  const navigate = useNavigate();

  return (
    <div className="not-found">
      <h1>404 — Page Not Found</h1>
      <p>The page you're looking for doesn't exist.</p>
      <button onClick={() => navigate("/")}>Go Home</button>
    </div>
  );
}
```

### Nested 404

```jsx
<Route path="/dashboard" element={<DashboardLayout />}>
  <Route index element={<DashboardHome />} />
  <Route path="settings" element={<Settings />} />
  {/* 404 within dashboard */}
  <Route path="*" element={<DashboardNotFound />} />
</Route>
```

---

## Active Link Styling with NavLink

`NavLink` is like `Link` but adds styling when the route is active:

```jsx
import { NavLink } from "react-router-dom";

function Navbar() {
  return (
    <nav>
      {/* className receives an object with isActive and isPending */}
      <NavLink
        to="/"
        className={({ isActive }) =>
          isActive ? "nav-link active" : "nav-link"
        }
      >
        Home
      </NavLink>

      <NavLink
        to="/about"
        className={({ isActive }) =>
          isActive ? "nav-link active" : "nav-link"
        }
      >
        About
      </NavLink>

      <NavLink
        to="/products"
        className={({ isActive }) =>
          isActive ? "nav-link active" : "nav-link"
        }
      >
        Products
      </NavLink>
    </nav>
  );
}
```

### Inline Styles

```jsx
<NavLink
  to="/dashboard"
  style={({ isActive }) => ({
    fontWeight: isActive ? "bold" : "normal",
    color: isActive ? "#3b82f6" : "#666",
    borderBottom: isActive ? "2px solid #3b82f6" : "none",
  })}
>
  Dashboard
</NavLink>
```

### Reusable NavLink Component

```jsx
function AppNavLink({ to, children }) {
  return (
    <NavLink
      to={to}
      className={({ isActive }) =>
        `px-4 py-2 rounded ${isActive ? "bg-blue-500 text-white" : "text-gray-600 hover:bg-gray-100"}`
      }
    >
      {children}
    </NavLink>
  );
}

// Usage
<AppNavLink to="/">Home</AppNavLink>
<AppNavLink to="/about">About</AppNavLink>
```

---

## Protected Routes Pattern

Redirect unauthenticated users away from private pages:

```jsx
import { Navigate, Outlet } from "react-router-dom";
import { useAuth } from "./AuthContext";

function ProtectedRoute() {
  const { user, loading } = useAuth();

  if (loading) return <p>Loading...</p>;
  if (!user) return <Navigate to="/login" replace />;

  return <Outlet />; // Render child routes if authenticated
}

// Usage
function App() {
  return (
    <Routes>
      <Route path="/login" element={<Login />} />
      <Route path="/register" element={<Register />} />

      {/* All routes inside are protected */}
      <Route element={<ProtectedRoute />}>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/profile" element={<Profile />} />
        <Route path="/settings" element={<Settings />} />
      </Route>

      <Route path="*" element={<NotFound />} />
    </Routes>
  );
}
```

---

## Best Practices

1. **Use `<Link>` and `<NavLink>`** — never `<a>` for internal navigation.
2. **Use nested routes** for shared layouts (dashboard, admin panels) — keeps code DRY.
3. **Always include a `*` catch-all route** — graceful 404 handling.
4. **Use `useSearchParams` for filters/pagination** — makes URLs shareable and bookmarkable.
5. **Use `replace: true`** for redirects after login — prevents "back" button going to login again.
6. **Keep route definitions centralized** — easier to understand the app's navigation structure.
7. **Use `index` routes** for default content in nested layouts.
8. **Lazy load route components** — combine with `React.lazy` and `Suspense` for code splitting.

---

## Common Mistakes

| Mistake                                                 | Why It's Wrong                                       |
| ------------------------------------------------------- | ---------------------------------------------------- |
| Using `<a href>` for internal links                     | Causes full page reload — loses state, slow          |
| Placing `<Routes>` outside `<BrowserRouter>`            | Router context not available — crashes               |
| Forgetting `<Outlet />` in layout routes                | Child routes have nowhere to render                  |
| Not handling 404 (no `*` route)                         | Blank screen on unknown URLs                         |
| Hardcoding URLs instead of using route params           | Brittle — breaking changes when paths change         |
| Putting routes in wrong order (catch-all first)         | `*` matches everything — put it last                 |
| Using `useNavigate` in render (not in handlers/effects) | Can cause infinite loops or navigation during render |

---

## Summary

- **React Router** enables client-side navigation without page reloads.
- Use **`<Link>`** instead of `<a>` for internal navigation — preserves state and performance.
- **Route params** (`:id`) capture dynamic URL segments; read them with `useParams()`.
- **Nested routes** with `<Outlet />` create shared layouts (sidebars, dashboards).
- **`useSearchParams`** manages query parameters for filters, search, and pagination.
- **`useNavigate`** handles programmatic navigation (after form submit, conditionally).
- **`NavLink`** provides active state styling for navigation menus.
- Always include a **`*` catch-all route** for 404 handling.
