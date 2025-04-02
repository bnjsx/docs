## CORS Middleware

The **CORS (Cross-Origin Resource Sharing) middleware** allows you to configure how your server handles requests from different origins. It ensures that only authorized domains can access your API while enabling secure cross-origin requests.

### How to Use CORS Middleware

To apply CORS settings to all requests, register the middleware globally:

```js
const { App, Router, cors } = require('bnjsx');

const app = new App();
const router = new Router();

// Enable CORS globally
app.use(cors);
```

### CORS Configuration Options

You can customize the CORS settings in `bnjsx.config.js`:

```js
cors: {
  origin: ["https://example.com", "https://api.example.com"], // Allowed origins
  credentials: true, // Allow credentials (cookies, authorization headers)
  methods: ["GET", "POST", "PUT", "DELETE"], // Allowed HTTP methods
  headers: ["Content-Type", "Authorization"], // Allowed request headers
  expose: ["X-Custom-Header"], // Expose custom headers to the client
  maxAge: 600 // Cache preflight response for 600 seconds
}
```

### CORS Options Overview

| Option        | Default                                                        | Description                             |
| ------------- | -------------------------------------------------------------- | --------------------------------------- |
| `origin`      | `"*"`                                                          | Defines allowed origins.                |
| `credentials` | `false`                                                        | Enables credentials (cookies/auth).     |
| `methods`     | `['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS', 'HEAD']` | Allowed HTTP methods.                   |
| `headers`     | `undefined`                                                    | Allowed request headers.                |
| `expose`      | `undefined`                                                    | Response headers visible to the client. |
| `maxAge`      | `86400`                                                        | Cache preflight response (in seconds).  |

### 1. `origin` – Controlling Access

Defines which origins are allowed to access your API.

#### Allow All Origins

```js
cors: {
  origin: '*',
}
```

✔ Any origin can access your API.  
❌ Not safe for sensitive data.

#### Allow a Single Origin

```js
cors: {
  origin: 'https://example.com',
}
```

✔ Only `https://example.com` can make requests.  
✔ Good for private APIs.

#### Allow Multiple Origins

```js
cors: {
  origin: ['https://site1.com', 'https://site2.com'],
}
```

✔ Only `site1.com` and `site2.com` are allowed.  
✔ More secure than `"*"`.

#### Synchronous Function

```js
cors: {
  origin: (origin) => origin === 'https://trusted.com',
}
```

✔ Only allows `https://trusted.com`.  
✔ Rejects all other origins.

#### Asynchronous Function

```js
cors: {
  origin: async (origin) => await checkInDatabase(origin),
}
```

✔ Fetches allowed origins from a database.  
✔ Great for dynamic access control.

---

### 2. `credentials` – Sending Cookies & Auth

Allows sending cookies, authorization headers...

#### Enable Credentials

```js
cors: {
  credentials: true,
}
```

✔ Required for cookies and `Authorization` headers.  
❌ Cannot use with `origin: "*"` (must specify origin).

#### Disable Credentials

```js
cors: {
  credentials: false,
}
```

✔ API is accessible without credentials.

---

### 3. `methods` – Allowed HTTP Methods

Defines which HTTP methods can be used in requests.

#### Allow All Methods

```js
cors: {
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS', 'HEAD'],
}
```

✔ Allows all common HTTP methods.

#### Restrict to Read-Only

```js
cors: {
  methods: ['GET', 'HEAD'],
}
```

✔ Only allows safe read operations.

---

### 4. `headers` – Custom Request Headers

Defines which request headers are allowed in cross-origin requests.

#### Allow Specific Headers

```js
cors: {
  headers: ['Content-Type', 'Authorization'],
}
```

✔ Only allows `Content-Type` and `Authorization`.

---

### 5. `expose` – Custom Response Headers

Controls which response headers are exposed to the client.

#### Expose Custom Headers

```js
cors: {
  expose: ['X-Custom-Header', 'X-Request-ID'],
}
```

✔ Allows the client to read `X-Custom-Header` and `X-Request-ID`.

---

### 6. `maxAge` – Preflight Caching

Defines how long preflight requests (OPTIONS) are cached by the browser.

#### Cache for 24 Hours

```js
cors: {
  maxAge: 86400,
}
```

✔ Reduces preflight requests for 24 hours.

#### No Caching

```js
cors: {
  maxAge: 0,
}
```

✔ Preflight requests sent on every request.  
✔ Useful for frequently changing policies.

## CSRF Middleware

Cross-Site Request Forgery (**CSRF**) is an attack where an attacker tricks an authenticated user into making unwanted requests to a website where they are logged in. Since browsers **automatically send cookies with requests**, the attacker can execute unauthorized actions on behalf of the user—like changing their account settings or making transactions.

