# Security Best Practices for Node.js/Express Applications

## What & Why

Security isn't a feature you add at the end — it's a practice woven throughout development. A single vulnerability (exposed secrets, unvalidated input, missing headers) can compromise your entire application and user data.

**Analogy:** Security is like locks on a house. No single lock makes you safe — you need locks on the door (Helmet), a fence (CORS), a guard dog (rate limiting), and hidden valuables (env variables) working together.

---

## 1. Helmet.js — HTTP Security Headers

Helmet sets various HTTP headers to protect against common attacks (XSS, clickjacking, MIME sniffing, etc.).

### Installation & Usage

```bash
npm install helmet
```

```javascript
const express = require("express");
const helmet = require("helmet");

const app = express();

// Use all default protections
app.use(helmet());
```

### What Helmet Sets

| Header                      | Protection Against   | What It Does                                      |
| --------------------------- | -------------------- | ------------------------------------------------- |
| `X-Content-Type-Options`    | MIME sniffing        | Prevents browsers from guessing file types        |
| `X-Frame-Options`           | Clickjacking         | Prevents your site from being embedded in iframes |
| `Strict-Transport-Security` | Protocol downgrade   | Forces HTTPS connections                          |
| `X-XSS-Protection`          | Cross-site scripting | Enables browser XSS filter                        |
| `Content-Security-Policy`   | XSS, injection       | Controls which resources can load                 |
| `X-DNS-Prefetch-Control`    | Privacy              | Controls DNS prefetching                          |

### Custom Configuration

```javascript
app.use(
  helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", "'unsafe-inline'", "cdn.example.com"],
        styleSrc: ["'self'", "'unsafe-inline'", "fonts.googleapis.com"],
        imgSrc: ["'self'", "data:", "res.cloudinary.com"],
        connectSrc: ["'self'", "api.example.com"],
      },
    },
    crossOriginEmbedderPolicy: false, // Disable if using external images/CDN
  }),
);
```

---

## 2. CORS — Cross-Origin Resource Sharing

CORS controls which domains can make requests to your API. Without it, any website could call your API.

### Installation & Usage

```bash
npm install cors
```

```javascript
const cors = require("cors");

// Allow all origins (development only!)
app.use(cors());

// Production: whitelist specific origins
const corsOptions = {
  origin: ["https://myapp.com", "https://admin.myapp.com"],
  methods: ["GET", "POST", "PUT", "DELETE", "PATCH"],
  allowedHeaders: ["Content-Type", "Authorization"],
  credentials: true, // Allow cookies/auth headers
  maxAge: 86400, // Cache preflight for 24 hours
};
app.use(cors(corsOptions));
```

### Dynamic Origin (for Multiple Environments)

```javascript
const allowedOrigins = [
  "http://localhost:3000",
  "https://myapp.com",
  "https://staging.myapp.com",
];

const corsOptions = {
  origin: function (origin, callback) {
    // Allow requests with no origin (mobile apps, curl, Postman)
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error("Not allowed by CORS"));
    }
  },
  credentials: true,
};

app.use(cors(corsOptions));
```

---

## 3. Rate Limiting

Prevents brute-force attacks and API abuse by limiting how many requests a client can make.

### Installation & Usage

```bash
npm install express-rate-limit
```

```javascript
const rateLimit = require("express-rate-limit");

// General API limiter
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: {
    error: "Too many requests, please try again after 15 minutes",
  },
  standardHeaders: true, // Return rate limit info in headers
  legacyHeaders: false, // Disable X-RateLimit-* headers
});

app.use("/api/", apiLimiter);

// Stricter limiter for auth routes
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5, // Only 5 login attempts per 15 minutes
  message: { error: "Too many login attempts. Try again in 15 minutes." },
  skipSuccessfulRequests: true, // Don't count successful logins
});

app.use("/api/auth/login", authLimiter);
app.use("/api/auth/register", authLimiter);
```

### Rate Limit Headers Returned

```
RateLimit-Limit: 100
RateLimit-Remaining: 95
RateLimit-Reset: 1689127856
```

---

## 4. Environment Variables with dotenv

Never hardcode secrets (API keys, DB passwords, JWT secrets) in your code.

### Installation & Usage

```bash
npm install dotenv
```

```javascript
// Load at the very top of your entry file
require("dotenv").config();

// Access variables
const dbUri = process.env.MONGO_URI;
const jwtSecret = process.env.JWT_SECRET;
const port = process.env.PORT || 3000;
```

### .env File

```env
# .env — NEVER commit this file
NODE_ENV=development
PORT=3000

# Database
MONGO_URI=mongodb+srv://user:password@cluster0.xxx.mongodb.net/myapp

# Auth
JWT_SECRET=a1b2c3d4e5f6-your-256-bit-secret-key-here
JWT_EXPIRES_IN=7d
REFRESH_SECRET=different-secret-for-refresh-tokens

# External APIs
CLOUDINARY_API_KEY=123456789
STRIPE_SECRET_KEY=sk_test_xxxxxxxxxxxx
```

