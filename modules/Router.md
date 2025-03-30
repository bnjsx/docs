# Bnjsx Router

The Router module provides a clean way to define and match routes in your bnjsx application. It supports all standard HTTP methods and flexible path matching patterns.

## Built-in Methods

Below is the complete list of route registration methods:

| **Method**   | **Description**                        | **Example**                       |
| ------------ | -------------------------------------- | --------------------------------- |
| `.get()`     | Registers a GET route                  | `router.get('/users', ...)`       |
| `.post()`    | Registers a POST route                 | `router.post('/users', ...)`      |
| `.put()`     | Registers a PUT route                  | `router.put('/users/:id', ...)`   |
| `.delete()`  | Registers a DELETE route               | `router.delete('/users/:id',...)` |
| `.patch()`   | Registers a PATCH route                | `router.patch('/users/:id',...)`  |
| `.options()` | Registers an OPTIONS route             | `router.options('/users',...)`    |
| `.head()`    | Registers a HEAD route                 | `router.head('/users',...)`       |
| `.all()`     | Registers a route for ALL HTTP methods | `router.all('*', ...)`            |

## Usage Examples

Creates a new router instance:

```javascript
const router = new Router();
```

### 1. Static Paths

Matches exact URL paths without any variables:

```javascript
router.get('/about', ...);
// Matches: /about
// Doesn't match: /about-us, /about/team
```

### 2. Named Parameters

Captures dynamic path segments as named variables:

```javascript
router.get('/users/:userId/posts/:postId', ...);
// Matches: /users/123/posts/456
// params: { userId: '123', postId: '456' }
```

### 3. Optional Parameters

Makes route segments optional with the ? suffix:

```javascript
router.get('/posts/:id?', ...);
// Matches both:
// - /posts/123 → params: { id: '123' }
// - /posts → params: { id: undefined }
```

### 4. Mixed Parameters

Combines required and optional parameters in one route:

```javascript
router.get('/items/:category/:id?/edit', ...);
// Matches:
// - /items/books/123/edit → { category: 'books', id: '123' }
// - /items/books/edit → { category: 'books', id: undefined }
// Doesn't match: /items/edit (missing required category)
```

### 5. Wildcards

Matches any remaining path segments after the specified prefix:

```javascript
router.get('/files/*', ...);
// Matches:
// - /files/image.jpg
// - /files/docs/report.pdf
// - /files/archives/2023/january/data.xls
```

### 6. Regular Expressions

Provides advanced pattern matching with regex capture groups:

```javascript
router.get(/^\/api\/v(\d+)\/users\/([a-z]+)$/, ...);
// Matches: /api/v1/users/admin
// params: ['1', 'admin']
// Doesn't match: /api/vx/users/admin (version must be numeric)
```

### 7. Basic CRUD Routes

The following example demonstrates how to define basic CRUD routes using the Router module.

```javascript
const router = new Router();

// Create
router.post('/users', ...);

// Read
router.get('/users', ...);
router.get('/users/:id', ...);

// Update
router.put('/users/:id', ...);

// Delete
router.delete('/users/:id', ...);
```

### 8. Advanced Matching

The example below defines a route that captures UUID patterns

```javascript
router.get(/^\/users\/([0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12})$/, ...);
```

You can use regex captures to extract route parameters as an array instead of an object.

```javascript
router.get(/^\/api\/(v\d+)\/users$/, ...);

const match = router.match('/api/v2/users', 'GET');
// match.params = ['v2']
```

### 9. Catch-All Route

Handle any HTTP method for a specific path or any path.

```javascript
// Matches any HTTP method for a specific path
router.all('/path', ...);

// Matches any HTTP method and any path
router.all('*', ...);
```

### 10. Route Matching

Matches a URL path against registered routes:

```javascript
// Register route
router.get('/users/:id', ...);

// Match routes
router.match('/users/123', 'GET');

// Returns:
// {
//   params: { id: '123' },
//   middlewares: [...] // Array of route middlewares
// }

// Match again
router.match('/users/123', 'POST');

// Returns:
// undefined (no matching route found)
```

## Order Matters

Place specific routes before generic ones

```javascript
// Good - Specific first
router.get('/users/active', ...);
router.get('/users/:id', ...);

// Bad - Generic first would shadow specific routes
router.get('/users/:id', ...);
router.get('/users/active', ...); // Never matches
```
