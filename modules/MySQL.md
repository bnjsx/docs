# Bnjsx MySQL

This module provides a simple, high-level, unified API for interacting with MySQL databases. It simplifies creating connections, executing queries, and managing transactions.

## Table of Contents

1. **[Features](#features)**
2. **[Create Connection](#create-connection)**
3. **[Execute Queries](#execute-queries)**
4. **[Close Connection](#close-connection)**
5. **[Transactions](#transactions)**
6. **[Usage Example](#usage-example)**
7. **[Driver Options](#driver-options)**

## Features

- Easy connection setup with MySQL databases
- Support for parameterized queries to prevent SQL injection
- Built-in transaction management
- Simple, high-level, unified API for all `Bnjsx` drivers
- Typescript support

## Create Connection

To start interacting with your MySQL database, you need to **create a connection**.

1. First, import `MySQL` driver from `bnjsx` to use it in your project.

```js
const { MySQL } = require('bnjsx');
```

2. Next, create an instance of `MySQL` and provide the necessary database configuration.

```js
const driver = new MySQL({
  database: 'test', // The name of the database you're connecting to
  user: 'root', // The username to access the database
  password: 'root', // The password for the database user
  host: 'localhost', // The host where the MySQL server is running
});
```

3. Finally, use the `create()` method to establish a connection to the database.

```js
driver
  .create()
  .then((result) => console.log(result)) // `Connection`
  .catch((error) => console.log(error)); // Handles errors
```

> Throws a `CreateConnectionError` if there was an issue creating the connection.

## Execute Queries

Once you’ve established a connection, you can start executing SQL queries on your MySQL database.

1. For select queries, the result is an array of objects representing the rows from the query. Each object corresponds to a row, with the column names as keys.

```js
connection
  .query('SELECT * FROM users;')
  .then((result) => console.log(result)) // [{name: 'John', id: 1}, ...]
  .catch((error) => console.log(error)); // Handles errors
```

2. For inserting a single row, the result will contain the inserted row’s ID. This ID is the auto-incremented value for the primary key, for example.

```js
const data = ['user1@gmail.com', 'pass1'];
connection
  .query('INSERT INTO users (email, password) VALUES (?, ?);', data)
  .then((result) => console.log(result)) // Inserted ID
  .catch((error) => console.log(error)); // Handles errors
```

3. When inserting multiple rows, the result will typically be undefined because no specific data is returned for bulk inserts.

```js
const data = ['user2@gmail.com', 'pass2', 'user3@gmail.com', 'pass3'];
connection
  .query('INSERT INTO users (email, password) VALUES (?, ?), (?, ?);', data)
  .then((result) => console.log(result)) // `undefined`
  .catch((error) => console.log(error)); // Handles errors
```

4. For updates, the result will generally be undefined when the operation is successful.

```js
const data = ['updated_email@example.com', 22];
connection
  .query('UPDATE users SET email = ? WHERE id = ?;', data)
  .then((result) => console.log(result)) // `undefined`
  .catch((error) => console.log(error)); // Handles errors
```

5. Similar to the update query, the result will be undefined after a successful delete operation. You won’t receive any data back.

```js
const data = [33];
connection
  .query('DELETE FROM users WHERE id = ?;', data)
  .then((result) => console.log(result)) // `undefined`
  .catch((error) => console.log(error)); // Handles errors
```

> For queries like `CREATE TABLE` or `DROP TABLE`, the result will be `undefined`, since no specific data is returned.

## Close Connection

Always **close the connection** after you're done using it. This is important because it frees up resources and prevents problems like memory leaks.

```js
connection
  .close()
  .then((r) => console.log(r)) // `undefined`
  .catch((e) => console.log(e)); // Handles errors
```

> Throws a `CloseConnectionError` if there was an issue closing the connection.

## Transactions

A **transaction** ensures that a group of database operations is treated as a single unit. Either **all operations succeed** (commit), or **none of them** are applied (rollback). This helps maintain data integrity.

```js
// Begin transaction
await connection.beginTransaction();

try {
  // Insert user
  const userId = await connection.query(
    'INSERT INTO users (email, password) VALUES (?, ?)',
    ['john@example.com', 'password']
  );

  // Insert related profile
  await connection.query(
    'INSERT INTO profiles (user_id, city, age) VALUES (?, ?, ?)',
    [userId, 'Tokyo', 30]
  );

  // Commit if everything is successful
  await connection.commit();
} catch (error) {
  // Rollback if something goes wrong
  await connection.rollback();
}
```

- `beginTransaction()`: Throws `BeginTransactionError` if there was an issue
- `commit()`: Throws `CommitTransactionError` if there was an issue
- `rollback()`: Throws `RollbackTransactionError` if there was an issue.

## Usage Example

In this example, we’ll walk through the process of creating a connection to your `MySQL` database, executing a query to fetch data from a table, and then closing the connection once you’re done. This example uses an async function to handle the asynchronous operations.

```js
// Import MySQL Driver
const { MySQL } = require('bnjsx');

// Define an async function
const app = async () => {
  // Create driver instance with database configuration
  const driver = new MySQL({
    database: 'test', // The database name
    user: 'root', // MySQL username
    password: 'root', // MySQL password
    host: 'localhost', // Database host
  });

  // Establish a connection to your MySQL database
  const connection = await driver.create();

  // Execute a query to fetch all records from the 'users' table
  const users = await connection.query('SELECT * FROM users');

  // Log the result of the query (list of users)
  console.log(users);

  // Close the connection to the database
  await connection.close();

  // The connection is now closed; you should not use it anymore!
};

// Execute your app
app();
```

## Driver Options

1. **`user?: string`**

   - Specifies the MySQL username for authentication.

2. **`password?: string`**

   - The password associated with the MySQL username provided above.

3. **`password1?: string`**

   - An alias for the MySQL user password, useful in scenarios with multi-factor authentication.

4. **`password2?: string`**

   - The second factor authentication password, required if the MySQL user account uses multi-factor authentication (MFA).

5. **`password3?: string`**

   - The third factor authentication password, required if the MySQL user account uses two additional authentication methods.

6. **`database?: string`**

   - The name of the database to use for the connection. If not specified, it connects to the default database.

7. **`charset?: string`**

   - The character set for the connection. It defines the encoding to use for storing and comparing text. Defaults to `'UTF8_GENERAL_CI'`.

8. **`host?: string`**

   - The host where the MySQL database is located. Defaults to `'localhost'`.

9. **`port?: number`**

   - The port number to connect to. Default is `3306`, which is the standard MySQL port.

10. **`localAddress?: string`**

    - Specifies the source IP address to use for the TCP connection. This is useful if you have multiple network interfaces.

11. **`socketPath?: string`**

    - If provided, this specifies the path to a Unix domain socket for connecting to MySQL. This takes priority over the `host` and `port` settings.

12. **`flags?: Array<string>`**

    - A list of custom connection flags. You can use this to modify the default connection behavior or blacklist certain default flags.

13. **`ssl?: string | SslOptions`**

    - Specifies the SSL configuration. You can provide an SSL profile name as a string, or use an object with detailed SSL options.

14. **`bigNumberStrings?: boolean`**
    - If set to `true`, all `BIGINT` values are retrieved as strings instead of numbers, which is useful to avoid precision loss with large values.

These options allow you to customize your MySQL connection according to your specific requirements, including multi-factor authentication, SSL configuration, and advanced connection settings.
