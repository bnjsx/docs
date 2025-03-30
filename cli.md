## Bnjsx CLI

Bnjsx CLI is a powerful command-line interface designed to streamline development with Bnjsx. It provides built-in commands for managing models, seeders, generators, and database operations, as well as support for defining and executing custom commands.

While this section focuses on how to define, register, and execute custom commands, explanations for specific built-in commands can be found in their respective module documentation

## Built-in Commands

Below is the full list of built-in commands:

| **Command**                       | **Description**                                 |
| --------------------------------- | ----------------------------------------------- |
| `node exec -v`                    | Outputs Bnjsx's current version.                |
| `node exec mk:command <!name>`    | Adds a command file.                            |
| `node exec mk:cmd <!name>`        | Shortcut for `mk:command`.                      |
| `node exec mk:model <!table>`     | Adds a model file for a table.                  |
| `node exec mk:seeder <!table>`    | Adds a seeder file for a table.                 |
| `node exec mk:generator <!table>` | Adds a generator file for a table.              |
| `node exec mk:gen <!table>`       | Shortcut for `mk:generator`.                    |
| `node exec mk:for <!table>`       | Adds generator, seeder, and model files.        |
| `node exec rm:command <!name>`    | Removes a command file.                         |
| `node exec rm:cmd <!name>`        | Shortcut for `rm:command`.                      |
| `node exec rm:model <!table>`     | Removes a model file.                           |
| `node exec rm:seeder <!table>`    | Removes a seeder file.                          |
| `node exec rm:generator <!table>` | Removes a generator file.                       |
| `node exec rm:gen <!table>`       | Shortcut for `rm:generator`.                    |
| `node exec rm:for <!table>`       | Removes generator, seeder, and model files.     |
| `node exec generate`              | Executes generators and creates tables.         |
| `node exec gen`                   | Shortcut for `generate`.                        |
| `node exec rollback`              | Executes generators and drops tables.           |
| `node exec roll`                  | Shortcut for `rollback`.                        |
| `node exec reset`                 | Drops all your tables.                          |
| `node exec seed <?table>`         | Seeds all tables or a specific table.           |
| `node exec clear <?table>`        | Clears all tables or a specific table.          |
| `node exec fetch <!table> <?id>`  | Fetches data from a table, with an optional ID. |

## Custom Commands

You can extend `Command` to create custom commands for your application. Here's a simple step-by-step guide to define custom commands like a pro:

### Adding New Command

To create a new command, run the following in your terminal:

```bash
node exec mk:cmd GreetCommand
```

This creates a new file at: `./commands/GreetCommand.js`

### Template Overview

When you open the `GreetCommand.js` file, you'll see this:

```js
const { Command } = require('bnjsx');

class GreetCommand extends Command {
  static syntax = '';
  static exec() {
    // Your command logic here...
  }
}

module.exports = { GreetCommand };
```

This is a basic template for building a command. Let's break it down:

1. `syntax`: Define what arguments or options your command accepts.
2. `exec()`: Implement the logic that runs when the command is executed.

### Command Syntax

The `syntax` property lets you describe the inputs for your command. Inputs can be:

- Required arguments: `<! name>`
- Optional arguments: `<? name>`
- Options (flags): `<- name>`

For example, let’s make a command that says:

- `"Hello There!"` if no name is given.
- `"Hello Name!"` if a name is provided.

Since the name is optional, the syntax is:

```js
syntax = '<? name>';
```

### Command Logic

The `exec()` method contains the code that runs when the command is executed.  
Use `this.argument(name)` to get the value of arguments defined in the `syntax`. For example:

```js
exec() {
  // Fetch the optional argument 'name'
  const name = this.argument('name');

  // If a name is provided
  if (name) console.log(`Hello ${name}!`);

  // If no name is provided
  else console.log('Hello There!');
}
```

### Command Registration

To make your command executable, register it in the `exec.js` file in your project’s root:

1. Import the command:

```js
const { GreetCommand } = require('./commands/GreetCommand');
```

2. Register it with a name:

```js
register('greet', GreetCommand);
```

Your `exec.js` file will look like this:

```js
// Import modules
const { register, execute } = require('bnjsx');
const { GreetCommand } = require('./commands/GreetCommand');

// Register commands
register('greet', GreetCommand);

// Execute commands
execute()
  .then(() => process.exit())
  .catch((error) => console.error(error.message));
```

### Command Execution

Run your command from the terminal:

```bash
node exec greet
# Outputs: "Hello There!"

node exec greet john
# Outputs: "Hello john!"
```

