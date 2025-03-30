# Bnjsx SQLite

This module provides a simple, high-level, unified API for interacting with SQLite databases. It simplifies creating connections, executing queries, and managing transactions.

## Table of Contents

1. **[Features](#features)**
2. **[Create Connection](#create-connection)**
3. **[Execute Queries](#execute-queries)**
4. **[Close Connection](#close-connection)**
5. **[Transactions](#transactions)**
6. **[Usage Example](#usage-example)**

## Features

- Easy connection setup with SQLite databases
- Support for parameterized queries to prevent SQL injection
- Built-in transaction management
- Unified, high-level API compatible with all Bnjsx drivers
- Typescript support

## Create Connection

To start interacting with your SQLite database, you need to **create a connection**.

1. First, import `SQLite` driver from `bnjsx` to use it in your project.

```js
const { SQLite } = require('bnjsx');
```

2. Next, create an instance of `SQLite` and provide the path to your database.

```js
const driver = new SQLite('./db.sqlite');
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

Once you’ve established a connection, you can start executing SQL queries on your SQLite database.

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
    'INSERT INTO users (email, password) VALUES (?, ?);',
    ['john@example.com', 'password']
  );

  // Insert related profile
  await connection.query(
    'INSERT INTO profiles (user_id, city, age) VALUES (?, ?, ?);',
    [userId, 'Tokyo', 30]
  );

  // Commit if everything is successful
  await connection.commit();
} catch (error) {
  // Rollback if something goes wrong
  await connection.rollback();
  throw error; // Re-Throw
}
```

- `beginTransaction()`: Throws `BeginTransactionError` if there was an issue
- `commit()`: Throws `CommitTransactionError` if there was an issue
- `rollback()`: Throws `RollbackTransactionError` if there was an issue.

## Usage Example

In this example, we’ll walk through the process of creating a connection to your `SQLite` database, executing a query to fetch data from a table, and then closing the connection once you’re done. This example uses an async function to handle the asynchronous operations.

```js
// Import SQLite Driver
const { SQLite } = require('bnjsx');

// Define an async function
const app = async () => {
  // Create driver instance
  const driver = new SQLite('./db.sqlite');

  // Establish a connection to your SQLite database
  const connection = await driver.create();

  // Execute a query to fetch all records from the 'users' table
  const users = await connection.query('SELECT * FROM users;');

  // Log the result of the query (list of users)
  console.log(users);

  // Close the connection to the database
  await connection.close();

  // The connection is now closed; you should not use it anymore!
};

// Execute your app
app();
```
