# Authentication & Security — User Auth with JWT

## What & Why

Authentication verifies **who you are**. Authorization determines **what you can do**. In modern web apps, we use **JWT (JSON Web Tokens)** to implement stateless authentication — the server doesn't store session data; instead, it issues a signed token the client sends with each request.

**Analogy:** A JWT is like a wristband at a concert. The bouncer (server) issues it after checking your ticket (login). For the rest of the night, you just flash your wristband — no need to re-check your ticket every time.

---

## The Full Auth Flow

```
┌─────────────┐         ┌─────────────┐         ┌─────────────┐
│   Client    │         │   Server    │         │  Database   │
└──────┬──────┘         └──────┬──────┘         └──────┬──────┘
       │                       │                       │
       │  POST /register       │                       │
       │──────────────────────>│                       │
       │                       │  Hash password        │
       │                       │  Save user            │
       │                       │──────────────────────>│
       │  201 Created          │                       │
       │<──────────────────────│                       │
       │                       │                       │
       │  POST /login          │                       │
       │──────────────────────>│                       │
       │                       │  Find user            │
       │                       │──────────────────────>│
       │                       │  Compare password     │
       │                       │  Generate JWT         │
       │  200 { token }        │                       │
       │<──────────────────────│                       │
       │                       │                       │
       │  GET /profile         │                       │
       │  Authorization: Bearer│token                  │
       │──────────────────────>│                       │
       │                       │  Verify JWT           │
       │                       │  Fetch user data      │
       │                       │──────────────────────>│
       │  200 { user data }    │                       │
       │<──────────────────────│                       │
```

---

## Setup

```bash
npm install express mongoose bcryptjs jsonwebtoken dotenv
```

```env
# .env
MONGO_URI=mongodb+srv://user:pass@cluster0.xxx.mongodb.net/authapp
JWT_SECRET=your-super-secret-key-change-in-production
JWT_EXPIRES_IN=7d
PORT=3000
```

---

## User Model with Password Hashing

```javascript
// models/User.js
const mongoose = require("mongoose");
const bcrypt = require("bcryptjs");

const userSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: [true, "Name is required"],
      trim: true,
    },
    email: {
      type: String,
      required: [true, "Email is required"],
      unique: true,
      lowercase: true,
      match: [/^\S+@\S+\.\S+$/, "Invalid email format"],
    },
    password: {
      type: String,
      required: [true, "Password is required"],
      minlength: [6, "Password must be at least 6 characters"],
      select: false, // Never return password in queries by default
    },
    role: {
      type: String,
      enum: ["user", "admin"],
      default: "user",
    },
  },
  { timestamps: true },
);

// Hash password before saving
userSchema.pre("save", async function (next) {
  // Only hash if password was modified
  if (!this.isModified("password")) return next();

  const salt = await bcrypt.genSalt(12);
  this.password = await bcrypt.hash(this.password, salt);
  next();
});

// Instance method: compare passwords
userSchema.methods.comparePassword = async function (candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};

module.exports = mongoose.model("User", userSchema);
```

---

## Registration Flow

```javascript
// controllers/authController.js
const User = require("../models/User");
const jwt = require("jsonwebtoken");

// Generate JWT token
const generateToken = (userId) => {
  return jwt.sign({ id: userId }, process.env.JWT_SECRET, {
    expiresIn: process.env.JWT_EXPIRES_IN,
  });
};

// POST /api/auth/register
exports.register = async (req, res) => {
  try {
    const { name, email, password } = req.body;

    // Check if user already exists
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(400).json({ message: "Email already registered" });
    }

    // Create user (password hashed by pre-save hook)
    const user = await User.create({ name, email, password });

    // Generate token
    const token = generateToken(user._id);

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
    res.status(500).json({ message: error.message });
  }
};
```

---

## Login Flow