To prevent CSRF attacks, our middleware **generates a unique CSRF token** and stores it in an **HTTP-only cookie**. When rendering forms, this token is also included as a hidden field. On submission, our middleware **validates the token from the request body against the token stored in the cookie**. Since the attacker **does not have access to the CSRF token stored in cookies**, they cannot forge a valid request, preventing the attack.

### Setup Your App and Router

First, create an application instance and import the necessary dependencies:

```js
const { App, Router, csrf } = require('bnjsx');

const app = new App();
const router = new Router();
```

### Add CSRF Token

To protect your form from CSRF attacks, use the `csrf` middleware in your GET request. This middleware **creates and stores a CSRF token** inside the request object.

```js
router.get('/form', csrf, async (req, res) => {
  // CSRF middleware generates a token and stores it in req.csrfToken
  console.log(req.csrfToken);

  // Pass the CSRF token to your form
  return res.render('pages.form', { csrfToken: req.csrfToken });
});
```

Inside your form template, include a **hidden input** field to send the CSRF token with every request:

```html
<input type="hidden" name="csrfToken" value="$(csrfToken)" />
```

### Validate CSRF Tokens

Before handling form submissions, use the `parse` middleware to parse the request body and make it available for the CSRF middleware.

```js
const { parse } = require('bnjsx'); // Parse middleware to read request body

router.post('/form', parse, csrf, async (req, res) => {
  // CSRF middleware validates req.body.csrfToken against the stored CSRF token.
  // If the token is valid, the request proceeds.
  console.log('CSRF validation passed!');
});
```

### Handle CSRF Validation Errors

If CSRF validation **fails**, the middleware **rejects the request** with a `BadRequestError`:

```js
reject(new BadRequestError('Invalid or missing CSRF token'));
```

- **If using `bnjet` (our scaffolding tool)** → The default error handler will display the `views/errors/400.fx` error page.
- **If running in API mode** → The error handler returns a JSON response instead.
- You can also define your own error handlers and handle errors however you like.

For more details on error handling, see the **App module documentation**.

### CSRF Protection in Web Mode

Our **Auth module** uses **cookies for authentication in web mode**, meaning CSRF protection is **mandatory** in this case. Since browsers automatically attach authentication cookies, a malicious site could attempt to perform actions on behalf of a logged-in user. By enforcing CSRF token validation, our middleware ensures that every request originates from the intended user, blocking unauthorized actions.

For **API mode**, where authentication tokens are sent in headers (not cookies), CSRF protection is **not required** because attackers cannot access or reuse these tokens.

## Security Middleware

Bnjsx provides built-in security middleware to enhance your application's security. It helps mitigate various attacks, such as Cross-Site Scripting (XSS), Clickjacking, and other security vulnerabilities.

We already provide a default security configuration, so in most cases, no additional setup is required. However, if you have specific security requirements, you can customize the security option in your `bnjsx.config.js` file.

This guide explains each security option and provides configuration examples.

### 1. Content Security Policy (CSP)

The Content Security Policy (CSP) defines which resources can be loaded in the browser, reducing the risk of XSS attacks.

**Available Options:**

| Directive                   | Purpose                       | Example                             |
| --------------------------- | ----------------------------- | ----------------------------------- |
| `default-src`               | Fallback for all resources    | `'self'` (Same origin only)         |
| `script-src`                | Controls JavaScript sources   | `'self' https://cdn.example.com'`   |
| `style-src`                 | Controls CSS sources          | `'self' 'unsafe-inline'`            |
| `img-src`                   | Allowed image sources         | `'self' data:'`                     |
| `font-src`                  | Allowed font sources          | `'self' https://fonts.example.com'` |
| `frame-ancestors`           | Restricts embedding           | `'self'` (Only same origin)         |
| `object-src`                | Controls `<object>`/`<embed>` | `'none'` (Disables them)            |
| `form-action`               | Limits form submissions       | `'self'`                            |
| `upgrade-insecure-requests` | Forces HTTPS                  | Enabled                             |

> Full list of options: [MDN CSP Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)

**Default:**

