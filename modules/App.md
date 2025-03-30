# Bnjsx App Module

The **App Module** in Bnjsx is the main entry point for your application. It is responsible for:

- Starting and stopping the server.
- Registering **middlewares**.
- Registering **routers**.

The App module loads the configuration from your `bnjsx.config.js` file and creates the server for you.

## Getting Started

### (1) Create Your App (Server)

```javascript
const app = new App(); // Creates your web server
```

### (2) Define a Router (Group of Routes)

```javascript
const userRouter = new Router(); // Handles all user-related routes
```

### (3) Add Routes to the Router

```javascript
// GET /profile → Shows user profile
userRouter.get('/profile', async (req, res) => {
  return res.json({ email: 'foo@gmail.com' });
});

// POST /login → Handles login
userRouter.post('/login', async (req, res) => {
  return res.json({ message: 'Welcome back foo!' });
});
```

### (4) Register Router Under a Namespace

```javascript
// Option A: Mount at root (`/`)
app.namespace('/', userRouter);
// → Routes: `/profile`, `/login`

// Option B: Mount under `/api`
app.namespace('/api', userRouter);
// → Routes: `/api/profile`, `/api/login`

// Option C: Mount under `/auth`
app.namespace('/auth', userRouter);
// → Routes: `/auth/profile`, `/auth/login`
```

### (5) Start the Server

```javascript
app.start(); // Server runs on http://localhost:2025
```

## How Namespaces Work (URL Structure)

| **Namespace** | **Route**    | **Final URL**      | **When to Use**     |
| ------------- | ------------ | ------------------ | ------------------- |
| `/`           | `/profile`   | `/profile`         | Main website routes |
| `/api`        | `/users`     | `/api/users`       | API endpoints       |
| `/admin`      | `/dashboard` | `/admin/dashboard` | Admin panel routes  |
| `/v1`         | `/posts`     | `/v1/posts`        | API versioning      |

## Real-World Web Use Cases

### Example 1: Blog Website

```javascript
const postRouter = new Router();
postRouter.get('/', listPosts); // GET /posts
postRouter.get('/:id', getPost); // GET /posts/123

app.namespace('/posts', postRouter);
```

**Result:**

- `/posts` → Lists all posts
- `/posts/123` → Shows post with ID 123

### Example 2: Admin Dashboard

```javascript
const adminRouter = new Router();
adminRouter.get('/', getStats); // GET /admin
adminRouter.post('/ban-user', banUser); // POST /admin/ban-user

app.namespace('/admin', adminRouter);
```

**Result:**

- `/admin` → View admin home page
- `/admin/ban-user` → Ban a user

## Key Takeaways

✔ **Clean organization** (No messy routes)  
✔ **Easy restructuring** (Change `/api` to `/v2/api` in one place)  
✔ **Better readability** (Routes grouped logically)
✔ **`App`** = Your web server.  
✔ **`Router`** = Groups related routes (like "User Routes").  
✔ **`namespace`** = URL prefix (`/api`, `/admin`, `/v2`).  
✔ **Same router, different namespaces** = Reuse routes under different paths.

## Middlewares

### What Are Middlewares?

Middlewares are functions that run during the request-response cycle. They can modify the request or response, or even terminate the request early.

### Global vs. Local Middlewares

- **Global Middlewares**: These are executed for every request. They are registered using the `app.use()` method.
- **Local Middlewares**: These are executed only when a specific route matches. You define them within the router.

#### Registering Global Middleware

```javascript
app.use(async () => {
  console.log('This runs for every request!');
});
```

#### Registering Local Middleware

```javascript
router.get('/', async () => {
  console.log('This runs when a GET request is received at /');
});
```

## How Bnjsx Middleware Works

Unlike traditional frameworks (like Express) that rely on `next()` callbacks, **Bnjsx uses Promises** for middleware flow control.  
This makes error handling **automatic** and eliminates callback hell.

### 1. No `next()` Function

✔ **Middleware execution is Promise-based.**  
✔ **No manual error catching required.**  
✔ **Next middleware runs automatically on `resolve()`.**

### 2. Three Possible Outcomes

| **Outcome**                             | **Behavior**           |
| --------------------------------------- | ---------------------- |
| **Promise resolves** (no response sent) | Next middleware runs   |
| **Promise resolves** (response sent)    | Middleware chain stops |
| **Promise rejects** (throws error)      | Error handler executes |

## Comparison: Bnjsx vs Express

### Express (Manual Control)

