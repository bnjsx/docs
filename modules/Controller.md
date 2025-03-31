# Bnjsx Controller Module

The **Bnjsx Controller** module simplifies the process of requesting and managing database connections. It automatically requests a connection from the pool and releases it back once you're done, effectively eliminating connection issues.

## Methods

The `Bnjsx Controller` currently includes two primary methods that can be accessed by extending the Controller class.

### 1. `Controller.prototype.connection`

The `connection` method allows you to request a connection from a specific pool and automatically releases it back when done.

```js
class PostController extends Controller {
  // Read all posts
  async readPosts(req, res) {
    // Request a connection from the pool and execute a query to fetch all posts
    const posts = await this.connection((con) =>
      con.query('SELECT * FROM posts;')
    );

    // Send response with posts data
    return res.json(posts);
  }
}
```

> **Note:** The `connection` method automatically releases the connection back to the pool after the query is executed.

#### Requesting a Connection from a Specific Pool

You can also specify a pool by name as the second argument to the `connection` method:

```js
this.connection((con) => console.log('Asian connection'), 'asia');
```

In this case, a connection from the 'asia' pool will be requested.

The `connection` method resolves with the same value from the callback function.

### 2. `Controller.prototype.builder`

The `builder` method allows you to build and execute SQL queries in a more structured and cleaner manner. This method is very useful when you need to build complex queries dynamically.

```js
class PostController extends Controller {
  // Read all posts
  async readPosts(req, res) {
    // Example method for reading posts (same as previous)
  }

  // Read post by ID
  async readPost(req, res) {
    // Get post ID from the URL parameter
    const id = req.params.id;

    // Use builder to fetch the post by ID
    const post = await this.builder((builder) =>
      builder
        .select()
        .from('posts')
        .where((col) => col('id').equal(id))
        .first()
    );

    // Return the post data as a response
    return res.json(post);
  }
}
```

The `builder` method simplifies the process of creating and executing queries.

### Manual Usage of Cluster and Builder

If you prefer doing things manually, you can still access the Cluster and Builder directly:

1. First, get a reference to your cluster:

   ```js
   const cluster = config().loadSync().cluster;
   ```

2. Request a connection from the cluster:

   ```js
   const connection = await cluster.request();
   ```

3. Create a builder instance:

   ```js
   const builder = new Builder(connection);
   ```

4. Build and execute your query:

   ```js
   const user = await builder
     .select()
     .from('users')
     .where((col) => col('id').equal(id))
     .first();
   ```

5. Finally, release the connection back to the pool:

   ```js
   connection.release();
   ```

6. Dereference the builder and connection objects:

   ```js
   builder = null;
   connection = null;
   ```

> **Note:** The Controller module abstracts these steps for you, making database operations much simpler.

### Pool Name Configuration

You can request a connection from a specific pool by providing the pool name as the second argument, as shown in the examples above. Alternatively, you can update the default pool in your configuration file:

#### Update Default Pool

```js
// Using the config helper
config().loadSync().default = 'asia';

// Alternatively, using Bnjsx
Bnjsx.loadSync().default = 'africa';
```

Once the default pool is updated, all calls to `this.connection()` or `this.builder()` will use the default pool unless you specify a different one.

### Future Plans

We plan to improve the Controller module further. Upcoming features include:

- Global access to the request and response objects.
- More powerful methods for query handling and connection management.

Stay tuned as we continue to enhance the Controller module. Things are about to get even better!
