# Bnjsx PostgreSQL

This module provides a simple, high-level, unified API for interacting with PostgreSQL databases. It simplifies creating connections, executing queries, and managing transactions.

## Table of Contents

1. **[Features](#features)**
2. **[Create Connection](#create-connection)**
3. **[Execute Queries](#execute-queries)**
4. **[Close Connection](#close-connection)**
5. **[Transactions](#transactions)**
6. **[Usage Example](#usage-example)**
7. **[Driver Options](#driver-options)**

## Features

- Easy connection setup with PostgreSQL databases
- Support for parameterized queries to prevent SQL injection
- Built-in transaction management
- Simple, high-level, unified API for all Bnjsx drivers
- Typescript support

## Create Connection

To start interacting with your PostgreSQL database, you need to **create a connection**.

1. First, import `PostgreSQL` driver from `bnjsx` to use it in your project.

```js
const { PostgreSQL } = require('bnjsx');
```

2. Next, create an instance of `PostgreSQL` and provide the necessary database configuration.

```js
const driver = new PostgreSQL({
  database: 'test', // The name of the database you're connecting to
  user: 'postgres', // The username to access the database
  password: 'postgres', // The password for the database user
  host: 'localhost', // The host where the PostgreSQL server is running
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

Once you’ve established a connection, you can start executing SQL queries on your PostgreSQL database.

1. For select queries, the result is an array of objects representing the rows from the query. Each object corresponds to a row, with the column names as keys.

```js
connection
  .query('SELECT * FROM users;')
  .then((result) => console.log(result)) // [{name: 'John', id: 1}, ...]
  .catch((error) => console.log(error)); // Handles errors
```

2. For inserting a single row, the result will be `undefined` unless you specify the columns to return.

```js
const data = ['user1@gmail.com', 'pass1'];
connection
  .query('INSERT INTO users (email, password) VALUES (?, ?);', data)
  .then((result) => console.log(result)) // `undefined`
  .catch((error) => console.log(error)); // Handles errors
```

```js
const data = ['user2@gmail.com', 'pass2'];
connection
  .query(
    'INSERT INTO users (email, password) VALUES (?, ?) RETURNING id;',
    data
  )
  .then((result) => console.log(result)) // {id: 2}
  .catch((error) => console.log(error)); // Handles errors
```

3. When inserting multiple rows, the result will be `undefined` unless you specify the columns to return.

```js
const data = ['user3@gmail.com', 'pass3', 'user4@gmail.com', 'pass4'];
connection
  .query(
    'INSERT INTO users (email, password) VALUES (?, ?), (?, ?) RETURNING id;',
    data
  )
  .then((result) => console.log(result)) // [{id:3}, {id: 4}]
  .catch((error) => console.log(error)); // Handles errors
```

4. For updates, the result will generally be `undefined` when the operation is successful.

```js
const data = ['updated_email@example.com', 22];
connection
  .query('UPDATE users SET email = ? WHERE id = ?;', data)
  .then((result) => console.log(result)) // `undefined`
  .catch((error) => console.log(error)); // Handles errors
```

5. Similar to the update query, the result will be `undefined` after a successful delete operation.

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
  const { id } = await connection.query(
    'INSERT INTO users (email, password) VALUES (?, ?) RETURNING id;',
    ['john@example.com', 'password']
  );

  // Insert related profile
  await connection.query(
    'INSERT INTO profiles (user_id, city, age) VALUES (?, ?, ?);',
    [id, 'Tokyo', 30]
  );

  // Commit if everything is successful
  await connection.commit();
} catch (error) {
  // Rollback if something goes wrong
  await connection.rollback();
  throw error; // Re-throw
}
```

- `beginTransaction()`: Throws `BeginTransactionError` if there was an issue
- `commit()`: Throws `CommitTransactionError` if there was an issue
- `rollback()`: Throws `RollbackTransactionError` if there was an issue.

## Usage Example

In this example, we’ll walk through the process of creating a connection to your `PostgreSQL` database, executing a query to fetch data from a table, and then closing the connection once you’re done. This example uses an async function to handle the asynchronous operations.

```js
// Import PostgreSQL Driver
const { PostgreSQL } = require('bnjsx');

// Define an async function
const app = async () => {
  // Create driver instance with database configuration
  const driver = new PostgreSQL({
    database: 'test', // The database name
    user: 'postgres', // PostgreSQL username
    password: 'postgres', // PostgreSQL password
    host: 'localhost', // Database host
  });

  // Establish a connection to your PostgreSQL database
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

- **`user`**  
  The username used for authentication with the PostgreSQL server.  
  **Type:** `string | undefined`

- **`database`**  
  The name of the PostgreSQL database to connect to.  
  **Type:** `string | undefined`

- **`password`**  
  The password for the user. Can also be a function that returns the password or a promise for the password.  
  **Type:** `string | (() => string | Promise<string>) | undefined`

- **`port`**  
  The port number on which the PostgreSQL server is listening. Defaults to `5432`.  
  **Type:** `number | undefined`

- **`host`**  
  The host address of the PostgreSQL server.  
  **Type:** `string | undefined`

- **`ssl`**  
  SSL settings for the connection. Can be a boolean or an object with specific SSL configuration options.  
  **Type:** `boolean | ConnectionOptions | undefined`

- **`bigNumberStrings`**  
  When set to `true`, `BIGINT` and `NUMERIC` values are returned as strings to avoid precision loss.  
  **Type:** `boolean | undefined`
