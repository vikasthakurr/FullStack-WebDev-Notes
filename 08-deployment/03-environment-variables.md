# Environment Variables in Production

## What / Why

**Environment variables** are key-value pairs set outside your code that configure how your application behaves. They let you change behavior without changing code — different values for development, staging, and production.

**Analogy:** Environment variables are like the settings on a TV. The TV (your code) stays the same, but you adjust brightness, volume, and input source (env vars) depending on the room you're in.

**Why use them?**

- **Security** — Keep secrets (API keys, passwords) out of source code
- **Flexibility** — Same code runs differently per environment
- **Portability** — Deploy anywhere without code changes
- **Collaboration** — Team members use their own credentials locally

```mermaid
graph TD
    A[Your Application Code] --> B{process.env}
    B -->|Development| C[".env file (dotenv)"]
    B -->|Production| D[Hosting Platform Env Vars]
    B -->|CI/CD| E[GitHub Secrets]
    C --> F["PORT=3000<br/>DB_URL=localhost<br/>DEBUG=true"]
    D --> G["PORT=10000<br/>DB_URL=atlas://...<br/>DEBUG=false"]
    E --> H["API_KEY=***<br/>DEPLOY_TOKEN=***"]
```

---

## The dotenv Package (Development)

### Installation

```bash
npm install dotenv
```

### Create a `.env` File

```env
# .env - Local development configuration
PORT=3000
DATABASE_URL=mongodb://localhost:27017/myapp
JWT_SECRET=dev-secret-key-not-for-production
JWT_EXPIRES_IN=7d
API_KEY=sk-test-abc123
NODE_ENV=development
CORS_ORIGIN=http://localhost:5173
```

### Load in Your App

```javascript
// server.js - Load at the very top, before other imports
require("dotenv").config();

const express = require("express");
const app = express();

// Access variables via process.env
const PORT = process.env.PORT || 3000;
const dbUrl = process.env.DATABASE_URL;

console.log(`Running in ${process.env.NODE_ENV} mode`);

app.listen(PORT, () => {
  console.log(`Server on port ${PORT}`);
});
```

### How `process.env` Works

```javascript
// process.env is a plain object with string values
console.log(typeof process.env.PORT); // "string" (always strings!)

// Convert to number when needed
const port = parseInt(process.env.PORT, 10);

// Boolean check
const isProduction = process.env.NODE_ENV === "production";

// Provide defaults
const timeout = process.env.REQUEST_TIMEOUT || "5000";
```

---

## The `.env.example` Pattern

Create a `.env.example` file that documents required variables without real values:

```env
# .env.example - Commit this file to git
# Copy to .env and fill in your values: cp .env.example .env

PORT=3000
DATABASE_URL=mongodb://localhost:27017/your-db-name
JWT_SECRET=generate-a-random-string-here
JWT_EXPIRES_IN=7d
API_KEY=your-api-key-here
NODE_ENV=development
CORS_ORIGIN=http://localhost:5173
```

**Why?**

- New developers know what variables are needed
- Serves as documentation
- Safe to commit (no real secrets)

---

## Never Commit `.env` to Git

### `.gitignore` Setup

```gitignore
# Environment variables
.env
.env.local
.env.production
.env.*.local

# Don't ignore the example
!.env.example
```

### What Happens If You Accidentally Commit `.env`

```bash
# Remove from tracking (file stays locally but is untracked)
git rm --cached .env
git commit -m "Remove .env from tracking"

# The damage: Even after removal, it exists in git history
# You need to rotate ALL exposed secrets immediately
```

> ⚠️ If you push a `.env` file with real secrets to GitHub, consider those secrets compromised. Regenerate them immediately.

---

## Setting Env Vars in Hosting Platforms

### Render

**Dashboard** → Your Service → **Environment** tab → Add key-value pairs

```
Key                Value
─────────────────  ──────────────────────────────
PORT               10000
DATABASE_URL       mongodb+srv://user:pass@cluster.mongodb.net/prod
JWT_SECRET         a1b2c3d4-random-production-secret
NODE_ENV           production
```

Or use `render.yaml`:

```yaml
services:
  - type: web
    name: my-api
    envVars:
      - key: NODE_ENV
        value: production
      - key: DATABASE_URL
        sync: false # Set in dashboard manually
```

### Railway

**Dashboard** → Project → **Variables** tab → Add variables

```bash
# Or via Railway CLI
railway variables set DATABASE_URL="mongodb+srv://..."
railway variables set JWT_SECRET="production-secret"
railway variables set NODE_ENV="production"
```

### Heroku

```bash
# Via CLI
heroku config:set DATABASE_URL="mongodb+srv://..."
heroku config:set JWT_SECRET="production-secret"
heroku config:set NODE_ENV="production"

# View all
heroku config
```

### Vercel

**Dashboard** → Project → **Settings** → **Environment Variables**

Can scope variables to environments: Production, Preview, Development.

---

## NODE_ENV: Development vs Production

```javascript
// NODE_ENV controls app behavior
const isProduction = process.env.NODE_ENV === "production";

// Express uses NODE_ENV internally
// In production: caches view templates, less verbose errors
app.get("env"); // returns process.env.NODE_ENV

// Conditional logic based on environment
if (isProduction) {
  // Use production database
  // Enable response compression
  // Disable detailed error messages
  app.use(compression());
  app.use(helmet());
} else {
  // Use local database
  // Enable CORS for localhost
  // Show stack traces in errors
  app.use(morgan("dev"));
}
```

