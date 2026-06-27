# Express Router (Modular Routing)

## What is Express Router?

`express.Router()` is a mini Express application that handles only routes and middleware. Think of it like departments in a company — instead of the CEO (main `app`) handling every single task, each department (router) manages its own domain independently.

Without Router, all routes live in one file. As your app grows, this becomes unmanageable. Router lets you split routes into separate files organized by feature or resource.

```javascript
// Without Router — everything in one file (messy at scale)
app.get('/api/users', ...);
app.post('/api/users', ...);
app.get('/api/products', ...);
app.post('/api/products', ...);
app.get('/api/orders', ...);
// 50+ more routes...

// With Router — clean, modular separation
app.use('/api/users', userRouter);
app.use('/api/products', productRouter);
app.use('/api/orders', orderRouter);
```

---

## Creating a Router

```javascript
const express = require("express");
const router = express.Router();

// Define routes on the router (paths are relative to the mount point)
router.get("/", (req, res) => {
  res.json({ message: "Get all users" });
});

router.get("/:id", (req, res) => {
  res.json({ message: `Get user ${req.params.id}` });
});

router.post("/", (req, res) => {
  res.status(201).json({ message: "User created" });
});

router.put("/:id", (req, res) => {
  res.json({ message: `User ${req.params.id} updated` });
});

router.delete("/:id", (req, res) => {
  res.status(204).send();
});

module.exports = router;
```

---

## Separating Routes into Files

### Project Structure

```
project/
├── src/
│   ├── routes/
│   │   ├── index.js          ← Central route registry
│   │   ├── userRoutes.js     ← User routes
│   │   ├── productRoutes.js  ← Product routes
│   │   └── orderRoutes.js    ← Order routes
│   ├── controllers/
│   │   ├── userController.js
│   │   ├── productController.js
│   │   └── orderController.js
│   └── app.js                ← Main Express app
├── package.json
└── server.js
```

### routes/userRoutes.js

```javascript
const express = require("express");
const router = express.Router();
const userController = require("../controllers/userController");

router.get("/", userController.getAllUsers);
router.get("/:id", userController.getUserById);
router.post("/", userController.createUser);
router.put("/:id", userController.updateUser);
router.delete("/:id", userController.deleteUser);

module.exports = router;
```

### controllers/userController.js

```javascript
let users = [
  { id: 1, name: "Vikas", email: "vikas@example.com" },
  { id: 2, name: "Priya", email: "priya@example.com" },
];

exports.getAllUsers = (req, res) => {
  res.json({ data: users });
};

exports.getUserById = (req, res) => {
  const user = users.find((u) => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ error: "User not found" });
  res.json({ data: user });
};

exports.createUser = (req, res) => {
  const { name, email } = req.body;
  const newUser = { id: users.length + 1, name, email };
  users.push(newUser);
  res.status(201).json({ data: newUser });
};

exports.updateUser = (req, res) => {
  const user = users.find((u) => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ error: "User not found" });

  user.name = req.body.name || user.name;
  user.email = req.body.email || user.email;
  res.json({ data: user });
};

exports.deleteUser = (req, res) => {
  users = users.filter((u) => u.id !== parseInt(req.params.id));
  res.status(204).send();
};
```

---

## Mounting Routers with app.use()

The **mount path** in `app.use()` acts as a prefix. Routes inside the router are relative to this prefix.

### app.js

```javascript
const express = require("express");
const app = express();

// Middleware
app.use(express.json());

// Import routers
const userRoutes = require("./routes/userRoutes");
const productRoutes = require("./routes/productRoutes");
const orderRoutes = require("./routes/orderRoutes");

// Mount routers with prefixes
app.use("/api/users", userRoutes); // /api/users, /api/users/:id
app.use("/api/products", productRoutes); // /api/products, /api/products/:id
app.use("/api/orders", orderRoutes); // /api/orders, /api/orders/:id

// Error handling
app.use((err, req, res, next) => {
  res.status(err.status || 500).json({ error: err.message });
});

module.exports = app;
```

### How Paths Work

```javascript
// In userRoutes.js
router.get("/"); // Matches: GET /api/users
router.get("/:id"); // Matches: GET /api/users/5
router.get("/search"); // Matches: GET /api/users/search

// The router doesn't know about "/api/users" — it only sees "/" and "/:id"
// The prefix is defined where you mount it: app.use('/api/users', router)
```

---

## Central Route Registry

For larger apps, use an index file to register all routers in one place.

### routes/index.js

```javascript
const express = require("express");
const router = express.Router();

const userRoutes = require("./userRoutes");
const productRoutes = require("./productRoutes");
const orderRoutes = require("./orderRoutes");
const authRoutes = require("./authRoutes");

router.use("/users", userRoutes);
router.use("/products", productRoutes);
router.use("/orders", orderRoutes);
router.use("/auth", authRoutes);

module.exports = router;
```

