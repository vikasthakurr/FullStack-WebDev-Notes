# Hosting Node.js Apps

## What / Why

Hosting means putting your Node.js app on a server that's accessible via the internet 24/7. Your local machine isn't designed to serve production traffic — you need a hosting provider that handles uptime, scaling, SSL, and infrastructure.

**Analogy:** Your local machine is like cooking at home. Hosting is like renting a commercial kitchen — it has the equipment, space, and licenses to serve thousands of customers.

```mermaid
graph LR
    A[Your Code on GitHub] --> B{Hosting Platform}
    B --> C[Render]
    B --> D[Railway]
    B --> E[AWS EC2/ECS]
    B --> F[Vercel Serverless]
    B --> G[Heroku]
    C --> H[Live App URL]
    D --> H
    E --> H
    F --> H
    G --> H
```

---

## Hosting Options Overview

### 1. Render

- Free tier available (spins down after 15 min inactivity)
- Auto-deploy from GitHub
- Built-in SSL, environment variables, and managed databases
- Best for: Small to medium projects, APIs, full-stack apps

### 2. Railway

- Simple deploy from GitHub or CLI
- Usage-based pricing (free trial credits)
- Built-in databases (Postgres, Redis, MongoDB)
- Best for: Rapid prototyping, full-stack apps

### 3. Heroku

- Pioneer of PaaS (Platform as a Service)
- No longer has a free tier (removed Nov 2022)
- Excellent add-on ecosystem
- Best for: Teams familiar with the platform

### 4. AWS EC2 / ECS

- Full control over server configuration
- EC2 = Virtual machines, ECS = Container orchestration
- Pay for what you use
- Best for: Production apps needing full control and scaling

### 5. Vercel (Serverless)

- Designed for frontend frameworks (Next.js) and serverless functions
- Not ideal for long-running processes or WebSocket connections
- Best for: API routes, serverless functions, JAMstack

---

## Deploying to Render

### Step 1: Create a Web Service

