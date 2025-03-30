# Bnjsx Model

This module is designed to simplify database interactions in `Bnjsx`, providing a high-level API for seamless CRUD operations (`INSERT`, `SELECT`, `UPDATE`, `DELETE`). With support for:

- Complex queries
- Relationship management (`OneToOne`, `OneToMany`, `ManyToMany`)
- Advanced features such as joins, pagination, and dynamic filtering, and more...

## Table of Contents

1. **[Getting Started](#getting-started)**
   - [Creating Model Files](#creating-model-files)
   - [Customizing Model Names](#customizing-model-names)
   - [Model Files Template](#model-files-template)
   - [Bnjsxuration Options](#configuration-options)
2. **[CRUD Operations](#crud-operations)**
   - [Create Users](#create-users)
   - [Read Users](#read-users)
   - [Update Users](#update-users)
   - [Delete Users](#delete-users)
   - [Where RUD (Read, Update, Delete)](#where-rud-read-update-delete)
3. **[Relationships](#relationships)**
   - [OneToOne Relationship](#onetoone-relationship)
   - [OneToOne Reverse Relationship](#onetoone-reverse-relationship)
   - [OneToMany Relationship](#onetomany-relationship)
   - [OneToMany Reverse Relationship](#onetomany-reverse-relationship)
   - [ManyToMany Relationship](#manytomany-relationship)
4. **[Linking Models](#linking-models)**
   - [What is Linking?](#what-is-linking)
   - [Link Methods](#link-methods)
   - [Unlink Methods](#unlink-methods)
5. **[Modifiers](#modifiers)**
6. **[Getter Methods](#getter-methods)**
7. **[Event Handling](#event-handling)**

## Getting Started

### Creating Model Files

Use the following command to generate a new model file for your table:

```bash
node exec mk:model <table>
```

To create a model for the `users` table:

```bash
node exec mk:model users
```

This will create a `User.js` model file in `./models` (or `User.ts` in `./src/models` if TypeScript is enabled).

**Naming Conventions:**  
The model name is automatically generated as the singular form of your table name. Examples:

- `users` → `User`
- `profiles` → `Profile`
- `products` → `Product`

If the singular form cannot be resolved or you want a custom name, you can register a new singular/plural mapping in the Bnjsx dictionary.

### Customizing Model Names

To change the default singular form, add a mapping in your `exec.js` file:

```js
const { Command } = require('bnjsx');
const { execute } = require('bnjsx');

// Import dictionary
const { dictionary } = require('bnjsx');

// Register 'loser' as the singular form of 'users'
dictionary('loser', 'users');

// Execute commands
execute()
  .then(() => process.exit())
  .catch((error) => Command.error(error.message));
```

Now, running the command:

```bash
node exec mk:model users
```

Will create a `Loser.js` model file instead of `User.js`.

### Model Files Template

Here’s what the generated `User.js` file looks like:

```js
const { Model } = require('bnjsx');

class User extends Model {
  // The name of the table associated with this model
  static table = 'users';
}

module.exports = { User };
```

You can now use the `User` model to interact with the `users` table and perform CRUD operations effortlessly.

### Bnjsxuration Options

`Model` provides several static properties to configure your models:

| **Property** | **Description**                                            | **Default**  |
| ------------ | ---------------------------------------------------------- | ------------ |
| `builder`    | Query builder instance                                     | Required     |
| `table`      | The associated database table name                         | Required     |
| `timestamps` | Automatically manage `created_at` and `updated_at` columns | `true`       |
| `createdAt`  | Custom name for the `created_at` column                    | `created_at` |
| `updatedAt`  | Custom name for the `updated_at` column                    | `updated_at` |
| `primaryKey` | Name of the primary key column                             | `id`         |
| `foreignKey` | Name of the foreign key column (e.g: `user_id`)            | `<model>_id` |
| `columns`    | Columns to select by default                               | All columns  |
| `ignore`     | Columns to ignore during updates                           | Empty array  |
| `modifiers`  | Modify model values after selection                        | Empty object |

```js
const { Model } = require('bnjsx');

class User extends Model {
  // The name of the table associated with this model
  static table = 'users';

  // Enable timestamps handling (default)
  static timestamps = true;

  // The users table createdAt column is `created_on`
  static createdAt = 'created_on';

  // The users table updatedAt column is `updated_on`
  static updatedAt = 'updated_on';

  // The users table primary key is id (default)
  static primaryKey = 'id';

  // The users table foreign key is user_id (default)
  static foreignKey = 'user_id';

  // The columns I want to select (default)
  static columns = ['users.*'];

  // The columns I want to ignore on update
  static ignore = [User.primaryKey, User.createdAt];
}

module.exports = { User };
```

> We will see how to use `modifiers` and how to set up your `builder` later

## CRUD Operations

CRUD stands for Create, Read, Update, and Delete, and `Model` provides an easy way to perform these operations.

### Create Users

`insert(row)` and `insertMany(rows)` allow you to insert one or more rows into the database.

```js
// Import modules
const { Bnjsx } = require('bnjsx');
const { Builder } = require('bnjsx');
const { User } = require('./models/User');

const app = async () => {
  // Load config and create builder
  const config = await Bnjsx.load();
  const connection = await config.cluster.request(config.default);
  const builder = new Builder(connection);

  // Set the builder for the User model
  User.builder = builder;

  // Insert One User
  const user = await User.insert({
    email: 'user1@gmail.com',
    password: '123',
  });

  console.log(user instanceof User); // true
  console.log(user); // { id: 1, email: 'user1@gmail.com', ... }

  // Insert Many Users
  const users = await User.insertMany([
    {
      email: 'user2@gmail.com',
      password: '123',
    },
    {
      email: 'user3@gmail.com',
      password: '123',
    },
  ]);

  console.log(users.every((user) => user instanceof User)); // true
  console.log(users); // [{ email: 'user2@gmail.com', ... }, ...]

  // Release the connection
  connection.release();
};

// Execute
app();
```

- Notes for `insert(row)`:

  - For all drivers: The resolved model instances will include their primary keys.
  - Emits an `INSERT` event before the row is inserted and `INSERTED` event after.
  - If `User.timestamps` is true, the `created_at` and `updated_at` columns will be set automatically.

- Notes for `insertMany(rows)`:

  - For `PostgreSQL`: The resolved model instances include their primary keys.
  - For `MySQL` & `SQLite`: The resolved model instances do not include primary keys.
  - Emits an `INSERT_MANY` event before the rows are inserted and `INSERTED_MANY` event after.
  - If `User.timestamps` is true, the `created_at` and `updated_at` columns will be set automatically.

### Read Users

`find(key)` and `findMany(keys)` allow you to retrieve one or more models by their primary keys.

```js
// Import modules
const { Bnjsx } = require('bnjsx');
const { Builder } = require('bnjsx');
const { User } = require('./models/User');

const app = async () => {
  // Load config and create builder
  const config = await Bnjsx.load();
  const connection = await config.cluster.request(config.default);
  const builder = new Builder(connection);

  // Set the builder for the User model
  User.builder = builder;

  // Find the user where id = 1
  console.log(await User.find(1));
  // Resolves with a User instance
  // or undefined if no matching User is found

  // Find users where id is in [1, 3, 5, 7]
  console.log(await User.findMany(1, 3, 5, 7));
  // Resolves with an array of User instances
  // or an empty array if no matches are found

  // Release the connection
  connection.release();
};

// Execute
app();
```

> The primary key does not have to be a `number`; it can be any value that uniquely identifies a record, such as `emails`, `UUIDs`, or other custom unique identifiers.

`select()` returns a `Selector` instance, which you can use to build and execute a `SELECT` query. The key difference between `Selector` and `Select` is in the `exec` method:

- The `exec` method in `Select` resolves with an array of rows (anonymous objects).
- The `exec` method in `Selector` resolves with an array of `Model` instances.

In `Selector`, you can also use the `all()` method to execute your queries.

```js
// Import modules
const { Bnjsx } = require('bnjsx');
const { Builder, ref } = require('bnjsx');
const { User } = require('./models/User');

const app = async () => {
  // Load config and create builder
  const config = await Bnjsx.load();
  const connection = await config.cluster.request(config.default);
  const builder = new Builder(connection);

  // Set the builder for the User model
  User.builder = builder;

  // Get all users
  console.log(await User.select().all());

  // Get users with their profiles using an inner join
  console.log(
    await User.select()
      .col('users.*', 'profiles.age', 'profiles.gender')
      .join('profiles', (col) => {
        col('users.id').equal(ref('profiles.user_id'));
      })
      .exec()
  ); // Array of User models with associated profile data

  // Release the connection
  connection.release();
};

// Execute
app();
```

- You can chain additional methods on the `Selector` instance to refine your query, such as `.where()`, `.groupBy()`, `.orderBy()`, and `.limit()`. The API is identical to the `Select` module.

### Update Users

The `update()` method updates the record associated with the current instance using the model's primary key. Additionally, it automatically updates the `updatedAt` timestamp if timestamp handling is enabled.

```js
// Import modules
const { Bnjsx } = require('bnjsx');
const { Builder } = require('bnjsx');
const { User } = require('./models/User');

const app = async () => {
  // Load config and create builder
  const config = await Bnjsx.load();
  const connection = await config.cluster.request(config.default);
  const builder = new Builder(connection);

  // Set the builder for the User model
  User.builder = builder;

  // First, find the user you want to update
  const user = await User.find(1);

  // Update the user's properties
  user.email = 'updated@gmail.com';
  user.password = 'updated123';

  // Save the changes
  await user.update();

  // If you have the primary key of the user!
  // You can do this instead:
  const secondUser = new User({ id: 2 });

  secondUser.email = 'updated2@gmail.com';
  secondUser.password = 'updated456';

  // Save the changes
  await secondUser.update();

  // Release the connection
  connection.release();
};

// Execute
app();
```

> The `update()` method triggers the `UPDATE` event before the update and the `UPDATED` event after the update.

### Delete Users

The `delete()` method deletes the record associated with the current instance using the model's primary key.

```js
// Import modules
const { Bnjsx } = require('bnjsx');
const { Builder } = require('bnjsx');
const { User } = require('./models/User');

const app = async () => {
  // Load config and create builder
  const config = await Bnjsx.load();
  const connection = await config.cluster.request(config.default);
  const builder = new Builder(connection);

  // Set the builder for the User model
  User.builder = builder;

  // First, find the user you want to delete
  const user = await User.find(1);

  // Delete the user
  await user.delete();

  // If you have the primary key of the user!
  // You can do this instead:
  await new User({ id: 3 }).delete();

  // Release the connection
  connection.release();
};

// Execute
app();
```

> The `delete()` method triggers the `DELETE` event before the deletion and the `DELETED` event after the deletion.

### Where RUD (Read, Update, Delete)

`where(condition)` allows you to build and execute `SELECT`, `UPDATE`, and `DELETE` queries based on specific conditions.

```js
// Import modules
const { Bnjsx } = require('bnjsx');
const { Builder } = require('bnjsx');
const { User } = require('./models/User');

const app = async () => {
  // Load config and create builder
  const config = await Bnjsx.load();
  const connection = await config.cluster.request(config.default);
  const builder = new Builder(connection);

  // Set the builder for the User model
  User.builder = builder;

  // Select a user with a specific ID
  await User.where((col) => col('id').equal(20)).select();

  // Delete a user with a specific ID
  await User.where((col) => col('id').equal(30)).delete();

  // Update a user's information by ID
  await User.where((col) => col('id').equal(45)).update({
    email: 'updated@gmail.com',
  });

  // Release the connection
  connection.release();
};

// Execute
app();
```

> This method does not emit any events, unlink `insert`, `insertMany`, `update` and `delete`.

## Relationships

Model provides built-in methods to simplify loading related models. with support for common relationships like **OneToOne**, **OneToMany**, and **ManyToMany**. You might already be familiar with these concepts, but even if you're not, this guide should be sufficient to get you started.

### OneToOne Relationship

Imagine we have a `User` model and a `Profile` model. Each user has exactly one profile, representing a **OneToOne** relationship. Here's how you can load the profile of any user:

```js
// Import modules
const { Model } = require('bnjsx');
const { Profile } = require('./models/Profile');

// In your User.js file
class User extends Model {
  // Set the table name
  static table = 'users';

  // This property will store the profile data
  profile;

  // Define a method to load the user's profile
  loadProfile() {
    return new Promise((resolve, reject) => {
      // Check if the profile is already loaded
      if (this.profile) return resolve(this.profile);

      // Fetch the profile using the OneToOne method
      this.OneToOne(Profile)
        .then((profile) => {
          this.profile = profile; // Cache the profile
          resolve(profile); // Resolve with the fetched profile
        })
        .catch(reject); // Reject if there's an error
    });
  }
}
```

To use this method to load any user's profile, follow the steps below:

```js
// Import modules
const { Bnjsx } = require('bnjsx');
const { Builder } = require('bnjsx');
const { User } = require('./models/User');

const app = async () => {
  // Load config and create builder
  const config = await Bnjsx.load();
  const connection = await config.cluster.request(config.default);
  const builder = new Builder(connection);

  // Set the builder for the User model
  User.builder = builder;

  // Fetch the first user
  const user = await User.find(1);

  // Load the user's profile
  await user.loadProfile();

  // The profile is now cached on the User instance
  console.log(user.profile);

  // Release the connection
  connection.release();
};

// Execute
app();
```

### OneToOne Reverse Relationship

You can also load the user associated with a profile using the `References(model)` method. This is the reverse of the OneToOne relationship.

```js
// Import modules
const { Model } = require('bnjsx');
const { User } = require('./models/User');

// In your Profile.js file
class Profile extends Model {
  // Set the table name
  static table = 'profiles';

  // This property will store the user data
  user;

  // Define a method to load the user of the profile
  loadUser() {
    return new Promise((resolve, reject) => {
      // Check if the user is already loaded
      if (this.user) return resolve(this.user);

      // Fetch the user using the References method
      this.References(User)
        .then((user) => {
          this.user = user; // Cache the user
          resolve(user); // Resolve with the fetched user
        })
        .catch(reject); // Reject if there's an error
    });
  }
}
```

Here's how you can load the user for any profile:

```js
// Import modules
const { Bnjsx } = require('bnjsx');
const { Builder } = require('bnjsx');
const { Profile } = require('./models/Profile');

const app = async () => {
  // Load config and create builder
  const config = await Bnjsx.load();
  const connection = await config.cluster.request(config.default);
  const builder = new Builder(connection);

  // Set the builder for the Profile model
  Profile.builder = builder;

  // Fetch the first profile
  const profile = await Profile.find(1);

  // Load the user of the profile
  await profile.loadUser();

  // The user is now cached on the Profile instance
  console.log(profile.user);

  // Release the connection
  connection.release();
};

// Execute
app();
```

- Use `OneToOne(model)` to load the **child model** (e.g., a user loads their profile).
- Use `References(model)` to load the **parent model** (e.g., a profile loads its user).
- The child model typically has a foreign key referencing the parent model.
- Both `OneToOne(model)` and `References(model)` will throw an error if multiple related models are found, as this violates the `OneToOne` relationship. They return `undefined` if no related model is found.
- In a `OneToOne` relationship, each instance of the parent model should correspond to exactly one instance of the child model (e.g., each user has exactly one profile).

### OneToMany Relationship

In a **OneToMany** relationship, one record in a parent model is associated with multiple records in a child model. For example, a `User` can have multiple `Post` records.

To demonstrate, let’s say we have a `User` model and a `Post` model. Each `Post` references a `User` (the parent model) via a foreign key (`user_id`). Using Bnjsx, you can load all posts associated with a user as shown below:

```js
// Import modules
const { Model } = require('bnjsx');
const { Post } = require('./models/Post');

// In your User.js file
class User extends Model {
  // Set the table name
  static table = 'users';

  // This property will store the user's posts
  posts;

  // Define a method to load the user's posts
  loadPosts() {
    return new Promise((resolve, reject) => {
      // Check if posts are already loaded and resolve from cache
      if (this.posts) return resolve(this.posts);

      // Fetch posts using the OneToMany method
      this.OneToMany(Post)
        .then((posts) => {
          this.posts = posts; // Cache the posts
          resolve(posts); // Resolve with the fetched posts
        })
        .catch(reject); // Reject if there's an error fetching posts
    });
  }
}
```

In your main application file (`index.js`), you can use the `loadPosts` method to load the posts for any user instance:

```js
// Import Bnjsx, Builder, and the User model
const { Bnjsx } = require('bnjsx');
const { Builder } = require('bnjsx');
const { User } = require('./models/User');

const app = async () => {
  // Load config and create builder
  const config = await Bnjsx.load();
  const connection = await config.cluster.request(config.default);
  const builder = new Builder(connection);

  // Set the query builder for the User model
  User.builder = builder;

  // Retrieve the first user
  const user = await User.find(1);

  // Load the user's posts
  await user.loadPosts();

  // The posts are now cached on the User instance
  console.log(user.posts);

  // Release the connection
  connection.release();
};

// Execute
app();
```

### OneToMany Reverse Relationship

Each `Post` references a `User`, making it easy to navigate back to the parent model. To load the `User` associated with a `Post`, use the `References` method:

```js
// Import the User model
const { User } = require('./models/User');

// In your Post.js file
class Post extends Model {
  // Set the table name
  static table = 'posts';

  // This property will store the user data
  user;

  // Define a method to load the user of the post
  loadUser() {
    return new Promise((resolve, reject) => {
      // Check if the user is already loaded
      if (this.user) return resolve(this.user);

      // Fetch the user using the References method
      this.References(User)
        .then((user) => {
          this.user = user; // Cache the user
          resolve(user); // Resolve with the fetched user
        })
        .catch(reject); // Reject if there's an error
    });
  }
}
```

> You can now load the user of any post by simply executing the `loadUser()` method.

### ManyToMany Relationship

Consider a scenario where we have a `Post` model and a `Category` model. Each post can have many categories, and each category can have many posts (ManyToMany relationship). In this case, we can load the posts of a category and the categories of a post using the following approach.

```js
// Import modules
const { Model } = require('bnjsx');
const { Category } = require('./models/Category');

// In your Post.js file
class Post extends Model {
  // Set the table name
  static table = 'posts';

  // This property will store the categories
  categories;

  // Define a method to load the post's categories
  loadCategories() {
    return new Promise((resolve, reject) => {
      // Check if categories are already loaded
      if (this.categories) return resolve(this.categories);

      // Fetch categories using the ManyToMany method
      this.ManyToMany(Category)
        .then((categories) => {
          this.categories = categories; // Cache the categories
          resolve(categories); // Resolve with the fetched categories
        })
        .catch(reject); // Reject if there's an error
    });
  }
}
```

Next, you should define another method in the `Category` model to load all posts associated with a specific category.

```js
// Import modules
const { Model } = require('bnjsx');
const { Post } = require('./models/Post');

// In your Category.js file
class Category extends Model {
  // Set the table name
  static table = 'categories';

  // This property will store the posts
  posts;

  // Define a method to load the category's posts
  loadPosts() {
    return new Promise((resolve, reject) => {
      // Check if posts are already loaded
      if (this.posts) return resolve(this.posts);

      // Fetch posts using the ManyToMany method
      this.ManyToMany(Post)
        .then((posts) => {
          this.posts = posts; // Cache the posts
          resolve(posts); // Resolve with the fetched posts
        })
        .catch(reject); // Reject if there's an error
    });
  }
}
```

Finally, in your `index.js`, you can use these methods to load the posts of any category and the categories of any post.

```js
// Import modules
const { Bnjsx } = require('bnjsx');
const { Builder } = require('bnjsx');
const { Post } = require('./models/Post');
const { Category } = require('./models/Category');

const app = async () => {
  // Load config and create builder
  const config = await Bnjsx.load();
  const connection = await config.cluster.request(config.default);
  const builder = new Builder(connection);

  // Set the builder for the Post and Category models
  Post.builder = builder;
  Category.builder = builder;

  // Get the first post and category
  const post = await Post.find(1);
  const category = await Category.find(1);

  // Load the post's categories
  await post.loadCategories();

  // Load the category's posts
  await category.loadPosts();

  // Posts are now cached on the Category instance
  console.log(category.posts);

  // Categories are now cached on the Post instance
  console.log(post.categories);

  // Release the connection
  connection.release();
};

// Execute
app();
```

- This method assumes a `ManyToMany` relationship, where a link table (e.g., `category_post`) connects the parent model (`Post`) and the child model (`Category`).

- Optionally, you can specify the name of the link table as the second argument in `ManyToMany(model, table?)`. If not provided, the default table name is generated by combining the parent and child model names alphabetically.

- You can also specify the columns you want to retrieve from the link table as the third argument in `ManyToMany(model, table?, ...columns)`. Although it is not recommended to have other columns in the link table besides foreign keys, you can include additional columns if needed, and they will be included in the result.

## Linking Models

In addition to fetching related models, you can easily **link** and **unlink** models to **create** or **remove** `ManyToMany` relationships.

### What is Linking?

In a `ManyToMany` relationship, a row in one table can be associated with multiple rows in another table. For example, a post can belong to multiple categories, and each category can have multiple posts. To manage these relationships, we use three tables:

- `posts`: stores post data
- `categories`: stores category data
- `category_post`: the link table connecting posts and categories

### Link Methods

- `link(model, table?, row?)`: Creates a link between the current model and the provided model by inserting a record into the link table.
- `linkMany(models, table?, rows?)`: Creates links between the current model and multiple models by inserting multiple records into the link table.

```js
// Import modules
const { Bnjsx } = require('bnjsx');
const { Builder } = require('bnjsx');
const { Post } = require('./models/Post');
const { Category } = require('./models/Category');

const app = async () => {
  // Load config and create builder
  const config = await Bnjsx.load();
  const connection = await config.cluster.request(config.default);
  const builder = new Builder(connection);

  // Set builder for models
  Post.builder = builder;
  Category.builder = builder;

  // Fetch the post and category
  const post = await Post.find(1);
  const category = await Category.find(1);

  // Link post to category
  await post.link(category);

  // Link post to multiple categories
  const categories = await Category.findMany(2, 3, 4);
  await post.linkMany(categories);

  // Now the post is linked with 4 categories
  console.log(await post.loadCategories());

  // Release the connection
  connection.release();
};

// Execute
app();
```

- You can pass extra columns in the third argument, besides the foreign keys.
- `link` emits `LINK` before and `LINKED` after the link is created.
- `linkMany` emits `LINK_MANY` before and `LINKED_MANY` after creating multiple links.

### Unlink Methods

In addition to linking models, you can also **unlink** models in a ManyToMany relationship. This is done with the following methods:

- `unlink(model, table?)`: Removes a link between the current model and the provided model by deleting the corresponding record from the link table.
- `unlinkMany(models, table?)`: Removes links between the current model and multiple models by deleting multiple records from the link table.

```js
// Import modules
const { Bnjsx } = require('bnjsx');
const { Builder } = require('bnjsx');
const { Post } = require('./models/Post');
const { Category } = require('./models/Category');

const app = async () => {
  // Load config and create builder
  const config = await Bnjsx.load();
  const connection = await config.cluster.request(config.default);
  const builder = new Builder(connection);

  // Set builder for models
  Post.builder = builder;
  Category.builder = builder;

  // Fetch the post and category
  const post = await Post.find(1);
  const category = await Category.find(1);

  // Unlink the post from the category
  await post.unlink(category);

  // Unlink post from multiple categories
  const categories = await Category.findMany(2, 3, 4);
  await post.unlinkMany(categories);

  // Now the post has no categories
  console.log(await post.loadCategories());

  // Release the connection
  connection.release();
};

// Execute
app();
```

- `unlink` emits the `UNLINK` event before the unlink and the `UNLINKED` event after the unlink.
- `unlinkMany` emits the `UNLINK_MANY` event before and `UNLINKED_MANY` event after unlinking multiple models.
- You can specify the link table name in the second argument if you don't follow Bnjsx's default link table naming convention.

## Modifiers

Modifiers in Bnjsx let you apply transformations to column values when they are fetched from the database. You can register these modifiers for specific columns to ensure that data is always formatted the way you want.

### Format Post Title

If you want to ensure that the `title` of every post is formatted correctly (e.g., capitalizing the first letter of each word), you can create a modifier function like this:

```js
// Modifier function to format title
function formatTitle(title) {
  return title
    .trim()
    .toLowerCase()
    .split(' ')
    .map((word) => word[0].toUpperCase() + word.slice(1))
    .join(' ');
}

// In your Post model
class Post extends Model {
  static table = 'posts';

  // Register modifier for the 'title' column
  static modifiers = {
    title: [formatTitle],
  };
}
```

### How It Works:

- **When you fetch posts**, the `title` field will be automatically transformed by the `formatTitle` function.
- For example, a title like `" i love bnjsx "` will become `"I Love Bnjsx"`.

## Getter Methods

Our `Model` offers several useful getter methods that help you interact with your model's configuration and manage its state. These methods are particularly useful when extending Model or accessing model properties dynamically.

| **Method**              | **Description**                                           |
| ----------------------- | --------------------------------------------------------- |
| `get.builder()`         | Returns the `Builder` instance for the model.             |
| `get.table()`           | Returns the model's table name.                           |
| `get.columns()`         | Returns the model's selected columns.                     |
| `get.ignore()`          | Returns columns ignored during updates.                   |
| `get.createdAt()`       | Returns the `created_at` column name.                     |
| `get.updatedAt()`       | Returns the `updated_at` column name.                     |
| `get.timestamps()`      | Indicates if timestamp support is enabled.                |
| `get.link(model)`       | Returns the link table name for ManyToMany relationships. |
| `get.emitter()`         | Returns the model's `EventEmitter` instance.              |
| `get.modifiers(column)` | Returns modifiers for a specific column.                  |
| `model()`               | Returns the model class from an instance.                 |
| `valueOf(column)`       | Returns the value of a column, ensuring it's defined.     |

## Event Handling

These events define the lifecycle of various operations. Each operation emits specific events to allow for hooks or listeners that can execute additional logic before or after the operation. Here's a breakdown:

### Insert Events

1. `INSERT`

   - Emitted before a single row is inserted.
   - **Parameter:** `row` - The row data to be inserted.
   - Triggered by the `insert()` static method.

2. `INSERTED`

   - Emitted after a single row is inserted.
   - **Parameter:** `model` - The resulting model from the insert operation.
   - Triggered by the `insert()` static method.

3. `INSERT_MANY`

   - Emitted before multiple rows are inserted.
   - **Parameter:** `rows` - The rows data to be inserted.
   - Triggered by the `insertMany()` static method.

4. `INSERTED_MANY`
   - Emitted after multiple rows are inserted.
   - **Parameter:** `models` - The resulting models from the insert operation.
   - Triggered by the `insertMany()` static method.

### Update Events

5. `UPDATE`

   - Emitted before a model is updated.
   - **Parameter:** `model` - The model to be updated.
   - Triggered by the `update()` method.

6. `UPDATED`
   - Emitted after a model is updated.
   - **Parameter:** `model` - The updated model.
   - Triggered by the `update()` method.

### Delete Events

7. `DELETE`

   - Emitted before a model is deleted.
   - **Parameter:** `model` - The model to be deleted.
   - Triggered by the `delete()` method.

8. `DELETED`
   - Emitted after a model is deleted.
   - **Parameter:** `model` - The deleted model.
   - Triggered by the `delete()` method.

### Linking Events

9. `LINK`

- Emitted before linking one model to another.
- **Parameters:**
  - `main`: The model being linked.
  - `model`: The target model.
  - `data`: Additional data for the link operation.
- Triggered by the `link()` method.

10. `LINKED`

    - Emitted after linking one model to another.
    - Same parameters as `LINK`.
    - Triggered by the `link()` method.

11. `LINK_MANY`

    - Emitted before linking multiple models to another.
    - **Parameters:**
      - `main`: The model being linked.
      - `models`: The target models.
      - `data`: Additional data for the link operation.
    - Triggered by the `linkMany()` method.

12. `LINKED_MANY`
    - Emitted after linking multiple models to another.
    - Same parameters as `LINK_MANY`.
    - Triggered by the `linkMany()` method.

### Unlinking Events

13. `UNLINK`

    - Emitted before unlinking one model from another.
    - **Parameters:**
      - `main`: The model being unlinked.
      - `model`: The target model.
    - Triggered by the `unlink()` method.

14. `UNLINKED`

    - Emitted after unlinking one model from another.
    - Same parameters as `UNLINK`.
    - Triggered by the `unlink()` method.

15. `UNLINK_MANY`

    - Emitted before unlinking multiple models from another.
    - **Parameters:**
      - `main`: The model being unlinked.
      - `models`: The target models.
    - Triggered by the `unlinkMany()` method.

16. `UNLINKED_MANY`
    - Emitted after unlinking multiple models from another.
    - Same parameters as `UNLINK_MANY`.
    - Triggered by the `unlinkMany()` method.

### Usage Example

```js
// Import INSERT event
const { INSERT } = require('bnjsx');
const { UPDATE } = require('bnjsx');
const { DELETE } = require('bnjsx');

// Import toSlug
const { toSlug } = require('bnjdx');

// Import UTC
const { UTC } = require('bnjsx');

// In your Post.js file
class Post extends Model {
  static table = 'posts';
}

// Example 1: Prepare the row for insert
Post.get.emitter().on(INSERT, (row) => {
  // Here you can update the row before insert
  row.title.toLowerCase();
  row.slug = toSlug(row.title); // create a slug from title

  // You can also handle timestamps your self
  row.created_at = UTC.get.datetime(); // Creation datetime in UTC
  row.updated_at = null; // Set null
});

// Example 2: Prepare row for UPDATE
Post.get.emitter().on(UPDATE, (row) => {
  // Update the slug using the new title
  row.slug = toSlug(row.title);

  // Set updated_at
  row.update_at = UTC.get.datetime();
});

// Example 3: Prepare a model for DELETE
Post.get.emitter().on(DELETE, (post) => {
  // Perform any cleanup actions
  if (post.is_published) {
    throw new Error('Cannot delete a published post.');
  }
});
```

> You can also use the `INSERT` event to validate user inputs before insertion.
> I highly recommend checking out our Controller module—it simplifies builder initialization and automatically handles connection creation and release for you.