```javascript
// POST /api/auth/login
exports.login = async (req, res) => {
  try {
    const { email, password } = req.body;

    // Validate input
    if (!email || !password) {
      return res
        .status(400)
        .json({ message: "Email and password are required" });
    }

    // Find user and explicitly include password field
    const user = await User.findOne({ email }).select("+password");
    if (!user) {
      return res.status(401).json({ message: "Invalid email or password" });
    }

    // Compare passwords
    const isMatch = await user.comparePassword(password);
    if (!isMatch) {
      return res.status(401).json({ message: "Invalid email or password" });
    }

    // Generate token
    const token = generateToken(user._id);

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
    res.status(500).json({ message: error.message });
  }
};
```

> **Security Note:** Always return the same error message for wrong email or wrong password — don't reveal which one was incorrect.

---

## JWT Explained

A JWT has three parts separated by dots: `header.payload.signature`

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.    ← Header (base64)
eyJpZCI6IjY0YTdmMmIzYzllMWEyZDRlOGYwMTIzNCIsImlhdCI6MTY4OTEyMzQ1Nn0.  ← Payload (base64)
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c  ← Signature (encrypted)
```

| Part          | Contains               | Example                                                       |
| ------------- | ---------------------- | ------------------------------------------------------------- |
| **Header**    | Algorithm & token type | `{ "alg": "HS256", "typ": "JWT" }`                            |
| **Payload**   | Claims (data)          | `{ "id": "64a7f2...", "iat": 1689123456, "exp": 1689727456 }` |
| **Signature** | Verification hash      | HMAC-SHA256(header + payload, secret)                         |

**How it works:**

1. Server signs: `HMACSHA256(base64(header) + "." + base64(payload), SECRET_KEY)`
2. Client stores the token (localStorage, cookie, or memory)
3. Client sends token in `Authorization: Bearer <token>` header
4. Server verifies signature — if valid, trusts the payload without DB lookup

> **Important:** The payload is NOT encrypted — it's just base64 encoded. Anyone can decode it. Never put sensitive data (passwords, credit cards) in the payload.

---

## Auth Middleware — Protecting Routes

```javascript
// middleware/auth.js
const jwt = require("jsonwebtoken");
const User = require("../models/User");

const protect = async (req, res, next) => {
  try {
    // 1. Get token from header
    const authHeader = req.headers.authorization;
    if (!authHeader || !authHeader.startsWith("Bearer ")) {
      return res.status(401).json({ message: "Not authorized — no token" });
    }

    const token = authHeader.split(" ")[1];

    // 2. Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);

    // 3. Check if user still exists
    const user = await User.findById(decoded.id);
    if (!user) {
      return res.status(401).json({ message: "User no longer exists" });
    }

    // 4. Attach user to request
    req.user = user;
    next();
  } catch (error) {
    if (error.name === "JsonWebTokenError") {
      return res.status(401).json({ message: "Invalid token" });
    }
    if (error.name === "TokenExpiredError") {
      return res.status(401).json({ message: "Token expired" });
    }
    res.status(500).json({ message: "Authentication error" });
  }
};

module.exports = protect;
```

### Using the Middleware

```javascript
const protect = require("../middleware/auth");

// Public routes
router.post("/auth/register", register);
router.post("/auth/login", login);

// Protected routes — require valid JWT
router.get("/profile", protect, getProfile);
router.put("/profile", protect, updateProfile);
router.get("/orders", protect, getOrders);
```

---

## Role-Based Access Control (RBAC)

```javascript
// middleware/authorize.js
const authorize = (...roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        message: `Role '${req.user.role}' is not authorized to access this resource`,
      });
    }
    next();
  };
};

module.exports = authorize;
```

### Usage

```javascript
const protect = require("../middleware/auth");
const authorize = require("../middleware/authorize");

// Only admin can access
router.get("/admin/users", protect, authorize("admin"), getAllUsers);
router.delete("/admin/users/:id", protect, authorize("admin"), deleteUser);

// Admin and moderator can access
router.put(
  "/posts/:id/approve",
  protect,
  authorize("admin", "moderator"),
  approvePost,
);
```

---

## Refresh Tokens (Concept)

Access tokens are short-lived (15min–1hr). Refresh tokens are long-lived (7–30 days) and used to get new access tokens without re-login.

```javascript
// Login returns both tokens
const accessToken = jwt.sign({ id: user._id }, process.env.JWT_SECRET, {
  expiresIn: "15m",
});
const refreshToken = jwt.sign({ id: user._id }, process.env.REFRESH_SECRET, {
  expiresIn: "7d",
});

