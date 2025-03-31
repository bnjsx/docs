# Bnjsx Response Module

The **Response Module** extends Node.js `ServerResponse`, providing additional methods and properties while retaining the original API. This module simplifies handling HTTP responses, offering powerful utilities for sending data, streaming files, setting cookies, and more.

---

## Table of Contents

- [Response Methods & Properties](#response-methods--properties)
  - [Setting the HTTP Status Code](#setting-status-code)
  - [Sending Responses](#sending-responses)
  - [Redirecting Requests](#redirecting-requests)
  - [Managing Cookies](#managing-cookies)
  - [Clearing Cookies](#clearing-cookies)
  - [Serving Files](#serving-files)
  - [File Downloads](#file-downloads)
  - [Streaming Data](#streaming-data)
  - [Rendering Templates](#rendering-templates)
  - [Additional Utilities](#additional-utilities)

---

## Response Methods & Properties

Every middleware function has access to the response object in the second argument. Below is a list of useful properties and methods you can use.

### Setting Status Code

Sets the HTTP status code with an optional message.

```js
res.status(404, 'Not Found');
```

If no message is provided, it automatically sets the standard HTTP status message.

```js
res.status(404); // Not Found
res.status(200); // OK
res.status(500); // Internal Server Error
res.status(400); // Bad Request
```

This method returns the response instance, allowing for method chaining:

```js
res.status(201).json({ message: 'Created successfully' });
```

### Sending Responses

Sends a response with automatic content-type detection.

```js
res.send('Hello'); // text/html
res.send({ key: 'value' }); // application/json
```

You can also use `json` to send a JSON response.

```js
res.json({ user: 'John', age: 30 });
```

Here is how `send` handles different data types

| **Type**       | **Behavior**            | **Content-Type**           | **Example**               |
| -------------- | ----------------------- | -------------------------- | ------------------------- |
| `string`       | Sent as html            | `text/html`                | `res.send('Hello')`       |
| `Buffer`       | Sent as binary data     | `application/octet-stream` | `res.send(Buffer.from())` |
| `object/array` | Converted to JSON       | `application/json`         | `res.send({ key: 1 })`    |
| `number`       | Converted to string     | `text/plain`               | `res.send(404)`           |
| `boolean`      | Converted to string     | `text/plain`               | `res.send(false)`         |
| `undefined`    | Sends an empty response | No Content-Type set        | `res.send()`              |
| Other types... | Throws error            | N/A                        | `res.send(Symbol())`      |

### Redirecting Requests

Redirects to the specified URL.

```js
res.redirect('https://example.com');
```

### Managing Cookies

Sets an HTTP cookie.

```js
res.cookie('token', 'abc123', { httpOnly: true, secure: true });
```

#### Cookie Options

| Option        | Type                           | Description                                                                    |
| ------------- | ------------------------------ | ------------------------------------------------------------------------------ |
| `maxAge`      | `number`                       | The lifetime of the cookie in **seconds**. If set, it **overrides** `expires`. |
| `expires`     | `string\|Date`                 | The **expiration date** of the cookie. Can be a `Date` object or `UTC` string. |
| `domain`      | `string`                       | The domain for which the cookie is valid, including subdomains.                |
| `path`        | `string` _(default: `/`)_      | Limits the cookie to a specific **URL path**. Defaults to `/`.                 |
| `secure`      | `boolean` _(default: `false`)_ | Ensures the cookie is only sent over **HTTPS** connections.                    |
| `httpOnly`    | `boolean`                      | Prevents JavaScript from accessing the cookie (for security reasons).          |
| `partitioned` | `boolean`                      | Ensures the cookie is stored separately for each partition (browser feature).  |
| `sameSite`    | `'Strict' \| 'Lax' \| 'None'`  | Controls whether the cookie is sent with **cross-site requests**.              |
| `priority`    | `'Low' \| 'Medium' \| 'High'`  | Determines the **priority** of the cookie when browser limits apply.           |

#### 1. Set a Secure, HTTP-only Cookie

```js
res.cookie('session', 'xyz456', { httpOnly: true, secure: true });
```

> The cookie is **not accessible via JavaScript** and is only sent over **HTTPS**.

#### 2. Set a Cookie That Expires in 1 Hour

```js
res.cookie('user', 'john_doe', { maxAge: 3600 });
```

> The cookie expires **in 1 hour**.

#### 3. Set a Cookie That Expires in 2 Months

```js
res.cookie('remember_me', 'user_token', { expires: UTC.future.month(2) });
```

> The cookie expires **in 2 months**. _(See: our `UTC` helper docs)_

#### 4. Set a Cookie for a Specific Domain and Path

```js
res.cookie('theme', 'dark', { domain: 'example.com', path: '/account' });
```

> The cookie is only sent for `example.com/account` and its subdirectories.

#### 5. Set a Cross-Site Cookie

```js
res.cookie('tracking', 'xyz', { sameSite: 'None', secure: true });
```

> The cookie is sent with **cross-site requests**, but requires `secure: true`.

#### 6. Set a High-Priority Cookie

```js
res.cookie('cart', '123', { priority: 'High' });
```

> The browser will preserve this cookie over lower-priority cookies when reaching storage limits.

### Clearing Cookies

Removes a cookie by setting its expiration date in the past.

```js
res.clearCookie('token');
```

#### Important Notes:

- By default, the **cookie's path is `/`** and **domain is the request host**.
- If the cookie was set with a **custom `path` or `domain`**, you **must** provide the same values when clearing it.  
  Otherwise, the browser **won't know which cookie to remove**.

#### Clearing a Cookie with a Custom Path and Domain

```js
res.clearCookie('session_id', '/user', 'example.com');
```

👉 This ensures the correct cookie (`session_id`) is removed for **`example.com/user`**, rather than some other cookie with the same name but a different path or domain.

### Serving Files

Sends a file to the client with automatic **MIME type detection** and **range request support**.

```js
res.sendFile('/images/logo.png'); // Content-Type: image/png
```

```js
res.sendFile('/videos/movie.mp4'); // Allows seeking in the video
```

#### How It Works

| **Scenario**       | **Behavior**                                           |
| ------------------ | ------------------------------------------------------ |
| No `Range` header  | Sends the **entire file** with `200 OK`.               |
| `Range: bytes=X-Y` | Sends **part of the file** with `206 Partial Content`. |
| Invalid range      | Sends `416 Range Not Satisfiable`.                     |

#### Headers Set Automatically

| **Header**       | **Description**                                              |
| ---------------- | ------------------------------------------------------------ |
| `Content-Type`   | Determines file type (e.g., `image/png`, `application/pdf`). |
| `Content-Length` | Specifies the file size (for full responses).                |
| `Accept-Ranges`  | Informs the client that range requests are supported.        |
| `Content-Range`  | Used in partial responses (`206 Partial Content`).           |

### File Downloads

Forces a file **download** by setting the appropriate `Content-Disposition` header. This method uses `sendFile()` internally to send the file while instructing the browser to download it.

#### Download a PDF

```js
res.download('/files/report.pdf', 'report.pdf');
```

#### Download a ZIP File Without Changing the Name

```js
res.download('/backups/latest.zip'); // Uses the original filename
```

#### How It Works

- Calls `sendFile(path)` to **send** the file.
- Sets `Content-Disposition: attachment; filename="name"` to prompt a **download**.
- Detects the file **MIME type** automatically.

### Streaming Data

Streams data to the client using a readable stream.

```js
const fs = require('fs');
const stream = fs.createReadStream('/path/to/large-file.mp4');
res.stream(stream);
```

### Rendering Templates

Renders a template using our `Flex` template engine.

```js
res.render('pages.home', { title: 'Home Page' });
```

> This method renders your `Flex` component and sends the result to the clinet _(See: `Flex` template engine docs for more details...)_

### Additional Methods

Retrieves the default status message for an HTTP code.

```js
console.log(res.getMessage(200)); // 'OK'
```

Determines the appropriate `Content-Type` based on the provided data.

```js
console.log(res.contentType({})); // 'application/json'
```

---

## Conclusion

The **Response Module** enhances the native `ServerResponse` object, making it more convenient and powerful. If any section needs additional examples, please let us know!
