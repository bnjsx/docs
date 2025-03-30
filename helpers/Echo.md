# Bnjsx Echo

This module is designed to provide robust retry mechanisms for your operations, such as network requests or database queries. It allows you to configure retry logic with customizable delays and incremental backoffs.

## Creating an Instance

To start using `Echo`, create an instance with optional configurations:

```js
const { Echo } = require('bnjsx');

// Create an Echo instance with:
// - maxRetry: 10 (maximum retries)
// - retryDelay: 1000 ms (delay between retries)
// - extraDelay: 500 ms (incremental delay added after each retry)
const echo = new Echo(10, 1000, 500);
```

## Retrying Jobs

The `retry` method executes a job and retries it according to the configuration.

```js
const fetchSuccess = () => {
  return new Promise((resolve) => {
    setTimeout(() => resolve('Success'), 100);
  });
};

// Executes the job and resolves on the first attempt after 1000 ms.
echo.retry(fetchSuccess).then((result) => console.log(result));
```

```js
const fetchFail = () => {
  return new Promise((_, reject) => {
    setTimeout(() => reject(new Error('Fail')), 100);
  });
};

// Retries the job 10 times before rejecting.
// - Each retry waits 1000 ms (10 seconds total for 10 retries).
// - An additional 500 ms is added incrementally after each retry.
echo.retry(fetchFail).catch((error) => console.log(error.message));
```

## Setters

You can configure `Echo` using the following methods:

- **`set.maxRetry(number)`**: Sets the maximum number of retries. Must be an integer greater than 0.
- **`set.retryDelay(number)`**: Sets the delay (in milliseconds) between retries. Must be an integer greater than 0.
- **`set.extraDelay(number)`**: Adds an incremental delay (in milliseconds) after each retry. Must be an integer greater than or equal to 0.

```js
echo.set.maxRetry(5); // Retry job 5 times
echo.set.retryDelay(1000); // After 1000 ms
echo.set.extraDelay(200); // Add 200 ms after every retry attempt

// The first retry after: 1000 ms
// The second retry after: 1000 ms + 200 ms
// The third retry after: 1000 ms + 200 ms + 200 ms
// And so on...
```

## Getters

Retrieve current configurations:

- **`get.maxRetry()`**: Returns the current `maxRetry` value.
- **`get.retryDelay()`**: Returns the current `retryDelay` value.
- **`get.extraDelay()`**: Returns the current `extraDelay` value.

```js
console.log(echo.get.maxRetry()); // 5
console.log(echo.get.retryDelay()); // 1000
console.log(echo.get.extraDelay()); // 200
```

## API

- **`constructor(maxRetry, retryDelay, extraDelay)`**: Initializes a new `Echo` instance.

  - **`maxRetry`**: Maximum number of retries. Default: 3.
  - **`retryDelay`**: Delay between retries (in milliseconds). Default: 500.
  - **`extraDelay`**: Additional delay added incrementally after each retry (in milliseconds). Default: 0.

- **`retry(job)`**: Retries a given function for the configured number of attempts.

  - **`job`**: A sync/async function to retry.
  - **Returns**: A Promise that resolves with the result of the `job` or rejects with the error if all retries fail.

- **`Echo.sleep(delay)`**: Delays execution for the specified duration.

  - **`delay`**: Delay time in milliseconds.
  - **Returns**: A Promise that resolves after the delay.

- **`set`**: Setter instance.

  - **`set.maxRetry(maxRetry: number)`**: Sets the maximum number of retries.
  - **`set.retryDelay(retryDelay: number)`**: Sets the delay between retries.
  - **`set.extraDelay(extraDelay: number)`**: Sets the additional incremental delay.

- **`get`**: Getter instance.

  - **`get.maxRetry()`**: Gets the current `maxRetry` value.
  - **`get.retryDelay()`**: Gets the current `retryDelay` value.
  - **`get.extraDelay()`**: Gets the current `extraDelay` value.