```json
{
  // Allow loading all content from the same origin unless explicitly specified otherwise
  "defaultSrc": "'self'",

  // Only allow using a <base> tag that points to the same origin
  "baseUri": "'self'",

  // Allow fonts to be loaded from the same origin, any HTTPS source, or as a data URL
  "fontSrc": "'self' https: data:",

  // Only allow form submissions to the same origin
  "formAction": "'self'",

  // Only allow embedding this page in iframes on the same origin
  "frameAncestors": "'self'",

  // Allow loading images from the same origin or as base64-encoded data URLs
  "imgSrc": "'self' data:",

  // Completely block embedding objects like Flash, Java Applets, or other plugins
  "objectSrc": "'none'",

  // Only allow executing scripts that are hosted on the same origin
  "scriptSrc": "'self'",

  // Prevent inline JavaScript inside HTML attributes from executing
  "scriptSrcAttr": "'none'",

  // Allow styles from the same origin and any HTTPS source, but also allow inline styles
  "styleSrc": "'self' https: 'unsafe-inline'",

  // Automatically upgrade all HTTP requests to HTTPS, preventing mixed content issues
  "upgradeInsecureRequests": true
}
```

**Examples:**

```ts
security.contentSecurityPolicy = {
  defaultSrc: "'self'", // Only allow resources from the same origin
  scriptSrc: ["'self'", 'https://trusted-cdn.com'], // Allow scripts from self and trusted CDN
  imgSrc: "'self' data:", // Allow images from self and base64-encoded images
};
```

To disable CSP:

```ts
security.contentSecurityPolicy = false; // (not recommended)
```

### 2. Strict Transport Security (HSTS)

Enforces HTTPS connections to the server, preventing man-in-the-middle attacks.

**Available Options**

| Option                | Description                                | Effect                                |
| --------------------- | ------------------------------------------ | ------------------------------------- |
| `maxAge` _(required)_ | Time (in seconds) to enforce HTTPS         | Enforces HTTPS after first visit      |
| `includeSubDomains`   | Applies HSTS to all subdomains             | Protects subdomains after first visit |
| `preload`             | Requests inclusion in browser preload list | Enforces HTTPS before first visit     |

