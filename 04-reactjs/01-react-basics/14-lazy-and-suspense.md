# Lazy Loading & Suspense

## What Is Code Splitting?

By default, bundlers (Vite, Webpack) combine all your JavaScript into **one large file**. As your app grows, this bundle becomes huge — users download megabytes of code before seeing anything, even code for pages they may never visit.

**Code splitting** breaks this single bundle into smaller **chunks** that load on demand — only when needed.

**Analogy:** Imagine a library that forces you to check out ALL books before you can read one. Code splitting is like the library giving you one book at a time — you only carry what you're actually reading. The rest stays on the shelf until you ask for it.

---

## Why Code Splitting Matters

| Without Code Splitting                  | With Code Splitting                        |
| --------------------------------------- | ------------------------------------------ |
| One giant bundle (2MB+)                 | Small initial bundle (~200KB)              |
| Slow first load — entire app downloaded | Fast first load — only current page loaded |
| Users wait for code they'll never use   | Code loads on demand when needed           |
| Time-to-interactive is high             | Time-to-interactive is low                 |

### Real Impact Example

```
Without splitting:
  bundle.js — 1.8MB (includes Dashboard, Admin, Reports, Settings...)

With splitting:
  main.js — 180KB (core app + current route)
  dashboard.chunk.js — 95KB (loaded when visiting /dashboard)
  admin.chunk.js — 120KB (loaded when visiting /admin)
  reports.chunk.js — 200KB (loaded when visiting /reports)
```

---

## React.lazy() for Dynamic Imports

`React.lazy()` lets you define a component that is loaded dynamically — its code is fetched only when it's first rendered:

```jsx
import { lazy } from "react";

// Instead of static import:
// import Dashboard from "./pages/Dashboard";

// Dynamic import — creates a separate chunk
const Dashboard = lazy(() => import("./pages/Dashboard"));
const AdminPanel = lazy(() => import("./pages/AdminPanel"));
const Reports = lazy(() => import("./pages/Reports"));
const Settings = lazy(() => import("./pages/Settings"));
```

### How It Works

1. `import("./pages/Dashboard")` returns a **Promise** that resolves to the module.
2. `React.lazy()` wraps this Promise — the component loads only when React tries to render it.
3. The bundler (Vite/Webpack) automatically creates a **separate chunk** for each dynamic import.

### Requirements

The dynamically imported module must have a **default export** that is a React component:

```jsx
// ✅ pages/Dashboard.jsx — default export
export default function Dashboard() {
  return <h1>Dashboard</h1>;
}

// ❌ Named exports don't work directly with lazy
export function Dashboard() { ... }

// Workaround for named exports:
const Dashboard = lazy(() =>
  import("./pages/Dashboard").then(module => ({
    default: module.Dashboard,
  }))
);
```

---

## Suspense Component with Fallback

Since lazy components load asynchronously, React needs something to show while waiting. `Suspense` provides a **fallback UI** during loading:

```jsx
import { lazy, Suspense } from "react";

const Dashboard = lazy(() => import("./pages/Dashboard"));

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Dashboard />
    </Suspense>
  );
}

function LoadingSpinner() {
  return (
    <div className="loading">
      <div className="spinner"></div>
      <p>Loading...</p>
    </div>
  );
}
```

### Multiple Lazy Components Under One Suspense

```jsx
function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/admin" element={<AdminPanel />} />
        <Route path="/reports" element={<Reports />} />
      </Routes>
    </Suspense>
  );
}
```

### Nested Suspense Boundaries

Different loading states for different sections:

```jsx
function App() {
  return (
    <div>
      {/* Main page loading */}
      <Suspense fallback={<PageSkeleton />}>
        <Header />
        <main>
          <Suspense fallback={<SidebarSkeleton />}>
            <Sidebar />
          </Suspense>

          <Suspense fallback={<ContentSkeleton />}>
            <MainContent />
          </Suspense>
        </main>
      </Suspense>
    </div>
  );
}
```

---

## Route-Based Code Splitting

The most common and impactful pattern — split by route since users visit one page at a time:

```jsx
import { lazy, Suspense } from "react";
import { BrowserRouter, Routes, Route } from "react-router-dom";

// Eagerly loaded (always needed)
import Navbar from "./components/Navbar";
import Footer from "./components/Footer";

// Lazy loaded (loaded on demand per route)
const Home = lazy(() => import("./pages/Home"));
const About = lazy(() => import("./pages/About"));
const Dashboard = lazy(() => import("./pages/Dashboard"));
const ProductList = lazy(() => import("./pages/ProductList"));
const ProductDetail = lazy(() => import("./pages/ProductDetail"));
const AdminPanel = lazy(() => import("./pages/AdminPanel"));
const NotFound = lazy(() => import("./pages/NotFound"));

function App() {
  return (
    <BrowserRouter>
      <Navbar /> {/* Always loaded — part of every page */}
      <Suspense fallback={<PageLoader />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/products" element={<ProductList />} />
          <Route path="/products/:id" element={<ProductDetail />} />
          <Route path="/admin" element={<AdminPanel />} />
          <Route path="*" element={<NotFound />} />
        </Routes>
      </Suspense>
      <Footer /> {/* Always loaded */}
    </BrowserRouter>
  );
}

function PageLoader() {
  return (
    <div className="page-loader">
      <div className="shimmer-block" style={{ height: "60vh" }} />
    </div>
  );
}
```

