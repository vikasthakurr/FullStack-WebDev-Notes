# Logging with Morgan/Winston and Sending Emails with Nodemailer

## Why Logging and Emails Matter

Logging is your app's flight recorder — when something crashes at 3 AM, logs tell you what happened. Emails let your app communicate with users (verification, password resets, notifications). Both are essential for any production application.

---

## Part 1: Morgan — HTTP Request Logging

Morgan is an HTTP request logger middleware for Express. It logs every incoming request automatically.

### Installation

```bash
npm install morgan
```

### Basic Usage

```javascript
const express = require("express");
const morgan = require("morgan");
const app = express();

// Use predefined format
app.use(morgan("dev"));

app.get("/api/users", (req, res) => {
  res.json([{ id: 1, name: "Vikas" }]);
});

app.listen(3000);
```

### Predefined Formats

```javascript
// 'dev' — Colored, concise output for development
// GET /api/users 200 5.312 ms - 42
app.use(morgan("dev"));

// 'combined' — Apache-style logs for production
// ::1 - - [15/Jan/2024:10:30:00 +0000] "GET /api/users HTTP/1.1" 200 42
app.use(morgan("combined"));

// 'common' — Standard Apache format (no response time)
app.use(morgan("common"));

// 'short' — Shorter than default, includes response time
app.use(morgan("short"));

// 'tiny' — Minimal output
// GET /api/users 200 42 - 5.312 ms
app.use(morgan("tiny"));
```

### Custom Format

```javascript
// Custom tokens
app.use(morgan(":method :url :status :response-time ms - :date[iso]"));

// Output: GET /api/users 200 4.233 ms - 2024-01-15T10:30:00.000Z
```

### Log to a File

```javascript
const fs = require("fs");
const path = require("path");

// Create a write stream (append mode)
const accessLogStream = fs.createWriteStream(
  path.join(__dirname, "logs", "access.log"),
  { flags: "a" },
);

// Log to file in production, console in development
if (process.env.NODE_ENV === "production") {
  app.use(morgan("combined", { stream: accessLogStream }));
} else {
  app.use(morgan("dev"));
}
```

---

## Part 2: Winston — Application Logging

Winston is a versatile logging library with support for multiple transports (destinations), log levels, and formatting.

### Installation

```bash
npm install winston
```

### Basic Setup

```javascript
const winston = require("winston");

const logger = winston.createLogger({
  level: "info", // Minimum level to log
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),
  transports: [
    // Write to files
    new winston.transports.File({ filename: "logs/error.log", level: "error" }),
    new winston.transports.File({ filename: "logs/combined.log" }),
  ],
});

// Also log to console in development
if (process.env.NODE_ENV !== "production") {
  logger.add(
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple(),
      ),
    }),
  );
}

module.exports = logger;
```

### Log Levels (Severity)

```javascript
// Winston default levels (npm levels): lower number = higher severity
// { error: 0, warn: 1, info: 2, http: 3, verbose: 4, debug: 5, silly: 6 }

logger.error("Database connection failed", { host: "localhost", port: 5432 });
logger.warn("Deprecated API endpoint called", { endpoint: "/api/v1/old" });
logger.info("User registered successfully", { userId: 123 });
logger.http("GET /api/users 200");
logger.debug("Query result", { rows: 50, time: "23ms" });
```

### Transports (Where Logs Go)

```javascript
const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || "info",
  format: winston.format.combine(
    winston.format.timestamp({ format: "YYYY-MM-DD HH:mm:ss" }),
    winston.format.errors({ stack: true }), // Log stack traces
    winston.format.json(),
  ),
  defaultMeta: { service: "user-service" },
  transports: [
    // Error logs — separate file
    new winston.transports.File({
      filename: "logs/error.log",
      level: "error",
      maxsize: 5242880, // 5MB
      maxFiles: 5, // Keep 5 rotated files
    }),

    // All logs — combined file
    new winston.transports.File({
      filename: "logs/combined.log",
      maxsize: 5242880,
      maxFiles: 10,
    }),

    // Console output — development
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.printf(({ timestamp, level, message, ...meta }) => {
          return `${timestamp} [${level}]: ${message} ${Object.keys(meta).length ? JSON.stringify(meta) : ""}`;
        }),
      ),
    }),
  ],
});
```

### Using Winston with Express