### app.js (simplified)

```javascript
const routes = require("./routes");
app.use("/api", routes); // All routes live under /api
```

---

## Organizing by Feature/Resource

### Option A: Organize by Type (MVC-style)

```
src/
├── controllers/
│   ├── userController.js
│   └── productController.js
├── routes/
│   ├── userRoutes.js
│   └── productRoutes.js
├── models/
│   ├── User.js
│   └── Product.js
└── app.js
```

### Option B: Organize by Feature (Recommended for larger apps)

```
src/
├── features/
│   ├── users/
│   │   ├── userRoutes.js
│   │   ├── userController.js
│   │   ├── userModel.js
│   │   ├── userValidation.js
│   │   └── userService.js
│   ├── products/
│   │   ├── productRoutes.js
│   │   ├── productController.js
│   │   ├── productModel.js
│   │   └── productService.js
│   └── orders/
│       ├── orderRoutes.js
│       ├── orderController.js
│       └── orderModel.js
├── middleware/
│   ├── auth.js
│   └── errorHandler.js
└── app.js
```

---

## Route-Level Middleware on Routers

### Middleware for All Routes in a Router

```javascript
const express = require("express");
const router = express.Router();
const authenticate = require("../middleware/auth");

// Apply to ALL routes in this router
router.use(authenticate);

router.get("/", (req, res) => {
  // Only authenticated users reach here
  res.json({ data: "Protected resource" });
});

module.exports = router;
```

### Middleware for Specific Routes

```javascript
const express = require("express");
const router = express.Router();
const authenticate = require("../middleware/auth");
const authorize = require("../middleware/authorize");

// Public route — no middleware
router.get("/", (req, res) => {
  res.json({ data: "Public list" });
});

// Protected route — requires auth
router.post("/", authenticate, (req, res) => {
  res.status(201).json({ data: "Created" });
});

// Admin-only route — requires auth + admin role
router.delete("/:id", authenticate, authorize("admin"), (req, res) => {
  res.status(204).send();
});

module.exports = router;
```

### Authorization Middleware Factory

```javascript
// middleware/authorize.js
function authorize(...roles) {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: "Insufficient permissions" });
    }
    next();
  };
}

module.exports = authorize;
```

### Router-Level Parameter Middleware

```javascript
// Runs whenever :id param is present in this router
router.param("id", (req, res, next, id) => {
  if (isNaN(id)) {
    return res.status(400).json({ error: "ID must be a number" });
  }
  req.params.id = parseInt(id);
  next();
});

router.get("/:id", (req, res) => {
  // req.params.id is guaranteed to be a number here
  res.json({ id: req.params.id });
});
```

---

## Best Practices

1. **One router per resource** — `userRoutes.js`, `productRoutes.js`, etc.
2. **Keep routes thin** — Move business logic to controllers or services.
3. **Use a central registry** — `routes/index.js` to mount all routers in one place.
4. **Relative paths in routers** — Use `/` and `/:id`, not `/api/users` inside the router file.
5. **Scope middleware** — Apply auth/validation only where needed, not globally.
6. **Consistent naming** — `userRoutes.js`, `userController.js`, `userModel.js`.
7. **Feature-based organization** — Group related files together for larger apps.
8. **Export routers** — Always `module.exports = router` at the end of route files.

---

## Common Mistakes

| Mistake                          | Problem                                          | Fix                                             |
| -------------------------------- | ------------------------------------------------ | ----------------------------------------------- |
| Full path in router file         | `router.get('/api/users')` duplicates the prefix | Use relative paths: `router.get('/')`           |
| Forgetting `module.exports`      | Router isn't available to import                 | Always export: `module.exports = router`        |
| Using `app` inside route files   | Tight coupling to main app                       | Use `express.Router()` and export it            |
| Mounting order issues            | More specific routes shadowed by generic ones    | Mount specific routes (`/search`) before `/:id` |
| No error handling in controllers | Unhandled errors crash the server                | Wrap async code in try/catch                    |
| Middleware applied too broadly   | All routes require auth, even public ones        | Use route-level middleware selectively          |

---

## Summary

- `express.Router()` creates a mini-app for handling a group of related routes
- **Mount routers** with `app.use('/prefix', router)` — routes inside are relative to the prefix
- **Separate into files** — one file per resource or feature for clean organization
- Use a **central registry** (`routes/index.js`) to keep `app.js` clean
- Apply **route-level middleware** to protect specific routes without affecting others
- Choose **by-type** (MVC) for small apps, **by-feature** for larger apps
- `router.param()` validates URL parameters at the router level
