# Connecting to Cloud MongoDB (MongoDB Atlas)

## What / Why

**MongoDB Atlas** is a fully managed cloud database service by MongoDB. Instead of installing and maintaining MongoDB on your own server, Atlas handles backups, scaling, security patches, and monitoring for you.

**Analogy:** Running MongoDB locally is like hosting files on your own hard drive — it works, but you're responsible for backups and access. Atlas is like Google Drive — managed, always available, accessible from anywhere, and automatically backed up.

**Why use Atlas in production?**

- Your hosting platform (Render, Railway) doesn't come with MongoDB installed
- Local MongoDB only works on your machine
- Atlas provides automatic backups, monitoring, and global replication
- Free M0 tier is perfect for learning and small projects

```mermaid
graph LR
    A[Express App on Render] -->|Connection String| B[MongoDB Atlas Cluster]
    C[Express App on Railway] -->|Connection String| B
    D[Local Dev Machine] -->|Connection String| B
    B --> E[(Database: myapp)]
    E --> F[Collection: users]
    E --> G[Collection: posts]
```

---

## Setting Up MongoDB Atlas

### Step 1: Create an Account & Cluster

1. Go to [cloud.mongodb.com](https://cloud.mongodb.com)
2. Sign up / Log in
3. Click **Build a Database**
4. Select **M0 FREE** tier
5. Choose a cloud provider (AWS/GCP/Azure) and region closest to your hosting
6. Name your cluster (e.g., `Cluster0`)

### Step 2: Create a Database User

Navigate to **Database Access** → **Add New Database User**:

```
Username: myapp_user
Password: (generate a strong password)
Role: Read and Write to Any Database
```

> ⚠️ This is NOT your Atlas account password. This is the database-level authentication.

### Step 3: Configure Network Access (IP Whitelist)

Navigate to **Network Access** → **Add IP Address**:

```
# For development - allow your current IP
My Current IP: 203.0.113.42

# For production - allow access from anywhere
# (Hosting platforms use dynamic IPs)
Access from Anywhere: 0.0.0.0/0
```

> 🔒 `0.0.0.0/0` means any IP can attempt to connect. Security relies on your username/password. For extra security in production, use VPC peering or Atlas private endpoints.

### Step 4: Get the Connection String

1. Go to **Database** → **Connect** → **Connect your application**
2. Select **Node.js** driver, version **5.5 or later**
3. Copy the connection string:

```
mongodb+srv://myapp_user:<password>@cluster0.abc123.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0
```

4. Replace `<password>` with your actual database user password
5. Add your database name before the `?`:

```
mongodb+srv://myapp_user:MyP@ss123@cluster0.abc123.mongodb.net/myapp?retryWrites=true&w=majority&appName=Cluster0
```

---

## Connecting from Express with Mongoose

### Install Mongoose

```bash
npm install mongoose dotenv
```

### Project Structure

```
my-api/
├── .env                 # Local secrets (never commit)
├── .env.example         # Template for other developers
├── .gitignore
├── server.js
├── config/
│   └── db.js           # Database connection logic
├── models/
│   └── User.js
└── routes/
    └── users.js
```

### `.env` File (Local Development)

```env
PORT=3000
MONGODB_URI=mongodb+srv://myapp_user:MyP@ss123@cluster0.abc123.mongodb.net/myapp?retryWrites=true&w=majority&appName=Cluster0
NODE_ENV=development
```

### `.env.example` (Commit this — it's a template)

```env
PORT=3000
MONGODB_URI=mongodb+srv://<username>:<password>@<cluster>.mongodb.net/<dbname>?retryWrites=true&w=majority
NODE_ENV=development
```

### Database Connection (`config/db.js`)

```javascript
const mongoose = require("mongoose");

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGODB_URI);
    console.log(`MongoDB Connected: ${conn.connection.host}`);
  } catch (error) {
    console.error(`MongoDB Connection Error: ${error.message}`);
    process.exit(1); // Exit process with failure
  }
};

module.exports = connectDB;
```

### Server Entry Point (`server.js`)

```javascript
require("dotenv").config(); // Load .env variables
const express = require("express");
const connectDB = require("./config/db");

const app = express();
const PORT = process.env.PORT || 3000;

// Connect to MongoDB
connectDB();

// Middleware
app.use(express.json());

// Routes
app.get("/health", (req, res) => {
  res.json({
    status: "ok",
    db: mongoose.connection.readyState === 1 ? "connected" : "disconnected",
  });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT} in ${process.env.NODE_ENV} mode`);
});
```

---

## Handling Connection Errors

### Connection Events

```javascript
const mongoose = require("mongoose");

const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGODB_URI);
  } catch (error) {
    console.error("Initial connection failed:", error.message);
    process.exit(1);
  }
};

// Connection event listeners
mongoose.connection.on("connected", () => {
  console.log("✅ Mongoose connected to Atlas");
});

