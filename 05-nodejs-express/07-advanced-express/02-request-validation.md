# Request Validation

## Why Validate Server-Side?

Client-side validation (HTML forms, JavaScript) improves UX but can be bypassed entirely. Anyone can send requests directly via Postman, curl, or a script. Server-side validation is your **last line of defense** — like a bouncer at the door who checks IDs regardless of what the person says.

**Without validation:**

- Users can submit empty or malformed data
- SQL injection, XSS, and other attacks slip through
- Your database fills with garbage
- App crashes on unexpected input types

```javascript
// ❌ Without validation — what could go wrong?
app.post("/api/users", (req, res) => {
  db.createUser(req.body); // Trusting raw user input!
});

// ✅ With validation — safe and predictable
app.post("/api/users", validateUser, (req, res) => {
  db.createUser(req.body); // Already validated and sanitized
});
```

---

## express-validator

The most popular validation library for Express. Built on top of `validator.js`.

### Installation

```bash
npm install express-validator
```

### Basic Usage

```javascript
const { body, param, query, validationResult } = require("express-validator");
```

### Body Validators

```javascript
const express = require("express");
const { body, validationResult } = require("express-validator");
const app = express();

app.use(express.json());

// Validation rules as middleware array
const createUserValidation = [
  body("name")
    .trim()
    .notEmpty()
    .withMessage("Name is required")
    .isLength({ min: 2, max: 50 })
    .withMessage("Name must be 2-50 characters"),

  body("email")
    .trim()
    .notEmpty()
    .withMessage("Email is required")
    .isEmail()
    .withMessage("Invalid email format")
    .normalizeEmail(),

  body("age")
    .optional()
    .isInt({ min: 18, max: 120 })
    .withMessage("Age must be between 18 and 120"),

  body("password")
    .isLength({ min: 8 })
    .withMessage("Password must be at least 8 characters")
    .matches(/\d/)
    .withMessage("Password must contain a number")
    .matches(/[A-Z]/)
    .withMessage("Password must contain an uppercase letter"),
];

app.post("/api/users", createUserValidation, (req, res) => {
  const errors = validationResult(req);

  if (!errors.isEmpty()) {
    return res.status(422).json({ errors: errors.array() });
  }

  // Data is safe to use
  res.status(201).json({ data: req.body });
});
```

### Param Validators

```javascript
const { param } = require("express-validator");

const validateId = [
  param("id").isInt({ min: 1 }).withMessage("ID must be a positive integer"),
];

app.get("/api/users/:id", validateId, (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  // Proceed...
});
```

### Query Validators

```javascript
const { query } = require("express-validator");

const validatePagination = [
  query("page")
    .optional()
    .isInt({ min: 1 })
    .withMessage("Page must be a positive integer"),

  query("limit")
    .optional()
    .isInt({ min: 1, max: 100 })
    .withMessage("Limit must be between 1 and 100"),

  query("sort")
    .optional()
    .isIn(["name", "email", "createdAt"])
    .withMessage("Invalid sort field"),
];

app.get("/api/users", validatePagination, (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  // Proceed with safe query params...
});
```

### validationResult — Handling Errors

```javascript
const { validationResult } = require("express-validator");

app.post("/api/users", createUserValidation, (req, res) => {
  const errors = validationResult(req);

  if (!errors.isEmpty()) {
    // errors.array() returns:
    // [
    //   { type: 'field', msg: 'Email is required', path: 'email', location: 'body' },
    //   { type: 'field', msg: 'Password must be at least 8 chars', path: 'password', location: 'body' }
    // ]
    return res.status(422).json({
      error: "Validation failed",
      details: errors.array().map((err) => ({
        field: err.path,
        message: err.msg,
      })),
    });
  }

  res.status(201).json({ data: req.body });
});
```

---

## Joi Schema Validation

Joi provides a schema-based approach — you define the shape of valid data, and Joi checks it.

### Installation

```bash
npm install joi
```

### Defining Schemas

```javascript
const Joi = require("joi");

const userSchema = Joi.object({
  name: Joi.string().min(2).max(50).required(),
  email: Joi.string().email().required(),
  age: Joi.number().integer().min(18).max(120).optional(),
  password: Joi.string()
    .min(8)
    .pattern(/^(?=.*[A-Z])(?=.*\d)/)
    .required()
    .messages({
      "string.pattern.base":
        "Password must have at least one uppercase letter and one number",
    }),
  role: Joi.string().valid("user", "admin", "moderator").default("user"),
});

const updateUserSchema = Joi.object({
  name: Joi.string().min(2).max(50),
  email: Joi.string().email(),
  age: Joi.number().integer().min(18).max(120),
}).min(1); // At least one field required for update
```

### Joi Validation Middleware

```javascript
function validate(schema) {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, {
      abortEarly: false, // Report all errors, not just the first
      stripUnknown: true, // Remove fields not in schema
    });

    if (error) {
      const details = error.details.map((err) => ({
        field: err.path.join("."),
        message: err.message,
      }));

      return res.status(422).json({
        error: "Validation failed",
        details,
      });
    }

    req.body = value; // Replace with validated/sanitized data
    next();
  };
}

// Usage
app.post("/api/users", validate(userSchema), (req, res) => {
  res.status(201).json({ data: req.body });
});

app.put("/api/users/:id", validate(updateUserSchema), (req, res) => {
  res.json({ data: req.body });
});
```

### Joi for Params and Query

