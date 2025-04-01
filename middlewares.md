### Built-in Middlewares

Bnjsx provides a set of built-in middlewares to handle common tasks efficiently. Each middleware serves a specific purpose, such as parsing JSON, handling cookies, managing static assets, and more. Below is a breakdown of the available middlewares and how they work. 🚀

## Cookie Middleware

The **cookie** middleware automatically parses and stores cookies from incoming requests, making them accessible within your application.

### Importing the Middleware

To use the cookie middleware, import it from `bnjsx`:

```js
const { App, Router, cookie } = require('bnjsx');
```

### Registering the Middleware

First, create an instance:

```js
const app = new App();
const router = new Router();
```

Then, register the middleware:

```js
app.use(cookie);
```

### Accessing Cookies in Requests

Once registered, the middleware will parse cookies and make them available in `req.cookie` for every request.

```js
router.all('*', async (req, res) => {
  if (config().loadSync().env === 'dev') {
    console.log(`Cookies: ${req.cookie}`);
  }
});
```

> In development mode, this example will log incoming cookies to the console.

## JSON Middleware

The **JSON middleware** automatically parses JSON request bodies, making them accessible via `req.body`. This is essential for building APIs.

### Example Usage

First, import the required modules:

```js
const { App, Router, json } = require('bnjsx');
const app = new App();
const router = new Router();
```

#### Registering the Middleware Globally

To enable JSON parsing for all requests, register the middleware globally:

```js
app.use(json);
```

#### Logging JSON Body

You can log incoming JSON request bodies for debugging:

```js
router.all('*', async (req, res) => {
  if (config().loadSync().env === 'dev' && req.type('json')) {
    console.log(`JSON body: ${req.body}`);
  }
});
```

> In development mode, this logs the parsed JSON body if the request's content type is `application/json`.

#### Using Middleware Locally

You can also apply the middleware to specific routes:

```js
router.post('/json', json, async (req, res) => {
  console.log(req.body);
});
```

> In this example, the JSON middleware only parses the request body for `POST /json` requests.

## Text Middleware

The **Text middleware** parses plain text or HTML content from the request body, making it accessible via `req.body`.

### Example Usage

First, import the required modules:

```js
const { App, Router, text } = require('bnjsx');

const app = new App();
const router = new Router();
```

### Registering the Middleware Globally

To enable text parsing for all requests, register the middleware globally:

```js
app.use(text);
```

### Handling Text and HTML Requests

You can log incoming plain text or HTML request bodies for debugging:

```js
router.all('*', async (req, res) => {
  if (config().loadSync().env === 'dev') {
    if (req.type('html')) {
      return console.log(`HTML Body: ${req.body}`);
    }

    if (req.type('txt')) {
      return console.log(`Text Body: ${req.body}`);
    }
  }
});
```

> In development mode, this logs the parsed body when the request content type is `text/html` or `text/plain`.

## Asset Middleware

The **Asset middleware** serves static files with built-in support for caching, compression, and range requests.

By default, assets are served from the `public` directory in your project root. However, you can customize this path in `bnjsx.config.js`.

### Setting an Absolute Path

To change the default public directory, set the `root` option to an absolute path:

```js
public: {
  root: resolve('../path', './to', 'public'),
}
```

### Setting a Relative Path

You can also provide a relative path. In this case, it will be relative to your project root:

```js
public: {
  root: 'public/static',
}
```

> This configuration serves assets from a `static` folder inside a `public` directory located at the project root (where `bnjsx.config.js` exists).

### Using the Asset Middleware

To enable asset serving, register the middleware:

```js
const { App, Router, asset } = require('bnjsx');

const app = new App();
const router = new Router();

// Register the middleware globally
app.use(asset);
```

That's it! Your static assets will now be served automatically.

### Serving Static Assets

Assuming your project structure includes a `public` folder like this:

```
public
│── image.png
│── svg
│   ├── icon.svg
│── css
│   ├── style.css
│── js
│   ├── app.js
```

You can request assets using a relative path:

| Request Path     | Serves File            |
| ---------------- | ---------------------- |
| `/image.png`     | `public/image.png`     |
| `/svg/icon.svg`  | `public/svg/icon.svg`  |
| `/css/style.css` | `public/css/style.css` |

> **Note:** Do **not** include the `public` folder in the request path. For example, `/public/image.png` will incorrectly resolve to `/public/public/image.png`.

### Security Measures

Bnjsx prevents **directory traversal attacks** to ensure secure asset delivery. If a request attempts to access files outside the configured public directory, it will be **rejected**.

#### Example of a Forbidden Request:

```plaintext
GET ../src  ❌ (Blocked)
```

- Requests pointing to **folders** are automatically ignored, resulting in a `404 Not Found`.
- If a request attempts to access a file outside the `public` directory, a `ForbiddenError` is thrown:

```js
new ForbiddenError(`The requested resource is forbidden at: '${path}'`);
```

### Route Precedence Warning

If you register the asset middleware globally, it has **higher precedence** than local routes. Be mindful of conflicts.

```js
router.get('/icon.svg', async (req, res) => {
  return res.sendFile(path);
});
```

> If `icon.svg` exists in the `public` folder, the asset middleware **will serve the file first**, and the above route **will never be executed**. Plan your routes accordingly.

### Asset Caching

By default, the asset middleware instructs the browser to cache assets for **3600 seconds** (1 hour). This reduces the number of requests, improves performance, and enhances the user experience.

You can modify this caching duration in your `bnjsx.config.js` file:

```js
public: {
  root: 'public',
  cache: 3600 * 24 // 24 hour
}
```

#### Adjusting Cache Duration

- If your static files **never change**, you can increase the cache duration for better performance.
- If your files **change frequently**, set `cache: 0` to disable caching.

#### Optimizing Caching Smartly

A good practice is to **serve static assets globally** using the asset middleware while serving frequently updated files manually using `req.sendFile()`.

Bnjsx gives you **flexibility!** just use the right approach for your use case.

### Asset Compression

By default, **gzip compression** support is **disabled**, but you can enable it in your `bnjsx.config.js` file:

```js
public: {
  root: 'public',
  gzip: true // Enable gzip support
}
```

#### How It Works

When gzip support is enabled, the asset handler will **first look for a compressed (`.gz`) version** of the requested file. If a `.gz` version exists, it will be served; otherwise, the regular file will be returned.

#### Example Directory Structure

```
public
│── image.png
│── svg
│   ├── icon.svg
│   ├── icon.svg.gz
```

#### Example Requests

- `GET /image.png` → Serves `image.png`
- `GET /svg/icon.svg` → Serves `svg/icon.svg.gz` (if available), otherwise `svg/icon.svg`

Enabling gzip can **significantly reduce file sizes**, improving loading speeds and overall performance.