### .gitignore

```gitignore
# Environment variables — CRITICAL
.env
.env.local
.env.production

# Dependencies
node_modules/
```

### .env.example (Commit This)

```env
# .env.example — template for other developers
NODE_ENV=development
PORT=3000
MONGO_URI=mongodb+srv://<username>:<password>@cluster0.xxx.mongodb.net/<dbname>
JWT_SECRET=<your-secret-key>
JWT_EXPIRES_IN=7d
```

### Validating Required Env Variables

```javascript
// config/validateEnv.js
const requiredVars = ["MONGO_URI", "JWT_SECRET", "PORT"];

requiredVars.forEach((varName) => {
  if (!process.env[varName]) {
    console.error(`❌ Missing required env variable: ${varName}`);
    process.exit(1);
  }
});
```

---

## 5. Input Sanitization — Preventing NoSQL Injection

MongoDB queries accept objects, making them vulnerable to injection if user input isn't sanitized.

### The Attack

```javascript
// Malicious login request body:
{
  "email": { "$gt": "" },    // Matches any non-empty email
  "password": { "$gt": "" }  // Matches any non-empty password
}

// This query matches the first user in the database!
User.findOne({ email: req.body.email, password: req.body.password });
```

### Prevention: express-mongo-sanitize

```bash
npm install express-mongo-sanitize
```

```javascript
const mongoSanitize = require("express-mongo-sanitize");

// Remove any keys starting with $ or containing .
app.use(mongoSanitize());

// Or replace prohibited characters
app.use(mongoSanitize({ replaceWith: "_" }));
```

### Additional Input Validation with express-validator

```bash
npm install express-validator
```

```javascript
const { body, validationResult } = require("express-validator");

router.post(
  "/register",
  [
    body("email").isEmail().normalizeEmail(),
    body("password").isLength({ min: 6 }).trim().escape(),
    body("name").notEmpty().trim().escape(),
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    // Proceed with registration...
  },
);
```

### Prevent XSS with xss-clean

```bash
npm install xss-clean
```

```javascript
const xss = require("xss-clean");
app.use(xss()); // Sanitize user input in req.body, req.query, req.params
```

---

## 6. Password Hashing Rounds

bcrypt's salt rounds control how expensive (slow) hashing is — more rounds = more secure but slower.

```javascript
const bcrypt = require("bcryptjs");

// Salt rounds (cost factor)
const SALT_ROUNDS = 12; // Good balance for 2024

const hash = await bcrypt.hash(password, SALT_ROUNDS);
```

| Salt Rounds | Time (approx) | Use Case               |
| ----------- | ------------- | ---------------------- |
| 10          | ~65ms         | Minimum acceptable     |
| 12          | ~250ms        | Recommended default    |
| 14          | ~1s           | High-security apps     |
| 16          | ~4s           | Too slow for most apps |

> **Rule:** Increase rounds as hardware gets faster. Target ~250ms per hash on your server.

---

## 7. HTTPS in Production

Never serve an auth-based app over HTTP in production — all tokens and passwords would be visible in transit.

### With a Reverse Proxy (NGINX)

```nginx
server {
    listen 443 ssl;
    server_name myapp.com;

    ssl_certificate /etc/letsencrypt/live/myapp.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/myapp.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header X-Forwarded-Proto https;
    }
}
```

### Force HTTPS Redirect in Express

```javascript
// Trust proxy if behind NGINX/load balancer
app.set("trust proxy", 1);

// Redirect HTTP to HTTPS in production
if (process.env.NODE_ENV === "production") {
  app.use((req, res, next) => {
    if (req.headers["x-forwarded-proto"] !== "https") {
      return res.redirect(`https://${req.headers.host}${req.url}`);
    }
    next();
  });
}
```

### Secure Cookie Settings (for cookie-based tokens)

```javascript
res.cookie("token", jwt, {
  httpOnly: true, // Not accessible via JavaScript (prevents XSS)
  secure: true, // Only sent over HTTPS
  sameSite: "strict", // Prevents CSRF
  maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days
});
```

---

## 8. Dependency Auditing

Third-party packages can have known vulnerabilities. Audit regularly.

```bash
# Check for vulnerabilities
npm audit

# Auto-fix what's possible
npm audit fix

# Force fix (may have breaking changes)
npm audit fix --force

# See detailed report
npm audit --json
```

### Automated Auditing in CI/CD

```yaml
# .github/workflows/audit.yml
name: Security Audit
on:
  schedule:
    - cron: "0 0 * * 1" # Every Monday
  push:
    branches: [main]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - run: npm audit --audit-level=high
```

### Additional Tools

```bash
# Snyk — more comprehensive vulnerability scanning
npm install -g snyk
snyk test
snyk monitor  # Continuous monitoring

