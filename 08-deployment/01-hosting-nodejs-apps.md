# Hosting Node.js Apps

## What is Hosting?

Hosting is the process of making your application accessible on the internet by running it on a remote server. For Node.js apps, this means placing your code on a machine that can execute Node.js, listen on a port, and respond to HTTP requests from users worldwide.

Unlike static sites (which only serve HTML/CSS/JS files), Node.js apps require a **runtime environment** — a server that actively runs your JavaScript code, handles database connections, processes API requests, and manages state.

## The Deployment Landscape

```mermaid
flowchart TD
    A[Your Node.js App] --> B{Choose a Platform}
    B --> C[PaaS - Platform as a Service]
    B --> D[IaaS - Infrastructure as a Service]
    B --> E[Containerized - Docker/K8s]
    C --> C1[Render]
    C --> C2[Railway]
    C --> C3[Fly.io]
    C --> C4[Heroku]
    D --> D1[AWS EC2]
    D --> D2[DigitalOcean Droplets]
    D --> D3[Google Compute Engine]
    E --> E1[AWS ECS/Fargate]
    E --> E2[Google Cloud Run]
    E --> E3[Azure Container Apps]
```

### PaaS vs IaaS

| Aspect            | PaaS (Render, Railway)      | IaaS (AWS EC2, DigitalOcean)           |
| ----------------- | --------------------------- | -------------------------------------- |
| Setup complexity  | Low — push code, it deploys | High — configure OS, firewall, runtime |
| Server management | Platform handles it         | You manage everything                  |
| Scaling           | Auto-scaling built in       | Manual or custom auto-scaling          |
| Cost at scale     | Can get expensive           | More cost-efficient at scale           |
| Learning curve    | Beginner-friendly           | Requires DevOps knowledge              |
| Customization     | Limited                     | Full control                           |

---

## Render

Render is a modern cloud platform that has become one of the most popular Heroku alternatives. It offers automatic deployments from Git, free SSL, and a generous free tier.

### Deployment Process

1. **Push your code to GitHub/GitLab**
2. **Create a new Web Service on Render**
3. **Connect your repository**
4. **Configure build and start commands**
5. **Deploy**

### Render Configuration

```yaml
# render.yaml — Infrastructure as Code for Render
services:
  - type: web
    name: my-node-api
    env: node
    region: oregon
    plan: free
    buildCommand: npm install
    startCommand: node server.js
    envVars:
      - key: NODE_ENV
        value: production
      - key: MONGODB_URI
        fromDatabase:
          name: my-mongo-db
          property: connectionString
    autoDeploy: true
```

### Key Settings on Render Dashboard

| Setting           | Value                           | Purpose                        |
| ----------------- | ------------------------------- | ------------------------------ |
| Build Command     | `npm install`                   | Install dependencies           |
| Start Command     | `node server.js` or `npm start` | Run your app                   |
| Environment       | Node                            | Runtime environment            |
| Auto-Deploy       | Yes                             | Deploy on every push to branch |
| Health Check Path | `/health`                       | Monitors if app is alive       |

### Example Express App for Render

```javascript
// server.js
const express = require("express");
const app = express();

const PORT = process.env.PORT || 3000;

app.get("/", (req, res) => {
  res.json({ message: "Hello from Render!" });
});

app.get("/health", (req, res) => {
  res.status(200).json({ status: "healthy", timestamp: Date.now() });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

> **Important**: Always use `process.env.PORT` — Render assigns a dynamic port to your app.

---

## Railway

Railway focuses on developer experience with an intuitive UI and instant deployments. It detects your project type automatically and configures the environment.

### Deployment Process

1. **Connect GitHub repository to Railway**
2. **Railway auto-detects Node.js and installs dependencies**
3. **Configure environment variables in the dashboard**
4. **Deploy triggers automatically on push**

### Railway Configuration

```json
// railway.json (optional — Railway auto-detects most settings)
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": {
    "builder": "NIXPACKS",
    "buildCommand": "npm install && npm run build"
  },
  "deploy": {
    "startCommand": "npm start",
    "healthcheckPath": "/health",
    "healthcheckTimeout": 30,
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 3
  }
}
```

### Railway CLI Deployment

```bash
# Install Railway CLI
npm install -g @railway/cli

# Login to Railway
railway login

# Initialize project
railway init

# Deploy
railway up