```javascript
const paramSchema = Joi.object({
  id: Joi.number().integer().positive().required(),
});

const querySchema = Joi.object({
  page: Joi.number().integer().min(1).default(1),
  limit: Joi.number().integer().min(1).max(100).default(10),
  search: Joi.string().max(100).optional(),
});

function validateParams(schema) {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.params);
    if (error) return res.status(400).json({ error: error.details[0].message });
    req.params = value;
    next();
  };
}

function validateQuery(schema) {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.query);
    if (error) return res.status(400).json({ error: error.details[0].message });
    req.query = value;
    next();
  };
}

app.get("/api/users/:id", validateParams(paramSchema), (req, res) => {
  // req.params.id is guaranteed to be a positive integer
});
```

---

## Custom Validation Middleware

When you need logic that libraries don't cover.

```javascript
// Check if email already exists (async validation)
async function checkDuplicateEmail(req, res, next) {
  const existingUser = await User.findOne({ email: req.body.email });

  if (existingUser) {
    return res.status(409).json({
      error: "Email already registered",
    });
  }

  next();
}

// Validate that referenced resource exists
async function validateProductExists(req, res, next) {
  const product = await Product.findById(req.body.productId);

  if (!product) {
    return res.status(400).json({
      error: "Referenced product does not exist",
    });
  }

  req.product = product; // Attach for later use
  next();
}

// Combine multiple validations
app.post(
  "/api/users",
  validate(userSchema), // Schema validation
  checkDuplicateEmail, // Business logic validation
  userController.createUser, // Handler
);
```

---

## Sanitization

Sanitization cleans data — removes dangerous characters, trims whitespace, normalizes formats.

### With express-validator

```javascript
const { body } = require("express-validator");

const sanitizeUser = [
  body("name").trim().escape(), // Remove HTML tags, trim spaces
  body("email").trim().normalizeEmail(), // Lowercase, remove dots in gmail
  body("bio").trim().stripLow(), // Remove control characters
  body("website")
    .trim()
    .customSanitizer((value) => {
      // Add https:// if missing
      if (value && !value.startsWith("http")) {
        return `https://${value}`;
      }
      return value;
    }),
];
```

### With Joi (Built-in)

```javascript
const schema = Joi.object({
  name: Joi.string().trim().required(), // Auto-trims
  email: Joi.string().email().lowercase(), // Converts to lowercase
  tags: Joi.array().items(Joi.string().trim()), // Trims each array item
  role: Joi.string().valid("user", "admin").default("user"), // Sets default
});

// stripUnknown removes fields not in the schema
schema.validate(data, { stripUnknown: true });
```

---

## Error Response Formatting

Keep error responses consistent across your entire API.

### Reusable Validation Middleware

```javascript
// middleware/validate.js
const { validationResult } = require("express-validator");

function handleValidationErrors(req, res, next) {
  const errors = validationResult(req);

  if (!errors.isEmpty()) {
    return res.status(422).json({
      success: false,
      error: {
        code: "VALIDATION_ERROR",
        message: "Request validation failed",
        details: errors.array().map((err) => ({
          field: err.path,
          value: err.value,
          message: err.msg,
          location: err.location,
        })),
      },
    });
  }

  next();
}

module.exports = handleValidationErrors;
```

### Usage Pattern

```javascript
const handleValidationErrors = require("../middleware/validate");

// Define rules separately, check results with reusable handler
app.post(
  "/api/users",
  createUserValidation, // Validation rules (array)
  handleValidationErrors, // Check & format errors
  userController.createUser, // Only runs if valid
);

app.put(
  "/api/users/:id",
  updateUserValidation,
  handleValidationErrors,
  userController.updateUser,
);
```

### Example Error Response

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "value": "not-an-email",
        "message": "Invalid email format",
        "location": "body"
      },
      {
        "field": "password",
        "value": "123",
        "message": "Password must be at least 8 characters",
        "location": "body"
      }
    ]
  }
}
```

---

## Best Practices

1. **Always validate server-side** — Client validation is a convenience, not security.
2. **Validate early** — Place validation middleware before controllers.
3. **Use schema-based validation** — Joi or express-validator, not manual if/else chains.
4. **Sanitize inputs** — Trim, escape, and normalize before processing.
5. **Return all errors at once** — Use `abortEarly: false` to show all issues, not just the first.
6. **Consistent error format** — Same structure for all validation errors across the API.
7. **Separate validation from logic** — Validation middleware should only validate, not handle business logic.
8. **Validate params and query too** — Not just the request body.
9. **Use `stripUnknown`** — Remove unexpected fields that could cause issues.

---

## Common Mistakes

| Mistake                             | Problem                                     | Fix                                        |
| ----------------------------------- | ------------------------------------------- | ------------------------------------------ |
| Only validating on frontend         | Users bypass forms easily                   | Always validate on the server              |
| Manual if/else validation           | Verbose, inconsistent, error-prone          | Use Joi or express-validator               |
| Not sanitizing input                | XSS attacks, data inconsistency             | Use `.trim()`, `.escape()`, `stripUnknown` |
| Returning only first error          | User fixes one error, submits, sees another | Use `abortEarly: false`                    |
| Validating after database call      | Wasted DB queries on invalid data           | Validate first, then query                 |
| No error response standard          | Frontend can't parse errors consistently    | Define one error format and stick to it    |
| Forgetting to validate `:id` params | Invalid IDs cause DB errors or crashes      | Use `param('id').isInt()`                  |
| Not handling `optional()` correctly | Fields are required by default in Joi       | Explicitly mark optional fields            |

---

## Summary

- **Server-side validation is mandatory** — never trust client input
- **express-validator**: Chainable validators for `body()`, `param()`, `query()` with `validationResult()`
- **Joi**: Schema-based validation — define the shape, Joi enforces it
- **Custom middleware**: For business logic validation (duplicates, references, permissions)
- **Sanitization**: Clean input with trim, escape, normalize, and stripUnknown
- **Error formatting**: Use a consistent structure so frontends can parse errors reliably
- Validate → Sanitize → Process — always in this order
