# Bnjsx Authentication Module

The **Bnjsx Auth module** provides a built-in **JWT-based authentication system** that works seamlessly for both APIs and websites. It is highly configurable, allowing you to customize almost everything to fit your needs.

## Features

- **Sign Up** (User Registration)
- **Sign In** (User Login)
- **Sign Out** (User Logout)
- **Password Reset** (Forgot Password Functionality)
- **Input Validation** (Ensures data integrity)
- **Flexible Configuration** (Easily customizable settings)
- **Authorization Middlewares** (Protect routes with authentication checks)

## Getting Started

### 1. Import and Initialize Auth

First, import the `Auth` module from `bnjsx` and create an instance:

```javascript
const auth = new Auth({ mailer });
```

### 2. Configuring the Mailer

The `Auth` constructor requires a **mailer function** to handle password reset emails. During development, you can use a simple mailer that logs emails to the console:

```javascript
const mailer = async (to, subject, html) => console.log(html);
```

> ⚠ **In production**, use a real mailer service such as [Nodemailer](https://nodemailer.com/) or any email-sending technology of your choice.

### 3. Recommended Instantiation

To avoid potential issues with method binding, it's **highly recommended** to use `Auth.break()` instead of `new Auth()`:

```javascript
const auth = Auth.break({ mailer }); // Recommended ✅
```

Instead of:

```javascript
const auth = new Auth({ mailer }); // Not recommended ❌
```

---

## Implementing Authentication in Your App

### 1. Setting Up the App and Router

Before adding authentication, set up your `App` and `Router` instances:

```javascript
const app = new App();
const router = new Router();
```

### 2. Registration Route (GET)

Create a route to render the **registration page**:

```javascript
router.get('/register', async (req, res) => {
  return res.render('pages.register');
});
```

> If you're using `bnjet` (our scaffolding tool), the **`pages.register` view** should already be available.

### 3. Handling User Registration (POST)

To process user registration, use the `auth.register` middleware:

```javascript
router.post('/register', auth.register, async (req, res) => {
  // Check if validation fails
  if (req.errors) {
    console.log(req.errors); // Log validation errors

    return res.render('pages.register', {
      error: Object.values(req.errors).flat()[0], // Display the first error
    });
  }

  // If registration is successful, redirect to the login page
  return res.redirect('/login');
});
```

### 4. Start the Server

```javascript
app.start();
```

---

## How Registration Works

- The `auth.register` middleware **validates** the user's email and password.
- It ensures the user has accepted the **terms and conditions**.
- If validation passes, the user is **inserted into the database**.

### Setting Up the Database

Ensure you have a **`users` table** with at least `email` and `password` columns.

If you're using `bnjet`, a generator file named `01_generate_users_table` is included. Run the following command to create the table:

```sh
node exec gen
```

Make sure your **database is set up** before running the application.

> **By default,** `bnjet` uses an **SQLite** database, which is great for development. However, **MySQL** and **PostgreSQL** are also supported.

## Customizing Validation Rules

By default, the **Auth** system expects the following fields in your registration form:

- `email`
- `password`
- `terms`

However, you can customize the validation to fit your needs. For example, you can add a `confirmation` field to ensure the user confirms their password. This is where the `validation` property comes into play.

The `auth.validation` object allows you to configure form validation rules. It has four properties, each representing a different form:

- `register`
- `login`
- `passwordReset`
- `requestReset`

Each property defines the required fields and their validation rules. If you log `auth.validation.register`, you will see something like this:

```js
{
  email: this.testers.email,
  password: this.testers.password,
  terms: this.testers.terms,
}
```

This means the **register** form requires `email`, `password`, and `terms` fields, and each field is validated using predefined testers.

## Adding a Custom Validation Rule

Let's add validation for a `confirmation` field in the registration form:

```js
auth.validation.register.confirmation = [
  {
    test: (value, body) => value && value === body.password,
    message: 'Password must match',
  },
];
```

### How It Works:

- The `test` function checks whether the `confirmation` field value matches the `password` field value.
- If the validation fails, the provided error message (`Password must match`) will be added to `req.errors` like this:

```js
{
  confirmation: ['Password must match'];
}
```

## Handling Multiple Validation Rules

You can define multiple validation testers for a single field. Each tester runs in sequence, and if any tester fails, its corresponding error message is added to `req.errors`.

### Example:

```js
auth.validation.register.email = [
  { test: (value) => value.includes('@'), message: 'Invalid email format' },
  {
    test: (value) => value.length > 5,
    message: 'Email must be at least 6 characters long',
  },
];
```

### Important:

Always check for `req.errors` in your request handler before proceeding, as shown in the example above!

By customizing `auth.validation`, you have full control over your authentication form validation.

## Predefined Testers

The Auth module provides a set of predefined validation testers to save you time. You can access them under `auth.testers`.

```javascript
{
  email: [
    {
      test: (email) => isEmail(email),
      message: this.messages.invalidEmail,
    },
    {
      test: (email) => email && email.length < 250,
      message: this.messages.longEmail,
    },
  ],
  password: [
    {
      test: (pass) => pass && pass.length >= 8,
      message: this.messages.shortPassword,
    },
    {
      test: (pass) => pass && pass.length < 20,
      message: this.messages.longPassword,
    },
    {
      test: (pass) => isPassword(pass),
      message: this.messages.weakPassword,
    },
  ],
  terms: [
    {
      test: (value) => typeof value === 'string',
      message: this.messages.termsNotAccepted,
    },
  ],
  confirmation: [
    {
      test: (pass, body) => pass && pass === body.password,
      message: this.messages.passwordMismatch,
    },
  ],
}
```

These testers cover common fields like **email**, **password**, **terms**, and **confirmation**. If you decide to add a `terms` or `confirmation` field in a specific form, you can simply reference these testers instead of writing custom validation logic.

### Adding Confirmation Field Validation

For example, to add a confirmation field validation to the registration form, do the following:

```javascript
auth.validation.register.confirmation = auth.testers.confirmation;
```

Now, whenever a user registers, they will be required to confirm their password. However, ensure that you provide a corresponding input field with a matching name in your form.

- If your validation rule is:

  ```javascript
  auth.validation.register.confirmation;
  ```

  Then the input field must be named `confirmation`.

- If your validation rule is:
  ```javascript
  auth.validation.register.confirm_password;
  ```
  Then the input field must be named `confirm_password`.

This ensures the validation works correctly.

### Disabling Validation

Although not recommended, you can disable validation for specific fields or entire forms.

#### Disable Validation for a Specific Field in a Form

- Disable email validation in the registration form:
  ```javascript
  auth.validation.register.email = undefined;
  ```
- Disable password validation in the login form:
  ```javascript
  auth.validation.login.password = undefined;
  ```

#### Disable Validation for an Entire Form

- Disable validation for the login form:
  ```javascript
  auth.validation.login = undefined;
  ```

#### Disable Validation Entirely

- Turn off all validations:
  ```javascript
  auth.validation = {};
  ```

### Warning ⚠️

Disabling validation can lead to errors, especially when inserting values into the database. For example, if both email and password validations are disabled, and these fields are `undefined`, the database insert operation may fail.

Always validate user inputs to ensure data integrity and prevent unexpected errors.

## Customize Error Messages

Bnjsx Auth allows you to customize default error messages. This is particularly useful if you're building an application in a different language (e.g., Arabic, French) or if you simply want to modify the default messages to better suit your application.

### Default Error Messages

Below is the list of default error messages used by the Auth module:

```javascript
{
  invalidEmail: 'Invalid email address.',
  longEmail: 'Email address must not exceed 254 characters.',
  shortPassword: 'Password must be at least 8 characters long.',
  longPassword: 'Password must not exceed 20 characters.',
  weakPassword: 'Password must contain letters, numbers, and symbols.',
  emailTaken: 'This email is already registered.',
  userNotFound: 'Invalid email or password.',
  termsNotAccepted: 'You must accept the terms to continue.',
  passwordMismatch: 'Passwords do not match.',
}
```

### Overriding Default Messages

You can easily override these messages by passing a `messages` object when creating an `Auth` instance:

```javascript
const auth = Auth.break({
  mailer,
  messages: {
    invalidEmail: 'Please enter a valid email address.',
    termsNotAccepted: 'You must agree to the terms before proceeding.',
  },
});
```

### How It Works

- If you provide a custom message, it replaces the default message.
- If you don’t provide a message, the default one is used.

That’s it! You now have full control over authentication error messages.

## Authentication Mode

The Bnjsx Auth module supports two authentication modes:

- **`lazy` mode (default)** – Requires the user to manually log in after registration.
- **`smart` mode** – Automatically logs the user in after successful registration.

### Enabling Smart Mode

To enable `smart` mode, pass the `mode` option when creating the `Auth` instance:

```javascript
const auth = Auth.break({ mode: 'smart' });
```

#### Automatic Login After Registration

When using `smart` mode, users are automatically logged in after registering, allowing you to redirect them to the home page instead of the login page.

```javascript
router.post('/register', auth.register, async (req, res) => {
  // Check if validation fails
  if (req.errors) {
    console.log(req.errors); // Log validation errors
    return res.render('pages.register', {
      error: Object.values(req.errors).flat()[0], // Display the first error
    });
  }

  // In lazy mode, you'd redirect the user to the login page:
  // return res.redirect('/login');

  // In smart mode, the user is already logged in, so redirect to home:
  return res.redirect('/');
});
```

---

## API Registration

If you are building an API, you can access the authentication token in the request and return it in the response.

```javascript
router.post('/register', auth.register, async (req, res) => {
  // Check if validation fails
  if (req.errors) {
    console.log(req.errors); // Log validation errors
    return res.json({ error: 'ValidationError', message: req.errors });
  }

  return res.json({ token: req.authToken });
});
```

> **Note:** The `authToken` is only available in **`smart` mode**.

Here's your improved documentation in Markdown format with better structure, clarity, and grammar:

---

## Implementing Login

The Bnjsx Auth module provides a `login` middleware to authenticate users. This middleware validates the user's email and password and logs them in if the credentials are correct.

If authentication fails, error messages will be available in `req.errors` as usual.

### Possible Errors

- **`invalidPassword`** – The password is incorrect.
- **`invalidEmail`** – The email format is invalid.
- **`userNotFound`** – No user was found with the provided email.

> **Note:** This middleware requires a **users table** in your database. Run `node exec gen` to generate the table if you haven't already.

### How It Works

The `login` middleware creates an authentication cookie in the user's browser. On subsequent requests, this cookie is validated (using Auth.any or Auth.auth) to determine whether the user is authenticated.

### Rendering the Login View

If you are using `bnjet`, the login page template (`pages.login`) should already be available.

```javascript
router.get('/login', async (_, res) => {
  return res.render('pages.login');
});
```

### Handling Login Requests

For `POST` requests, we use the `login` middleware to validate the email and password, generate an authentication token, and log the user in.

```javascript
router.post('/login', auth.login, async (req, res) => {
  // If validation fails, display the first error
  if (req.errors) {
    return res.render('pages.login', {
      error: Object.values(req.errors).flat()[0],
    });
  }

  // Redirect to home if authentication is successful
  return res.redirect('/');
});
```

---

## API Authentication

If you are building an API, the authentication token will be available in `req.authToken`.

```javascript
router.post('/login', auth.login, async (req, res) => {
  if (req.errors) {
    return res.json({ error: 'ValidationError', message: req.errors });
  }

  return res.json({ token: req.authToken });
});
```

---

## Authenticated Users

The Auth module provides the `auth` middleware, which you can use to validate user tokens and protect authenticated routes.

For example, let’s say you have a `/profile` route that should only be accessible by authenticated users. You can use the `auth` middleware to ensure only logged-in users can access it:

```javascript
router.get('/profile', Auth.auth, async (req, res) => {
  // Render the profile page and provide user data
  return res.render('pages.profile', { user: req.user });
});
```

The `auth` middleware ensures the request is made by an **authenticated user**. If a guest tries to access `/profile`, a **403 (Access Forbidden)** error page will be displayed. This view is available if you are using `bnjet`.

### API vs. Web Authentication

- **API mode**: Expects an `x-auth-token` header containing the authentication token.
- **Web mode**: Expects an `authToken` cookie for authentication.

If the authentication token is found and validated successfully, the user’s email will be accessible via `req.user.email`.

### Fetching User Data

Once the email is available in `req.user.email`, you can use it to retrieve the user’s data from the database using a custom middleware:

```javascript
export class Fetch extends Controller {
  async user(req, res) {
    req.user = await this.builder((builder) =>
      builder
        .select()
        .from('users')
        .where((col) => col('email').equal(req.user.email))
        .first()
    );
  }
}

// Export the middleware
const fetch = new Fetch();
export const user = fetch.user.bind(fetch);
```

Now, you can import this middleware and use it to fetch the user from the database.

> **Tip:** Instead of manually querying the database, use a **User Model** to easily load related data like user posts, categories, and more. Check out our **Model documentation** for details.

---

## Any User (Authenticated or Guest)

The `any` middleware allows both authenticated users and guests to access a route.

- If a **guest** accesses the route, they are allowed in, and `req.user` will be `undefined`.
- If an **authenticated user** accesses the route, their authentication token is validated, and `req.user` is set.

A common use case for this middleware is the **home page**, where both guests and logged-in users can access the page:

```javascript
router.get('/', Auth.any, async (req, res) => {
  // Check if the user is authenticated
  if (req.user) console.log('Welcome back!');
  else console.log('Join us!');

  return res.render('pages.home', { user: req.user });
});
```

---

## Restricting Routes to Guest Users

A **guest user** is someone who is **not authenticated**. Certain pages (e.g., **Login, Register, Password Reset**) should only be accessible to guest users, and logged-in users should be restricted from accessing them.

This is where the `guest` middleware comes into play.

```javascript
router.get('/register', Auth.guest, async (req, res) => {
  // req.user will always be undefined for guests
  console.log(req.user);

  return res.render('pages.register');
});
```

### Handling Forbidden Access

If an **authenticated user** tries to access a guest-only route, the `guest` middleware will throw a **ForbiddenError**, which is handled as follows:

- **Web mode**: The **403 Access Forbidden** page is displayed.
- **API mode**: The following JSON response is returned:

  ```json
  {
    "error": "ForbiddenError",
    "message": "Access Forbidden."
  }
  ```

## Password Reset

The password reset functionality consists of two middlewares:

- **`requestReset`**: Handles user requests for a password reset link.
- **`passwordReset`**: Resets the user's password once they provide a valid token.

### Requirements

To enable password reset functionality, ensure the following:

- A mailer function for sending emails.
- A `users` table exists with `email` and `password` fields.
- A `reset_tokens` table exists to store tokens and expiration times.
  - `bnjet` provides generators you need just run `node exec gen`
- An email component located in the `views` folder.
  - If using `bnjet`, a default email component is available.
  - To use a custom email component, set the `component` option to its path, e.g., `components.email`.

### Requesting a Password Reset

#### Display the Password Reset Request Page

The following route renders the password reset request form:

```js
router.get('/request-reset', async (req, res) => {
  return res.render('pages.request_reset');
});
```

#### Handle the Password Reset Request

The `requestReset` middleware sends an email containing the password reset link.

```js
router.post('/request-reset', auth.requestReset, async (req, res) => {
  if (req.errors) {
    return res.render('pages.request_reset', {
      error: Object.values(req.errors).flat()[0],
    });
  }

  // Display a success message prompting the user to check their email.
  return res.render('pages.request_reset', {
    success: 'Check your email for a password reset link.',
  });
});
```

#### What This Middleware Does

- Validates the given email address
- Generates a unique reset token and stores it in the database.
- Renders the email component with the reset link. _(See **`views/emails/reset_email.fx`**.)_
- Sends the email using the configured mailer function.

### Resetting the Password

#### Display the Password Reset Form

When the user clicks the reset link, they should be redirected to a page where they can enter a new password.

```js
router.get('/password-reset/:token', async (req, res) => {
  return res.render('pages.password_reset', { token: req.params.token });
});
```

> **Note:** Ensure that the form action is set to `/password-reset/:token` so that the reset request is correctly processed.

#### Handle the Password Reset Request

This route processes the password reset form submission.

```js
router.post('/password-reset/:token', auth.passwordReset, async (req, res) => {
  if (req.errors) {
    return res.render('pages.password_reset', {
      error: Object.values(req.errors).flat()[0],
      token: req.params.token, // Preserve token for resubmission.
    });
  }

  // Redirect the user to the login page after a successful reset.
  return res.redirect('/login');
});
```

#### How It Works

- The middleware validates the input fields.
- It verifies that the token is valid and has not expired.
- If valid, the password is updated in the database.
- The user is redirected to the login page.

### Alternative API Usage

If using an API, you can modify the process to suit your application. Instead of sending a reset link, send the token directly and allow the user to copy and paste it into a frontend component, similar to an OTP system.

- The user submits their email.
- Instead of receiving a clickable link, they receive a token.
- They enter this token into a form, which makes a `POST` request to `/password-reset/:token`.

#### Accessing the Token in Templates

- Use the `token` variable in `reset_email.fx` to insert the token in the email.
- Use the `link` variable to insert the full reset link.

> **Tip:** This allows flexibility depending on whether you’re building a web-based or mobile authentication system.

## Options

### `messages`

Allows you to customize validation error messages as needed.

---

### `mailer`

A mailer function that you must provide to the Auth module. This is used internally to send password reset emails.

---

### `mode`

Determines whether the user should be logged in automatically after registration.

- **`smart`**: Automatically logs in the user.
- **`lazy`**: Requires the user to log in manually after registration.

---

### `subject`

The email subject for password reset emails.  
**Default**: `"Password Reset Link"`

---

### `expires`

The expiration time for the JWT (JSON Web Token), in seconds.

---

### `component`

The name of your password reset component view.  
**Default**: `"emails.reset_email"` (stored in `views/email/reset_email.fx`)  
You can modify or replace this with another component using dot notation.

---

### `base`

The base path for your password reset link.  
**Default**: `"password-reset"` (final link will be `password-reset/:token`).

---

### `createdAt`

The column name for the creation timestamp in your users table.  
**Default**: `"created_at"`  
You can update this to a custom column name or set it to `false`.

> **Note**: Disabling the creation timestamp is **not recommended**.

---

### `updatedAt`

The column name for the update timestamp in your users table.  
**Default**: `"updated_at"`  
You can update this to a custom column name or set it to `false`.

> **Note**: Disabling the update timestamp is **not recommended**.

---

### `cookie`

Cookie options for the `authToken` cookie.  
_(See cookie middleware documentation for more details on `CookieOptions`.)_