# Check for outdated packages
npm outdated
```

---

## Complete Security Setup (All Together)

```javascript
const express = require("express");
const helmet = require("helmet");
const cors = require("cors");
const rateLimit = require("express-rate-limit");
const mongoSanitize = require("express-mongo-sanitize");
const xss = require("xss-clean");
require("dotenv").config();

const app = express();

// === SECURITY MIDDLEWARE === //

// Set security HTTP headers
app.use(helmet());

// CORS
app.use(
  cors({
    origin: process.env.ALLOWED_ORIGINS?.split(",") || "http://localhost:3000",
    credentials: true,
  }),
);

// Rate limiting
app.use(
  "/api/",
  rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100,
  }),
);

// Body parser with size limit
app.use(express.json({ limit: "10kb" })); // Prevent large payload attacks

// Data sanitization against NoSQL injection
app.use(mongoSanitize());

// Data sanitization against XSS
app.use(xss());

// === ROUTES === //
app.use("/api/auth", require("./routes/auth"));
app.use("/api/users", require("./routes/users"));

// === ERROR HANDLER === //
app.use((err, req, res, next) => {
  // Don't leak stack traces in production
  const message =
    process.env.NODE_ENV === "production"
      ? "Something went wrong"
      : err.message;

  res.status(err.statusCode || 500).json({
    status: "error",
    message,
  });
});

module.exports = app;
```

---

## Security Checklist

```markdown
- [ ] Helmet.js enabled
- [ ] CORS configured with specific origins (not \*)
- [ ] Rate limiting on all routes (stricter on auth)
- [ ] Environment variables for all secrets
- [ ] .env in .gitignore
- [ ] Input sanitization (mongo-sanitize, xss-clean)
- [ ] Password hashing with bcrypt (12+ rounds)
- [ ] HTTPS in production
- [ ] Body size limited (express.json({ limit: '10kb' }))
- [ ] No stack traces in production error responses
- [ ] npm audit runs regularly
- [ ] Dependencies kept up to date
- [ ] JWT secret is strong (256+ bit)
- [ ] Tokens have expiration
- [ ] Sensitive data not in JWT payload
```

---

## Best Practices

1. **Layer your security** — no single measure is enough; use helmet + cors + rate limit + sanitization together
2. **Never trust client input** — validate and sanitize everything from `req.body`, `req.query`, `req.params`
3. **Fail securely** — on error, deny access rather than grant it
4. **Keep dependencies updated** — set up automated auditing in CI
5. **Use HTTPS everywhere** — especially when handling tokens or credentials
6. **Limit request body size** — prevent denial-of-service via large payloads
7. **Don't expose error details in production** — attackers use stack traces to find vulnerabilities
8. **Rotate secrets periodically** — change JWT secrets, API keys on a schedule

---

## Common Mistakes

| Mistake                                 | Why It's Wrong                                    | Fix                                      |
| --------------------------------------- | ------------------------------------------------- | ---------------------------------------- |
| `cors({ origin: '*' })` in production   | Any website can call your API                     | Whitelist specific origins               |
| Committing `.env` to git                | Secrets exposed in repository history             | Add to `.gitignore`, use `.env.example`  |
| No rate limiting on login route         | Enables brute-force password attacks              | Use strict rate limit (5 attempts/15min) |
| `express.json()` without size limit     | Attackers can send massive payloads (DoS)         | Use `{ limit: '10kb' }`                  |
| Returning `err.stack` in production     | Reveals file paths, library versions to attackers | Use generic error messages in production |
| Not sanitizing MongoDB queries          | NoSQL injection can bypass authentication         | Use `express-mongo-sanitize`             |
| Using `npm install` in production       | Installs devDependencies, no lockfile guarantee   | Use `npm ci --production`                |
| Ignoring `npm audit` warnings           | Known vulnerabilities remain exploitable          | Run `npm audit fix` and update packages  |
| Same secret for access & refresh tokens | Compromised access token also compromises refresh | Use separate secrets                     |
| No `httpOnly` on auth cookies           | JavaScript (XSS) can steal tokens                 | Always set `httpOnly: true`              |

---

## Summary

- **Helmet.js** sets security headers automatically — always use it as the first middleware
- **CORS** restricts which origins can access your API — whitelist only your frontend domains
- **Rate limiting** prevents brute-force and DoS attacks — be stricter on auth endpoints
- **dotenv** keeps secrets out of code — never commit `.env`, always provide `.env.example`
- **Input sanitization** prevents NoSQL injection and XSS — use `express-mongo-sanitize` + `xss-clean`
- **bcrypt with 12+ rounds** makes password cracking computationally expensive
- **HTTPS** encrypts all data in transit — mandatory for production
- **npm audit** catches known vulnerabilities in dependencies — automate in CI/CD
- Security is **layers** — each measure covers different attack vectors