# Open the deployed app
railway open
```

---

## Heroku Alternatives Comparison

After Heroku removed its free tier in 2022, many developers migrated to alternatives:

| Platform     | Free Tier             | Sleep on Idle                 | Custom Domains | Auto-Deploy   | Database                   |
| ------------ | --------------------- | ----------------------------- | -------------- | ------------- | -------------------------- |
| Render       | Yes (750 hrs/month)   | Yes (spins down after 15 min) | Yes (free)     | Yes           | PostgreSQL free tier       |
| Railway      | $5 credit/month       | No                            | Yes            | Yes           | PostgreSQL, MySQL, Redis   |
| Fly.io       | Yes (3 shared VMs)    | No                            | Yes            | Yes (via CLI) | PostgreSQL (via extension) |
| Cyclic       | Yes                   | No (always on)                | Yes            | Yes           | DynamoDB integration       |
| Adaptable.io | Yes                   | Yes                           | Yes            | Yes           | MongoDB, PostgreSQL        |
| Koyeb        | Yes (1 nano instance) | No                            | Yes            | Yes           | External only              |

---

## AWS Overview for Node.js

AWS provides multiple ways to host Node.js apps, each suited to different scales:

```mermaid
flowchart LR
    A[AWS Node.js Hosting Options] --> B[EC2]
    A --> C[Elastic Beanstalk]
    A --> D[Lambda + API Gateway]
    A --> E[ECS/Fargate]
    A --> F[App Runner]

    B --> B1[Full VM control\nManual setup]
    C --> C1[PaaS-like experience\nAuto-scaling]
    D --> D1[Serverless\nPay per invocation]
    E --> E1[Container orchestration\nDocker-based]
    F --> F1[Simplified containers\nAuto-scaling]
```

### AWS Elastic Beanstalk (Simplest AWS Option)

```bash
# Install EB CLI
pip install awsebcli

# Initialize Elastic Beanstalk
eb init my-node-app --platform node.js --region us-east-1

# Create environment and deploy
eb create production-env

# Deploy updates
eb deploy

# Open in browser
eb open
```

### AWS Lambda (Serverless)

```javascript
// handler.js — AWS Lambda function
exports.handler = async (event) => {
  const response = {
    statusCode: 200,
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ message: "Hello from Lambda!" }),
  };
  return response;
};
```

---

## Connecting to Cloud MongoDB (MongoDB Atlas)

MongoDB Atlas is the official cloud-hosted MongoDB service. It provides a free tier (M0) with 512 MB storage — enough for development and small production apps.

### Setup Process

```mermaid
flowchart TD
    A[Create MongoDB Atlas Account] --> B[Create a Cluster - M0 Free]
    B --> C[Create Database User]
    C --> D[Whitelist IP Addresses]
    D --> E[Get Connection String]
    E --> F[Add to Environment Variables]
    F --> G[Connect from Node.js App]
```

### Step-by-Step Atlas Setup

1. **Sign up** at [mongodb.com/atlas](https://www.mongodb.com/atlas)
2. **Create a free cluster** (M0 Sandbox — AWS, GCP, or Azure)
3. **Create a database user** with a strong password
4. **Network Access**: Add `0.0.0.0/0` to allow connections from any IP (required for platforms like Render/Railway)
5. **Get connection string**: Click "Connect" → "Connect your application"

### Connecting with Mongoose

```javascript
// db.js
const mongoose = require("mongoose");

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGODB_URI, {
      // These options are set by default in Mongoose 6+
      // but explicit for clarity
    });
    console.log(`MongoDB Connected: ${conn.connection.host}`);
  } catch (error) {
    console.error(`Error: ${error.message}`);
    process.exit(1);
  }
};

module.exports = connectDB;
```

### Connection String Format

```
mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/<dbname>?retryWrites=true&w=majority
```

| Component                    | Description                                       |
| ---------------------------- | ------------------------------------------------- |
| `mongodb+srv://`             | Protocol with DNS seedlist                        |
| `<username>`                 | Database user (not Atlas account)                 |
| `<password>`                 | Database user password (URL-encode special chars) |
| `cluster0.xxxxx.mongodb.net` | Your cluster address                              |
| `<dbname>`                   | Default database name                             |
| `retryWrites=true`           | Automatically retry failed writes                 |
| `w=majority`                 | Write concern — acknowledged by majority of nodes |

---

## Configuring Environment Variables in Production

Environment variables keep sensitive data out of your codebase. Each platform handles them differently:

### On Render

```bash
# Via Dashboard: Dashboard → Service → Environment → Add Environment Variable

# Via render.yaml
envVars:
  - key: MONGODB_URI
    value: mongodb+srv://user:pass@cluster.mongodb.net/mydb
  - key: JWT_SECRET
    value: your-super-secret-key
  - key: NODE_ENV
    value: production
```

### On Railway

```bash
# Via Dashboard: Project → Variables → Add Variable

# Via Railway CLI
railway variables set MONGODB_URI="mongodb+srv://user:pass@cluster.mongodb.net/mydb"
railway variables set JWT_SECRET="your-super-secret-key"
railway variables set NODE_ENV="production"
```

### On AWS Elastic Beanstalk

```bash
# Via EB CLI
eb setenv MONGODB_URI="mongodb+srv://..." JWT_SECRET="..." NODE_ENV="production"

# Via AWS Console: Elastic Beanstalk → Configuration → Software → Environment Properties
```

### Accessing in Node.js