```javascript
const logger = require("./utils/logger");

// Replace morgan with Winston for HTTP logging
app.use((req, res, next) => {
  const start = Date.now();

  res.on("finish", () => {
    const duration = Date.now() - start;
    logger.http(`${req.method} ${req.url}`, {
      status: res.statusCode,
      duration: `${duration}ms`,
      ip: req.ip,
    });
  });

  next();
});

// In route handlers
app.post("/api/users", async (req, res) => {
  try {
    const user = await createUser(req.body);
    logger.info("User created", { userId: user.id, email: user.email });
    res.status(201).json(user);
  } catch (err) {
    logger.error("Failed to create user", {
      error: err.message,
      stack: err.stack,
    });
    res.status(500).json({ error: "Internal server error" });
  }
});
```

---

## Part 3: Nodemailer — Sending Emails

Nodemailer is the standard library for sending emails from Node.js.

### Installation

```bash
npm install nodemailer
```

### Basic SMTP Transport Setup

```javascript
const nodemailer = require("nodemailer");

// Create reusable transporter
const transporter = nodemailer.createTransport({
  host: process.env.SMTP_HOST, // e.g., 'smtp.gmail.com'
  port: process.env.SMTP_PORT || 587,
  secure: false, // true for 465, false for 587
  auth: {
    user: process.env.SMTP_USER,
    pass: process.env.SMTP_PASS, // App password, not account password
  },
});

// Verify connection
transporter.verify((error, success) => {
  if (error) {
    console.error("SMTP connection failed:", error);
  } else {
    console.log("SMTP server ready");
  }
});
```

### sendMail Options

```javascript
async function sendEmail() {
  const mailOptions = {
    from: '"My App" <noreply@myapp.com>', // Sender
    to: "user@example.com", // Recipient(s) - comma separated
    cc: "manager@example.com", // CC (optional)
    bcc: "admin@example.com", // BCC (optional)
    subject: "Welcome to Our App!", // Subject line
    text: "Hello! Welcome to our app.", // Plain text body
    html: "<h1>Hello!</h1><p>Welcome to our app.</p>", // HTML body
  };

  try {
    const info = await transporter.sendMail(mailOptions);
    console.log("Email sent:", info.messageId);
    return info;
  } catch (error) {
    console.error("Email failed:", error);
    throw error;
  }
}
```

### HTML Emails with Templates