### With Nested Routes

```jsx
const DashboardHome = lazy(() => import("./pages/dashboard/DashboardHome"));
const Analytics = lazy(() => import("./pages/dashboard/Analytics"));
const Settings = lazy(() => import("./pages/dashboard/Settings"));

function App() {
  return (
    <Routes>
      <Route path="/dashboard" element={<DashboardLayout />}>
        <Route
          index
          element={
            <Suspense fallback={<ContentSkeleton />}>
              <DashboardHome />
            </Suspense>
          }
        />
        <Route
          path="analytics"
          element={
            <Suspense fallback={<ContentSkeleton />}>
              <Analytics />
            </Suspense>
          }
        />
        <Route
          path="settings"
          element={
            <Suspense fallback={<ContentSkeleton />}>
              <Settings />
            </Suspense>
          }
        />
      </Route>
    </Routes>
  );
}
```

---

## Skeleton UI Concept (Loading Placeholders)

Instead of a generic spinner, skeleton screens show the **shape** of the content that's loading. This feels faster to users because they can anticipate the layout.

```jsx
// Generic skeleton components
function SkeletonText({ width = "100%" }) {
  return (
    <div
      className="skeleton"
      style={{ width, height: "1rem", borderRadius: "4px" }}
    />
  );
}

function SkeletonCard() {
  return (
    <div className="skeleton-card">
      <div className="skeleton" style={{ height: "200px" }} /> {/* Image */}
      <SkeletonText width="80%" /> {/* Title */}
      <SkeletonText width="60%" /> {/* Subtitle */}
      <SkeletonText width="40%" /> {/* Price */}
    </div>
  );
}

function ProductPageSkeleton() {
  return (
    <div className="product-grid">
      {Array.from({ length: 8 }).map((_, i) => (
        <SkeletonCard key={i} />
      ))}
    </div>
  );
}
```

```css
/* Skeleton animation */
.skeleton {
  background: linear-gradient(90deg, #e0e0e0 25%, #f0f0f0 50%, #e0e0e0 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
  border-radius: 4px;
}

@keyframes shimmer {
  0% {
    background-position: 200% 0;
  }
  100% {
    background-position: -200% 0;
  }
}
```

### Using Skeletons as Suspense Fallbacks

```jsx
<Suspense fallback={<ProductPageSkeleton />}>
  <ProductList />
</Suspense>

<Suspense fallback={<DashboardSkeleton />}>
  <Dashboard />
</Suspense>
```

---

## Error Boundaries for Lazy Components

If a lazy component fails to load (network error, chunk not found), you need an **error boundary** to catch it gracefully:

```jsx
import { Component } from "react";

class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error("Lazy load failed:", error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        this.props.fallback || (
          <div className="error-fallback">
            <h2>Something went wrong</h2>
            <p>Failed to load this section. Please try again.</p>
            <button onClick={() => this.setState({ hasError: false })}>
              Retry
            </button>
          </div>
        )
      );
    }

    return this.props.children;
  }
}
```

### Combining Error Boundary + Suspense

```jsx
function App() {
  return (
    <ErrorBoundary fallback={<ErrorPage />}>
      <Suspense fallback={<PageLoader />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Suspense>
    </ErrorBoundary>
  );
}
```

### Per-Route Error Handling

```jsx
function LazyRoute({ component: Component, fallback, errorFallback }) {
  return (
    <ErrorBoundary fallback={errorFallback || <DefaultError />}>
      <Suspense fallback={fallback || <PageLoader />}>
        <Component />
      </Suspense>
    </ErrorBoundary>
  );
}

// Usage
<Route
  path="/dashboard"
  element={
    <LazyRoute
      component={Dashboard}
      fallback={<DashboardSkeleton />}
      errorFallback={<DashboardError />}
    />
  }
/>;
```

---

## When to Use Lazy Loading

### ✅ Good Candidates for Lazy Loading

| Scenario                           | Why                                        |
| ---------------------------------- | ------------------------------------------ |
| Route-level pages                  | Users visit one page at a time             |
| Admin panels / dashboards          | Only admin users access them               |
| Modals and dialogs                 | Not visible on initial load                |
| Heavy components (charts, editors) | Large dependencies loaded only when needed |
| Below-the-fold content             | Not visible without scrolling              |
| Feature flags / A/B tests          | Only some users see them                   |

### ❌ Don't Lazy Load

