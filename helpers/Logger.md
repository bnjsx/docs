# Bnjsx Logger

This module is designed for managing log files, with support for logging messages, retrieving entries, filtering by date, and more.

## Features

- Retrieve all logs as objects or plain messages.
- Filter logs by a specific date.
- Clear log file content.
- Simple and intuitive API.

## Usage

Import Logger:

```js
const { Logger } = require('bnjsx');
```

Create an instance of the Logger class and specify the path to your log file:

```js
const logger = new Logger('./app.log');
```

Log a message to your log file:

```js
await logger.log('This is a log message');
```

Retrieve all log entries as an array of objects:

```js
const logs = await logger.get.logs();
console.log(logs);
// Output: [{ date: '2024-10-12 12:34:56', message: 'This is a log message' }, ...]
```

Retrieve all log messages as plain strings:

```js
const messages = await logger.get.messages();
console.log(messages);
// Output: ['This is a log message', ...]
```

Retrieve log messages starting from a specific date:

```js
const messages = await logger.get.from('2024-10-12 00:00:00');
console.log(messages);
// Output: ['Log message from 2024-10-12', ...]
```

Clear all log entries in the log file:

```js
await logger.clear();
```

## API

- **`constructor(path)`**: Initializes a new `Logger` instance.

  - **`path`**: The file path where logs are stored. Must be a valid string. If the file does not exist, it will be created automatically.

- **`log(message)`**: Logs a message with a UTC timestamp.

  - **`message`**: The message to log. Must be a string.

- **`clear()`**: Clears all log entries in the file.

- **`get.path()`**: Returns the file path of the log file.

- **`get.logs()`**: Returns all log entries as an array of objects with `date` and `message` properties.

- **`get.messages()`**: Returns all log messages as plain strings.

- **`get.from(date)`**: Returns log messages from a specific date onward.

  - **`date`**: A string in the format `YYYY-MM-DD HH:MM:SS`.

## Notes

- Logs are stored in the format: `[YYYY-MM-DD HH:MM:SS] message`.
- The log file is automatically created if it does not exist at the specified path.
- All dates and times are in UTC.
