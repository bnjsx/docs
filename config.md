## Bnjsx Config

Bnjsx requires a configuration file `bnjsx.config.js` at the root of your project. Below is a breakdown of the configuration options, their purposes, and examples.

### Server Options

| Property   | Description                                             | Default Value |
| ---------- | ------------------------------------------------------- | ------------- |
| `env`      | The environment the app is running in (`dev` or `pro`). | `dev`         |
| `mode`     | The application mode (`web` or `api`).                  | `web`         |
| `host`     | The hostname or IP address the server should bind to.   | `localhost`   |
| `port`     | The port number the server should listen on.            | `2025`        |
| `protocol` | The protocol used for the server (`http` or `https`).   | `http`        |
| `key`      | The private key path for HTTPS configuration.           | `undefined`   |
| `cert`     | The certificate path for HTTPS configuration.           | `undefined`   |

### Flex Options

| Property  | Description                           | Default Value | Notes                                                      |
| --------- | ------------------------------------- | ------------- | ---------------------------------------------------------- |
| `cache`   | Enable/disable component caching.     | `true`        | Set to `true` in production to cache components in memory. |
| `globals` | Global values for Flex components.    | `undefined`   | Define reusable values across components.                  |
| `tools`   | Global functions for Flex components. | `undefined`   | Define reusable functions across components.               |

### Required Options

| Property  | Description                                                 |
| --------- | ----------------------------------------------------------- |
| `cluster` | A `Cluster` instance that defines your full database setup. |
| `default` | The name of the default pool used for database connections. |

The `default` option is used when running commands like `node exec gen` to generate tables. It tells the system which pool to pull a connection from by default. This is especially useful when managing multiple databases.

**_For example:_**

- You create tables in the `A` pool.
- Then switch the default to `B` and run the same command again.
- Now the tables get created in the other database — clean and flexible.

The `cluster` is your database **structure** — think of it like this:

```
Cluster (house)
├── Pool (room)
│   └── Driver (door)
│       └── connection (key)
│           └── used to query: SELECT / INSERT / UPDATE / DELETE
├── Pool (another room)
│   └── ...
```

- **Cluster** is a collection of pools
- **Pool** manages a specific database (with its own driver and connection setup)
- **Driver** defines how to connect (MySQL, PostgreSQL, SQLite, etc)
- **Connection** gives you access to run queries

This setup gives you full control over multiple databases. Define everything in your config, load your cluster anywhere, request a connection, and run queries whenever and wherever needed.

For a detailed explanation and usage examples, take a look at:

