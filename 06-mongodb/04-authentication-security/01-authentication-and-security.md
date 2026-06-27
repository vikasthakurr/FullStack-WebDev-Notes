# Authentication & Security

## User Registration & Login Flow

```mermaid
flowchart TD
    A["User Submits Registration"] --> B["Validate Input"]
    B --> C["Hash Password (bcrypt)"]
    C --> D["Store User in DB"]
    D --> E["Return Success"]

    F["User Submits Login"] --> G["Find User by Email"]
    G --> H["Compare Password with Hash"]
    H -->|"Match"| I["Generate JWT Token"]
    H -->|"No Match"| J["Return 401 Error"]
    I --> K["Send Token to Client"]
```

---

## Password Hashing with bcrypt

Never store passwords in plain text. Use bcrypt to hash them.

```bash
npm install bcrypt
```

```javascript
const bcrypt = require("bcrypt");

// Hash a password
async function hashPassword(plainPassword) {
  const saltRounds = 12; // Higher = more secure but slower
  return bcrypt.hash(plainPassword, saltRounds);
}

// Compare password with hash
async function comparePassword(plainPassword, hashedPassword) {
  return bcrypt.compare(plainPassword, hashedPassword);
}
```

### In Mongoose Schema

```javascript
const userSchema = new mongoose.Schema({
  email: { type: String, required: true, unique: true, lowercase: true },
  password: { type: String, required: true, minlength: 8, select: false },
});

// Hash password before saving
userSchema.pre("save", async function (next) {
  if (!this.isModified("password")) return next();
  this.password = await bcrypt.hash(this.password, 12);
  next();
});

// Instance method to compare passwords
userSchema.methods.comparePassword = async function (candidatePassword) {
  return bcrypt.compare(candidatePassword, this.password);
};
```

`select: false` means password is excluded from queries by default — you must explicitly request it.

---

## JWT Authentication

JWT (JSON Web Token) is a stateless authentication mechanism — the server does not store session data.

```bash
npm install jsonwebtoken
```

### How JWT Works

```mermaid
flowchart LR
    A["Login"] --> B["Server creates JWT"]
    B --> C["Client stores JWT"]
    C --> D["Client sends JWT in header"]
    D --> E["Server verifies JWT"]
    E --> F["Access granted"]
```

### Token Structure

```
header.payload.signature

eyJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOiIxMjMiLCJyb2xlIjoiYWRtaW4ifQ.signature
```

- **Header** — algorithm and token type.
- **Payload** — claims (userId, role, expiration).
- **Signature** — verifies the token has not been tampered with.

### Generating Tokens

```javascript
const jwt = require("jsonwebtoken");

function generateToken(user) {
  return jwt.sign(
    { userId: user._id, role: user.role }, // Payload
    process.env.JWT_SECRET, // Secret key
    { expiresIn: "7d" }, // Options
  );
}

function generateRefreshToken(user) {
  return jwt.sign({ userId: user._id }, process.env.JWT_REFRESH_SECRET, {
    expiresIn: "30d",
  });
}
```

### Verifying Tokens (Auth Middleware)

```javascript
function authenticate(req, res, next) {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith("Bearer ")) {
    return res.status(401).json({ error: "No token provided" });
  }

  const token = authHeader.split(" ")[1];

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded; // { userId, role }
    next();
  } catch (error) {
    if (error.name === "TokenExpiredError") {
      return res.status(401).json({ error: "Token expired" });
    }
    return res.status(401).json({ error: "Invalid token" });
  }
}
```

---

## Complete Auth Controller

```javascript
const User = require("../models/User");
const jwt = require("jsonwebtoken");

// Register
exports.register = async (req, res) => {
  try {
    const { name, email, password } = req.body;

    // Check existing user
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(409).json({ error: "Email already registered" });
    }

    // Create user (password hashed by pre-save hook)
    const user = await User.create({ name, email, password });

    // Generate token
    const token = generateToken(user);

    res.status(201).json({
      message: "Registration successful",
      token,
      user: {
        id: user._id,
        name: user.name,
        email: user.email,
        role: user.role,
      },
    });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
};

// Login
exports.login = async (req, res) => {
  try {
    const { email, password } = req.body;

    if (!email || !password) {
      return res.status(400).json({ error: "Email and password required" });
    }

    // Find user WITH password (select: false is default)
    const user = await User.findOne({ email }).select("+password");

    if (!user || !(await user.comparePassword(password))) {
      return res.status(401).json({ error: "Invalid email or password" });
    }

    const token = generateToken(user);

    res.json({
      message: "Login successful",
      token,
      user: {
        id: user._id,
        name: user.name,
        email: user.email,
        role: user.role,
      },
    });
  } catch (error) {
    res.status(500).json({ error: "Server error" });
  }
};

// Get current user profile
exports.getProfile = async (req, res) => {
  const user = await User.findById(req.user.userId);
  res.json({ user });
};
```

