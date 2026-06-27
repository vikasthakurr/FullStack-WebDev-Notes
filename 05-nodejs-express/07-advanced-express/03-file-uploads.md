# File Uploads with Multer

## What is Multer?

Multer is a Node.js middleware for handling `multipart/form-data`, the encoding type used for file uploads. Think of Multer as a mailroom clerk — it receives packages (files), inspects them, decides where to store them, and passes them along to the right department (your route handler).

Express cannot parse file uploads on its own. `express.json()` and `express.urlencoded()` only handle text data. Multer fills this gap.

```bash
npm install multer
```

```javascript
const multer = require("multer");
```

---

## Configuring Storage

### Disk Storage (Save to filesystem)

```javascript
const multer = require("multer");
const path = require("path");

const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, "uploads/"); // Directory must exist
  },
  filename: (req, file, cb) => {
    // Create unique filename: timestamp-originalname
    const uniqueName = `${Date.now()}-${file.originalname}`;
    cb(null, uniqueName);
  },
});

const upload = multer({ storage });
```

### Memory Storage (Keep in buffer)

Useful when you want to process the file (resize, upload to cloud) without saving locally.

```javascript
const storage = multer.memoryStorage();
const upload = multer({ storage });

// file.buffer contains the file data
app.post("/upload", upload.single("avatar"), (req, res) => {
  console.log(req.file.buffer); // <Buffer ...>
  // Upload to S3, Cloudinary, etc.
});
```

---

## Single File Upload — upload.single()

Accepts one file from a field named in the argument.

```javascript
const upload = multer({ storage });

app.post("/api/users/avatar", upload.single("avatar"), (req, res) => {
  // req.file contains the uploaded file info
  console.log(req.file);
  // {
  //   fieldname: 'avatar',
  //   originalname: 'photo.jpg',
  //   encoding: '7bit',
  //   mimetype: 'image/jpeg',
  //   destination: 'uploads/',
  //   filename: '1705312000000-photo.jpg',
  //   path: 'uploads/1705312000000-photo.jpg',
  //   size: 245678
  // }

  res.json({
    message: "File uploaded",
    file: req.file.filename,
  });
});
```

### HTML Form for Single Upload

```html
<form action="/api/users/avatar" method="POST" enctype="multipart/form-data">
  <input type="file" name="avatar" />
  <button type="submit">Upload</button>
</form>
```

---

## Multiple Files — upload.array()

Accepts multiple files from the same field.

```javascript
// Accept up to 5 photos from the "photos" field
app.post("/api/gallery", upload.array("photos", 5), (req, res) => {
  // req.files is an array of file objects
  console.log(req.files.length); // Number of uploaded files

  const filenames = req.files.map((f) => f.filename);
  res.json({ message: "Files uploaded", files: filenames });
});
```

### Multiple Fields — upload.fields()

```javascript
const cpUpload = upload.fields([
  { name: "avatar", maxCount: 1 },
  { name: "documents", maxCount: 3 },
]);

app.post("/api/profile", cpUpload, (req, res) => {
  // req.files is an object keyed by field name
  console.log(req.files["avatar"]); // Array with 1 file
  console.log(req.files["documents"]); // Array with up to 3 files

  res.json({ message: "All files uploaded" });
});
```

---

## File Filtering (Mimetype and Size Limits)

### Filter by File Type

```javascript
const imageFilter = (req, file, cb) => {
  const allowedTypes = ["image/jpeg", "image/png", "image/gif", "image/webp"];

  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true); // Accept file
  } else {
    cb(new Error("Only JPEG, PNG, GIF, and WebP images are allowed"), false);
  }
};

const upload = multer({
  storage,
  fileFilter: imageFilter,
  limits: {
    fileSize: 5 * 1024 * 1024, // 5 MB max
  },
});
```

### Comprehensive Configuration

```javascript
const upload = multer({
  storage,
  fileFilter: (req, file, cb) => {
    const allowed = /jpeg|jpg|png|gif|pdf/;
    const extname = allowed.test(path.extname(file.originalname).toLowerCase());
    const mimetype = allowed.test(file.mimetype);

    if (extname && mimetype) {
      cb(null, true);
    } else {
      cb(new Error("Invalid file type. Allowed: jpg, png, gif, pdf"));
    }
  },
  limits: {
    fileSize: 10 * 1024 * 1024, // 10 MB
    files: 5, // Max 5 files per request
    fields: 10, // Max 10 non-file fields
  },
});
```

---

## Handling Upload Errors

Multer errors need special handling since they occur during middleware execution.

```javascript
app.post("/api/upload", (req, res) => {
  const uploadMiddleware = upload.single("file");

  uploadMiddleware(req, res, (err) => {
    if (err instanceof multer.MulterError) {
      // Multer-specific errors
      switch (err.code) {
        case "LIMIT_FILE_SIZE":
          return res
            .status(400)
            .json({ error: "File too large. Max 5MB allowed." });
        case "LIMIT_FILE_COUNT":
          return res
            .status(400)
            .json({ error: "Too many files. Max 5 allowed." });
        case "LIMIT_UNEXPECTED_FILE":
          return res.status(400).json({ error: "Unexpected field name." });
        default:
          return res.status(400).json({ error: err.message });
      }
    } else if (err) {
      // Custom filter errors or other errors
      return res.status(400).json({ error: err.message });
    }

    // No error — file uploaded successfully
    if (!req.file) {
      return res.status(400).json({ error: "No file provided" });
    }

    res.json({ message: "Upload successful", file: req.file.filename });
  });
});
```