1. Sign up at [render.com](https://render.com)
2. Click **New** → **Web Service**
3. Connect your GitHub repository

### Step 2: Configure the Service

```yaml
# Render settings
Name: my-express-api
Region: Oregon (US West)
Branch: main
Runtime: Node
Build Command: npm install
Start Command: node server.js
Plan: Free
```

### Step 3: Set Environment Variables

In the Render dashboard → **Environment** tab:

```
PORT=10000
DATABASE_URL=mongodb+srv://user:pass@cluster.mongodb.net/mydb
JWT_SECRET=your-secret-key
NODE_ENV=production
```

### Step 4: Auto-Deploy

Render auto-deploys on every push to your connected branch:

```mermaid
graph LR
    A[git push origin main] --> B[GitHub Repo Updated]
    B --> C[Render Detects Change]
    C --> D[Build: npm install]
    D --> E[Start: node server.js]
    E --> F[Live at myapp.onrender.com]
```

### Render `render.yaml` (Infrastructure as Code)

```yaml
services:
  - type: web
    name: my-express-api
    runtime: node
    repo: https://github.com/username/my-app
    buildCommand: npm install
    startCommand: node server.js
    envVars:
      - key: NODE_ENV
        value: production
      - key: DATABASE_URL
        sync: false # Set manually in dashboard
```

---

## Deploying to Railway

### Step 1: Connect Repository

1. Sign up at [railway.app](https://railway.app)
2. Click **New Project** → **Deploy from GitHub Repo**
3. Select your repository

### Step 2: Configure

Railway auto-detects Node.js projects. Override if needed:

```json
// package.json
{
  "scripts": {
    "start": "node server.js",
    "build": "npm install"
  }
}
```

### Step 3: Add Variables

In the Railway dashboard → **Variables** tab:

```bash
DATABASE_URL=mongodb+srv://...
JWT_SECRET=my-secret
NODE_ENV=production
```

### Step 4: Custom Domain (Optional)

```
Settings → Domains → Add Custom Domain → your-api.example.com
```

---

## Platform Comparison Table

| Feature            | Render           | Railway                  | Heroku           | AWS EC2             | Vercel               |
| ------------------ | ---------------- | ------------------------ | ---------------- | ------------------- | -------------------- |
| **Free Tier**      | Yes (spins down) | Trial credits            | No               | 12-month free tier  | Yes (serverless)     |
| **Ease of Use**    | ⭐⭐⭐⭐⭐       | ⭐⭐⭐⭐⭐               | ⭐⭐⭐⭐         | ⭐⭐                | ⭐⭐⭐⭐⭐           |
| **Auto-Deploy**    | Yes              | Yes                      | Yes              | Manual/CI           | Yes                  |
| **Custom Domains** | Yes              | Yes                      | Yes              | Yes                 | Yes                  |
| **SSL**            | Auto             | Auto                     | Auto             | Manual/ACM          | Auto                 |
| **WebSockets**     | Yes              | Yes                      | Yes              | Yes                 | No                   |
| **Databases**      | Postgres         | Postgres, Redis, MongoDB | Add-ons          | Any (self-managed)  | No                   |
| **Scaling**        | Manual           | Auto                     | Dynos            | Auto Scaling Groups | Auto                 |
| **Docker Support** | Yes              | Yes                      | Yes              | Yes                 | No                   |
| **Best For**       | APIs, full-stack | Rapid prototyping        | Enterprise teams | Full control        | Frontend, serverless |
| **Pricing**        | Free → $7/mo     | Usage-based              | $5/mo+           | Pay-per-use         | Free → $20/mo        |

---

## Preparing Your App for Deployment

```javascript
// server.js - Production-ready setup
const express = require("express");
const app = express();

// Use PORT from environment (hosting platforms assign their own)
const PORT = process.env.PORT || 3000;

// Trust proxy (for platforms behind a reverse proxy)
app.set("trust proxy", 1);

// Health check endpoint
app.get("/health", (req, res) => {
  res.status(200).json({ status: "ok", timestamp: Date.now() });
});

// Your routes
app.get("/", (req, res) => {
  res.json({ message: "API is running" });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

```json
// package.json - Essential fields for deployment
{
  "name": "my-express-api",
  "version": "1.0.0",
  "engines": {
    "node": ">=18.0.0"
  },
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  }
}
```

---

## Best Practices

1. **Always use `process.env.PORT`** — hosting platforms assign a dynamic port
2. **Specify Node.js version** in `engines` field of `package.json`
3. **Add a health check endpoint** — platforms use it to verify your app is alive
4. **Use `npm ci` over `npm install`** in production builds (faster, deterministic)
5. **Never hardcode secrets** — always use environment variables
6. **Set `NODE_ENV=production`** — enables optimizations in Express and other libraries
7. **Add a `.gitignore`** — never push `node_modules/` or `.env`
8. **Use a `Procfile` or start script** — be explicit about how to start your app

---

## Common Mistakes

| Mistake                          | Problem                                        | Fix                                               |
| -------------------------------- | ---------------------------------------------- | ------------------------------------------------- |
| Hardcoding `PORT=3000`           | App fails to start on platform's assigned port | Use `process.env.PORT \|\| 3000`                  |
| Pushing `node_modules`           | Bloats repo, causes version conflicts          | Add to `.gitignore`                               |
| Missing `start` script           | Platform doesn't know how to run your app      | Add `"start": "node server.js"` to `package.json` |
| Using `nodemon` in production    | Unnecessary restarts, not production-grade     | Use `node` directly or PM2                        |
| Not setting `NODE_ENV`           | Missing production optimizations               | Set `NODE_ENV=production` in env vars             |
| Ignoring cold start on free tier | First request after inactivity is slow         | Upgrade plan or use health check pings            |
| Forgetting `engines` field       | Platform may use wrong Node.js version         | Specify `"node": ">=18.0.0"`                      |

---

## Summary

- **Render & Railway** are the best starting points — free/cheap, easy GitHub integration, auto-deploy
- **AWS EC2** gives full control but requires more DevOps knowledge
- **Vercel** is ideal for serverless functions and frontend frameworks, not traditional Express apps
- Always use environment variables, specify your Node version, and listen on `process.env.PORT`
- Your app needs a health check endpoint and proper start script to deploy successfully