---

## Role-Based Access Control (RBAC)

```javascript
function authorize(...allowedRoles) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: "Authentication required" });
    }

    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({ error: "Insufficient permissions" });
    }

    next();
  };
}

// Usage
app.get(
  "/api/admin/dashboard",
  authenticate,
  authorize("admin"),
  (req, res) => {
    res.json({ message: "Admin dashboard" });
  },
);

app.delete(
  "/api/users/:id",
  authenticate,
  authorize("admin", "moderator"),
  (req, res) => {
    // Only admins and moderators can delete
  },
);

app.get("/api/profile", authenticate, (req, res) => {
  // Any authenticated user
});
```

---

## Security Best Practices

### Helmet (Security Headers)

```bash
npm install helmet
```

```javascript
const helmet = require("helmet");
app.use(helmet());
// Sets headers: X-Content-Type-Options, Strict-Transport-Security,
// X-Frame-Options, X-XSS-Protection, etc.
```

### CORS

```javascript
const cors = require("cors");
app.use(
  cors({
    origin: process.env.FRONTEND_URL, // Not "*" in production
    credentials: true,
  }),
);
```

### Rate Limiting

```bash
npm install express-rate-limit
```

```javascript
const rateLimit = require("express-rate-limit");

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: { error: "Too many requests, try again later" },
});

const authLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 5, // 5 login attempts
  message: { error: "Too many login attempts" },
});

app.use("/api", limiter);
app.use("/api/auth/login", authLimiter);
```

### Data Sanitization

```bash
npm install express-mongo-sanitize xss-clean
```

```javascript
const mongoSanitize = require("express-mongo-sanitize");
const xss = require("xss-clean");

app.use(mongoSanitize()); // Prevents NoSQL injection ({ $gt: "" })
app.use(xss()); // Sanitizes HTML in input (prevents XSS)
```

### Environment Variables

```bash
npm install dotenv
```

```
# .env
NODE_ENV=production
JWT_SECRET=your-very-long-random-secret-key-here
JWT_REFRESH_SECRET=another-random-key
MONGO_URI=mongodb+srv://user:pass@cluster.mongodb.net/db
FRONTEND_URL=https://myapp.com
```

**Never commit `.env`** — add to `.gitignore`.

### Parameter Pollution Prevention

```bash
npm install hpp
```

```javascript
const hpp = require("hpp");
app.use(hpp()); // Prevents HTTP parameter pollution
```

---

## Security Checklist

| Category       | Measure                                         |
| -------------- | ----------------------------------------------- |
| Passwords      | bcrypt with 12+ rounds, never stored plain      |
| Authentication | JWT with expiration, refresh token rotation     |
| Authorization  | RBAC middleware, check on every protected route |
| Input          | Validate (Joi), sanitize (mongo-sanitize, xss)  |
| Headers        | Helmet for security headers                     |
| Rate limiting  | express-rate-limit on all endpoints             |
| CORS           | Specific origins only, not `*`                  |
| Environment    | Secrets in env vars, never in code              |
| Dependencies   | `npm audit` regularly, pin versions             |
| HTTPS          | Always in production (TLS/SSL)                  |

---

## Summary

- Hash passwords with bcrypt (12+ salt rounds) — never store plain text.
- Use JWT for stateless authentication — include userId and role in payload.
- Implement RBAC with middleware that checks `req.user.role` against allowed roles.
- Layer security: Helmet (headers), CORS (origins), rate limiting (abuse prevention), sanitization (injection prevention).
- Store secrets in environment variables — never commit them to version control.
- Combine `authenticate` and `authorize` middleware for fine-grained access control on every route.
