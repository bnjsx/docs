## Bnjsx Builder

This module provides is flexible query builder, designed to simplify the process of constructing and executing SQL queries. With a clean, simple API.

## Table of Contents

1. **[Getting Started](#getting-started)**
2. **[SELECT Queries](#select-queries)**
3. **[Building Queries](#building-queries)**
4. **[Logging and Debugging](#logging-and-debugging)**
5. **[Where Clause](#where-clause)**
6. **[Condition Builder Methods](#condition-builder-methods)**
7. **[Joining Tables](#joining-tables)**
8. **[Grouping and Ordering](#grouping-and-ordering)**
9. **[Distinct, Limit and Offset](#distinct-limit-and-offset)**
10. **[Union and UnionAll](#union-and-unionall)**
11. **[Pagination and Count](#pagination-and-count)**
12. **[INSERT Queries](#insert-queries)**
13. **[UPDATE Queries](#update-queries)**
14. **[DELETE Queries](#delete-queries)**
15. **[Raw Queries](#raw-queries)**
16. **[Setter and Getter](#setter-and-getter)**
17. **[Query Classes](#query-classes)**

## Getting Started

1. Import `Builder` and `Bnjsx`

```js
const { Builder } = require('bnjsx');
const { Bnjsx } = require('bnjsx');
```

> You should be familiar with Bnjsx's `Pool` and `Cluster`.

2. Create a New Builder Instance

```js
const app = async () => {
  // Load your configuration
  const config = await Bnjsx.load();

  // Request a connection
  const connection = await config.cluster.request(config.default);

  // Create a builder instance with the connection
  const builder = new Builder(connection);

  // Fetch all users
  const users = await builder.select().from('users').exec();
  console.log(users);

  // Release the connection back when done
  connection.release();

  // Ensure not to reuse the connection after release...
};

app(); // Execute the app
```

> You must provide a `PoolConnection` instance for executing queries. `Connection` instances are not supported.

## SELECT Queries

The `select()` method returns a `Select` instance to construct and execute your `SELECT` queries:

```js
const { Select } = require('bnjsx');
console.log(builder.select() instanceof Select); // true
```

## Building Queries

The `build()` method is available on all query instances (`Select`, `Update`, `Delete`, `Insert`) to construct your SQL string.

```js
builder.insert().build();
// Throws an error because you haven't defined the table.
```

To select data from a table, use the `from(table)` method:

```js
builder.select().from('users').build();
// Returns: SELECT * FROM users;
```

By default, all columns (`*`) are selected. Use the `col` method to specify columns:

```js
builder.select().col('id', 'email').from('users').build();
// Returns: SELECT id, email FROM users;
```

You can also use aliases:

```js
builder.select().col('email AS mail').from('users').build();
// Returns: SELECT email AS mail FROM users;
```

## Logging and Debugging

You can log and retrieve your query and values using the `log` and `get` properties:

```js
const query = builder
  .select()
  .from('profiles')
  .where((col) => col('city').equal('Tokyo'));

query.log.query(); // Outputs: SELECT * FROM profiles WHERE city = ?;
query.log.values(); // Outputs: ['Tokyo'];

query.get.query(); // Returns: SELECT * FROM profiles WHERE city = ?;
query.get.values(); // Returns: ['Tokyo'];
```

> These features are available for `Select`, `Update`, `Delete`, and `Insert` queries.

## Where Clause

The `where(condition)` method lets you specify conditions for your query:

```js
builder
  .select()
  .from('users')
  .where((col) => col('id').equal(1))
  .build();
// Returns: SELECT * FROM users WHERE id = ?;
```

The `where(condition)` method uses a condition builder function. This function receives:

- **`col`**: A column selector function to specify the column and build conditions.
- **`con`**: A condition builder instance.

## Condition Builder Methods

`equal(values)`: Compares the column with a specified value for equality.

```js
builder
  .select()
  .from('profiles')
  .where((col) => col('age').equal(30))
  .build();
// Returns: SELECT FROM profiles WHERE age = ?;
```

`lessThan(value)`: Compares the column with a specified value to check if it's less than the value.

```js
builder
  .select()
  .from('products')
  .where((col) => col('price').lessThan(100))
  .build();
// Returns: SELECT FROM products WHERE price < ?;
```

`lessThanOrEqual(value)`: Compares the column with a specified value to check if it's less than or equal to the value.

```js
builder
  .select()
  .from('orders')
  .where((col) => col('order_date').lessThanOrEqual('2023-01-01'))
  .build();
// Returns: SELECT FROM orders WHERE order_date <= ?;
```

`greaterThan(value)`: Compares the column with a specified value to check if it's greater than the value.

```js
builder
  .select()
  .from('employees')
  .where((col) => col('salary').greaterThan(50000))
  .build();
// Returns: SELECT FROM employees WHERE salary > ?;
```

`greaterThanOrEqual(value)`: Compares the column with a specified value to check if it's greater than or equal to the value.

```js
builder
  .select()
  .from('profiles')
  .where((col) => col('age').greaterThanOrEqual(18))
  .build();
// Returns: SELECT FROM profiles WHERE age >= ?;
```

`between(start, end)`: Compares the column with a specified range to check if the value is between the `start` and `end` values.

```js
builder
  .select()
  .from('users')
  .where((col) => col('created_at').between('2023-01-01', '2023-12-31'))
  .build();
// Returns: SELECT FROM users WHERE created_at BETWEEN ? AND ?;
```

`in(values)`: Checks if the column's value is included in a list of specified values.

```js
builder
  .select()
  .from('profiles')
  .where((col) => col('status').in('active', 'pending'))
  .build();
// Returns: SELECT FROM profiles WHERE status IN (?, ?);
```

`inSubquery(subquery)`: Checks if the column's value is in the result set of a subquery.

```js
// Select customer ids from orders
const subquery = (select) => select.col('customer_id').from('orders');

// Select customers with at least one order
builder
  .select()
  .from('customers')
  .where((col) => col('id').inSubquery(subquery))
  .build();
// Returns:
// SELECT FROM customers
// WHERE id IN (SELECT customer_id FROM orders);
```

`like(value)`: Compares the column to a pattern using the `LIKE` operator.

```js
builder
  .select()
  .from('profiles')
  .where((col) => col('name').like('%john%'))
  .build();
// Returns: SELECT FROM profiles WHERE name LIKE ?;
```

> `%` means 0 or more characters.

`isNull()`: Checks if the column's value is `NULL`.

```js
builder
  .select()
  .from('profiles')
  .where((col) => col('gender').isNull())
  .build();
// Returns: SELECT FROM profiles WHERE gender IS NULL;
```

`not()`: Negates the current condition.

```js
builder
  .select()
  .from('profiles')
  .where((col) => col('name').not().like('%john%'))
  .build();
// Returns: SELECT FROM profiles WHERE name NOT LIKE ?;
```

`not()` must be called before the condition builder method

```js
builder
  .select()
  .from('profiles')
  .where((col) => col('name').like('%john%').not()) // No affect
  .build();
// Returns:  SELECT FROM profiles WHERE name LIKE ?;
```

> You can use `not()` with all condition builder methods

`and()`: Combines conditions with a logical **AND**.

```js
builder
  .select()
  .from('profiles')
  .where((col) => col('age').equal(20))
  .and()
  .where((col) => col('city').equal('NY'))
  .build();
// Returns: SELECT * FROM profiles WHERE age = ? AND city = ?;
```

You can chain `and()` directly.

```js
builder
  .select()
  .from('profiles')
  .where((col) => col('age').equal(20).and().col('city').equal('NY'))
  .build();
// Returns: SELECT * FROM profiles WHERE age = ? AND city = ?;
```

`or()`: Combines conditions with a logical **OR**.

```js
builder
  .select()
  .from('profiles')
  .where((col) => col('status').equal('inactive'))
  .or()
  .where((col) => col('status').equal('banned'))
  .build();
// Returns: SELECT * FROM profiles WHERE status = ? OR status = ?;
```

You can chain `or()` directly.

```js
builder
  .select()
  .from('profiles')
  .where((col) =>
    col('status').equal('inactive').or().col('status').equal('banned')
  )
  .build();
// Returns: SELECT * FROM profiles WHERE status = ? OR status = ?;
```

If the column is the same, call `col` only once.

```js
builder
  .select()
  .from('profiles')
  .where((col) => col('status').equal('inactive').or().equal('banned'))
  .build();
// Returns: SELECT * FROM profiles WHERE status = ? OR status = ?;
```

`open()` and `close()`: Adds explicit parentheses to group conditions.

```js
builder
  .select()
  .from('profiles')
  .where((col) => col('city').equal('NY'))
  .and()
  .open() // Start grouping with parentheses
  .where((col) => col('status').equal('inactive'))
  .or()
  .where((col) => col('status').equal('banned'))
  .close() // End grouping with parentheses
  .build();
// Returns:
// SELECT * FROM profiles
// WHERE city = ? AND (status = ? OR status = ?);
```

Nested parentheses are supported as well.

```js
builder
  .select()
  .from('profiles')
  .where((col) => col('city').equal('NY'))
  .and()
  .open() // Start outer parentheses
  .open() // Start inner parentheses
  .where((col) => col('status').equal('inactive'))
  .or()
  .where((col) => col('status').equal('banned'))
  .close() // Close inner parentheses
  .close() // Close outer parentheses
  .build();
// Returns:
// SELECT * FROM profiles
// WHERE city = ? AND ((status = ? OR status = ?));
```

`paren()`: Toggles parentheses, simplifying their usage.

```js
builder
  .select()
  .from('profiles')
  .where((col) => col('city').equal('NY'))
  .and()
  .paren() // Opens parentheses (or closes if already open)
  .where((col) => col('status').equal('inactive'))
  .or()
  .where((col) => col('status').equal('banned'))
  .paren() // Closes parentheses (or opens if already closed)
  .build();
// Returns:
// SELECT * FROM profiles
// WHERE city = ? AND (status = ? OR status = ?);
```

> `paren()` does not support nested parentheses. Use `open()` and `close()` for that.

`raw(condition, values)`: Adds a raw SQL condition string to the condition stack.

```js
builder
  .select()
  .from('profiles')
  .where((col) => col('status').equal('active').and().raw('age = ?', 18))
  .build();
// Returns: SELECT * FROM profiles WHERE status = ? AND age = ?;
```

`build()`: Builds and returns the final condition string.

```js
builder
  .select()
  .from('products')
  .where((col) => {
    const condition = col('price').between(20, 50).build();
    console.log(condition); // Outputs: price BETWEEN ? AND ?
  })
  .build();
// Returns: SELECT FROM products WHERE proce BETWEEN ? AND ?;
```

The build methods ensures your query is valid.

```js
builder
  .select()
  .from('products')
  .where(() => {}) // Empty condition
  .build();
// Throws: Invalid syntax: Condition cannot be empty.
```

```js
builder
  .select()
  .from('products')
  .where((col, con) => con.and()) // Invalid condition
  .build();
// Throws: Invalid syntax: Condition cannot start with an operator.
```

`exists(subquery)`: Checks if any rows exist based on the result of a subquery.

```js
// Import ref
const { ref } = require('bnjsx');

// Use `ref(column)` to ensure column references are not replaced with `?`
const subquery = (select) =>
  select
    .from('products')
    .where((col) => col('products.supplier_id').equal(ref('suppliers.id')))
    .and()
    .where((col) => col('products.price').lessThan(20));

// Select suppliers with at least one product
builder
  .select()
  .from('suppliers')
  .where((col, con) => con.exists(subquery));
// Returns:
// SELECT * FROM suppliers WHERE EXISTS (
// SELECT * FROM products
// WHERE products.supplier_id = suppliers.id AND products.price < ?
// );

// You can also use `raw(condition)` if you like
const subquery = (select) =>
  select
    .from('products')
    .where((col, con) => con.raw('products.supplier_id = suppliers.id'))
    .and()
    .where((col) => col('products.price').lessThan(20));

builder
  .select()
  .from('suppliers')
  .where((col, con) => con.exists(subquery));
// Returns:
// SELECT * FROM suppliers WHERE EXISTS (
// SELECT * FROM products
// WHERE products.supplier_id = suppliers.id AND products.price < ?
// );
```

`all(operator, subquery)`: Checks if the column value satisfies a condition when compared to **all results** from a subquery.

```js
// Import the Greater Than Operator
const { MORE } = require('bnjsx');

// Build a subquery to fetch the order_amounts of the first customer
const subquery = (select) =>
  select
    .col('order_amount')
    .from('orders')
    .where((col) => col('customer_id').equal(1));

// Fetch all orders with a higher order_amount
// than all orders placed by the first customer
builder
  .select()
  .from('orders')
  .where((col) => col('order_amount').all(MORE, subquery))
  .build();
// Returns:
// SELECT * FROM orders
// WHERE order_amount > ALL (
// SELECT order_amount FROM orders WHERE customer_id = ?
// );
```

> You can use these operators as well: `EQUAL`, `NOT_EQUAL`, `LESS`, `LESS_OR_EQUAL`, `MORE_OR_EQUAL`.

`any(operator, subquery)`: Checks if the column value satisfies a condition when compared to **any result** from a subquery.

```js
// Import the Greater Than Operator
const { MORE } = require('bnjsx');

// Build a subquery to fetch the order_amounts of the first customer
const subquery = (select) =>
  select
    .col('order_amount')
    .from('orders')
    .where((col) => col('customer_id').equal(1));

// Fetch all orders with a higher order_amount
// than any orders placed by the first customer
builder
  .select()
  .from('orders')
  .where((col) => col('order_amount').any(MORE, subquery))
  .build();
// Returns:
// SELECT * FROM orders
// WHERE order_amount > ANY (
// SELECT order_amount FROM orders WHERE customer_id = ?
// );
```

> You can use these operators as well: `EQUAL`, `NOT_EQUAL`, `LESS`, `LESS_OR_EQUAL`, `MORE_OR_EQUAL`.

`inDate(date)`: Compares the date portion of a `DATETIME`, `TIMESTAMP`, or `DATE` column with a specified date value.

```js
builder
  .select()
  .from('products')
  .where((col) => col('created_at').inDate('2023-05-01'))
  .build();
// Returns: SELECT * FROM products WHERE DATE(created_at) = ?;
```

`inTime(time)`: Compares the time portion of a `DATETIME`, `TIMESTAMP`, or `TIME` column with a specified time value.

```js
builder
  .select()
  .from('products')
  .where((col) => col('created_at').inTime('15:30:00'))
  .build();
// Returns: SELECT * FROM products WHERE TIME(created_at) = ?;
```

`inYear(year)`: Compares the year portion of a `DATETIME`, `TIMESTAMP`, or `DATE` column with a specified year value.

```js
builder
  .select()
  .from('products')
  .where((col) => col('created_at').inYear('2023'))
  .build();
// Returns: SELECT * FROM products WHERE YEAR(created_at) = ?;
```

`inMonth(month)`: Compares the month portion of a `DATETIME`, `TIMESTAMP`, or `DATE` column with a specified month value.

```js
builder
  .select()
  .from('products')
  .where((col) => col('created_at').inMonth(3)) // 1 - 12
  .build();
// Returns: SELECT * FROM products WHERE MONTH(created_at) = ?;
```

`inDay(day)`: Compares the day portion of a `DATETIME`, `TIMESTAMP`, or `DATE` column with a specified day value.

```js
builder
  .select()
  .from('products')
  .where((col) => col('created_at').inDay(20)) // 1 - 31
  .build();
// Returns: SELECT * FROM products WHERE DAY(created_at) = ?;
```

`inHour(hour)`: Compares the hour portion of a `DATETIME`, `TIMESTAMP`, or `TIME` column with a specified hour value.

```js
builder
  .select()
  .from('products')
  .where((col) => col('created_at').inHour(16)) // 0 - 23
  .build();
// Returns: SELECT * FROM products WHERE HOUR(created_at) = ?;
```

`inMinute(minute)`: Compares the minute portion of a `DATETIME`, `TIMESTAMP`, or `TIME` column with a specified minute value.

```js
builder
  .select()
  .from('products')
  .where((col) => col('created_at').inMinute(30)) // 0 - 59
  .build();
// Returns: SELECT * FROM products WHERE MINUTE(created_at) = ?;
```

`inSecond(second)`: Compares the second portion of a `DATETIME`, `TIMESTAMP`, or `TIME` column with a specified second value.

```js
builder
  .select()
  .from('products')
  .where((col) => col('created_at').inSecond(45)) // 0 - 59
  .build();
// Returns: SELECT * FROM products WHERE SECOND(created_at) = ?;
```

> The extract function changes based on your driver.

## Joining Tables

`join(table, condition)`: Adds an `INNER JOIN` clause to the query.

```js
// Import ref
const { ref } = require('bnjsx');

// Use `ref(column)` to ensure column references are not replaced with `?`
builder
  .select()
  .from('users')
  .join('orders', (col) =>
    col('users.id')
      .equal(ref('orders.user_id'))
      .and()
      .col('orders.status')
      .equal('pending')
  )
  .build();
// Returns:
// SELECT * FROM users
// INNER JOIN orders ON users.id = orders.user_id AND orders.status = ?;
```

You can join as many tables as you like.

```js
builder
  .select()
  .col('users.*', 'profiles.name', 'orders.status')
  .from('users')
  .join('profiles', (col) => col('users.id').equal(ref('profiles.user_id')));
  .join('orders', (col) => col('users.id').equal(ref('orders.user_id')))
  .build();
// Returns:
// SELECT users.*, profiles.name, orders.status FROM users
// INNER JOIN profiles ON users.id = profiles.user_id
// INNER JOIN orders ON user.id = orders.user_id;
```

`leftJoin(table, condition)`: Adds a `LEFT JOIN` clause to the query.

```js
// Import ref
const { ref } = require('bnjsx');

// Fetch all users and their orders
builder
  .select()
  .from('users')
  .leftJoin('orders', (col) => col('users.id').equal(ref('orders.user_id')))
  .build();
// Returns:
// SELECT * FROM users LEFT JOIN orders ON users.id = orders.user_id;
```

`rightJoin(table, condition)`: Adds a `LEFT JOIN` clause to the query.

```js
// Import ref
const { ref } = require('bnjsx');

// Fetch all orders and their users
builder
  .select()
  .from('users')
  .rightJoin('orders', (col) => col('users.id').equal(ref('orders.user_id')))
  .build();
// Returns:
// SELECT * FROM users RIGHT JOIN orders ON users.id = orders.user_id;
```

## Grouping and Ordering

`groupBy(columns)`: Adds a `GROUP BY` clause to the query.

```js
// Group products by category and price
builder.select().from('products').groupBy('category', 'price').build();
// Returns: SELECT * FROM products GROUP BY category, price;
```

`orderBy(column, type)`: Adds an `ORDER BY` clause to the query.

```js
// Import Order types
const { DESC, ASC } = require('bnjsx');

// Order products by price in descending order
new Select(connection).from('products').orderBy('price', DESC).build();
// Returns: SELECT * FROM products ORDER BY price DESC;

// Order products by price in ascending order
new Select(connection).from('products').orderBy('price', ASC).build();
// Returns: SELECT * FROM products ORDER BY price ASC;
```

> `type` is optional, and `ASC` is the default order type.

`having(condition)`: Adds a `HAVING` clause to the query, typically used after a `GROUP BY`. This allows you to filter the grouped results based on aggregate functions (like `SUM()`, `COUNT()`, etc.) which can't be filtered using the `WHERE` clause.

```js
// Get products with total sales greater than 1000
new Select(connection)
  .from('sales') // The sales table stores product_id, price
  .groupBy('product_id')
  .having((col) => col('SUM(price)').greaterThan(1000))
  .build();
// Returns:
// SELECT * FROM sales GROUP BY product_id HAVING SUM(price) > ?;
```

> Use `having()` after `groupBy()` when you need to apply conditions on aggregated results, such as filtering groups based on their summed, counted, or averaged values.

In summary:

- `groupBy()` organizes your data into groups.
- `orderBy()` arranges those groups in a specific order.
- `having()` applies filters to those grouped and ordered results, typically with aggregate functions.

## Distinct, Limit and Offset

`distinct()`: ensuring the results are unique. This eliminates duplicate rows from the result set.

```js
// Select distinct product names
builder.select().distinct().col('name').from('products').build();
// Returns: SELECT DISTINCT name FROM products;

// Select distinct countries
builder.select().distinct().col('country').from('profiles').build();
// Returns: SELECT DISTINCT country FROM profiles;
```

`limit(number)`: limits the number of rows to return.
`offset(number)`: skip a certain number of rows and start from a specific point.

```js
// Get the first 10 users
builder.select().from('users').limit(10).build();
// Returns: SELECT * FROM users LIMIT 10;

// Get 5 products, starting from the 6th
builder
  .select()
  .from('products')
  .offset(5) // Start from the 6th product
  .limit(5) // Limit to 5 products
  .build();
// Returns: SELECT * FROM products OFFSET 5 LIMIT 5;
```

## Union and UnionAll

`union(subquery)`: Adds a `UNION` clause to the query, combining the current query’s results with the results of a subquery.

```js
// Get users with a gold membership
// and combine them with VIP membership users
builder
  .select()
  .from('users')
  .where((col) => col('membership').equal('gold'))
  .union((select) => {
    select.from('users').where((col) => col('membership').equal('vip'));
  })
  .build();
// Returns:
// SELECT * FROM users WHERE membership = ?
// UNION
// SELECT * FROM users WHERE membership = ?;
```

> `union()` ensures that any duplicate rows are removed in the final result. It is useful when you want to merge results from multiple queries but avoid repetition.

`unionAll(subquery)`: Adds a `UNION ALL` clause to the query, combining the current query’s results with the results of a subquery. Unlike `UNION`, `UNION ALL` keeps all rows, including duplicates, in the combined result set.

```js
// Get users with a gold membership
// and combine them with VIP membership users
builder
  .select()
  .from('users') // First query: Get gold members
  .where((col) => col('membership').equal('gold'))
  .unionAll((select) => {
    select // Second query: Get VIP members
      .from('users')
      .where((col) => col('membership').equal('vip'));
  })
  .build();
// Returns:
// SELECT * FROM users WHERE membership = ?
// UNION ALL
// SELECT * FROM users WHERE membership = ?;
```

> `unionAll()` does **not** remove duplicates, allowing all results to be returned as they are. It is used when you want to merge results from multiple queries and keep all entries, including repetitions.

## Pagination and Count

`count()`: returns the total number of rows that match the current query conditions. This is particularly useful when you need to know the total number of items that meet specific filters or criteria, such as how many blog posts are published or how many products belong to a certain category.

```js
// Get the total number of products in the "Laptops" category
builder
  .select()
  .from('products')
  .where((col) => col('category').equal('Laptops'))
  .count()
  .then((count) => console.log(count));
// Outputs: The total number of laptops available
```

`paginate()`: breaks down large result sets into smaller, paginated chunks. This is useful when you want to display results in a paginated format, such as showing a subset of products or blog posts per page.

```js
// Get the first page of products in the "Laptops" category
// 10 products per page
builder
  .select()
  .from('products')
  .where((col) => col('category').equal('Laptops'))
  .paginate(1, 10)
  .then((pagination) => console.log(pagination));
// Example output:
// {
//   result: [{ id: 1, name: 'Laptop A' }, ...], // Products
//   page: {
//     current: 1,  // Current page number
//     prev: undefined,  // No previous page on the first page
//     next: 2,  // Next page is page 2
//     items: 10,  // 10 products per page
//   },
//   total: {
//     pages: 5,  // Total of 5 pages
//     items: 50,  // Total of 50 laptops available
//   }
// }
```

### Notes

- If `next` is `undefined`, it indicates the user is on the last page, so you can hide the `Next` button.
- If `prev` is `undefined`, it indicates the user is on the first page, so you can hide the `Back` button.
- Use `total.pages` and `total.items` to show total pages and items to users.
- You can also use `paginate` to implement **infinite scrolling** in your app, loading more items as the user scrolls down.

## INSERT Queries

The `builder.insert()` method allows you to create and execute `INSERT` queries.

- Insert a single row into your table.

```js
const id = await builder
  .insert()
  .into('users')
  .row({ email: 'example@gmail.com', password: '123' })
  .build();
// Returns:
// INSERT INTO users (email, password) VALUES (?, ?);
```

- Insert multiple rows into your table.

```js
await builder
  .insert()
  .into('users')
  .row([
    { email: 'example1@gmail.com', password: '123' },
    { email: 'example2@gmail.com', password: '123' },
  ])
  .build();
// Returns:
// INSERT INTO users (email, password) VALUES (?, ?), (?, ?);
```

`returning(...columns)`: Specify columns to return after inserting data (`PostgreSQL` only).

```js
// Insert one user and return the ID
const id = await builder
  .insert()
  .into('users')
  .row({ email: 'example@gmail.com', password: '123' })
  .returning('id')
  .exec();
console.log(id); // { id: 1 }

// Insert multiple users and return their IDs
const ids = await builder
  .insert()
  .into('users')
  .row([
    { email: 'example1@gmail.com', password: '123' },
    { email: 'example2@gmail.com', password: '123' },
  ])
  .returning('id')
  .exec();
console.log(ids); // [{ id: 1 }, { id: 2 }]
```

### Notes

- The `row` object represents the data you want to insert.
  - Keys are the column names like: `email`, `password`.
  - Values are the data to insert like: `'example@gmail.com'`, `123`.
- You can insert the following types:
  - `Strings` like: `'example@gmail.com'`
  - `Numbers` like: `18`, `20.15`
  - `Null` for nullable columns
- The `exec()` method returns the primary key value for `MySQL` and `SQLite` drivers when inserting a single row.
- The `exec()` method returns `undefined` for `MySQL` and `SQLite` drivers when inserting multiple rows.

## UPDATE Queries

The `builder.update()` method allows you to create and execute `UPDATE` queries.

```js
builder
  .update()
  .table('orders')
  .set({ status: 'shipped' })
  .paren()
  .where((col) => col('status').equal('pending'))
  .and()
  .where((col) => col('order_date').lessThan('2024-12-01'))
  .paren()
  .build();
// Returns:
// UPDATE orders SET status = ?
// WHERE (status = ? AND order_date < ?);
```

Set columns to `NULL`

```js
builder
  .update()
  .table('profiles')
  .set({ gender: 'male', city: null, bio: null })
  .where((col) => col('user_id').equal(100))
  .build();
// Returns:
// UPDATE profiles
// SET gender = ?, city = NULL, bio = NULL
// WHERE user_id = ?;
```

### Notes

- Use `table(name)` to specify the `UPDATE` table.
- Use `set(row)` to specify your `UPDATE` columns and values.
- Use `where(condition)` to build your `UPDATE` condition.
- Use `build()` to build and access your `UPDATE` query.
- Use `exec()` to execute your `UPDATE` query.
  - This method always resolves with `undefined` in `UPDATE` queries.

## DELETE Queries

The `builder.delete()` method allows you to create and execute `DELETE` queries.

```js
builder
  .delete()
  .from('profiles')
  .where((col) => col('user_id').equal(100))
  .build();
// Returns: DELETE FROM profiles WHERE user_id = ?;
```

### Notes

- Use `from(table)` to specify the `DELETE` table.
- Use `where(condition)` to build your `DELETE` condition.
- Use `build()` to build and access your `DELETE` query.
- Use `exec()` to execute your `DELETE` query.
  - This method always resolves with `undefined` in `DELETE` queries.

## Raw Queries

The `builder.raw(sql, ...values)` method allows you to execute raw queries.

```js
// Select user by ID
await builder.raw('SELECT * FROM users WHERE id = ?;', [1]);

// Delete user by ID
await builder.raw('DELETE FROM users WHERE id = ?;', [1]);

// Update user's email
await builder.raw('UPDATE users SET email = ? WHERE id = ?;', [
  'updated@gmail.com',
  1,
]);

// Insert a new user
await builder.raw('INSERT INTO users (email, password) VALUES (?, ?);', [
  'second@gmail.com',
  '123',
]);
```

> Use `?` as a placeholder for values in your query. Pass the corresponding values as arguments after the SQL string to prevent SQL injection.

## Setter and Getter

- `builder.set` and `builder.get` allows you set and get the connection your builder instance is using and that's very helpfull asspecially when you decide to build and execute queries in multiple databases

```js
const { Builder } = require('bnjsx');
const { Bnjsx } = require('bnjsx');

// Imagine you have two pools in your cluster: 'asia' and 'africa'
// You want to SELECT users from both the Asian and African pools
const app = async () => {
  // Load your configuration
  const config = await Bnjsx.load();

  // Request a connection from `asia`
  const con1 = await config.cluster.request('asia');

  // Create a builder instance with `con1`
  const builder = new Builder(con1);

  // Fetch all users
  console.log(await builder.select().from('users').exec());

  // Release the `con1` back when done
  con1.release();

  // Now request another connection from `africa`
  const con2 = await config.cluster.request('africa');

  // Update the builder connection
  builder.set.connection(con2);

  // Fetch all users
  console.log(await builder.select().from('users').exec());

  // Release the `con2` back when done
  con2.release();
};

app(); // Execute the app
```

## Query Classes

In Bnjsx, we have four query classes: `Select`, `Update`, `Delete`, and `Insert`. You can use them to build and execute `SELECT`, `UPDATE`, `DELETE`, and `INSERT` queries.

All query classes extend the base class `Query`. That's why you will see methods in common:

- `exec()`: executes the query.
- `build()`: builds and returns the query string.
- `raw(query, values)`: executes a raw query.
- `reset()`: resets the query instance state.
- `log`: logs the query string and values.
- `get`: retrieves the query string and values.

```js
// Import Select, Update, Delete, and Insert
const { Select } = require('bnjsx');
const { Update } = require('bnjsx');
const { Delete } = require('bnjsx');
const { Insert } = require('bnjsx');

// Import Bnjsx
const { Bnjsx } = require('bnjsx');

// Load config
const config = await Bnjsx.load();

// Request a connection
const connection = await config.cluster.request(config.default);

// Create instances
const _select = new Select(connection);
const _update = new Update(connection);
const _delete = new Delete(connection);
const _insert = new Insert(connection);

// Build & Execute SELECT query
const user = await _select
  .from('users')
  .where((col) => col('email').equal('example@gmail.com'))
  .exec();

// Log the result
console.log(user); // [ { id: 1, email: 'example@gmail.com', ... } ]

// Reset your query instance
_select.reset();

// Now you can build and execute another one
const profiles = await _select
  .from('profiles')
  .where((col) => col('user_id').equal(user.id))
  .exec();

// Log the result
console.log(profiles); // [ { id: 1, user_id: 1, name: 'john', ... } ]

// Build & Execute DELETE query
await _delete
  .from('profiles')
  .where((col) => col('user_id').equal(1))
  .exec();

await _delete
  .reset() // Reset
  .from('users')
  .where((col) => col('id').equal(1))
  .exec();

// Build & Execute UPDATE query
await _update
  .table('profiles')
  .set({ bio: 'Bnjsx is More Than Just ORM' })
  .where((col) => col('user_id').equal(22))
  .exec();

// Build & Execute INSERT query
await _insert
  .into('users')
  .row({ email: 'Bnjsx@gmail.com', password: '123' })
  .exec();

// Release the connection back when done!
connection.release();

// You should not use the connection or queries from this point on...
```

> I recommend using the builder because it simplifies your workflow. With the builder, you create one instance and can execute `SELECT`, `UPDATE`, `DELETE`, `INSERT`, and raw queries using the same connection.

> In contrast, if you use the `Select`, `Update`, `Delete`, or `Insert` classes directly, you must create a new instance every time you want to execute a query.
