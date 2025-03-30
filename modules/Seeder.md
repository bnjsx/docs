# Bnjsx Seeder

This module allows you to seed data into your database tables in Bnjsx. Each seeder file corresponds to a specific table, allowing for targeted and efficient seeding and clearing. Let's explore how to create and use seeders!

## Table of Contents

1. **[Adding Seeder Files](#adding-seeder-files)**
2. **[Seeder File Structure](#seeder-file-structure)**
3. **[Seeding Tables](#seeding-tables)**
4. **[Clearing Tables](#clearing-tables)**
5. **[Removing Seeder Files](#removing-seeder-files)**

## Adding Seeder Files

To create a seeder file, use the following command:

```bash
node exec mk:seeder <table>
```

To create a seeder for the `users` table, run:

```bash
node exec mk:seeder users
```

This command will output the following:

```bash
Seeder added in: ./seeders/01_seed_users_table.js
```

- Similar to `Generator` module, each seeder file is prefixed with a number, defining the order in which tables are seeded.
- This numbering is crucial, especially when your tables reference other tables through foreign keys. Always make sure seeder files are in the same order as generator files.

## Seeder File Structure

Below is an example of a seeder file for the `users` table:

```js
const { Seeder } = require('bnjsx');

class UsersTableSeeder extends Seeder {
  constructor() {
    super();

    this.set.rows(10); // Insert 10 rows
    this.set.table('users'); // Target the 'users' table
  }

  layout() {
    const faker = this.get.faker(); // Access a Faker instance

    return {
      username: faker.username(), // Generate a random username
      email: faker.email(), // Generate a random email
      password: faker.password(), // Generate a random password
      created_at: faker.datetime(), // Random datetime
      updated_at: faker.datetime(), // Random datetime
    };
  }
}

module.exports = new UsersTableSeeder(); // Export a new instance
```

### Code Overview

- **`this.set.rows(number)`**: Defines the number of rows to create (`number` must be > 0).
- **`this.set.table(name)`**: Specifies the target table for seeding.
- **`this.get.faker()`**: Accesses `Faker` instance to generate realistic, fake data for insertion.
  - You should be familiar with our `Faker` helper module.

### How it Works

1. The `layout` method is executed the specified number of times to generate rows.
2. Every time the `layout` method is called, it should return an object:
   - Object keys represent column names.
   - Object values represent data to insert.
3. An `INSERT` SQL query is built and executed.

## Seeding Tables

To seed all your tables, use:

```bash
node exec seed
```

To seed only one table, use:

```bash
node exec seed <table>
```

This command will seed the `users` table only:

```bash
node exec seed users
```

You can use this command to seed the same table multiple times!

- The first execution inserts the specified number of rows.
- The next execution inserts the same number of rows, and so on...

## Clearing Tables

To clear all tables with associated seeder files, use:

```bash
node exec clear
```

To clear only one table, use the following command:

```bash
node exec clear <table>
```

This command will clear all data from the `users` table only:

```bash
node exec clear users
```

## Removing Seeder Files

To remove a specific seeder file, use the following command:

```bash
node exec rm:seeder <table>
```

For example, consider the following seeders folder structure:

```
- 01_seed_users_table.js
- 02_seed_profiles_table.js
```

If you execute:

```bash
node exec rm:seeder users
```

The `users` seeder file will be removed, and the numbering will update automatically:

```
- 01_seed_profiles_table.js
```

That's it! Managing your seeder files has never been easier.