📌 **Note:** Without `preload`, HSTS only works **after a user’s first secure visit**. Preloading eliminates this delay but requires submitting your domain to [hstspreload.org](https://hstspreload.org). 🚀

**Default:**

```json
{
  "maxAge": 15552000,
  "includeSubDomains": true,
  "preload": false
}
```

**Examples:**

```ts
security.strictTransportSecurity = {
  maxAge: 31536000, // Enforce HTTPS for 1 year
  includeSubDomains: true,
  preload: true, // Allow inclusion in Chrome's preload list
};
```

To disable HSTS:

```ts
security.strictTransportSecurity = false;
```

### 3. Referrer Policy

Controls how much **referrer** information is included in requests when navigating between pages. The referrer is the URL of the page that made the request.

**Available Options:**

| Option                            | Behavior                                                                                    |
| --------------------------------- | ------------------------------------------------------------------------------------------- |
| `no-referrer`                     | No referrer data is sent.                                                                   |
| `no-referrer-when-downgrade`      | Referrer is stripped when downgrading HTTPS → HTTP.                                         |
| `origin`                          | Sends only the origin (`https://example.com`).                                              |
| `origin-when-cross-origin`        | Full referrer for same-origin, only origin for cross-origin.                                |
| `same-origin`                     | Sends referrer only within the same site.                                                   |
| `strict-origin`                   | Sends origin but removes it if downgrading to HTTP.                                         |
| `strict-origin-when-cross-origin` | Full referrer for same-origin, only origin for cross-origin, removes referrer on downgrade. |
| `unsafe-url`                      | Always sends the full URL (not recommended).                                                |

**Default:**

```json
"origin"
```

**Examples:**

_(Use no referrer at all (maximum privacy))_

```ts
security.referrerPolicy = 'no-referrer';
```

_(Allow referrer only when navigating within the same site)_

```ts
security.referrerPolicy = 'same-origin';
```

_(Send full referrer within the same site, but only the origin for cross-origin requests)_

```ts
security.referrerPolicy = 'origin-when-cross-origin';
```

_(Strict security: prevent referrer leaks on HTTP downgrades)_

```ts
security.referrerPolicy = 'strict-origin-when-cross-origin';
```

_(Disable Referrer Policy (not recommended))_

```ts
security.referrerPolicy = false;
```

### 4. Cross-Origin Resource Policy (CORP)

Restricts which origins can access your server's resources.

**Available Options:**

| Option         | Behavior                                                  |
| -------------- | --------------------------------------------------------- |
| `same-origin`  | Only same-origin requests are allowed.                    |
| `same-site`    | Allows requests from the same site, including subdomains. |
| `cross-origin` | Allows all origins to access resources.                   |
| `false`        | Disables CORP, no restrictions on resource access.        |

**Default:**

```json
"same-origin"
```

**Examples:**

```ts
security.crossOriginResourcePolicy = 'same-site'; // Allow subdomains
security.crossOriginResourcePolicy = 'cross-origin'; // Allow all origins
```

To disable:

```ts
security.crossOriginResourcePolicy = false;
```

### 5. Cross-Origin Opener Policy (COOP)

Prevents cross-origin interactions to enhance security.

**Available Options:**

| Option                     | Behavior                                                                 |
| -------------------------- | ------------------------------------------------------------------------ |
| `same-origin`              | Fully isolates the page from all other origins.                          |
| `same-origin-allow-popups` | Allows popups to open but keeps the main page isolated.                  |
| `unsafe-none`              | No isolation; all origins can interact with the page ( Not recommended). |
| `false`                    | Disables COOP entirely.                                                  |

**Default:**

```json
"same-origin"
```

**Examples:**

```ts
security.crossOriginOpenerPolicy = 'same-origin-allow-popups'; // Allow popups without full isolation
security.crossOriginOpenerPolicy = 'unsafe-none'; // Disable COOP (not recommended)
```

To disable:

```ts
security.crossOriginOpenerPolicy = false;
```

### 6. Cross-Origin Embedder Policy (COEP)

Specifies how cross-origin resources can be embedded in your page.

**Available Options**

| Option           | Behavior                                                                   |
| ---------------- | -------------------------------------------------------------------------- |
| `unsafe-none`    | No restrictions on embedding cross-origin resources.                       |
| `require-corp`   | Only allows embedding resources with a valid CORP header.                  |
| `credentialless` | Embeds resources but prevents credentials (cookies, auth) from being sent. |
| `false`          | Disables COEP entirely, no restrictions on embedding.                      |

**Default:**

```json
false
```

**Examples:**

```ts
security.crossOriginEmbedderPolicy = 'require-corp'; // Require CORP headers for embeds
security.crossOriginEmbedderPolicy = 'credentialless'; // Allow embeds without credentials
```

To disable:

```ts
security.crossOriginEmbedderPolicy = false;
```

### 7. Origin-Agent Cluster

Makes sure your website runs in its own isolated space in the browser, preventing certain cross-origin data leaks and side-channel attacks like `Spectre`

**Default:**

```json
true
```

**Example:**

```ts
security.originAgentCluster = false; // Disable memory protection (not recommended)
```

### 8. X-Content-Type-Options

Stops the browser from guessing file types, forcing it to use the correct one. This helps prevent sneaky attacks where a file pretends to be something it's not.

**Default:**

```json
true
```

**Example:**

```ts
security.xContentTypeOptions = false; // Allow MIME-type sniffing (not recommended)
```

### 9. X-DNS-Prefetch-Control

Controls DNS prefetching, which resolves domain names in the background to speed up browsing.

**Default:**

```json
true // Disables DNS prefetching to reduce privacy risks
```

**Example:**

```ts
security.xDnsPrefetchControl = false; // Enable DNS prefetching (not recommended)
```

### 10. X-Download-Options

Prevents automatic execution of downloaded files.

**Default:**

```json
true
```

**Example:**

```ts
security.xDownloadOptions = false; // Allow automatic execution (not recommended)
```

### 11. XSS Protection

Prevents some types of cross-site scripting attacks.

**Default:**

```json
true
```

**Example:**

```ts
security.xssProtection = false; // Disable XSS protection (not recommended)
```

### 12. X-Frame-Options

Prevents your site from being embedded in `<iframe>` elements to protect against **clickjacking attacks**.

#### **Available Options**

| Option       | Behavior                                                  |
| ------------ | --------------------------------------------------------- |
| `DENY`       | Blocks all embedding in `<iframe>`.                       |
| `SAMEORIGIN` | Allows embedding only from the same origin.               |
| `false`      | Disables this policy, allowing embedding from any origin. |

**Default:**

```json
"SAMEORIGIN"
```

**Examples:**

```ts
security.xFrameOptions = 'DENY'; // Block all iframes
security.xFrameOptions = false; // Allow all iframes (not recommended)
```

### 13. X-Permitted-Cross-Domain-Policies

Controls whether `Flash` and `Acrobat` can make cross-domain requests.

**Available Options**

| Option            | Behavior                                                              |
| ----------------- | --------------------------------------------------------------------- |
| `none`            | Blocks all cross-domain requests.                                     |
| `master-only`     | Only allows cross-domain requests if a master policy file is present. |
| `by-content-type` | Allows requests based on the file's content type.                     |
| `all`             | Allows all cross-domain requests.                                     |
| `false`           | Disables this policy, no restrictions.                                |

**Default:**

```json
"none"
```

**Examples:**

```ts
security.xPermittedCrossDomainPolicies = 'master-only';
security.xPermittedCrossDomainPolicies = 'all';
```

To disable:

```ts
security.xPermittedCrossDomainPolicies = false;
```