```mermaid
graph TD
    A[NODE_ENV] -->|"development"| B[Dev Behavior]
    A -->|"production"| C[Prod Behavior]
    B --> B1[Detailed errors]
    B --> B2[Hot reload]
    B --> B3[Debug logging]
    B --> B4[Local DB]
    C --> C1[Generic error pages]
    C --> C2[Minified assets]
    C --> C3[Structured logs]
    C --> C4[Remote DB + Caching]
```

### What Changes in Production

| Behavior       | Development                        | Production                                  |
| -------------- | ---------------------------------- | ------------------------------------------- |
| Error messages | Full stack trace                   | Generic "Something went wrong"              |
| View caching   | Disabled (re-compile each request) | Enabled (compile once)                      |
| Logging        | Verbose (morgan 'dev')             | Structured JSON                             |
| Dependencies   | All installed                      | Only `dependencies` (not `devDependencies`) |
| Performance    | Unoptimized                        | Compression, caching enabled                |

---

## Common Environment Variables

```env
# Server Configuration
PORT=3000                    # Server port (hosting platforms set this)
HOST=0.0.0.0                # Listen on all interfaces
NODE_ENV=production          # Environment mode

# Database
DATABASE_URL=mongodb+srv://user:pass@host/db    # MongoDB connection
REDIS_URL=redis://localhost:6379                  # Redis connection

# Authentication
JWT_SECRET=your-256-bit-secret      # Token signing key
JWT_EXPIRES_IN=7d                    # Token expiration
SESSION_SECRET=random-session-key    # Express session secret

# External APIs
STRIPE_SECRET_KEY=sk_live_...       # Payment processor
SENDGRID_API_KEY=SG.xxx            # Email service
CLOUDINARY_URL=cloudinary://...     # Image hosting
AWS_ACCESS_KEY_ID=AKIA...          # AWS services
AWS_SECRET_ACCESS_KEY=xxx           # AWS services

# App Configuration
CORS_ORIGIN=https://myapp.com       # Allowed frontend origin
RATE_LIMIT_MAX=100                  # Requests per window
UPLOAD_MAX_SIZE=5242880             # 5MB in bytes

# Logging & Monitoring
LOG_LEVEL=info                      # Log verbosity
SENTRY_DSN=https://xxx@sentry.io   # Error tracking
```

---

## Validation Pattern

```javascript
// config/env.js - Validate required env vars at startup
function validateEnv() {
  const required = ["DATABASE_URL", "JWT_SECRET", "NODE_ENV"];

  const missing = required.filter((key) => !process.env[key]);

  if (missing.length > 0) {
    console.error("❌ Missing required environment variables:");
    missing.forEach((key) => console.error(`   - ${key}`));
    process.exit(1);
  }

  // Validate specific formats
  if (!process.env.DATABASE_URL.startsWith("mongodb")) {
    console.error("❌ DATABASE_URL must be a valid MongoDB URI");
    process.exit(1);
  }

  console.log("✅ All environment variables validated");
}

module.exports = validateEnv;
```

```javascript
// server.js
require("dotenv").config();
const validateEnv = require("./config/env");
validateEnv(); // Fail fast if config is wrong

// ... rest of app
```

---

## Best Practices

1. **Never commit `.env` files** — add to `.gitignore` immediately
2. **Always provide `.env.example`** — document required variables
3. **Validate at startup** — fail fast with clear error messages
4. **Use descriptive names** — `DATABASE_URL` not `DB`, `JWT_SECRET` not `SECRET`
5. **Remember values are strings** — parse numbers and booleans explicitly
6. **Don't use dotenv in production** — hosting platforms inject vars natively
7. **Rotate secrets periodically** — especially after team member changes
8. **Use different secrets per environment** — dev and prod must not share JWT keys
9. **Prefix related vars** — `DB_HOST`, `DB_PORT`, `DB_NAME` or use a single URL
10. **Don't log secrets** — sanitize `process.env` before logging

---

## Common Mistakes

| Mistake                             | Problem                                   | Fix                                     |
| ----------------------------------- | ----------------------------------------- | --------------------------------------- |
| Committing `.env` to git            | Secrets exposed in repo history           | Add to `.gitignore`, rotate secrets     |
| Using same secret in dev & prod     | Compromised dev key affects production    | Generate unique secrets per environment |
| Not calling `dotenv.config()` first | `process.env.X` returns `undefined`       | Put `require('dotenv').config()` at top |
| Expecting non-string values         | `process.env.PORT` is `"3000"` not `3000` | Use `parseInt()` or `=== 'true'`        |
| Hardcoding fallback secrets         | Production uses dev defaults              | Validate required vars at startup       |
| Adding spaces around `=` in `.env`  | Variable includes the space               | Use `KEY=value` not `KEY = value`       |
| Using quotes inconsistently         | Quotes become part of the value           | `KEY=value` (no quotes needed usually)  |
| Forgetting `NODE_ENV` in production | Express runs in development mode          | Always set `NODE_ENV=production`        |

---

## Summary

- **Environment variables** separate configuration from code — different values for different environments
- Use **dotenv** in development to load from a `.env` file; hosting platforms handle production
- **Never commit `.env`** to version control; provide `.env.example` as a template
- **Validate** required variables at startup to catch misconfigurations early
- Common vars include `PORT`, `DATABASE_URL`, `JWT_SECRET`, `NODE_ENV`, and API keys
- `NODE_ENV=production` triggers optimizations in Express and skips devDependencies
- All values in `process.env` are **strings** — convert to numbers/booleans explicitly