```javascript
// Always access via process.env
const mongoURI = process.env.MONGODB_URI;
const jwtSecret = process.env.JWT_SECRET;
const nodeEnv = process.env.NODE_ENV;

// Validate required variables on startup
const requiredEnvVars = ["MONGODB_URI", "JWT_SECRET", "NODE_ENV"];
for (const envVar of requiredEnvVars) {
  if (!process.env[envVar]) {
    console.error(`FATAL: ${envVar} is not defined`);
    process.exit(1);
  }
}
```

---

## Free Tier Options Comparison

| Platform           | Free Compute     | RAM         | Storage      | Bandwidth          | Sleep Behavior                 | Database Free Tier         |
| ------------------ | ---------------- | ----------- | ------------ | ------------------ | ------------------------------ | -------------------------- |
| Render             | 750 hrs/month    | 512 MB      | —            | 100 GB/month       | Sleeps after 15 min inactivity | PostgreSQL: 256 MB         |
| Railway            | $5 credit/month  | 512 MB      | 1 GB         | $5 worth           | No sleep                       | PostgreSQL, Redis included |
| Fly.io             | 3 shared-cpu VMs | 256 MB each | 3 GB volumes | —                  | No sleep                       | Postgres via Supabase      |
| MongoDB Atlas (M0) | —                | —           | 512 MB       | —                  | —                              | Free forever (512 MB)      |
| PlanetScale        | —                | —           | 5 GB         | 1B row reads/month | —                              | MySQL free tier            |
| Supabase           | —                | —           | 500 MB       | 2 GB bandwidth     | Pauses after 1 week            | PostgreSQL free            |

---

## Complete Deployment Example

Here is a full example deploying an Express + MongoDB app to Render:

### Project Structure

```
my-api/
├── server.js
├── package.json
├── .env              ← local only (gitignored)
├── .gitignore
├── config/
│   └── db.js
├── routes/
│   └── users.js
└── models/
    └── User.js
```

### package.json

```json
{
  "name": "my-api",
  "version": "1.0.0",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.6.0",
    "dotenv": "^16.3.1",
    "cors": "^2.8.5",
    "helmet": "^7.1.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.2"
  },
  "engines": {
    "node": ">=18.0.0"
  }
}
```

### .gitignore

```
node_modules/
.env
.env.local
```

### server.js

```javascript
const express = require("express");
const cors = require("cors");
const helmet = require("helmet");
const connectDB = require("./config/db");

// Load env vars only in development
if (process.env.NODE_ENV !== "production") {
  require("dotenv").config();
}

const app = express();

// Connect to MongoDB
connectDB();

// Middleware
app.use(helmet());
app.use(cors());
app.use(express.json());

// Routes
app.use("/api/users", require("./routes/users"));

// Health check
app.get("/health", (req, res) => {
  res.status(200).json({ status: "ok" });
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`Server running in ${process.env.NODE_ENV} mode on port ${PORT}`);
});
```

---

## Best Practices

- Always use `process.env.PORT` — cloud platforms assign dynamic ports to your app.
- Set `"engines"` in `package.json` to lock the Node.js version on the server.
- Add a `/health` endpoint for platform health checks and monitoring.
- Never commit `.env` files — use platform-specific environment variable settings.
- Use `helmet()` middleware for security headers in production.
- Enable CORS only for your frontend domain in production, not `*`.
- Set `NODE_ENV=production` to enable framework optimizations and disable verbose errors.
- Whitelist `0.0.0.0/0` on MongoDB Atlas only if your hosting platform uses dynamic IPs.
- Use connection pooling and handle MongoDB connection errors gracefully.
- Add a `render.yaml` or `railway.json` for Infrastructure as Code — reproducible deployments.

## Common Mistakes

| Mistake                                      | Why It Is a Problem                                        |
| -------------------------------------------- | ---------------------------------------------------------- |
| Hardcoding the port (`app.listen(3000)`)     | Cloud platforms inject their own PORT; your app will crash |
| Committing `.env` to Git                     | Exposes secrets (database passwords, API keys) publicly    |
| Not whitelisting IPs on MongoDB Atlas        | Connection refused errors in production                    |
| Using `nodemon` in production start script   | Unnecessary overhead; `nodemon` is a dev tool              |
| Forgetting `engines` field in `package.json` | Platform may use an incompatible Node.js version           |
| Not handling graceful shutdown               | Database connections stay open; data corruption risk       |
| Ignoring free tier sleep behavior            | First request after sleep takes 30–50 seconds (cold start) |

## Summary

- Hosting a Node.js app means running it on a remote server with a Node.js runtime that listens for HTTP requests.
- PaaS platforms (Render, Railway, Fly.io) are the easiest path — push code, get a URL.
- AWS offers more control but requires DevOps knowledge (EC2, Elastic Beanstalk, Lambda, App Runner).
- MongoDB Atlas provides free cloud database hosting with 512 MB storage on the M0 tier.
- Environment variables are the standard way to manage secrets in production — never hardcode them.
- Always use `process.env.PORT`, add health checks, set engine versions, and validate required env vars on startup.
- Render and Railway have replaced Heroku as the go-to platforms for indie developers and small teams.
