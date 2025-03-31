# Bnjsx Form Module

The **Bnjsx Form Module** allows you to handle form data efficiently, supporting both field validation and file uploads. It works with the following encodings:

- `multipart/form-data` (for file uploads)
- `application/x-www-form-urlencoded` (for standard form fields)

## Getting Started

First, import the `Form` module and create a new instance:

```javascript
const form = new Form(mode);
```

You can optionally specify the **form mode** to determine how unexpected files and fields should be handled:

- **`strict`**: Rejects unexpected files and fields.
- **`strong`** _(default)_: Rejects unexpected files but allows unexpected fields.
- **`ignore`**: Ignores unexpected files and fields.

By default, the `strong` mode is used. This mode allows unexpected fields but rejects unexpected files to prevent processing potentially harmful or excessively large files (e.g., a 100GB file that could crash your server).

### Handling Errors

If an unexpected file or field is detected, **Bnjsx** will display a `Bad Request` error page. This error page is located at:

```
views/errors/400.fx
```

You can customize this page if needed. It should be available in your project if you are using `bnjet`.

However, if you register your own error handlers, this file is not required since the default error handler won't be executed.

> **⚠️ Security Note:**
> In production, an unexpected file means someone has injected an unauthorized `<input>` field into your form. This is why it's **highly recommended** to set the mode to `strong` or `strict` to reject such requests.

While Bnjsx already enforces strong security, an additional layer of validation is always advisable.

---

## Handling Form Fields

In `strong` mode, all form fields will be parsed and included in the `req.body`. To parse the form, use the `form.parse(req, res)` middleware.

### Example Usage

```javascript
const { parse } = new Form();

router.post('/register', parse, async (req, res) => {
  // Log all fields
  console.log(req.body);
});
```

A typical request body may look like this:

```js
{ email: "foo@gmail.com", password: "foo_1234", terms: "true" }
```

Or, if the user submits an empty form:

```js
{ email: undefined, password: undefined, terms: undefined }
```

### Validating Fields

Before processing, always validate user input. For example:

```javascript
router.post('/register', parse, async (req, res) => {
  if (!req.body || !req.body.email) {
    return res.json({ error: 'ValidationError', message: 'Email is required' });
  }

  // Further processing...
});
```

### Using the `field` Method (Recommended)

A better approach is to use the `field` method to register expected field names and validation rules:

```javascript
const { parse, field } = new Form();

// Register validation rules
field('email', {
  test: (email) => email && /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email),
  message: 'Missing or invalid email format',
});

router.post('/register', parse, async (req, res) => {
  if (req.errors) {
    console.log(req.errors);
    return res.json({ error: 'ValidationError', message: req.errors });
  }

  // If validation passes
  return res.json({ message: 'Success' });
});
```

### How Validation Works

When a form is submitted, the `Form` instance validates each field based on the registered rules:

- If the test function **returns `true`**, validation **passes**.
- If it **returns `false`**, validation **fails**, and an error message is stored in the `req.errors` object.

Example error object:

```json
{
  "email": ["Missing or invalid email format"],
  "password": ["Password must contain at least 8 characters"]
}
```

By structuring your validation this way, you ensure a robust and secure form-handling process.

## Form Files

The **Form Module** provides a way to validate files **while they are being uploaded**. This means we don't need to upload the entire file before detecting issues. If an issue is encountered, a response is returned immediately.

### Example Usage

```javascript
const { parse, file, MB } = new Form();

file('profile_pic', {
  count: 1,
  type: ['image/jpg', 'image/png'],
  size: {
    min: MB,
    max: MB * 10,
  },
  required: true,
});

router.post('/profile/edit', parse, async (req, res) => {
  if (req.errors) {
    console.log(req.errors);
    return res.json({ error: 'ValidationError', message: req.errors });
  }

  // Log file info
  console.log(req.body.profile_pic);

  // If validation passes
  return res.json({ message: 'Profile updated' });
});
```

### Explanation

In the above example, we expect the user to submit a `profile_pic` image with the following conditions:

- The file must be exactly **one file** (`count: 1`).
- The file **is required** (`required: true`).
- The file size must be between **1MB and 10MB**.
- The file type must be **JPEG or PNG** (`type: ['image/jpg', 'image/png']`).

If all validation checks pass, the file info will be available under `req.body.profile_pic`. Example output:

```js
{
  // File size in bytes after validation.
  size: 1542454,

  // MIME type of the validated file.
  type: "image/png",

  // Absolute path where the validated file is stored.
  path: "/app/uploads/Xaysnnjs78a.png",

  // Original name of the uploaded file.
  name: "nice_pic.png"
}
```

### Handling Validation Errors

If validation fails, errors will be stored in `req.errors`. The **Form Module** provides default error messages:

```json
{
  "count": "Too many files. Maximum allowed is 1.",
  "required": "This field is required.",
  "type": "File must be image/jpeg, image/png.",
  "size": {
    "min": "File must be at least 1MB.",
    "max": "File must not exceed 10MB."
  }
}
```

### Customizing Error Messages

You can provide your own error messages:

```javascript
file('profile_pic', {
  count: 1,
  type: ['image/jpg', 'image/png'],
  size: {
    min: MB,
    max: MB * 10,
  },
  required: true,
  messages: {
    count: 'Custom message for count error',
    required: 'Custom message for required field',
    type: 'Custom message for file type error',
    size: {
      min: 'Custom message for minimum size',
      max: 'Custom message for maximum size',
    },
  },
});
```

### Storing Uploaded Files

By default, all uploaded files are stored in the `uploads` directory. However, you can specify a custom upload path for more flexibility:

```javascript
file('profile_pic', {
  count: 1,
  type: ['image/jpg', 'image/png'],
  size: {
    min: MB,
    max: MB * 10,
  },
  required: true,
  location: resolve('./public'), // Store this file in the public folder
  messages: {
    count: 'Custom message',
    required: 'Custom message',
    type: 'Custom message',
    size: {
      min: 'Custom message',
      max: 'Custom message',
    },
  },
});
```

### Public vs. Private File Storage

- **Public files** (e.g., profile pictures) can be stored in the `public` folder for direct access.
- **Private files** should be stored in the `uploads` folder and handled securely by your application.
