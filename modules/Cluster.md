# Bnjsx Cluster

This module is designed to manage multiple database connection pools efficiently. It allows you to add, remove, freeze, and unfreeze pools, providing a flexible connection request distribution.

## Table of Contents

1. **[Overview](#overview)**
2. **[Requesting Connections](#requesting-connections)**
3. **[Event Handling and Logging](#event-handling-and-logging)**
4. **[Cluster Getters](#cluster-getters)**
5. **[Cluster Setters](#cluster-setters)**
6. **[Cluster Checkers](#cluster-checkers)**

## Overview

To create a `Cluster` instance, you can provide one or more `ClusterPool` instances during initialization. However, this step is optional since you can always add additional pools at runtime using the `add()` method.

### ClusterPool vs Pool

- **`ClusterPool`**: Named pools for `Cluster`.
- **`Pool`**: Anonymous pools not identified by name.

### Creating a ClusterPool

To create a `ClusterPool`, use the following parameters:

1. **`name`**: A unique string to identify the pool.
2. **`driver`**: The instance of the database driver.
3. **`options`**: Optional configuration settings.

Import `ClusterPool` from `bnjsx`

```js
const { ClusterPool } = require('bnjsx');
```

Create a `ClusterPool` instance

```js
const asia = new ClusterPool('asia', driver, options);
const africa = new ClusterPool('africa', driver, options);
```

### Creating a Cluster

You can create a `Cluster` by passing one or more `ClusterPool` instances

```js
const { Cluster } = require('bnjsx');
```

Now create a `Cluster` instance with the named pools

```js
const cluster = new Cluster(asia, africa);
```

### Adding Pools at Runtime

If you don’t want to provide pools initially, you can always add them later using the `add()` method. This allows you to add `ClusterPool` instances at runtime, ensuring flexibility in managing your cluster.

```js
cluster.add(new ClusterPool('europe', driver, options));
```

> Make sure the pool name is unique within the cluster. If a pool with the same name already exists, it will throw an error.

### Removing Pools from the Cluster

The `remove()` method allows you to remove one or more pools from the cluster. Before using this method, ensure the following:

1. **Release all connections**: Make sure all active connections are released back to the pool. If connections are not released, the removal will fail.
2. **No queued requests**: The method will fail if there are any pending connection requests in the queue.
3. **Freeze the pool**: It is highly recommended to freeze the pool for a few seconds before removing it. Freezing ensures no new requests are made to the pool, making it safe to shut down.

```js
// Remove a specific pool by name
await cluster.remove('africa_1');

// Remove multiple pools using a wildcard ('*')
await cluster.remove('asia_*');

// Remove multiple pools using a regular expression
await cluster.remove(/^asia_/);
```

### Freezing Pools

The `freeze()` method temporarily disables your pool by moving it from the active pool list to the frozen list. While frozen, no new connections can be made to the pool.

```js
// Freeze a specific pool
cluster.freeze('asia_1');

// Freeze pools using wildcard
cluster.freeze('asia_*');

// Freeze pools using RegExp
cluster.freeze(/^asia_[0-9]+$/);
```

Once frozen:

- No new connections can be requested from the pool.
- You can still remove it using the `remove()` method.

```js
// Requesting a connection after freezing will throw an error
cluster.request('asia_1');

// Attempting to get the pool will also fail
cluster.get.pool('asia_1');
```

### Unfreezing Pools

The `unfreeze()` method moves a pool back from the frozen list to the active list, making it available to handle new connection requests again.

```js
// Unfreeze a specific pool
cluster.unfreeze('asia_1');

// Unfreeze pools using a wildcard
cluster.unfreeze('asia_*');

// Unfreeze pools using a RegExp
cluster.unfreeze(/^asia_[0-9]+$/);
```

Once unfrozen:

- The pool is active again and can accept new connection requests.

### Freezing and Removing Pools Safely

Before removing a pool, especially if there are queued connection requests, it’s important to **freeze** the pool to stop new requests from coming in. This ensures a smooth removal process without errors.

1. **Ensure** you have other pools to hanlde requests.
2. **Freeze** the pool to stop new connection requests.
3. **Wait** for a few seconds.
4. **Remove** the pool safely.

Here is an example (Not Recommended)

```js
// Freeze the pool
cluster.freeze('asia_1');

// Wait for a few seconds
setTimeout(
  // Then remove the pool
  () => cluster.remove('asia_1'),
  3000
);
```

> You should build an **Admin Dashboard** with buttons for freezing, unfreezing, and removing pools providing an easy and user-friendly way to manage everything.

### Handling PendingConnections

**PendingConnections** are connections that could not be closed. When this occurs, the pool emits a `CLOSE_FAIL` event, passing the `PendingConnection` instance for you to manage manually.

To simplify this process, **Cluster** automatically handles this event and stores all `PendingConnection` instances for you. and provides utilities to manage these connections:

- Use `cluster.has.pending()` to determine if there are any pending connections that need attention.

```js
if (cluster.has.pending()) {
  console.log('There are pending connections to handle.');
}
```

- Use `cluster.closePendingConnections()` to close all pending connections.

```js
// Close all connections, keep failed ones for future attempts.
await cluster.closePendingConnections();

// Closes all connections, ignoring any errors during closure.
await cluster.closePendingConnections(true);

// Check for pending connections
if (cluster.has.pending()) {
  console.log('Attempting to close pending connections...');

  try {
    // Close pending connections
    await cluster.closePendingConnections();
    console.log('All pending connections closed successfully.');
  } catch (error) {
    console.error('Failed to close some connections:', error);
  }
}
```

### Shutting Down the Cluster

To properly shut down the cluster and ensure all resources are cleaned up, you can use the `shutdown()` method. This method performs the following:

1. Removes all pools from the cluster using `remove('*')`.
2. Closes any pending connections by calling `closePendingConnections(force)`.

Shutdown without forcing closure of pending connections

```js
cluster
  .shutdown()
  .then(() => console.log('Cluster shut down successfully.'))
  .catch((error) => console.error('Failed to shut down'));
```

Shutdown with forcing closure of pending connections

```js
cluster
  .shutdown(true) // Force
  .then(() => console.log('Cluster forcefully shut down.'))
  .catch((error) => console.error('Failed to forcefully shut down'));
```

> This ensures that the cluster is cleanly shut down and no resources are left hanging, preserving the health of your system.

## Requesting Connections

The **`request()`** method allows you to request a connection from an active pool within the cluster. You can optionally specify the pool(s) from which to request the connection. If no pool name is provided, the connection will be requested based on the current mode (either `ORDER_MODE` or `RANDOM_MODE`).

### Request a Connection from a Specific Pool

You can specify the name of the pool from which you want to request a connection. This ensures that the connection comes from the selected pool.

```js
const cluster = new Cluster();

// Add group of pools (african pools)
cluster.add(new ClusterPool('africa_1', driver));
cluster.add(new ClusterPool('africa_2', driver));

// Add group of pools (asian pools)
cluster.add(new ClusterPool('asia_1', driver));
cluster.add(new ClusterPool('asia_2', driver));

// Request a connection from a specific pool
const africa1Con = await cluster.request('africa_1');
```

### Request a Connection in Order

If no specific pool name is provided, the connection is requested based on the current mode. By default, the mode is set to `ORDER_MODE`, meaning connections will be provided in the order the pools were added.

```js
// Import ORDER_MODE
const { ORDER_MODE } = require('bnjsx');

// Set mode to ORDER_MODE (default mode)
cluster.set.mode(ORDER_MODE);

// Request a connection in order
const con = await cluster.request();
```

### Request a Connection from a Random Pool

You can change the mode to `RANDOM_MODE` if you want to request a connection from a randomly selected pool.

```js
// Import RANDOM_MODE
const { RANDOM_MODE } = require('bnjsx');

// Set mode to RANDOM_MODE
cluster.set.mode(RANDOM_MODE);

// Request a random connection
const randomCon = await cluster.request();
```

### Request a Connection from a Group of Pools in Order

You can also request connections from a group of pools using a pattern (e.g., `'africa*'` for all pools starting with `africa`). By default, the connection will be provided in order.

```js
// Import ORDER_MODE
const { ORDER_MODE } = require('bnjsx');

// Set mode to ORDER_MODE (default mode)
cluster.set.mode(ORDER_MODE);

// Request a connection from a group of pools in order
const africanCon = await cluster.request('africa*');
const asianCon = await cluster.request('asia*');
```

### Request a Random Connection from a Group of Pools

To request a random connection from a group of pools, you can switch the mode to `RANDOM_MODE`.

```js
// Import RANDOM_MODE
const { RANDOM_MODE } = require('bnjsx');

// Set mode to RANDOM_MODE
cluster.set.mode(RANDOM_MODE);

// Request a random connection from a group of pools
const randomAfricanCon = await cluster.request('africa*');
const randomAsianCon = await cluster.request('asia*');
```

### Use Regular Expressions

You can also use a regular expression to match pool names, which allows for more flexible matching of pools.

```js
// Request a connection using a regular expression
const africanCon = await cluster.request(/^africa.+$/);
const asianCon = await cluster.request(/^asia.+$/);
```

### Notes

- **Active Pools Only**: You can only request connections from active pools.
- **Mode**: By default, the mode is `ORDER_MODE`. If you want a random connection, you must set the mode to `RANDOM_MODE`.

This method provides a flexible way to request connections from specific pools or groups of pools, whether in order or randomly, with the ability to use patterns or regular expressions for more complex matching.

## Event Handling and Logging

Bnjsx's `Cluster` automatically listens for several `Pool` events, handling them in the background. Whenever an event occurs, and registers a warning message, which can be accessed later. By using the `cluster.get.warnings()` method.

In addition to warnings, `Cluster` can log error messages to a file, These logs can be useful for persistent error tracking, even if the server restarts.

### How Cluster Handles Events

By default, `Cluster` listens for the following events and registers corresponding warnings:

- `CLOSE_FAIL`: Triggered when closing a connection fails.
- `CREATE_FAIL`: Triggered when creating a connection fails.
- `MAX_CONNECTION`: Triggered when the maximum number of connections is reached.
- `MAX_QUEUE_TIME`: Triggered when the maximum request queue time is exceeded.
- `MAX_QUEUE_SIZE`: Triggered when the maximum request queue size is reached.
- `COMMIT_FAIL`: Triggered when a commit transaction fails.
- `QUERY_FAIL`: Triggered when a query execution fails.
- `ROLLBACK_FAIL`: Triggered when a rollback fails.
- `SHUTDOWN_FAIL`: Triggered when a pool shutdown fails.
- `TRANSACTION_FAIL`: Triggered when a transaction fails.

For each of these events, `Cluster` logs the associated error message to a log file (if you set up a `Logger` instance) and saves `warnings` for you.

### Using Logger Helper

If you'd like to persist these error messages in a log file, you should set up a `Logger` instance. This will allow `Cluster` to write the error messages to the specified log file.

```js
// Import Logger
const { Logger } = require('bnjsx');

// Create a new Logger instance
const logger = new Logger('./app.log');

// Set the logger instance for the cluster
cluster.set.logger(logger);
```

> Once the logger is set, `Cluster` will log error messages whenever an issue occurs related to the events mentioned above.

### Accessing Logs

You can retrieve all log messages by calling `get.messages()` on the logger instance.

```js
const logs = await cluster.get.logger().get.messages();
console.log(logs); // Outputs an array of logged messages
```

### Warnings vs Logs

- **Warnings**: These are stored in memory. If the server restarts, the warnings will be lost.
- **Logs**: These are stored in a `.log` file, meaning they persist even if the server crashes or restarts.

## Cluster Getters

The **`Getter`** interface provides methods to retrieve various details about the cluster and its pools, including the current mode, warnings, pools, and specific information about the cluster and pools.

### Methods Overview

- **`mode()`**: Retrieves the current mode of the cluster (`ORDER_MODE` or `RANDOM_MODE`).
- **`logger()`**: Retrieves the current logger instance.
- **`warnings()`**: Retrieves the list of warnings associated with the cluster.
- **`pools.frozen()`**: Retrieves the list of frozen pools within the cluster.
- **`pools.active()`**: Retrieves the list of active pools within the cluster.
- **`pool(name)`**: Retrieves a pool by name or a pool based on the current mode if no name is provided.
- **`info()`**: Retrieves detailed information about the cluster, including pool statistics and warnings.
- **`infoAbout(name)`**: Retrieves detailed information about a specific pool by name.

### Get the Current Mode of the Cluster

Use the `mode()` method to get the current select mode for the cluster, which can either be `ORDER_MODE` or `RANDOM_MODE`.

```js
const { ORDER_MODE, RANDOM_MODE } = require('bnjsx');

if (cluster.get.mode() === ORDER_MODE) {
  console.log('Pools selected in order');
}

if (cluster.get.mode() === RANDOM_MODE) {
  console.log('Pools selected randomly');
}
```

### Get the Current Logger Instance

The `logger()` method provides access to the current logger instance.

```js
console.log(cluster.get.logger());
```

### Get the List of Warnings in the Cluster

You can use the `warnings()` method to retrieve an array of warnings stored in the cluster.

```js
console.log(cluster.get.warnings());
```

### Get Frozen and Active Pools

You can use `get.pools.frozen()` and `get.pools.active()` to get the list of frozen and active pools in the cluster.

```js
console.log(cluster.get.pools.frozen()); // Frozen pools
console.log(cluster.get.pools.active()); // Active pools
```

### Get a Pool by Name or Based on the Current Mode

If you provide a name, the `pool(name)` method will return the pool matching that name. If no name is provided, the pool will be selected based on the current mode (either `ORDER_MODE` or `RANDOM_MODE`).

```js
// Import RANDOM_MODE
const { RANDOM_MODE } = require('bnjsx');

// Get a pool by name
const africa1Pool = cluster.get.pool('africa_1');

// Get a pool based on the current mode
const pool = cluster.get.pool();

// Get a random pool
cluster.set().mode(RANDOM_MODE);
const randomPool = cluster.get.pool();

// Get a pool from a group of pools
const africanPool = cluster.get.pool('africa*');
```

### Get Full Cluster Information

You can use the `info()` method to get detailed information about the cluster, including the number of active and frozen pools, warnings, and the cluster's select mode.

```js
console.log(cluster.get.info());
```

### Get Information About a Specific Pool

Use the `infoAbout(name)` method to get detailed information about a specific pool by its name. This includes the pool's ID, creation date, driver, performance, and connection counts.

```js
console.log(cluster.get.infoAbout('asia_1'));
```

## Cluster Setters

The **`Setter`** interface provides methods to configure various aspects of the cluster, such as setting the pool resolving mode, logger instance, and managing warning messages.

### Methods Overview

- **`mode(mode)`**: Sets the mode for how pools are selected (`ORDER_MODE` or `RANDOM_MODE`).
- **`logger(logger)`**: Sets a custom logger instance for the cluster.
- **`warnings(warnings)`**: Registers or clears warning messages for the cluster.

### Set the Pool Resolving Mode

Use the `mode()` method to set the mode for how pools are selected. You can choose between `ORDER_MODE` (sequential order) and `RANDOM_MODE` (random selection).

```js
// Import ORDER_MODE & RANDOM_MODE
const { ORDER_MODE, RANDOM_MODE } = require('bnjsx');

// Set the cluster to use ORDER_MODE (sequential order)
cluster.set.mode(ORDER_MODE);

// Set the cluster to use RANDOM_MODE (random order)
cluster.set.mode(RANDOM_MODE);
```

### Set a Custom Logger Instance

Use the `logger()` method to set a custom logger instance for the cluster. This allows you to log messages to a log file.

```js
// Import Logger
const { Logger } = require('bnjsx');

// Create a custom logger instance
const logger = new Logger('./app.log');

// Set the custom logger for the cluster
cluster.set.logger(logger);
```

> If the provided logger instance is invalid, an error will be thrown.

### Register or Clear Warning Messages

You can use the `warnings()` method to register one or more warning messages for the cluster or clear all existing warnings.

```js
// Register a single warning message
cluster.set.warnings('There was an issue with the connection');

// Register multiple warning messages
cluster.set.warnings(['Warning 1', 'Warning 2', 'Warning 3']);

// Clear all warning messages
cluster.set.warnings([]);
```

> If the provided messages are invalid, the method will throw an error.

## Cluster Checkers

The **`Checker`** interface provides methods to check the presence or state of various cluster properties. It helps you determine if certain resources are available or in a specific state.

### Methods Overview

- **`logger()`**: Checks if the cluster has a valid logger instance.
- **`pending()`**: Checks if there are pending connections in the cluster.
- **`warnings()`**: Checks if any warnings have been recorded for the cluster.
- **`frozen(name)`**: Checks if a frozen pool exists, optionally by name or pattern.
- **`active(name)`**: Checks if an active pool exists, optionally by name or pattern.

### Check if the Logger Exists

Use the `logger()` method to check if a logger instance is available in the cluster.

```js
if (cluster.has.logger()) console.log('Available');
else console.log('No logger found');
```

### Check for Pending Connections

The `pending()` method allows you to check if there are any pending connections in the cluster.

```js
if (cluster.has.pending()) console.log('Available');
else console.log('No pending connections');
```

### Check for Warnings

Use the `warnings()` method to check if any warnings have been recorded for the cluster.

```js
if (cluster.has.warnings()) console.log('Available');
else console.log('No warnings');
```

### Check for Frozen and Active Pools

You can use the `frozen()` and `active()` methods to check for the presence of frozen and active pools, respectively.

```js
if (cluster.has.frozen()) console.log('Available');
else console.log('No frozen pools');

if (cluster.has.active()) console.log('Available');
else console.log('No active pools');
```

### Check for a Specific Frozen or Active Pool

You can also check for a frozen or active pool by its name or pattern.

```js
// Check for a frozen pool by name
if (cluster.has.frozen('asia_1')) {
  console.log('Frozen pool "asia_1" exists');
}

// Check for an active pool by name
if (cluster.has.active('asia_1')) {
  console.log('Active pool "asia_1" exists');
}

// Check for all pools starting with "asia"
if (cluster.has.frozen(/^asia_/)) {
  console.log('There are frozen pools matching "asia_"');
}
```