```javascript
function getWelcomeEmailHTML(userName) {
  return `
    <!DOCTYPE html>
    <html>
    <head>
      <style>
        .container { max-width: 600px; margin: 0 auto; font-family: Arial; }
        .header { background: #4F46E5; color: white; padding: 20px; text-align: center; }
        .body { padding: 20px; }
        .button { background: #4F46E5; color: white; padding: 12px 24px;
                  text-decoration: none; border-radius: 4px; display: inline-block; }
        .footer { color: #666; font-size: 12px; padding: 20px; text-align: center; }
      </style>
    </head>
    <body>
      <div class="container">
        <div class="header">
          <h1>Welcome, ${userName}!</h1>
        </div>
        <div class="body">
          <p>Thanks for signing up. Click below to verify your email:</p>
          <a href="https://myapp.com/verify" class="button">Verify Email</a>
        </div>
        <div class="footer">
          <p>If you didn't sign up, ignore this email.</p>
        </div>
      </div>
    </body>
    </html>
  `;
}

// Usage
await transporter.sendMail({
  from: '"MyApp" <noreply@myapp.com>',
  to: user.email,
  subject: "Welcome! Verify your email",
  html: getWelcomeEmailHTML(user.name),
});
```

### Sending Attachments

```javascript
await transporter.sendMail({
  from: '"Reports" <reports@myapp.com>',
  to: "admin@myapp.com",
  subject: "Monthly Report - January 2024",
  text: "Please find the monthly report attached.",
  attachments: [
    {
      filename: "report-jan-2024.pdf",
      path: "./reports/january.pdf", // File path
    },
    {
      filename: "data.csv",
      content: "name,email\nVikas,vikas@example.com\n", // String content
    },
    {
      filename: "logo.png",
      path: "./assets/logo.png",
      cid: "logo", // Use in HTML as <img src="cid:logo">
    },
  ],
});
```

---

## Environment-Based Configuration

### .env File

```env
# Server
NODE_ENV=development
PORT=3000
LOG_LEVEL=debug

# SMTP
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password

# Email
EMAIL_FROM="MyApp <noreply@myapp.com>"
```

### Config Module

```javascript
// config/index.js
require("dotenv").config();

module.exports = {
  env: process.env.NODE_ENV || "development",
  port: process.env.PORT || 3000,

  logging: {
    level:
      process.env.LOG_LEVEL ||
      (process.env.NODE_ENV === "production" ? "info" : "debug"),
    file: process.env.NODE_ENV === "production",
  },

  email: {
    host: process.env.SMTP_HOST,
    port: parseInt(process.env.SMTP_PORT) || 587,
    user: process.env.SMTP_USER,
    pass: process.env.SMTP_PASS,
    from: process.env.EMAIL_FROM || "noreply@myapp.com",
  },
};
```

### Environment-Aware Logger

```javascript
const winston = require("winston");
const config = require("../config");

const logger = winston.createLogger({
  level: config.logging.level,
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json(),
  ),
  transports: [],
});

// Always log errors to file
logger.add(
  new winston.transports.File({ filename: "logs/error.log", level: "error" }),
);

// File logging in production
if (config.logging.file) {
  logger.add(new winston.transports.File({ filename: "logs/combined.log" }));
}

// Console in development
if (config.env !== "production") {
  logger.add(
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple(),
      ),
    }),
  );
}

module.exports = logger;
```

### Environment-Aware Email Service

```javascript
const nodemailer = require("nodemailer");
const config = require("../config");
const logger = require("./logger");

let transporter;

if (config.env === "production") {
  // Real SMTP in production
  transporter = nodemailer.createTransport({
    host: config.email.host,
    port: config.email.port,
    secure: config.email.port === 465,
    auth: { user: config.email.user, pass: config.email.pass },
  });
} else {
  // Use Ethereal (fake SMTP) in development
  nodemailer.createTestAccount().then((account) => {
    transporter = nodemailer.createTransport({
      host: "smtp.ethereal.email",
      port: 587,
      auth: { user: account.user, pass: account.pass },
    });
  });
}

async function sendEmail({ to, subject, html, text, attachments }) {
  const mailOptions = {
    from: config.email.from,
    to,
    subject,
    html,
    text,
    attachments,
  };

  try {
    const info = await transporter.sendMail(mailOptions);
    logger.info("Email sent", { to, subject, messageId: info.messageId });

    // In dev, log the preview URL
    if (config.env !== "production") {
      logger.debug("Preview URL", { url: nodemailer.getTestMessageUrl(info) });
    }

    return info;
  } catch (error) {
    logger.error("Email send failed", { to, subject, error: error.message });
    throw error;
  }
}

module.exports = { sendEmail };
```

---

## Best Practices

1. **Use Morgan for HTTP logs, Winston for app logs** — They serve different purposes.
2. **Set log levels per environment** — `debug` in dev, `info` or `warn` in production.
3. **Log to files in production** — Console output is lost when the process restarts.
4. **Never log sensitive data** — No passwords, tokens, or credit card numbers in logs.
5. **Use structured logging (JSON)** — Easier to parse and search in log management tools.
6. **Use app passwords for Gmail** — Regular passwords won't work with 2FA enabled.
7. **Use Ethereal for development** — Catch emails without sending real ones.
8. **Handle email failures gracefully** — Don't crash the app if SMTP is down.
9. **Rotate log files** — Use `maxsize` and `maxFiles` to prevent disk from filling up.
10. **Keep email templates separate** — Don't embed HTML in business logic.

---

## Common Mistakes

| Mistake                                  | Problem                                 | Fix                                                |
| ---------------------------------------- | --------------------------------------- | -------------------------------------------------- |
| Logging passwords/tokens                 | Security breach if logs are exposed     | Redact sensitive fields before logging             |
| No log rotation                          | Disk fills up, server crashes           | Use `maxsize` and `maxFiles` in Winston            |
| Using `console.log` in production        | No levels, no persistence, no structure | Use Winston with proper transports                 |
| Real SMTP credentials in dev             | Sending test emails to real users       | Use Ethereal or Mailtrap in development            |
| Not handling transporter errors          | App crashes if SMTP unreachable         | Wrap sendMail in try/catch, log errors             |
| Morgan in production without file stream | Logs lost on restart                    | Stream to access.log file in production            |
| Hardcoding SMTP credentials              | Secrets in source code                  | Use environment variables                          |
| Synchronous file logging                 | Blocks event loop on heavy traffic      | Winston transports are async by default — use them |

---

## Summary

### Morgan

- HTTP request logger middleware for Express
- Use **`'dev'`** format in development (colored, concise)
- Use **`'combined'`** format in production (detailed, file-based)
- Stream to a file for persistent request logs

### Winston

- Full-featured application logger with levels, transports, and formatting
- **Levels**: error → warn → info → http → debug (set minimum to log)
- **Transports**: Console (dev), File (production), or external services
- Always log structured JSON for searchability

### Nodemailer

- Standard email library: create transporter → define mail options → sendMail
- Use **SMTP transport** with proper credentials (app passwords for Gmail)
- Support **HTML emails** with inline styles and **attachments**
- Use **Ethereal** in development to preview without sending

### Key Principle

- Configure everything via environment variables — different behavior for dev/staging/production