### Command Options

To add options (flags) to your command, include them in the syntax using `<- name>` and retrieve them with `this.option(name)`. Let’s add a `-lazy` option:

- If `-lazy` is used, the command says `"Hi!"` or `"Hi Name!"` instead of `"Hello"`.

```js
syntax = '<? name> <- lazy>';
```

```js
exec() {
  const name = this.argument('name');

  // Check if the -lazy option is used
  const lazy = this.option('lazy');

  if (lazy && name) console.log(`Hi ${name}!`);
  else if (lazy) console.log('Hi!');
  else if (name) console.log(`Hello ${name}!`);
  else console.log('Hello There!');
}
```

### Command Input and Casting

All command-line inputs are treated as **strings**. So you should convert them as needed.

```js
const age = Number(this.argument('age')); // Convert '23' to 23
```

### More Command Examples

Here’s a more advanced example of a command that inserts a user into a database:

- Required arguments: `email`, `password`
- Optional argument: `age`
- Option: `-male` (indicates gender)

```js
syntax = '<! email> <! password> <? age> <- male>';
```

```js
exec() {
  const email = this.argument('email');
  const password = this.argument('password');
  const gender = this.option('male') ? 'male' : 'female';
  const age = this.argument('age')
  ? Number(this.argument(age))
  : undefined;

  // Insert the user into the database (simplified example)
  console.log(
    `email: ${email}
     password: ${password}
     age: ${age}
     gender: ${gender}`
  );
}
```

Here’s a practical example of how to use `Command` to insert a user into a database.

```js
const { Command } = require('bnjsx');
const { Bnjsx } = require('bnjsx');

class InsertUserCommand extends Command {
  /**
   * Define the command syntax, as the following:
   * - `<! name>`: required argument
   * - `<? name>`: optional argument
   * - `<- name>`: option
   */
  static syntax = '<! email> <! password> <? age> <- male>';

  /**
   * This method is called when the command is executed.
   *
   * @returns No return value is required.
   */
  static async exec() {
    // Reference arguments and options
    const row = {
      email: this.argument('email'),
      password: this.argument('password'),
      age: this.argument('age') ? Number(this.argument('age')) : undefined,
      gender: this.option('male') ? 'male' : 'female',
    };

    // Build query
    const keys = Object.keys(row).filter((k) => row[k] !== undefined);
    const values = keys.map((k) => row[k]);
    const placeholders = keys.map(() => '?').join(', ');
    const columns = keys.join(', ');
    const query = `INSERT INTO users (${columns}) VALUES (${placeholders});`;

    // Load Bnjsx Config
    const config = await Bnjsx.load();

    // Request a Connection
    const connection = await config.cluster.request(config.default);

    // Execute an Insert Query
    await connection.query(query, values);

    // Log Success Message
    this.success(`User has been successfully created!`);
  }
}

module.exports = { InsertUserCommand };
```

Register this command in your `exec.js` file:

```js
// Import modules
const { register, execute } = require('bnjsx');
const { GreetCommand } = require('./commands/GreetCommand');
const { InsertUserCommand } = require('./commands/InsertUserCommand');

// Register commands
register('greet', GreetCommand);
register('insert:user', InsertUserCommand);

// Execute commands
execute()
  .then(() => process.exit())
  .catch((error) => console.error(error.message));
```

You can now use the command to insert a user into the database:

```bash
# Insert a user with email, password, age, and gender
node exec insert:user user1@example.com password 25 -male

# Insert a user with only required fields
node exec insert:user user2@example.com password
```

- **Command** makes it easy to define and execute custom commands.
- Use `syntax` to describe inputs and `exec()` to write logic.
- Register your command and run it from the terminal.

Now you're ready to build powerful commands in your app! 🚀

## Output Methods

The `Command` class provides built-in methods for displaying messages in the terminal, each with a specific color to represent the message type:

1. `this.success(message)`: **Green** color is used for success messages, indicating that an operation has completed successfully.

```js
this.success('User created successfully!');
```

2. `this.info(message)`: **Blue** color is used for informational messages, providing helpful details or updates without indicating any issue.

```js
this.info('Current Bnjsx version is 1.x.x');
```

3. `this.warning(message)`: **Yellow** color is used for warning messages, alerting the user to something that might require attention or could cause problems later.

```js
this.warning('Deprecated option, please update your configuration.');
```

4. `this.error(message)`: **Red** color is used for error messages, signaling that something went wrong and needs immediate attention.

```js
this.error('Invalid email address provided.');
```