- [SQLite Driver](https://github.com/bnjsx/docs/blob/main/modules/SQLite.md)
- [MySQL Driver](https://github.com/bnjsx/docs/blob/main/modules/MySQL.md)
- [PostgreSQL Driver](https://github.com/bnjsx/docs/blob/main/modules/PostgreSQL.md)
- [Pool Module](https://github.com/bnjsx/docs/blob/main/modules/Pool.md)
- [Cluster Module](https://github.com/bnjsx/docs/blob/main/modules/Cluster.md)

### Other Options

1. `paths`: An object that specifies custom paths for Bnjsx folders. If not provided, Bnjsx uses the default paths.

| Property           | Description                         | Default Path        |
| ------------------ | ----------------------------------- | ------------------- |
| `paths.generators` | Path to your **generators** folder. | `<root>/generators` |
| `paths.seeders`    | Path to your **seeders** folder.    | `<root>/seeders`    |
| `paths.models`     | Path to your **models** folder.     | `<root>/models`     |
| `paths.commands`   | Path to your **commands** folder.   | `<root>/commands`   |
| `paths.views`      | Path to your **views** folder.      | `<root>/views`      |

> If typescript is enabled `paths` must be relative

2. `typescript`: An object to enable and configure TypeScript support.

| Property             | Description                                       | Default       |
| -------------------- | ------------------------------------------------- | ------------- |
| `typescript.enabled` | Enable TypeScript support (boolean).              | `false`       |
| `typescript.src`     | Path to the **source** folder when TS is enabled. | `<root>/src`  |
| `typescript.dist`    | Path to the **dist** folder when TS is enabled.   | `<root>/dist` |

- When `typescript.enabled` is `false`, commands like `node exec mk:model <table>` will add js files.
- When `typescript.enabled` is `true`, the same command will add TypeScript files.

3. `public`: Options for serving public assets

| Property | Description                                                               | Default |
| -------- | ------------------------------------------------------------------------- | ------- |
| `root`   | The root directory from which assets are served.                          | `false` |
| `cache`  | The number of seconds an asset should be cached.                          | `false` |
| `gzip`   | Whether to serve a precompressed `.gz` version of the asset if available. | `false` |

> For a detailed explanation and usage examples, take a look at the [middlewares docs](https://github.com/bnjsx/docs/blob/main/middlewares.md#asset-middleware).

4. `cors`: Configuration options specific to CORS (Cross-Origin Resource Sharing) behavior.

| Property      | Description                                                                      |
| ------------- | -------------------------------------------------------------------------------- |
| `origin`      | Specifies allowed origins for CORS requests.                                     |
| `methods`     | Specifies allowed HTTP methods for CORS requests.                                |
| `credentials` | Indicates whether credentials (cookies, HTTP authentication)                     |
| `headers`     | Specifies allowed headers in the CORS request.                                   |
| `expose`      | Specifies which headers are exposed to the browser.                              |
| `maxAge`      | Specifies how long the results of a preflight request can be cached, in seconds. |

> For a detailed explanation and usage examples, take a look at the [security docs](https://github.com/bnjsx/docs/blob/main/security.md#cors-middleware).

4. `security`: Configuration options for security-related HTTP headers

| Property                        | Description                                                             |
| ------------------------------- | ----------------------------------------------------------------------- |
| `contentSecurityPolicy`         | Restricts the resources that can be loaded and executed by the browser. |
| `strictTransportSecurity`       | Enforces HTTPS connections to prevent man-in-the-middle attacks.        |
| `referrerPolicy`                | Controls the referrer information sent with requests.                   |
| `crossOriginResourcePolicy`     | Controls which origins can access resources from your server.           |
| `crossOriginOpenerPolicy`       | Isolates the page from cross-origin interactions.                       |
| `crossOriginEmbedderPolicy`     | Controls restrictions on cross-origin resource embedding.               |
| `originAgentCluster`            | Prevents cross-origin memory leaks (e.g., Spectre).                     |
| `xContentTypeOptions`           | Prevents MIME type sniffing.                                            |
| `xDnsPrefetchControl`           | Disables DNS prefetching to reduce privacy risks.                       |
| `xDownloadOptions`              | Prevents automatic opening of downloaded files.                         |
| `xssProtection`                 | Prevents certain types of cross-site scripting attacks.                 |
| `xFrameOptions`                 | Prevents your site from being embedded in frames to avoid clickjacking. |
| `xPermittedCrossDomainPolicies` | Controls whether Flash or Acrobat files can make cross-domain requests. |

> For a detailed explanation and usage examples, take a look at the [security docs](https://github.com/bnjsx/docs/blob/main/security.md#security-middleware).

Here’s a complete example combining all options:

```js
const { Cluster } = require('bnjsx');
const { ClusterPool } = require('bnjsx');
const { MySQL } = require('bnjsx');
const { PostgreSQL } = require('bnjsx');
const { SQLite } = require('bnjsx');
const { UTC, Vite } = require('bnjsx');

module.exports = {
  env: 'env',
  mode: 'web',
  host: 'localhost',
  port: 3030,
  protocol: 'http',
  key: undefined,
  cert: undefined,
  cache: false,
  default: 'mysql',
  cors: { origin: '*' },
  security: { contentSecurityPolicy: false },
  globals: { app: { name: 'bnjsx', entry: 'https://bnjsx.com' } },
  tools: { year: UTC.get.year.bind(UTC), vite: Vite.assets.bind(Vite) },
  public: { root: './public' },
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
    new ClusterPool('sqlite', new SQLite('./database/main.sqlite'))
  ),
  paths: {
    generators: './gens', // Store generator files in gens folder
    seeders: './seeders',
    models: './models',
    commands: './cmds', // Store command files in cmds folder
    views: './views',
  },
  typescript: {
    enabled: true, // Enable typescript support
    src: './src',
    dist: './dist',
  },
};
```

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

**_Notes_**

- _You can also use `LoadSync` if you prefer to load your `bnjsx.config.js` configuration synchronously. Both `LoadSync` and `Load` work perfectly, so there's no need to worry about performance issues. Feel free to use whichever method you prefer._

- _You can also use `config` instead of `Bnjsx` if you preffer here is an example:_

```js
const { loadSync } = require('bnjsx').config();

// Load and log your configuration synchronously
console.log(loadSync());
```

> I implemented the `config` helper function so that if I ever needed to change the class name _(`Bnjsx`)_, I could do it in one place instead of updating multiple files.

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

> See the [Config](https://github.com/bnjsx/docs/blob/main/helpers/Config.md) helper docs for more examples and a detailed explanation.