mongoose.connection.on("error", (err) => {
  console.error("❌ Mongoose connection error:", err.message);
});

mongoose.connection.on("disconnected", () => {
  console.warn("⚠️ Mongoose disconnected from Atlas");
});

// Graceful shutdown
process.on("SIGINT", async () => {
  await mongoose.connection.close();
  console.log("MongoDB connection closed due to app termination");
  process.exit(0);
});

module.exports = connectDB;
```

### Connection with Retry Logic

```javascript
const mongoose = require("mongoose");

const connectDB = async (retries = 5) => {
  for (let i = 0; i < retries; i++) {
    try {
      await mongoose.connect(process.env.MONGODB_URI);
      console.log("MongoDB Connected");
      return;
    } catch (error) {
      console.error(`Connection attempt ${i + 1} failed: ${error.message}`);
      if (i < retries - 1) {
        const delay = Math.pow(2, i) * 1000; // Exponential backoff
        console.log(`Retrying in ${delay / 1000}s...`);
        await new Promise((res) => setTimeout(res, delay));
      }
    }
  }
  console.error("All connection attempts failed. Exiting.");
  process.exit(1);
};

module.exports = connectDB;
```

---

## Connection String Anatomy

```
mongodb+srv://myapp_user:MyP@ss123@cluster0.abc123.mongodb.net/myapp?retryWrites=true&w=majority
└──┬──────┘  └───┬────┘ └───┬───┘ └─────────┬─────────────────┘└─┬─┘ └───────────┬────────────┘
 Protocol    Username  Password        Cluster Host          DB Name    Options
```

| Part                          | Description                                       |
| ----------------------------- | ------------------------------------------------- |
| `mongodb+srv://`              | SRV protocol (auto-discovers replica set nodes)   |
| `myapp_user`                  | Database username (created in Step 2)             |
| `MyP@ss123`                   | Database password (URL-encoded if special chars)  |
| `cluster0.abc123.mongodb.net` | Atlas cluster hostname                            |
| `/myapp`                      | Default database name                             |
| `retryWrites=true`            | Automatically retry failed writes                 |
| `w=majority`                  | Write concern — acknowledged by majority of nodes |

---

## Using Environment Variables in Hosting

### On Render

Dashboard → Your Service → **Environment** tab:

```
Key: MONGODB_URI
Value: mongodb+srv://myapp_user:MyP@ss123@cluster0.abc123.mongodb.net/myapp?retryWrites=true&w=majority
```

### On Railway

Dashboard → Your Project → **Variables** tab:

```
MONGODB_URI = mongodb+srv://myapp_user:MyP@ss123@cluster0.abc123.mongodb.net/myapp?retryWrites=true&w=majority
```

> 💡 No need for `dotenv` in production — hosting platforms inject env vars directly into `process.env`.

---

## Best Practices

1. **Never hardcode the connection string** — always use environment variables
2. **Use a dedicated database user per app** — principle of least privilege
3. **Allow `0.0.0.0/0` for production** on platforms with dynamic IPs, rely on strong credentials
4. **Choose the Atlas region closest to your hosting** — reduces latency
5. **Use connection pooling** — Mongoose handles this by default (pool size 100)
6. **Add retry logic** with exponential backoff for production resilience
7. **Monitor with Atlas dashboard** — track slow queries, connections, and storage
8. **URL-encode special characters** in passwords (`@` → `%40`, `#` → `%23`)
9. **Set `appName` in connection string** — helps identify your app in Atlas monitoring

---

## Common Mistakes

| Mistake                             | Problem                        | Fix                                 |
| ----------------------------------- | ------------------------------ | ----------------------------------- |
| Forgetting to replace `<password>`  | Authentication fails           | Replace with actual password        |
| Not adding IP to whitelist          | Connection timeout             | Add `0.0.0.0/0` or your specific IP |
| Special characters in password      | Connection string breaks       | URL-encode the password             |
| Missing database name in URI        | Data goes to `test` database   | Add `/myapp` before the `?`         |
| Using `localhost` URI in production | Can't connect to remote DB     | Use Atlas connection string         |
| Committing `.env` to GitHub         | Credentials exposed publicly   | Add `.env` to `.gitignore`          |
| Not handling connection errors      | App crashes silently           | Add try/catch and event listeners   |
| Wrong driver version selected       | Incompatible connection string | Choose Node.js 5.5+ in Atlas UI     |

---

## Summary

- **MongoDB Atlas** provides a free managed MongoDB cluster accessible from anywhere
- Setup flow: Create Cluster → Create DB User → Whitelist IPs → Get Connection String
- Use **Mongoose** to connect from Express, storing the URI in environment variables
- Always handle connection errors with retry logic and event listeners
- The connection string contains: protocol, credentials, host, database name, and options
- In production, set `MONGODB_URI` in your hosting platform's environment variables — no `.env` file needed