### Simpler Error Handling with Error Middleware

```javascript
app.post("/api/upload", upload.single("file"), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: "No file provided" });
  }
  res.json({ file: req.file.filename });
});

// Catch multer errors globally
app.use((err, req, res, next) => {
  if (err instanceof multer.MulterError) {
    return res.status(400).json({ error: `Upload error: ${err.message}` });
  }
  if (err.message.includes("Invalid file type")) {
    return res.status(400).json({ error: err.message });
  }
  next(err);
});
```

---

## Serving Uploaded Files with express.static

```javascript
const path = require("path");

// Serve files from "uploads" directory
app.use("/uploads", express.static(path.join(__dirname, "uploads")));

// Now accessible at: http://localhost:3000/uploads/1705312000000-photo.jpg
```

### Complete Upload + Serve Example

```javascript
const express = require("express");
const multer = require("multer");
const path = require("path");
const fs = require("fs");

const app = express();

// Ensure upload directory exists
const uploadDir = path.join(__dirname, "uploads");
if (!fs.existsSync(uploadDir)) {
  fs.mkdirSync(uploadDir, { recursive: true });
}

// Multer config
const storage = multer.diskStorage({
  destination: (req, file, cb) => cb(null, "uploads/"),
  filename: (req, file, cb) => {
    const ext = path.extname(file.originalname);
    cb(null, `${file.fieldname}-${Date.now()}${ext}`);
  },
});

const upload = multer({
  storage,
  limits: { fileSize: 5 * 1024 * 1024 },
  fileFilter: (req, file, cb) => {
    const allowed = ["image/jpeg", "image/png", "image/webp"];
    if (allowed.includes(file.mimetype)) cb(null, true);
    else cb(new Error("Only JPEG, PNG, and WebP allowed"));
  },
});

// Serve uploaded files
app.use("/uploads", express.static(uploadDir));

// Upload endpoint
app.post("/api/upload", upload.single("image"), (req, res) => {
  if (!req.file) return res.status(400).json({ error: "No file" });

  const fileUrl = `${req.protocol}://${req.get("host")}/uploads/${req.file.filename}`;
  res.status(201).json({ url: fileUrl });
});

// Delete endpoint
app.delete("/api/upload/:filename", (req, res) => {
  const filePath = path.join(uploadDir, req.params.filename);

  if (!fs.existsSync(filePath)) {
    return res.status(404).json({ error: "File not found" });
  }

  fs.unlinkSync(filePath);
  res.status(204).send();
});

app.listen(3000);
```

---

## Best Practices

1. **Always set file size limits** — Prevent users from uploading 2GB files.
2. **Validate file types** — Check both mimetype and extension (attackers rename files).
3. **Generate unique filenames** — Prevent overwrites and path traversal attacks.
4. **Create upload directory on startup** — Don't rely on it existing.
5. **Use memory storage for cloud uploads** — No need to save locally if sending to S3/Cloudinary.
6. **Don't serve uploads from app root** — Use a dedicated `/uploads` path.
7. **Add cleanup logic** — Delete orphaned files when associated records are deleted.
8. **Never trust `originalname`** — It can contain path separators or special characters.
9. **Use environment variables** for max file size and allowed types — different envs need different limits.

---

## Common Mistakes

| Mistake                                    | Problem                                          | Fix                                                    |
| ------------------------------------------ | ------------------------------------------------ | ------------------------------------------------------ |
| Upload directory doesn't exist             | Multer throws `ENOENT` error                     | Create directory on app startup with `mkdirSync`       |
| Field name mismatch                        | `upload.single('photo')` but form sends `avatar` | Match field name in multer with HTML/client field name |
| No file size limit                         | Server runs out of disk/memory                   | Always set `limits.fileSize`                           |
| Only checking extension                    | Renamed `.exe` to `.jpg` passes                  | Check both mimetype AND extension                      |
| Using `originalname` as filename           | Path traversal, overwrites, encoding issues      | Generate unique names with timestamp or UUID           |
| Forgetting `enctype="multipart/form-data"` | File arrives as `undefined`                      | Must set enctype on HTML form                          |
| Not handling "no file" case                | `req.file` is `undefined`, code crashes          | Check `if (!req.file)` before accessing properties     |
| Serving uploads without access control     | Anyone can access any uploaded file              | Add auth middleware before static serving if needed    |

---

## Summary

- **Multer** handles `multipart/form-data` file uploads that Express can't parse natively
- **Storage options**: `diskStorage` (save to disk) or `memoryStorage` (keep in buffer)
- **upload.single('field')** for one file, **upload.array('field', max)** for multiple
- **File filtering**: Validate mimetype and extension, reject everything else
- **Size limits**: Always set `limits.fileSize` to prevent abuse
- **Error handling**: Catch `multer.MulterError` for size/count/field errors
- **Serve files**: Use `express.static()` to make uploaded files accessible via URL
- Always generate unique filenames and validate file types on the server
