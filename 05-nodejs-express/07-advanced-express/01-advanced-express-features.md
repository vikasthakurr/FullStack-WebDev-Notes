# Advanced Express Features

## Express Router (Modular Routing)

Organize routes into separate files for maintainability:

```javascript
// routes/userRoutes.js
const express = require("express");
const router = express.Router();
const userController = require("../controllers/userController");

router.get("/", userController.getAll);
router.get("/:id", userController.getOne);
router.post("/", userController.create);
router.put("/:id", userController.update);
router.delete("/:id", userController.delete);

module.exports = router;
```

```javascript
// routes/index.js
const express = require("express");
const router = express.Router();

router.use("/users", require("./userRoutes"));
router.use("/posts", require("./postRoutes"));
router.use("/auth", require("./authRoutes"));

module.exports = router;
```

```javascript
// app.js
const express = require("express");
const routes = require("./routes");

const app = express();
app.use(express.json());
app.use("/api/v1", routes); // All routes under /api/v1

module.exports = app;
```

---

## Request Validation

### Using Joi

```bash
npm install joi
```

```javascript
const Joi = require("joi");

const userSchema = Joi.object({
  name: Joi.string().min(2).max(50).required(),
  email: Joi.string().email().required(),
  age: Joi.number().integer().min(18).max(120),
  role: Joi.string().valid("user", "admin", "moderator").default("user"),
});

function validate(schema) {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, { abortEarly: false });

    if (error) {
      const details = error.details.map((d) => d.message);
      return res.status(400).json({ error: "Validation failed", details });
    }

    req.body = value; // Use validated/sanitized values
    next();
  };
}

// Usage
app.post("/api/users", validate(userSchema), (req, res) => {
  // req.body is validated and sanitized
  res.status(201).json(req.body);
});
```

### Using express-validator

```bash
npm install express-validator
```

```javascript
const { body, validationResult } = require("express-validator");

app.post(
  "/api/users",
  [
    body("name").trim().notEmpty().withMessage("Name is required"),
    body("email")
      .isEmail()
      .normalizeEmail()
      .withMessage("Valid email required"),
    body("password").isLength({ min: 8 }).withMessage("Min 8 characters"),
    body("age").optional().isInt({ min: 18 }),
  ],
  (req, res) => {
    const errors = validationResult(req);

    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }

    res.status(201).json(req.body);
  },
);
```

---

## File Uploads with Multer

```bash
npm install multer
```

```javascript
const multer = require("multer");
const path = require("path");

// Configure storage
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, "uploads/");
  },
  filename: (req, file, cb) => {
    const uniqueName = `${Date.now()}-${Math.round(Math.random() * 1e9)}`;
    cb(null, `${uniqueName}${path.extname(file.originalname)}`);
  },
});

// File filter
const fileFilter = (req, file, cb) => {
  const allowedTypes = ["image/jpeg", "image/png", "image/gif"];
  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error("Only JPEG, PNG, and GIF images are allowed"), false);
  }
};

const upload = multer({
  storage,
  fileFilter,
  limits: { fileSize: 5 * 1024 * 1024 }, // 5MB
});

// Single file upload
app.post("/api/avatar", upload.single("avatar"), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: "No file uploaded" });
  }
  res.json({
    message: "File uploaded",
    filename: req.file.filename,
    size: req.file.size,
  });
});

// Multiple files
app.post("/api/gallery", upload.array("photos", 10), (req, res) => {
  res.json({ uploaded: req.files.length });
});
```

---

## Logging with Morgan & Winston

### Morgan (HTTP Request Logging)

```bash
npm install morgan
```

```javascript
const morgan = require("morgan");

// Predefined formats
app.use(morgan("dev")); // Colored concise output for development
app.use(morgan("combined")); // Apache-style logs for production

// Custom format
app.use(
  morgan(":method :url :status :res[content-length] - :response-time ms"),
);

// Log to file
const fs = require("fs");
const logStream = fs.createWriteStream("./logs/access.log", { flags: "a" });
app.use(morgan("combined", { stream: logStream }));
```

