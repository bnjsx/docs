# Bnjsx Request Module

The **Request Module** extends Node.js `IncomingMessage`, providing additional methods and properties while retaining the original API.

## Additional Properties & Methods

Every middleware function has access to the request object in the first argument. Below is a list of useful properties and methods you can use:

#### 1. `response`

- The response object associated with this request.
- **Type:** `Response`

#### 2. `cookies`

- Stores cookies sent with the request.
- **Type:** `Record<string, string>`

  ```ts
  console.log(req.cookies); // { sessionId: 'abc123' }
  ```

**Note:** This property is `undefined` unless you use our cookie middleware, which is responsible for parsing and filling the `req.cookies` property.

#### 3. `protocol`

- Determines the request protocol.
- **Type:** `'http' | 'https'`

  ```ts
  console.log(req.protocol); // 'http'
  ```

#### 4. `host`

- The hostname extracted from the request URL.
- **Type:** `string`

  ```ts
  console.log(req.host); // 'localhost'
  ```

#### 5. `port`

- The port number used in the request.
- **Type:** `string`

  ```ts
  console.log(req.port); // '2025'
  ```

#### 6. `path`

- The request path without query parameters.
- **Type:** `string`

  ```ts
  console.log(req.path); // '/api/users'
  ```

#### 7. `query`

- Parsed query parameters from the request URL.
- **Type:** `URLSearchParams`

  ```ts
  console.log(req.query.get('id')); // '123'
  ```

#### 8. `params`

- Extracted route parameters.
- **Type:** `Record<string, string> | string[]`

  ```ts
  console.log(req.params); // { id: '42' } for named params
  console.log(req.params); // ['42'] for regex-based routes
  ```

#### 9. `body`

- The parsed body of the request.
- **Type:** `any`

  ```ts
  console.log(req.body); // { username: 'john', password: 'secure123' }
  ```

**Note:** The `body` is filled using the `Form` module (handling fields and file uploads) and can also be populated by other middlewares such as `json` and `text` middleware.

#### 10. `errors`

- Stores validation errors related to form submissions.
- **Type:** `Record<string, Array<string>>`

  ```ts
  console.log(req.errors); // { email: ['Invalid email format'] }
  ```

#### 11. `type(name)`

- Checks if the `Content-Type` header matches a given MIME type.
- **Parameters:**
  - `name` - The MIME type or file extension.
- **Returns:** `boolean`

  ```ts
  // Using file extension
  if (req.type('json')) {
    console.log('Request contains JSON');
  }
  ```

#### 12. `accepts(name)`

- Checks if the request accepts a specific MIME type.
- **Parameters:**
  - `name` - The MIME type or file extension.
- **Returns:** `boolean`

  ```ts
  if (req.accepts('html')) {
    console.log('Client accepts HTML responses');
  }
  ```

#### 13. `getHeader(name)`

- Retrieves a specified request header.
- **Parameters:**
  - `name` - The header name.
- **Returns:** `string | string[] | undefined`

  ```ts
  console.log(req.getHeader('User-Agent')); // 'Mozilla/5.0'
  ```

## Conclusion

This **Request Module** enhances the native `IncomingMessage` object with additional utilities, making it easier to handle requests in a structured way. Feel free to extend or modify it based on your project's needs!
