## Bnjsx Generator

This module is designed to manage tables creation and deletion for you. Each generator file represents a table and enables you to easily create or drop the table as needed.

## Table of Contents

1. **[Adding Generator Files](#adding-generator-files)**
2. **[Generator Files Order](#generator-files-order)**
3. **[Generator File Structure](#generator-file-structure)**
4. **[Creating Tables](#creating-tables)**
5. **[Dropping Tables](#dropping-tables)**
6. **[Rolling Back](#rolling-back)**
7. **[Removing Generator Files](#removing-generator-files)**
8. **[Data Types](#data-types)**
9. **[Constraints](#constraints)**

## Adding Generator Files

To create a new generator file, use the following command:

```bash
node exec mk:gen <table>
```

or

```bash
node exec mk:generator <table>
```

Let's add a new generator file for our users table:

```bash
node exec mk:gen users
```

You will see this response:

```
Generator added in: ./generators/01_generate_users_table.js
```

Each generator file has a **numbered prefix** followed by a descriptive name, This prefix determines the creation order of your tables.

## Generator Files Order

Let’s say your `./generators` folder contains the following files:

```
01_generate_users_table.js
02_generate_profiles_table.js
03_generate_products_table.js
```

This order means:

1. The `users` table is created first.
2. The `profiles` table is created next.
3. The `products` table is created last.

**Why Order Matters**  
If the `profiles` table includes a foreign key that references the `users` table, the `users` table must exist before creating the `profiles` table. Always ensure referenced tables are created earlier (has a smaller number).

## Generator File Structure

Here’s an example of the users table generator file:

```js
const { Generator } = require('bnjsx');

class UsersTableGenerator extends Generator {
  constructor() {
    super();

    // Set the table name associated with this generator.
    this.set.table('users');
  }

  create() {
    return this.schema(
      this.primaryKey(), // Primary key column
      this.createdAt(), // created_at column
      this.updatedAt(), // updated_at column

      this.column('username').varChar(200).unique().notNull(), // username column
      this.column('email').varChar(200).unique().notNull(), // email column
      this.column('password').varChar(200).notNull() // password column
    );
  }
}

module.exports = new UsersTableGenerator(); // Export an instance
```

1. Use `this.set.table(name)` to define the table name.

```js
this.set.table('users');
```

2. Use `this.column(name)` to create a column and chain methods to define its properties.

```js
this.column('email')
  .varChar(200) // Variable string with a max length of 255
  .unique() // Ensures the value is unique
  .notNull(); // Ensures the column cannot be null
```

3. Pass your column(s) to `this.schema(columns)` to build and execute the `CREATE TABLE` query.

```js
create() {
  return this.schema(
    this.primaryKey(), // Primary key column
    this.createdAt(), // created_at column
    this.updatedAt(), // updated_at column

    this.column('username').varChar(200).unique().notNull(), // username column
    this.column('email').varChar(200).unique().notNull(), // email column
    this.column('password').varChar(200).notNull() // password column
  );
}
```

The `create()` function will create a `users` table with a primary key column (`id`), along with `created_at` and `updated_at` columns to track record creation and updates. Additionally, it will create three required columns: `username` (unique and non-nullable), `email` (unique and non-nullable), and `password` (non-nullable). The `username` and `email` columns must be unique across all records, ensuring no duplicates.

## Creating Tables

Once you have defined your generator files, use the following command to create your tables:

```bash
node exec gen
```

This command will execute all the generator files that have not been run yet, creating the tables in your database based on the schema you defined in each generator file. The output should be:

```
1/1 tables created
```

This output means the first generator file (e.g., `01_generate_users_table.js`) successfully created the `users` table. If there are more generator files, it will execute those in order as well.

## Dropping Tables

If you need to reset your database and drop all tables, use the following command:

```bash
node exec reset
```

This will **drop all tables** in your database, allowing you to recreate them from scratch by running the generator files again using `node exec gen`.

## Rolling Back

If you only want to drop tables created the last time `node exec gen` command is executed, use:

```bash
node exec roll
```

This will **drop only the tables created in the last execution of `node exec gen`**, allowing you to undo your most recent table creation without affecting older tables. This is helpful if you want to remove a specific table but keep others intact.

## Removing Generator Files

If you no longer need a specific generator file, you can remove it by running the following command:

```bash
node exec rm:gen <table>
```

Replace `<table>` with the name of the table associated with the generator file. For example, if you want to remove the generator for the `users` table:

```bash
node exec rm:gen users
```

This will remove the generator file associated with the `users` table (`01_generate_users_table.js`), Keep in mind that this command will first drop all your tables, remove the generator file, and then reorder the remaining generators.

These commands provide an easy way to manage your tables and generator files, ensuring you can create, modify, and clean up your database schema as needed.

## Data Types

In this section, you'll find different types you can use to define the kind of data each column will hold. These methods are important for making sure your data is stored in the right format, and they help keep your database organized and efficient.

`tinyInt()`: Used for very small integer values and ranges from -128 to 127 `SIGNED` or 0 to 255 `UNSIGNED`.

```js
this.column('age').tinyInt();
```

`smallInt()`: Used for small integer values and ranges from -32,768 to 32,767 `SIGNED` or 0 to 65,535 `UNSIGNED`.

```js
this.column('quantity').smallInt();
```

`mediumInt()`: Used for medium-sized integer values and ranges from -8,388,608 to 8,388,607 `SIGNED` or 0 to 16,777,215 `UNSIGNED`.

```js
this.column('count').mediumInt();
```

`int()`: Used for standard integer values and ranges from -2,147,483,648 to 2,147,483,647 `SIGNED` or 0 to 4,294,967,295 `UNSIGNED`.

```js
this.column('total').int();
```

`bigInt()`: Used for very large integer values and ranges from -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 `SIGNED` or 0 to 18,446,744,073,709,551,615 `UNSIGNED`.

```js
this.column('id').bigInt();
```

`float()`: Used for single-precision floating-point numbers. It can handle up to approximately 7 decimal places.

```js
this.column('rating').float();
```

> Inserting `3.1415926535` will result in `3.141593` (rounded-up).

`double()`: Used for double-precision floating-point numbers. It can handle up to approximately 15 decimal places.

```js
this.column('distance').double();
```

> Inserting `3.141592653589793238` will result in `3.141592653589794` (rounded-up).

`decimal()`: Used for exact numeric values. It stores numbers with a fixed number of decimal places.

```js
this.column('price').decimal(10, 2); // from 0.00 to 99999999.99
this.column('discount').decimal(8, 3); // from 0.000 to 99999.999
```

`char()`: Used for storing fixed-length strings. will always store that number of characters, padding with spaces if necessary.

```js
this.column('size').char(5);
```

- Inserting `'abc'` will result in `'abc\s\s'`
- Inserting `'abcdef'` will result in `'abcde'` (truncated)
- Use it to store fixed-length strings like: `'xs'`, `'sm'`, `'md'`, `'lg'`, etc.
- The `length` must be <= 255 and >= 0

`varChar()`: Used for storing variable-length strings.

```js
this.column('size').varChar(5);
```

- Inserting `'abc'` will result in `'abc'`
- Inserting `'abcdef'` will result in `'abcde'` (truncated)
- Use it to store `names`, `addresses`, `phones`, etc.
- The `length` must be <= 65,535 and >= 0

`tinyText()`: Used for storing variable-length strings with a maximum length of `255` characters.

```js
this.column('bio').tinyText();
```

`mediumText()`: Used for storing variable-length strings with a maximum length of 16,777,215 characters.

```js
this.column('description').mediumText();
```

`text()`: Used for storing variable-length strings with a maximum length of 65,535 characters.

```js
this.column('article').text();
```

`longText()`: Used for storing variable-length strings with a maximum length of 4,294,967,295 characters.

```js
this.column('book').longText();
```

`boolean()`: Used for storing `TRUE` or `FALSE` values.

```js
this.column('is_active').boolean();
```

> To insert `TRUE` use `1` and `0` for `FALSE`.

`date()`: Used for storing date values in the format `YYYY-MM-DD`.

```js
this.column('birth_date').date();
```

`time()`: Used for storing time values in the format `HH:MM:SS`.

```js
this.column('birth_time').time();
```

`datetime()`: Used for storing date and time values in the format `YYYY-MM-DD HH:MM:SS`.

```js
this.column('updated_at').datetime();
```

`timestamp()`: Used for storing date and time values, but it's represented as Unix epoch format.

```js
this.column('created_at').timestamp();
```

`year()`: Used for storing a 4-digit year value.

```js
this.column('year').year();
```

`json()`: Used for storing JSON data (js Object Notation).

```js
this.column('details').json();
```

> The object must be stringified before insertion.

`enum()`: Used for storing a string value chosen from a list of values.

```js
this.column('size').enum('sm', 'md', 'lg');
```

> The column will only accept one of the following values: `'sm'`, `'md'`, or `'lg'`.

## Constraints

In this section, we'll go over different types of constraints you can use when defining your columns. Constraints are essential for maintaining data integrity and ensuring data in your database follows certain rules. They help enforce uniqueness, and data consistency in your tables. Below, you'll find the most commonly used constraints that you can apply to your columns.

`unique()`: Creates a unique constraint for the column. A unique constraint ensures that all values in the column are distinct, preventing duplicate entries.

```js
// Define a unique column
this.column('email').unique();

// Always Specify the column type first
this.column('email').varChar(200).unique();

// You can combine unique with other constraints, like NOT NULL
this.column('barcode').varChar(200).unique().notNull();
```

`primaryKey()`: Creates a primary key constraint for the column. A primary key constraint uniquely identifies each record in the table.

```js
// Define a primary key column
this.column('id').primaryKey();

// Always Specify the column type first
this.column('id').bigInt();

// Positive big integers column
this.column('id').bigInt().unsigned();

// Auto-incrementing positive big integers column
this.column('id').bigInt().unsigned().autoIncrement();

// Auto-incrementing positive big integers primary key column
this.column('id').bigInt().unsigned().autoIncrement().primaryKey();

// Auto-incrementing positive big integers primary key (short-cut)
this.column('id').pk();

// Auto-incrementing positive big integers primary key (short-cut)
this.primaryKey();
```

`foreignKey()`: Creates a foreign key constraint for the column. A foreign key constraint establishes a relationship between the current table and another table.

```js
const { CASCADE, SET_NULL } = require('bnjsx');

// Foreign key referencing the user from the profiles table
this.column('user_id').foreignKey().references('users', 'id');

// Foreign key with CASCADE ON DELETE
this.column('user_id').foreignKey().references('users', 'id').onDelete(CASCADE);

// Foreign key with SET NULL ON UPDATE
this.column('user_id')
  .foreignKey()
  .references('users', 'id')
  .onUpdate(SET_NULL);

// Foreign key with SET NULL ON UPDATE (short-cut)
this.column('user_id').fk().ref('users', 'id').onUpdate(SET_NULL);

// Foreign key without references (this will throw an error)
this.column('user_id').foreignKey(); // Undefined foreign key reference
```

- You must call `references()` to specify the referenced table and column.
- You must also specify the type of the column as well.
- The types of primary key and foreign key columns must be the same.
- I recommend using `fk()` and `pk()` to define your keys.
- `ref()` shortcut for: `references()`
- `fk()` shortcut for: `bigInt().unsigned().foreignKey()`
- `pk()` shortcut for: `bigInt().unsigned().autoIncrement().primaryKey()`
- `onUpdate(action)`: Specifies the behavior when the referenced row is updated.
- `onDelete(action)`: Specifies the behavior when the referenced row is deleted.
- Full list of actions:

  - `RESTRICT`: Prevents the referenced row from being updated/deleted.
  - `NO_ACTION`: Takes no action when the referenced row is updated/deleted.
  - `CASCADE`: Automatically updates/deletes the foreign key in the child table.
  - `SET_NULL`: Sets the foreign key column in the child table to `NULL`.
  - `SET_DEFAULT`: Sets the foreign key column in the child table to its default value.

`check(condition)`: Creates a CHECK constraint for the column. A CHECK constraint allows you to specify a condition that must be met for each row in the table.

```js
// Define a check constraint ensure the value is >= 18
this.column('age').check('age >= 18');

// Define a check constraint to ensure positive values
this.column('salary').check('salary > 0');

// Multiple check constraints can be applied to a single column
this.column('score')
  .check('score BETWEEN 0 AND 100')
  .check('score IS NOT NULL');
```

> I recommand to validate your values before insert and avoid using check constraint for faster insert queries

`index()`: Creates an index for the column. An index improves the speed of data retrieval operations on a database table. It allows the database engine to find and access data more efficiently.

```js
// Create an index for a username column
this.column('username').index();

// Create an index for an email column
this.column('email').index();
```

`default(value)`: Creates a default value for the column. This method sets a default value that will be used when no value is provided during insertion.

```js
// Set a default string value
this.column('status').default('active');

// Set a default numeric value
new SQLiteColumn('views').default(0);

// Set a default boolean value
new PostgreSQLColumn('is_active').default(false);

// Set a default null value
this.column('deleted_at').default(null);

// Attempting to set an invalid default value throws an error
this.column('invalid').default([]); // Invalid default value

// Function values are treated as strings (for now)
this.column('user_id').default('UUID()');
```