```javascript
app.get(async (req, res, next) => {
  try {
    const user = await fetchUser(); // Async operation
    if (!user) return next(new Error('User not found')); // Manual error
    req.user = user; // Save user
    next(); // Must call next() to continue
  } catch (err) {
    next(err); // Must catch and forward errors
  }
});
```

### **Bnjsx (Automatic Flow)**

```javascript
router.get(async (req, res) => {
  const user = await fetchUser(); // No try/catch needed
  if (!user) throw new Error('User not found'); // Auto error handling
  res.user = user; // No next() required
});
```

✔ **No `next()`** → Cleaner code  
✔ **Errors auto-caught** → No `try/catch`  
✔ **Stops if response sent** → No extra calls

## Key Rules for Bnjsx Middlewares

### 1. Always Use `async`

Even if your code is synchronous:

```javascript
// Correct (async)
app.use(async (req, res) => {
  console.log('This runs');
});

// Wrong (non-async)
app.use((req, res) => {
  console.log('This will break');
});
```

### 2. Throwing Errors = Automatic Handling

```javascript
app.get(async (req, res) => {
  if (!req.body) throw new BadRequestError(); // Auto-handled
});
```

### 3. Response Sent = Stops Execution

```javascript
app.use(async (req, res) => {
  if (req.isBot) {
    return res.status(403).send('Bots not allowed');
  }
  // Continue if not a bot
});
```

## Why This Design?

### 1. Cleaner Code

No more:

- `next()` spaghetti
- Manual `try/catch` blocks
- Forgetting to forward errors

### 2. Better Error Handling

- Uncaught errors **automatically** go to error handler middlewares.
- No risk of silent failures.

### 3. Predictable Flow

- If a response is sent → **Middleware chain stops**.
- If an error is thrown → **Error handler takes over**.

## Real-World Example

```javascript
// Clean, automatic error handling
app.use(async (req, res) => {
  const token = req.headers.authorization;
  if (!token) throw new UnauthorizedError('Missing token'); // Auto-handled

  const user = await verifyToken(token); // No try/catch needed
  req.user = user; // Attach user to request
});

// Error handler (optional)
app.use(async (req, res, error) => {
  if (error instanceof UnauthorizedError) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
});
```

## Summary

| **Feature**         | **Bnjsx**                 | **Traditional (Express)** |
| ------------------- | ------------------------- | ------------------------- |
| **Flow Control**    | Promises                  | `next()` callback         |
| **Error Handling**  | Automatic                 | Manual `try/catch`        |
| **Middleware Stop** | Auto-detects `res.send()` | Needs `return next()`     |
| **Code Style**      | Clean, modern             | Callback-heavy            |

**Bnjsx middleware system is designed for modern, clean, and maintainable code.**

## Error Handlers

You can register **error handlers** using `app.use()`. Error handlers are just like middlewares but specifically handle errors.

```javascript
app.use(async (req, res, err) => {
  console.log('Error handler executed...');

  // Handle errors and send a response
  if (err instanceof BadRequestError) {
    return res.status(400).json({
      error: 'BadRequestError',
      message: 'Bad Request.',
    });
  }
});
```

## Starting and Stopping the Server

Once you’ve registered your routes and middlewares, it’s time to **start** the server:

```javascript
app.start();
```

You can also **stop** the server at any time:

```javascript
app.stop();
```

## Singleton Pattern

Bnjsx follows the **Singleton Pattern**. This means there can only be one instance of the `App` class.

```javascript
console.log(new App() === new App()); // true
```

This ensures that you have a single, consistent instance throughout your application.

## Multiple Applications

If you want to run multiple applications (e.g., on different ports or hosts), you can easily achieve this by creating **multiple Bnjsx projects** using the `bnjet` scaffolding tool.

## Conclusion

With the Bnjsx App module, you can:

- Define and register routers to handle requests.
- Use middlewares to handle requests globally or locally.
- Organize routers using namespaces.
- Handle errors automatically with async middlewares.
- Easily start and stop your server.

This simple and flexible approach gives you full control over your application while keeping things clean and efficient.

### Key Points for Beginners

- **Namespaces**: They allow you to group and organize your routers.
- **Middlewares**: Functions that handle requests and responses. Bnjsx uses `async` middlewares, making the flow simpler and cleaner than traditional frameworks that require `next()`.
- **Error Handlers**: You don't need to manually catch errors. Bnjsx automatically handles errors and allows you to define custom error responses.
- **Single App Instance**: Bnjsx uses the Singleton Pattern, ensuring there is only one `App` instance for the entire application.