| Scenario                                       | Why                               |
| ---------------------------------------------- | --------------------------------- |
| Small, frequently used components              | Loading overhead > bundle savings |
| Components visible on initial render           | Adds unnecessary loading state    |
| Critical above-the-fold content                | Delays first meaningful paint     |
| Components used on every page (Navbar, Footer) | Better to include in main bundle  |

---

## Advanced: Preloading Lazy Components

Load a component **before** the user navigates — on hover or when idle:

```jsx
const Dashboard = lazy(() => import("./pages/Dashboard"));

// Preload on hover
function Navbar() {
  const preloadDashboard = () => {
    import("./pages/Dashboard"); // Starts downloading the chunk
  };

  return (
    <nav>
      <Link to="/">Home</Link>
      <Link
        to="/dashboard"
        onMouseEnter={preloadDashboard} // Preload on hover
      >
        Dashboard
      </Link>
    </nav>
  );
}
```

### Preload on Idle

```jsx
// Preload components when the browser is idle
function usePreloadOnIdle(importFn) {
  useEffect(() => {
    const id = requestIdleCallback(() => {
      importFn();
    });
    return () => cancelIdleCallback(id);
  }, [importFn]);
}

// Usage
function App() {
  usePreloadOnIdle(() => import("./pages/Dashboard"));
  usePreloadOnIdle(() => import("./pages/AdminPanel"));
  // ...
}
```

---

## Complete Example: App with Lazy Loading

```jsx
import { lazy, Suspense } from "react";
import { BrowserRouter, Routes, Route, NavLink } from "react-router-dom";

// Always loaded
import Navbar from "./components/Navbar";

// Lazy loaded
const Home = lazy(() => import("./pages/Home"));
const Products = lazy(() => import("./pages/Products"));
const ProductDetail = lazy(() => import("./pages/ProductDetail"));
const Cart = lazy(() => import("./pages/Cart"));
const Checkout = lazy(() => import("./pages/Checkout"));
const NotFound = lazy(() => import("./pages/NotFound"));

function App() {
  return (
    <BrowserRouter>
      <Navbar />
      <ErrorBoundary>
        <Suspense
          fallback={
            <div className="page-skeleton">
              <div className="skeleton" style={{ height: "80vh" }} />
            </div>
          }
        >
          <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/products" element={<Products />} />
            <Route path="/products/:id" element={<ProductDetail />} />
            <Route path="/cart" element={<Cart />} />
            <Route path="/checkout" element={<Checkout />} />
            <Route path="*" element={<NotFound />} />
          </Routes>
        </Suspense>
      </ErrorBoundary>
    </BrowserRouter>
  );
}

export default App;
```

---

## Best Practices

1. **Split at the route level first** — biggest impact with least complexity.
2. **Use meaningful skeleton fallbacks** over generic spinners — feels faster to users.
3. **Wrap with error boundaries** — handle chunk load failures gracefully.
4. **Don't lazy load tiny components** — the loading overhead outweighs the bundle savings.
5. **Preload on hover** for likely navigation targets — eliminates perceived loading time.
6. **Keep the main bundle small** — lazy load anything not needed for the initial view.
7. **Test on slow networks** — use Chrome DevTools throttling to verify the loading experience.
8. **Name your chunks** for easier debugging:
   ```jsx
   const Dashboard = lazy(
     () => import(/* webpackChunkName: "dashboard" */ "./pages/Dashboard"),
   );
   ```

---

## Common Mistakes

| Mistake                                            | Why It's Wrong                                                |
| -------------------------------------------------- | ------------------------------------------------------------- |
| Lazy loading everything (including Navbar, Footer) | Critical UI delays for tiny savings                           |
| No Suspense boundary around lazy components        | React throws an error — Suspense is required                  |
| No error boundary                                  | Network failures crash the entire app                         |
| Using lazy inside a component function             | Creates a new lazy component on every render — never resolves |
| Generic "Loading..." text instead of skeletons     | Feels slower — layout shift when content appears              |
| Not testing on slow connections                    | Loading states may be invisible on fast dev machines          |

### The "Lazy Inside Component" Mistake

```jsx
// ❌ WRONG — creates new lazy component every render
function App() {
  const Page = lazy(() => import("./Page")); // Recreated each render!
  return (
    <Suspense fallback={<Loader />}>
      <Page />
    </Suspense>
  );
}

// ✅ CORRECT — define lazy outside the component
const Page = lazy(() => import("./Page"));

function App() {
  return (
    <Suspense fallback={<Loader />}>
      <Page />
    </Suspense>
  );
}
```

---

## Summary

- **Code splitting** breaks one large bundle into smaller chunks loaded on demand.
- `React.lazy(() => import("./Component"))` creates a dynamically-loaded component.
- **`Suspense`** shows a fallback UI while the lazy component loads — required for lazy components.
- **Route-based splitting** is the most impactful pattern — one chunk per page.
- Use **skeleton UIs** as fallbacks for a smoother loading experience.
- Wrap lazy components in **error boundaries** to handle network failures.
- **Preload on hover** for instant-feeling navigation.
- Define `lazy()` calls **outside** component functions — never inside render.
