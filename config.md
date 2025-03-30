## Bnjsx Config

Bnjsx requires a configuration file `bnjsx.config.js` at the root of your project. Below is a breakdown of the configuration options, their purposes, and examples.

### Required Options

1. `cluster`: A `Cluster` instance representing your database setup. It can contain multiple pools, with different drivers (e.g., `MySQL`, `PostgreSQL`, `SQLite`).

2. `default`: The name of the pool `Bnjsx` will use for connections by default.

Here’s an example of a `bnjsx.config.js` file:

```js
const { resolve } = require('path');
const { Cluster } = require('bnjsx');
const { ClusterPool } = require('bnjsx');
const { MySQL } = require('bnjsx');
const { PostgreSQL } = require('bnjsx');
const { SQLite } = require('bnjsx');

module.exports = {
  // Default pool name Bnjsx will use
  default: 'mysql',

  // Cluster setup with three database pools
  cluster: new Cluster(
    new ClusterPool(
      'mysql', // Pool name
      new MySQL({
        host: 'localhost',
        user: 'root',
        password: 'root',
        database: 'main',
      })
    ),
    new ClusterPool(
      'pg', // Pool name
      new PostgreSQL({
        host: 'localhost',
        user: 'postgres',
        password: 'root',
        database: 'main',
      })
    ),
    new ClusterPool(
      'sqlite', // Pool name
      new SQLite(resolve(__dirname, './database.sqlite'))
    )
  ),
};
```

In this setup:

- `mysql` is the default pool to request connections from.
- The `cluster` contains three database pools: `mysql`, `pg`, and `sqlite`. Each pool is defined with its respective driver and connection details.

### Optional Options

1. `paths`: An object that specifies custom paths for Bnjsx folders. If not provided, Bnjsx uses the default paths.

| Property           | Description                         | Default Path        |
| ------------------ | ----------------------------------- | ------------------- |
| `paths.generators` | Path to your **generators** folder. | `<root>/generators` |
| `paths.seeders`    | Path to your **seeders** folder.    | `<root>/seeders`    |
| `paths.models`     | Path to your **models** folder.     | `<root>/models`     |
| `paths.commands`   | Path to your **commands** folder.   | `<root>/commands`   |

2. `typescript`: An object to enable and configure TypeScript support.

| Property             | Description                                       | Default       |
| -------------------- | ------------------------------------------------- | ------------- |
| `typescript.enabled` | Enable TypeScript support (boolean).              | `false`       |
| `typescript.src`     | Path to the **source** folder when TS is enabled. | `<root>/src`  |
| `typescript.dist`    | Path to the **dist** folder when TS is enabled.   | `<root>/dist` |

- When `typescript.enabled` is `false`, commands like `node mega add:model <table>` will add js files.
- When `typescript.enabled` is `true`, the same command will add TypeScript files.

Here’s a complete example combining all options:

```js
const { resolve } = require('path');
const { Cluster } = require('bnjsx');
const { ClusterPool } = require('bnjsx');
const { MySQL } = require('bnjsx');
const { PostgreSQL } = require('bnjsx');
const { SQLite } = require('bnjsx');

module.exports = {
  default: 'mysql',
  cluster: new Cluster(
    new ClusterPool(
      'mysql',
      new MySQL({
        host: 'localhost',
        user: 'root',
        password: 'root',
        database: 'main',
      })
    ),
    new ClusterPool(
      'pg',
      new PostgreSQL({
        host: 'localhost',
        user: 'postgres',
        password: 'root',
        database: 'main',
      })
    ),
    new ClusterPool(
      'sqlite',
      new SQLite(resolve(__dirname, './database.sqlite'))
    )
  ),
  paths: {
    generators: './gens', // Store generator files in gens folder
    seeders: './seeders',
    models: './models',
    commands: './cmds', // Store command files in cmds folder
  },
  typescript: {
    enabled: true, // Enable typescript support
    src: './src',
    dist: './dist',
  },
};
```

> If typescript is enabled `paths` must be relative

### Loading Configuration

You can also load your `bnjsx.config.js` file from anywhere in your project. This is very useful because it provides easy access to your `Cluster` and allows you to define custom options if needed.

Start by importing **Bnjsx** from `bnjsx` into your project:

```js
import { Bnjsx } from 'bnjsx';
```

Now you can load `bnjsx.config.js` from anywhere in your project using the `Bnjsx.load()` method. This method will return a promise that resolves with the configuration object.

```js
Bnjsx.load().then((config) => console.log(config));
```

- The `load` method:
  - Loads the configuration from the `bnjsx.config.js` file.
  - Caches the configuration after the first load for better performance.
  - Runs registered validators to ensure the configuration is valid.

### Loading Other Configurations

If you need to load a custom configuration file, you can extend `Config` helper to create your own configuration loader. For example:

```js
// Import Config
const { Config } = require('bnjsx');

// Extend Config
class NodeJSConfig extends Config {
  static file = 'package.json'; // Load package.json
  static default = { version: '1.0.0' }; // Default package.json config
}

// Export NodeJSConfig
module.exports = { NodeJsConfig };
```

Now you can use `NodeJSConfig` to load `package.json` anywhere you like

```js
// Load and log package.json
NodeJSConfig.load().then((config) => console.log(config));

// Get the project root from any sub-folder
NodeJSConfig.resolveSync();

// Load any JSON config
NodeJSConfig.loadJSON(path);

// Load any JS config
NodeJSConfig.loadJS(path);
```

> See the `Config` helper docs for more examples and a detailed explanation.