// Store refresh token in DB (to enable revocation)
user.refreshToken = refreshToken;
await user.save();

// Refresh endpoint
exports.refresh = async (req, res) => {
  const { refreshToken } = req.body;

  if (!refreshToken) {
    return res.status(401).json({ message: "Refresh token required" });
  }

  try {
    const decoded = jwt.verify(refreshToken, process.env.REFRESH_SECRET);
    const user = await User.findById(decoded.id);

    // Verify stored refresh token matches
    if (!user || user.refreshToken !== refreshToken) {
      return res.status(403).json({ message: "Invalid refresh token" });
    }

    // Issue new access token
    const newAccessToken = jwt.sign({ id: user._id }, process.env.JWT_SECRET, {
      expiresIn: "15m",
    });

    res.json({ accessToken: newAccessToken });
  } catch (error) {
    res.status(403).json({ message: "Invalid or expired refresh token" });
  }
};
```

### Token Flow

```
1. Login → receive accessToken (15min) + refreshToken (7d)
2. Use accessToken for API requests
3. When accessToken expires → POST /refresh with refreshToken
4. Receive new accessToken → continue making requests
5. When refreshToken expires → user must login again
```

---

## Complete Route Setup

```javascript
// routes/auth.js
const express = require("express");
const router = express.Router();
const {
  register,
  login,
  getProfile,
} = require("../controllers/authController");
const protect = require("../middleware/auth");

router.post("/register", register);
router.post("/login", login);
router.get("/profile", protect, getProfile);

module.exports = router;

// app.js
app.use("/api/auth", require("./routes/auth"));
```

---

## Best Practices

1. **Never store plain-text passwords** — always hash with bcrypt (12+ salt rounds)
2. **Use short-lived access tokens** (15min–1hr) with refresh tokens for long sessions
3. **Store JWT secret in environment variables** — never hardcode
4. **Return generic error messages** for auth failures — don't reveal if email or password was wrong
5. **Use `select: false`** on password field to prevent accidental leaks
6. **Validate input before processing** — check for empty fields, email format
7. **Use HTTPS in production** — JWTs are readable; HTTP exposes them to man-in-the-middle attacks
8. **Set appropriate `expiresIn`** — shorter = more secure, longer = better UX

---

## Common Mistakes

| Mistake                                 | Why It's Wrong                              | Fix                                          |
| --------------------------------------- | ------------------------------------------- | -------------------------------------------- |
| Storing passwords in plain text         | One breach exposes all users                | Always hash with bcrypt                      |
| Putting sensitive data in JWT payload   | JWT payload is base64 (readable by anyone)  | Only store user ID and role                  |
| Using a weak/short JWT secret           | Brute-force attacks can crack it            | Use 256+ bit random string                   |
| Not checking if user still exists       | Deleted user's token still works            | Verify user exists in DB on each request     |
| Same error for expired vs invalid token | Harder to debug for clients                 | Return specific error messages               |
| Storing tokens in localStorage only     | Vulnerable to XSS attacks                   | Consider httpOnly cookies for refresh tokens |
| Not handling `select: false` on login   | `findOne({ email })` won't include password | Use `.select('+password')` explicitly        |
| Hardcoding `expiresIn` values           | Difficult to change across environments     | Use environment variables                    |

---

## Summary

- **Registration:** validate input → check if user exists → hash password → save to DB → return JWT
- **Login:** find user → compare password with bcrypt → generate JWT → return token
- **JWT** is a signed token with header.payload.signature — stateless, no server-side sessions
- **Auth middleware** extracts token from header, verifies it, attaches user to `req.user`
- **RBAC** restricts routes by checking `req.user.role` against allowed roles
- **Refresh tokens** allow long sessions without long-lived access tokens — store in DB for revocation
- Never store secrets in code, never put sensitive data in JWT payloads, always hash passwords