### Winston (Application Logging)

```bash
npm install winston
```

```javascript
const winston = require("winston");

const logger = winston.createLogger({
  level: "info",
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),
  transports: [
    new winston.transports.File({ filename: "logs/error.log", level: "error" }),
    new winston.transports.File({ filename: "logs/combined.log" }),
  ],
});

// Add console in development
if (process.env.NODE_ENV !== "production") {
  logger.add(
    new winston.transports.Console({
      format: winston.format.simple(),
    }),
  );
}

// Usage
logger.info("Server started", { port: 3000 });
logger.error("Database connection failed", { error: err.message });
logger.warn("High memory usage", { usage: process.memoryUsage() });
```

---

## Sending Emails with Nodemailer

```bash
npm install nodemailer
```

```javascript
const nodemailer = require("nodemailer");

const transporter = nodemailer.createTransport({
  host: "smtp.gmail.com",
  port: 587,
  secure: false,
  auth: {
    user: process.env.EMAIL_USER,
    pass: process.env.EMAIL_PASS, // App password, not regular password
  },
});

async function sendWelcomeEmail(to, name) {
  const mailOptions = {
    from: '"My App" <noreply@myapp.com>',
    to,
    subject: "Welcome to Our App!",
    html: `
      <h1>Welcome, ${name}!</h1>
      <p>Thanks for signing up. Get started by exploring our features.</p>
      <a href="https://myapp.com/dashboard">Go to Dashboard</a>
    `,
  };

  const info = await transporter.sendMail(mailOptions);
  console.log("Email sent:", info.messageId);
}

// In route handler
app.post("/api/register", async (req, res) => {
  // ... create user ...
  await sendWelcomeEmail(user.email, user.name);
  res.status(201).json({ message: "User registered" });
});
```

---

## Environment Variables

```bash
npm install dotenv
```

```
# .env (never commit this file)
PORT=3000
NODE_ENV=development
DB_URI=mongodb://localhost:27017/myapp
JWT_SECRET=your-secret-key-here
EMAIL_USER=you@gmail.com
EMAIL_PASS=app-specific-password
```

```javascript
require("dotenv").config(); // Load at the very top

const PORT = process.env.PORT || 3000;
const dbUri = process.env.DB_URI;
```

Add `.env` to `.gitignore`:

```
.env
.env.local
.env.production
```

---

## CORS (Cross-Origin Resource Sharing)

```bash
npm install cors
```

```javascript
const cors = require("cors");

// Allow all origins (development only)
app.use(cors());

// Specific origins (production)
app.use(
  cors({
    origin: ["https://myapp.com", "https://admin.myapp.com"],
    methods: ["GET", "POST", "PUT", "DELETE"],
    allowedHeaders: ["Content-Type", "Authorization"],
    credentials: true,
  }),
);
```

---

## Project Structure (Full)

```
project/
├── src/
│   ├── config/
│   │   └── db.js
│   ├── controllers/
│   │   ├── authController.js
│   │   └── userController.js
│   ├── middleware/
│   │   ├── auth.js
│   │   ├── errorHandler.js
│   │   └── validate.js
│   ├── models/
│   │   └── User.js
│   ├── routes/
│   │   ├── index.js
│   │   ├── authRoutes.js
│   │   └── userRoutes.js
│   ├── utils/
│   │   ├── email.js
│   │   └── logger.js
│   ├── app.js            ← Express config (middleware, routes)
│   └── server.js         ← Entry point (listen)
├── uploads/
├── logs/
├── .env
├── .gitignore
└── package.json
```

---

## Summary

- Use Express Router to split routes into modular files by resource.
- Validate input with Joi or express-validator — never trust client data.
- Handle file uploads with Multer — configure storage, size limits, and file type filters.
- Log HTTP requests with Morgan and application events with Winston.
- Use Nodemailer for transactional emails (welcome, password reset, notifications).
- Protect sensitive config with environment variables (`.env` + `dotenv`).
- Enable CORS for frontend-backend communication across different origins.
