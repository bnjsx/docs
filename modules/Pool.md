## Bnjsx Pool

This module is a connection manager designed to optimize database connection handling. Rather than creating and closing a new connection for each request, which can slow down performance under heavy loads, our `Pool` ensures efficient management of connections.

## Table of Contents

1. **[Overview](#overview)**
2. **[How It Works](#how-it-works)**
3. **[Using the Pool](#using-the-pool)**
4. **[Shutting Down the Pool](#shutting-down-the-pool)**
5. **[Pool Config](#pool-config)**
6. **[Pool Events](#pool-events)**
7. **[Pool Getters](#pool-getters)**
8. **[Pool Checkers](#pool-checkers)**
9. **[Additional Note](#notes-on-pool)**

## Overview

Pools are connection managers that optimizes the process of creating, reusing, and closing database connections. When your application handles a high number of requests per second (e.g., 10000), repeatedly opening and closing connections is inefficient. `Bnjsx` solves this problem by maintaining a pool of connections, which can be reused for multiple requests.

Instead of opening a new connection for every request, our `Pool` creates connections when needed and stores them for future use. If a connection is not used for a specific amount of time, it is closed automatically.

## How It Works

Bnjsx's `Pool` module operates with two types of connections:

- **Acquired connections**: Connections that are currently in use.
- **Idle connections**: Connections that are available for use.

When you need a connection, the pool checks if there are any idle connections available. If so, it provides one. If there are no idle connections, our pool automatically creates a new one. When you’re done with the connection, you release it back to the pool for future use. This eliminates the overhead of creating and closing connections for every request.

## Using the Pool

### Create a Driver Instance

Start by creating an instance of the database driver (e.g., MySQL) with your connection configuration:

```js
// Import MySQL driver
const { MySQL } = require('bnjsx');

// Creating a MySQL driver instance
const driver = new MySQL({
  host: 'localhost',
  user: 'root',
  password: 'root',
  db: 'test',
});
```

### Create a Pool Instance

Next, create a pool instance using your driver:

```js
// Import Pool
const { Pool } = require('bnjsx');

// Creating a Pool instance using a MySQL driver
const pool = new Pool(driver);
```

### Requesting a Connection

To request a connection from the pool, use the `request` method:

```js
pool
  .request() // Request a connection
  .then((con) => con.query('SELECT * FROM users;')) // Execute an SQL query
  .then((result) => console.log(result)); // Log the result
```

Alternatively, using `async/await` syntax:

```js
const con = await pool.request(); // Request a connection
const result = await con.query('SELECT * FROM users'); // Execute an SQL query
console.log(result); // Log the result
```

If you just want to execute a single query, you can do this:

```js
await pool
  .query('SELECT * FROM users') // Execute an SQL query
  .then((result) => console.log(result)); // Log the result
```

## Shutting Down the Pool

If you no longer need the pool, you can shut it down. This will close all connections and release the resources:

```js
pool
  .shutdown()
  .then(() => console.log('Pool has been shut down successfully.'))
  .catch((error) => console.log(error));
```

After shutting down, any further attempts to use the pool will result in a rejection:

```js
// This will reject because the pool has been shut down
await pool.request();
```

## Pool Config

You can configure the pool with a set of options to control how connections are managed. Here’s an explanation of each option along with a practical example of how to use them.

1. **`maxConnection`**: Defines the maximum number of connections that can be created in the pool (default: `10`).

```js
// Create a Pool instance with a max of 5 connections
const pool = new Pool(driver, { maxConnection: 5 });
```

> This configuration ensures that no more than 5 connections are created at any given time.

2. **`maxIdleTime`**: Sets the lifetime (in milliseconds) for idle connections. If a connection remains idle for longer than this time, it will be closed automatically (default: `60000`).

```js
// Create a Pool instance where idle connections will be closed after 2 minutes
const pool = new Pool(driver, { maxIdleTime: 120000 });
```

> This means that if a connection is idle for 2 minutes, it will be closed to free up resources.

3. **`shouldQueue`**: Determines whether connection requests should be queued when there are no idle connections available. If set to `true`, the request will be placed in a queue until an idle connection is available (default: `true`).

```js
// Create a Pool instance with request queuing disabled
const pool = new Pool(driver, { shouldQueue: false });
```

> With `shouldQueue: false`, if there are no idle connections, the request will fail immediately.

4. **`maxQueueSize`**: Limits the number of connection requests that can be queued when there are no idle connections available (default: `Infinity`).

```js
// Create a Pool instance with a queue size limit of 5
const pool = new Pool(driver, { maxQueueSize: 5 });
```

> This limits the queue to 5 requests. If there are more than 5 requests while the pool is exhausted, the extra requests will be rejected.

5. **`maxQueueTime`**: Sets the maximum amount of time (in milliseconds) a request can remain in the queue before it is rejected (default: `1000`).

```js
// Create a Pool instance where queued requests will be rejected after 2 seconds
const pool = new Pool(driver, { maxQueueTime: 2000 });
```

> With this setting, any request that has been in the queue for longer than 2 seconds will be rejected.

6. **`shouldRetry`**: Determines whether failed connection creation or closure attempts should be retried (default: `true`).

```js
// Create a Pool instance that disables retrying on connection failure
const pool = new Pool(driver, { shouldRetry: false });
```

> Setting this to `false` means that if a connection creation or closure fails, it will not be retried.

7. **`maxRetry`**: Specifies the maximum number of retry attempts to make when connection creation or closure fails (default: `3`).

```js
// Create a Pool instance that will retry up to 5 times on failure
const pool = new Pool(driver, { maxRetry: 5 });
```

> This ensures that if connection creation fails, the system will try up to 5 times before giving up.

8. **`retryDelay`**: Defines the delay (in milliseconds) between retry attempts when a failure occurs (default: `500`).

```js
// Create a Pool instance that waits 1 second between retries
const pool = new Pool(driver, { retryDelay: 1000 });
```

> This sets a 1-second delay between retry attempts when a failure happens.

9. **`extraDelay`**: Adds extra time (in milliseconds) between retry attempts. This increases the delay incrementally with each retry (default: `500`).

```js
// Create a Pool instance with additional delay between retries
const pool = new Pool(driver, { extraDelay: 1000 });
```

> This ensures that the delay between retries increases by an additional second for each retry attempt.

Here’s how you can configure a **Pool** with all the options set:

```js
const pool = new Pool(driver, {
  // Max number of connections in the pool
  maxConnection: 15,

  // Idle connections will be closed after 3 minutes
  maxIdleTime: 180000,

  // Queue requests when no idle connections are available
  shouldQueue: true,

  // Max 10 requests can be queued
  maxQueueSize: 10,

  // Reject requests if they stay in the queue for more than 3 seconds
  maxQueueTime: 3000,

  // Retry failed connection creation/closure
  shouldRetry: true,

  // Retry up to 5 times on failure
  maxRetry: 5,

  // Wait 1 second between retries
  retryDelay: 1000,

  // Add an additional 500ms delay after each retry
  extraDelay: 500,
});
```

In this example, you have customized all the connection pool options, which would suit a high-traffic environment where connection reuse is critical but failure resilience and queue handling are also important.

## Pool Events

In Bnjsx's Pool, several events are emitted during the execution of various operations such as querying, connection management, transaction handling, and pool shutdown. Developers can listen to these events to handle success or failure scenarios.

Below are examples of how to handle these events:

```js
const { QUERY_FAIL, QUERY_SUCCESS } = require('bnjsx');

// Handle when a query fails
pool.on(QUERY_FAIL, (error) => {
  console.log('Query execution failed:', error);
});

// Handle when a query succeeds
pool.on(QUERY_SUCCESS, (result) => {
  console.log('Query executed successfully:', result);
});
```

```js
const { CREATE_FAIL, CREATE_SUCCESS } = require('bnjsx');

// Handle when a connection creation fails
pool.on(CREATE_FAIL, (error) => {
  console.log('Connection creation failed:', error);
});

// Handle when a connection creation succeeds
pool.on(CREATE_SUCCESS, (connection) => {
  console.log('Connection created successfully:', connection);
});
```

```js
const { CLOSE_FAIL, CLOSE_SUCCESS } = require('bnjsx');

// Handle when closing a connection fails
pool.on(CLOSE_FAIL, (connection, error) => {
  console.log('Connection closing failed for', connection, 'Error:', error);
});

// Handle when closing a connection succeeds
pool.on(CLOSE_SUCCESS, () => {
  console.log('Connection closed successfully');
});
```

> You must handle `CLOSE_FAIL` event and store `PendingConnection` for later closing attempts.
> This is automatically managed in Bnjsx `Cluster`.

```js
const { TRANSACTION_FAIL, TRANSACTION_SUCCESS } = require('bnjsx');

// Handle when a transaction fails
pool.on(TRANSACTION_FAIL, (error) => {
  console.log('Transaction failed:', error);
});

// Handle when a transaction succeeds
pool.on(TRANSACTION_SUCCESS, () => {
  console.log('Transaction started successfully');
});
```

```js
const { COMMIT_FAIL, COMMIT_SUCCESS } = require('bnjsx');

// Handle when committing a transaction fails
pool.on(COMMIT_FAIL, (error) => {
  console.log('Transaction commit failed:', error);
});

// Handle when committing a transaction succeeds
pool.on(COMMIT_SUCCESS, () => {
  console.log('Transaction committed successfully');
});
```

```js
const { ROLLBACK_FAIL, ROLLBACK_SUCCESS } = require('bnjsx');

// Handle when rolling back a transaction fails
pool.on(ROLLBACK_FAIL, (error) => {
  console.log('Transaction rollback failed:', error);
});

// Handle when rolling back a transaction succeeds
pool.on(ROLLBACK_SUCCESS, () => {
  console.log('Transaction rolled back successfully');
});
```

```js
const { SHUTDOWN_FAIL, SHUTDOWN_SUCCESS } = require('bnjsx');

// Handle when the shutdown process of the pool fails
pool.on(SHUTDOWN_FAIL, (error) => {
  console.log('Pool shutdown failed:', error);
});

// Handle when the shutdown process of the pool succeeds
pool.on(SHUTDOWN_SUCCESS, () => {
  console.log('Pool shut down successfully');
});
```

```js
const { MAX_QUEUE_TIME } = require('bnjsx');

// Handle when the maximum allowed queue time is reached
pool.on(MAX_QUEUE_TIME, (error) => {
  console.log('Max queue time reached:', error);
});
```

```js
const { MAX_QUEUE_SIZE } = require('bnjsx');

// Handle when the maximum allowed queue size is reached
pool.on(MAX_QUEUE_SIZE, (error) => {
  console.log('Max queue size reached:', error);
});
```

```js
const { MAX_CONNECTION } = require('bnjsx');

// Handle when the maximum number of allowed connections is reached
pool.on(MAX_CONNECTION, (error) => {
  console.log('Max connections reached:', error);
});
```

```js
const { RELEASED } = require('bnjsx');

// Handle when a connection is released back to the pool
pool.on(RELEASED, (connection) => {
  console.log('Connection released back to the pool:', connection);
});
```

## Pool Getters

The **Pool Getters** provides an easy-to-use interface for accessing real-time information about the connection pool's state, configuration, and performance. This enables you to monitor and adjust the pool’s behavior effectively.

1. **`get.count.idle()`**: Returns the number of idle (unused) connections in the pool.

```js
console.log(`Idle: ${pool.get.count.idle()}`);
```

2. **`get.count.acquired()`**: Returns the number of active (acquired) connections in the pool.

```js
console.log(`Acquired: ${pool.get.count.acquired()}`);
```

3. **`get.count.request()`**: Returns the number of requests currently queued in the pool.

```js
console.log(`Requests: ${pool.get.count.request()}`);
```

4. **`get.options()`**: Returns the current configuration options of the pool.

```js
console.log('Options:', pool.get.options());
```

5. **`get.driver()`**: Returns the database driver used by the pool.

```js
console.log('Driver:', pool.get.driver());
```

6. **`get.performance()`**: Evaluates the performance of the pool based on the number of active connections.
   - **`GREAT_POOL`**: Less than 50% of `maxConnection` in use.
   - **`GOOD_POOL`**: Around 50% of `maxConnection` in use.
   - **`BAD_POOL`**: More than 80% of `maxConnection` in use.

```js
const { GREAT_POOL, GOOD_POOL, BAD_POOL } = require('bnjsx');

const performance = pool.get.performance();

if (performance === GREAT_POOL) console.log('GREAT!');
else if (performance === GOOD_POOL) console.log('GOOD!');
else if (performance === BAD_POOL) console.log('BAD!');
```

## Pool Checkers

The **Pool Checkers** provides a quick way to verify specific conditions or states within the connection pool. These methods help ensure that the pool is operating as expected or to make decisions based on its current status.

1. **`has.acquired()`**: Checks if the pool has any active (acquired) connections.
   - **Returns**: `true` if there are acquired connections, otherwise `false`.

```js
if (pool.has.acquired()) console.log('The pool has acquired connections.');
else console.log('No connections are currently acquired.');
```

2. **`has.idle()`**: Checks if the pool has any idle (unused) connections available.
   - **Returns**: `true` if there are idle connections, otherwise `false`.

```js
if (pool.has.idle()) console.log('The pool has idle connections available.');
else console.log('No idle connections in the pool.');
```

3. **`has.request()`**: Checks if there are any connection requests queued in the pool.
   - **Returns**: `true` if there are queued requests, otherwise `false`.

```js
if (pool.has.request()) console.log('There are connection requests.');
else console.log('No connection requests are queued.');
```

## Additional Note

- Bnjsx's `Pool` module automatically manages connections by creating new ones when needed and closing idle ones after a set period.
- Always release connections back to the pool using the `release()` method when you’re done with them. Once released, the connection cannot be reused.
- After shutting down the pool, it becomes a dummy pool and can no longer process connection requests.
- Before shutting down the pool, ensure all connections are released and there are no pending requests in the queue. Freeze the pool first to prevent new requests.
- You can use Bnjsx's `Cluster` module to handle pool freezing and shutdown or implement your own logic.
- If you choose not to use our `Cluster`, handle the `CLOSE_FAIL` event and store `PendingConnection` for retrying closure later.
- The main difference between a driver-created connection (`Connection`) and a pool-created connection (`PoolConnection`) is the way they are closed:
  - **Connection** uses the `close()` method.
  - **PoolConnection** uses the `release()` method.
- Both types of connections can execute queries and manage transactions.
- A `PendingConnection` is a connection that could not be closed by the pool for some reason (e.g., internet issues, server restarts). In this case, the `Pool` emits a `CLOSE_FAIL` event and provides these connections for you to store somewhere and attempt to close them when you are free. You can implement this logic yourself or simply use our `Cluster` for that.
