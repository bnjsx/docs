# Bnjsx Flex Template Engine

**Flex** is a dynamic, **component-based template engine** that lets you create reusable UI components with ease. It adapts to different data inputs, so each component renders uniquely based on the context, making your interface more txtible and responsive.

**Why Use It?**
Use Flex to streamline your development with clean, modular, and reusable components that scale with your project. It simplifies dynamic content rendering and reduces the complexity of building responsive, data-driven UIs.

## Table of Contents

1. [Official VS Code Extension](#official-vs-code-extension)
2. [What is Rendering in Flex?](#what-is-rendering-in-flex)
3. [Flex Statements](#flex-statements)
   - [Print Statement](#print-statement)
   - [Short Print Syntax](#short-print-syntax)
   - [Log Statement](#log-statement)
   - [If Statement](#if-statement)
   - [Foreach Statement](#foreach-statement)
   - [Render Statement](#render-statement)
   - [Replacements](#replacements)
4. [Tools And Globals](#tools-and-globals)
   - [Global Variables](#global-variables)
   - [Tools (aka template superpowers)](#tools-aka-template-superpowers)

## Official VS Code Extension

To make working with Flex templates even easier, we’ve created **Flexer**, the official VS Code extension for **Flex**. It provides syntax highlighting, auto-completion, and formatting for `.fx` files, making your development experience smoother and more efficient.

You can install **Flexer** from the [VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=BnjsxTeam.txter), and start using its features right away:

- **Syntax Highlighting** for Flex statements, HTML, and JavaScript.
- **Auto-Completion** for Flex statements, HTML elements, and Tailwind CSS classes.
- **Code Formatting** to keep your `.fx` files clean and properly indented.

We encourage you to contribute to **Flexer** if you can. Whether it’s fixing bugs, adding new features, or improving the code, your help will make it even better. You can find the extension's repository on [GitHub](https://github.com/bnjsx/txter) and submit your pull requests.

Your contributions are highly appreciated!

## What is Rendering in Flex?

In Flex, rendering is the process of displaying a component with specific values provided at runtime. A component is essentially a template that can display dynamic content based on the values you pass to it. Each time you render a component with different data, the output will be unique, making your UI txtible and adaptive.

For example, consider a simple component for a button:

```txt
<button>$(title)</button>
```

When you render this component, you can provide the value of `title`:

```js
render('button', { title: 'Hello World' }).then(console.log).catch(console.log);
```

The output would be:

```txt
<button>Hello World</button>
```

Alternatively, if you render it with a different `title`:

```js
render('button', { title: 'Nice' }).then(console.log).catch(console.log);
```

The output would change accordingly:

```txt
<button>Nice</button>
```

Components in Flex are defined using `.fx` files. You can render these components in two primary ways:

1. **Using the `render` function**: This method renders a component and returns the resulting template.
2. **Using the `$render` statement**: This allows you to render a component inside another component.

**Component-Based Architecture**

Flex is a component-based template engine, similar to React, where you can create reusable components that can be included or rendered in any part of your application. Let's explore how this works in practice.

**Setting Up Your Views Folder**

To render a component, it must be stored in a `views` directory (or any directory you choose). You can configure the path to this folder in your `bnjsx.config.js`.

**Example Configuration**

```js
paths: {
  views: 'absolute/path/to/views/folder',
}
```

This tells Flex where to look for your components when rendering them. Now, whenever you render a component, Flex will automatically search the `views` folder and return the parsed result.

**Rendering Components with the `render` Function**

To render a component from the `views` folder, you simply provide the component name without including the `views` folder in the path.

**Example: Render a Component**

```js
const { render } = require('bnjsx');

render('button').then(console.log).catch(console.log);
```

Here, we’re rendering a `button.fx` component located in the `views` folder. Notice that we don't need to specify the folder itself in the path—just the component's name.

**Using Dot Notation for Nested Components**

Flex supports dot notation for paths, which makes referencing nested components cleaner and more intuitive. Consider the following folder structure:

```
views
├── components
│   ├── button.fx
│   └── input.fx
├── pages
│   ├── home.fx
│   ├── register.fx
│   └── layout.fx
```

To render the `input.fx` component located in the `components` folder, use:

```js
render('components.input').then(console.log).catch(console.log);
```

To render the `home.fx` component in the `pages` folder, use:

```js
render('pages.home').then(console.log).catch(console.log);
```

This dot notation approach makes it easier to organize and access components, especially as your project grows.

**Asynchronous Rendering**

All render operations are asynchronous, ensuring high performance and speed. The `render` function returns a promise that resolves with the rendered component.

**Error Handling**

Flex’s error messages are descriptive and provide exact line numbers where the error occurred, just like in JavaScript. These messages will help you identify parsing or logic errors quickly. As long as your components are defined correctly, there’s no need to worry about errors in production.

**Rendering and Sending Response with `Response.proyotype.render`**

For a simpler workflow, you can use `Response.proyotype.render` to render and send the component to the client in one step. This method simplifies the process for you.

**Using `Response.proyotype.render`**

```js
const { App, Router } = require('bnjsx');

const app = new App();
const router = new Router();

router.get('/', async (_, res) => res.render('pages.home'));

// Register router
app.namespace('/', router);

// Start the app
app.start();
```

This method automatically handles the rendering and sends the result to the client, which can save time and simplify your code.

**Manual Rendering**

If you prefer more control, you can manually handle the rendering like this:

```js
// Send has automatic content type detection
// Status code is `200 OK` by default
router.get('/', async (req, res) => {
  return res.send(await render('pages.home'));
});
```

While this approach works, **using `Response.proyotype.render`** is cleaner and more concise, reducing the potential for messy, spaghetti code.

Now that you know how to render components in Flex, you can start building dynamic, reusable components that can be easily nested and composed. In the next section, we’ll dive into the basics of the **Flex Engine** and learn how to render components from within other components.

## Flex Statements

Flex includes a set of **statements** that you can use within your templates to manage dynamic content.

### Print Statement

The **print statement** allows you to output values within your template. While you can use it to print static values, it's primarily used for displaying dynamic data.

#### Supported Values

You can use `$print()` to print the following types of values:

- **Scalar** – Strings, numbers, booleans, `null`, and `undefined`.
- **Tool** – The return value of a tool function (explained later).
- **Local** – A reference to a value accessible within a specific component.
- **Global** – A reference to a globally accessible value.

#### Printing Scalar Values

```txt
$print('Hello, World!')
```

```txt
$print(123)
```

```txt
$print(true)
```

```txt
$print(undefined)
```

```txt
$print(null)
```

#### Printing a Tool Return Value

```txt
$print(@year())
```

> A **tool** is a function that you can define and execute. We will cover tools in detail later.

#### Printing a Local Reference

```txt
$print(user.name)
```

```txt
$print(friends.length)
```

```txt
$print(friends[0].name)
```

> Use **dot notation** for object properties (`user.name`) and **bracket notation** for array items (`friends[0]`). However, bracket notation for object properties (`user['name']`) is not supported yet.

#### Printing a Global Reference

```txt
$print(#app_name)
```

> A **global reference** is a variable defined and accessible across all components.

### Short Print Syntax

Flex also provides a **short print statement**: `$(value)`, which is a more concise way to print values.

Here are the examples split individually:

```txt
$(@year())
```

```txt
$(@month())
```

```txt
$(#app_name)
```

```txt
$(name)
```

```txt
$(age)
```

### Log Statement

The **log statement** is useful for debugging. It allows you to log various values directly within your template, including **scalars, tool return values, global variables, and local variables**.

#### ⚠️ Limitations

- **Only one value at a time** – Logging multiple values like `$log(name, age)` **is not supported** and will result in an error.
- **Execution-dependent logging** – The log statement will only output the value **if execution reaches it**. If an error occurs before the log statement, it will not be executed.

#### Logging a Tool Return Value

```txt
$log(@year())
```

#### Logging a Global Value

```txt
$log(#app_name)
```

#### Logging a Local Value

```txt
$log(age)
```

#### Logging a Scalar Value

```txt
$log('Hello, World!')
```

### If Statement

Conditions are essential in any template engine, in `Flex` template engine, you can use `$if`, `$elseif`, and `$else` to control what gets rendered dynamically. Let’s dive in!

#### Basic If Statement

Need to show something only if a condition is met? Use `$if`.

```txt
$if(user)
  Welcome back $(user.name)!
$endif
```

> If `user` exists, the template will show "Welcome back [user's name]!" Otherwise, nothing will appear.

#### If-Else Statement

What if you want to show different content when the condition is false? Use `$else`!

```txt
$if(user.active)
  $(user.name) is active!
$else
  $(user.name) is not active!
$endif
```

> If `user.active` is `true`, it displays "[user's name] is active!" Otherwise, it says "not active".

#### If-ElseIf-Else Statement

Sometimes, you need more than just two options. That’s where `$elseif` comes in!

```txt
$if(user.country === 'japan')
  $(user.name) is living in Japan!
$elseif(user.country === 'france')
  $(user.name) is living in France!
$else
  $(user.name) is living somewhere!
$endif
```

> This checks the user’s country. If they’re from Japan, it displays that. If not, it checks for France. If neither, it shows a fallback message.

> Of course! You can use as many `$elseif` statements as you need, but just like in JavaScript, you can only have one `$if` to start and a single `$else` at the end.

#### Nesting If Statements

Yes, you can go as deep as you want! Here’s an example:

```txt
$if(user)
  $if(user.name)
   User name is: $(user.name)
  $endif
  $if(user.country)
   User country is: $(user.country)
  $endif
  $if(user.friends)
    $if(user.friends[0])
      First friend: $(user.friends[0].name)
    $endif
    $if(user.friends[1])
     Second friend: $(user.friends[1].name)
    $endif
  $endif
$endif
```

> This ensures that each piece of data exists before accessing it, preventing errors while dynamically displaying information.

#### Complex Conditions? No Problem!

Flex supports JavaScript-like conditions, so you can combine logic as needed.

```txt
$if(user && user.name)
  User name is: $(user.name)
$endif
```

```txt
$if(!user.country)
  User country is unknown
$endif
```

> Here, the first condition ensures both `user` and `user.name` exist before printing. The second condition checks if `user.country` is missing and prints "unknown" if so.

#### Supported Operators

| Type        | Operators Supported                            |
| ----------- | ---------------------------------------------- |
| Comparison  | `===`, `==`, `!==`, `!=`, `>=`, `<=`, `<`, `>` |
| Logical     | `&&`, `\|\|`                                   |
| Negation    | `!`                                            |
| Parentheses | `()` (higher precedence)                       |

#### More Condition Examples

```txt
$if('hello' === 'hello') Content shown $endif
```

```txt
$if(true) Content shown $endif
```

```txt
$if(false) No content $endif
```

```txt
$if(123) Content shown $endif
```

```txt
$if(123 < 0) No content $endif
```

```txt
$if(true && (false || true) && 'hello') Content shown $endif
```

> You can use `numbers`, `strings`, `booleans`, `null` and `undefined` directly inside conditions.

```txt
$if(@year() === 2025) Happy New Year 2025! $endif
```

```txt
$if(@fetchUserById(2000).name === 'james')
  If user 2000 is James, show this.
$endif
```

> You can even define and execute functions, pass arguments, and use their return values in conditions, _(whether they’re sync or async)_.

> The example above fetches a user from the database and checks if the name is "James." While this shows what `Flex` is capable of, I recommend keeping complex logic, like database queries, in your controllers for better structure and maintainability.

```txt
$if(#app_name === 'bnjsx') Welcome to Bnjsx! $endif
```

> You can use global variables in your conditions too! In this example, if the global variable `#app_name` is set to `"bnjsx"`, the message **"Welcome to Bnjsx!"** will be displayed.

#### Final Thoughts

Use `$if`, `$elseif`, and `$else` to make your templates dynamic and responsive. Go wild with nested conditions, combine logic, and let Flex handle the rest!

### Foreach Statement

`Flex` supports looping, which is useful when you need to iterate over a list of items _(like posts)_ and display them dynamically on your page. To achieve this, you can use the `$foreach` statement.

```txt
$foreach(post, posts)
  Post Title: $(post.title)
$endforeach
```

> This will repeat the content for each item in the `posts` variable.

#### Nested Foreach Statements

You can also use nested `$foreach` statements to iterate over sub-collections within the main collection:

```txt
$foreach(post, posts)
  Post Title: $(post.title)
  $foreach(category, post.categories)
    Post Category: $(category.name)
  $endforeach
$endforeach
```

> In this example, we iterate over the `posts` collection. For each post, we iterate over its `categories` and display each category's name.

#### Combining Multiple Statements

You can combine various statements to control the flow of the loop. For example, you can check if a collection exists before iterating over it:

```txt
$if(posts)
  $foreach(post, posts)
    Post Title: $(post.title)
    $if(post.categories)
      $foreach(category, post.categories)
        Post Category: $(category.name)
      $endforeach
    $endif
  $endforeach
$endif
```

> In this example, we first check if the `posts` collection exists. Then, for each post, we check if it has categories. Only if it does, we iterate over the categories and display them.

#### Scope in Foreach Loops

The first argument of `$foreach` represents the current item in the iteration. Inside, you can use this variable, but it will be **undefined** outside.

`Flex` supports **scopes**, similar to JavaScript. Variables declared in a `$foreach` statement are available in all nested statements. However, if you declare a variable with the same name in a nested `$foreach` statement, it will **overwrite** the outer variable.

Here’s an example demonstrating how scope works in `Flex`:

```txt
$foreach(post, posts)
  $log(post)
  $foreach(category, post.categories)
    $log(post)
    $log(category)
  $endforeach
$endforeach
```

> In this example, we first iterate over the `posts` collection and log each post. Then, we iterate over each post's `categories` and log both the post and category. The outer `post` variable is accessible in all nested loops unless overwritten.

#### Variable Shadowing

If you use the same variable name inside a nested loop, it will **overwrite** the outer variable. Here's an example:

```txt
$foreach(post, posts)
  $log(post)
  $foreach(post, post.categories)
    $log(post)
  $endforeach
$endforeach
```

> In this case, we're iterating over the `categories` of each post. Inside the nested `$foreach`, the `post` variable now refers to a category, not the original post. So, be careful when reusing variable names in nested loops.

#### Accessing Loop Index

You can also access the index of each item in the loop. The second argument in `$foreach` represents the index of the current item in the collection.

```txt
$foreach(post, index, posts)
  $log(post)
  $log(index)
$endforeach
```

> You can name your variables whatever you like; they don't have to be `post` and `index`. Just ensure you reference them correctly.

### Render Statement

While you can render a Flex component using the `render` method, you can also use the `$render` statement to achieve the same result with more txtibility.

Imagine we have two pages, `register.fx` and `login.fx`, both requiring input fields and buttons with different titles. Instead of copying and pasting the same HTML and customizing it, you can create dynamic components and render them wherever needed.

Let’s start by creating a dynamic `button.fx` component:

```txt
<button> $(title) </button>
```

This component expects a `title` to be provided during rendering. If you don’t pass a title, the button will show `undefined`:

```txt
<button> undefined </button>
```

To handle this, you can check if the title is provided and use a default value:

```txt
<button> $if(title) $(title) $else Submit $endif </button>
```

Now, if no title is provided, the button will display `Submit` by default:

```js
render('components.button').then(console.log).catch(console.log);
```

This outputs:

```txt
<button> Submit </button>
```

If you provide a title:

```js
render('components.button', { title: 'Login' })
  .then(console.log)
  .catch(console.log);
```

The result will be:

```txt
<button> Login </button>
```

Now, in your `login.fx` page, you can simply use this component:

```txt
<form>
  <input name='email' type='text'/>
  <input name='password' type='password'/>
  $render('components.button', title='Login')$endrender
</form>
```

The output will be:

```txt
<form>
  <input name='email' type='text'/>
  <input name='password' type='password'/>
  <button> Login </button>
</form>
```

As you can see, the component is rendered exactly where you placed the `$render` statement. However, the button type is not set, so it won’t submit the form yet.

Let’s update the `button.fx` component to accept the `type` attribute:

```txt
<button $if(type) type='$(type)' $endif>
  $if(title) $(title) $else Submit $endif
</button>
```

Now, in the `login.fx` page, we can pass the `type` as `submit`:

```txt
<form action='/login' method='POST'>
  <input name='email' type='text'/>
  <input name='password' type='password'/>
  $render('components.button', title='Login', type='submit')$endrender
</form>
```

The result will now be:

```txt
<form action='/login' method='POST'>
  <input name='email' type='text'/>
  <input name='password' type='password'/>
  <button type='submit'> Login </button>
</form>
```

And there you go! You've just created a reusable, dynamic `button` component that you can use across your project without duplicating code. The real power comes when you need to make updates or style changes—you only need to modify the component in one place!

The beauty of this approach is that you can create as many dynamic components as you need. Just make sure they’re useful and txtible.

One more thing to note: the `$render` statement has an `$endrender` tag. This is because you can pass additional values into the render statement, which we’ll cover next with **replacements**!

### Replacements

Replacements let you take placeholders in your templates and replace them with dynamic values. It’s like giving your template a bit of txtibility and creativity.

Let’s look at a simple example with a `button` component:

```txt
<button> $place('title') </button>
```

In this case, we’ve set up a `title` placeholder inside the button. When we render this component, we can replace that placeholder with whatever we like.

Now, in the `login.fx` page, we render the button and pass a value

```txt
<form>
  $render('components.button')
    $replace('title') Login $endreplace
  $endrender
</form>
```

The result?

```txt
<form>
  <button> Login </button>
</form>
```

Just like that, we’ve replaced `title` with `Login`. But what if you want to replace more than one value?

```txt
<button type="$place('type')"> $place('title') </button>
```

Here, we can replace both the `title` and `type` placeholders

```txt
<form>
  $render('components.button')
    $replace('title') Login $endreplace
    $replace('type') submit $endreplace
  $endrender
</form>
```

The result now looks like this:

```txt
<form>
  <button type="submit"> Login </button>
</form>
```

**Why Do Replacements Matter?**

You might be thinking, “Can’t I just use variables and pass them when rendering the component?” Sure, but **replacements** are much more powerful.

You’re not just replacing static values; you can use **templates** inside replacements. This opens up a whole new level of txtibility.

Imagine you’re building a `login.fx` and a `register.fx` page. Both pages use the same layout _(header, footer, etc.)_ Instead of copy-pasting the same layout into each page, you can use replacements to dynamically inject the content.

Let’s create a `layout.fx` template:

```txt
<!DOCTYPE html>
<html>
  <head></head>
  <body>
    <header> </header>
    $place('main')
    <footer></footer>
  </body>
</html>
```

Now, in `login.fx`, you can render the layout and provide your content to replace the `main` placeholder:

```txt
$render('layout')
  $replace('main')
    <main>
      <form action="/login" method="POST">
        <input name='email' type='text'/>
        <input name='password' type='password'/>
        $render('components.button', title='Login', type='submit')$endrender
      </form>
    </main>
  $endreplace
$endrender
```

When you render this, Flex will:

1. Resolve the `main` template first _($render, $foreach, $if...)_.
2. Render the layout and fill in the `main` section.

Yes! First, we resolve the replacement templates. These templates have access to the state of the parent component _(`login.fx`)_. Once all replacements are resolved, we render the component and inject the final replacement output.

The final result?

```txt
<!DOCTYPE html>
<html>
  <head></head>
  <body>
    <header> </header>
    <main>
      <form action="/login" method="POST">
        <input name='email' type='text'/>
        <input name='password' type='password'/>
        <button type="submit"> Login </button>
      </form>
    </main>
    <footer></footer>
  </body>
</html>
```

**Key Takeaways**

- **You must provide a replacement** for every `$place`. If not, Flex will throw an error.
- **`$place` is a placeholder**, and `$replace` fills it—either with a value or even a full template.
- **Replacements keep your components clean and reusable.** Update once, reflect everywhere.
- **This is what Flex is built for**—composable, dynamic templates without repetitive code.

**Limits**

- You **can’t use `$place` inside a `$replace`**. Think of them as separate steps—one defines the place, the other fills it.

Absolutely—let’s make it clear, friendly, and human!

---

### Include Statement

The `$include` statement lets you **drop in static components** wherever you need them. It's perfect for content that never changes but appears in multiple places—like logos, dividers, or visual elements.

Think of it like copy-paste... but smarter.

Instead of writing the same HTML again and again, you put it in a component file and include it wherever you need it.

**When to Use `$include`**

Use `$include` only for **purely static components**:

- No variables (`$(...)`)
- No logic (`$if`, `$foreach`, etc.)
- Just plain markup

If your component has logic or needs data, use `$render` instead.

**Usage Example**

Let’s say you’ve got this in `components/divider.fx`:

```txt
<hr class="my-6 border-gray-200" />
```

You can now include it in any template like this:

```txt
<h2>Blog Posts</h2>
$include('components.divider')
<ul>
  <li>First Post</li>
  <li>Second Post</li>
</ul>
```

This will be rendered as:

```html
<h2>Blog Posts</h2>
<hr class="my-6 border-gray-200" />
<ul>
  <li>First Post</li>
  <li>Second Post</li>
</ul>
```

Nice and simple. No need to repeat the `<hr>` tag in every page—it’s just included.

**⚠️ Notes**

- If your component has **any** statements _(like `$foreach` or `$if` and so on...)_, `$include` **won’t process them**. It will just paste the file content as-is.
- If you need dynamic behavior, switch to `$render`.
- Great use-cases for `$include` statement includes:
  - Logos & Static SVG icons
  - Badges
  - Horizontal dividers
  - Skeleton loaders

Keep them simple, clean, and logic-free.

## Tools And Globals

**Tools and Globals** are born from my own experience. As a developer, I’ve always wanted a way to define small, reusable functions and variables that I can easily access in my templates. The best part? They should be **globally accessible**.

### Global Variables

Let’s say you’re working on a project with 20 pages, and across those pages, you reference the **application name** in multiple places. Now imagine your client wants to change the app name. Do you really want to go through every single page and replace that name manually? That’s a nightmare.

Here’s where **global variables** come in handy.

Globals let you define values once and access them everywhere in your templates. Need to change something like the app name or domain? Just update it **in one place**, no more searching through every page.

Let me show you how to use globals in our Flex template engine.

**Defining Global Variables**

In your `bnjsx.config.js` file, you can define global variables using the `globals` property. For example, you can define the app name, entry URL, and more:

```js
globals: {
  app: {
    name: 'bnjsx',
    entry: 'https://bnjsx.com',
  }
},
```

**Using Global Variables**

Now that the global variables are defined, you can access and use them anywhere in your templates. For instance, let’s log the app name and domain:

```txt
$log(#app.name)
```

```txt
$log(#app.entry)
```

Or you can even use them directly in HTML:

```html
<a href="$(#app.entry)/docs/v1">Docs</a>
```

**Passing Globals When Rendering Components**

Although it’s not really necessary to pass a global variable when rendering a component (since it’s globally available), Flex does support it anyway. Here’s an example of passing the global `entry` variable to a component:

```txt
$render('components.button', entry=#app.entry) $endrender
```

**Using Globals in Conditions**

Global variables are also perfect for conditions. You can use them to control what’s displayed based on their values. For example, let’s check if the app name is `bnjsx` and display a message accordingly:

```txt
$if(#app.name === 'bnjsx')
  <p>Welcome to Bnjsx!</p>
$else
  <p>Welcome to our application!</p>
$endif
```

**Why It’s Awesome**

The power of global variables is that they centralize your values, making updates across your whole project easier. If your client wants to change the app name or domain, you can do it in **one place** _(your `bnjsx.config.js` file)_. This means no more hunting down values across multiple pages or components.

### Tools (aka template superpowers)

So, what are **tools**?

They're just little functions you can call from _anywhere_ inside your templates. Think of them like helpers that live in your `bnjsx.config.js` file and are always ready when you need them.

**Defining tools**

Here’s how you register your tools in `bnjsx.config.js`.

Let’s say you want to check if something is an array, object, or string:

```js
// bnjsx.config.js
const { isArr, isObj, isStr } = require('bnjsx');

module.exports = {
  tools: {
    isArr,
    isObj,
    isStr,
  },
};
```

Now you can use `@isArr()`, `@isObj()`, or `@isStr()` in your templates.

**Real-world usage**

Say you’re rendering a login page and you want to show error messages. But sometimes the error might be:

- an array,
- an object,
- or just a plain string.

```txt
$if(@isArr(error))
  <ul>
    $foreach(message, error)
      <li>$(message)</li>
    $endforeach
  </ul>
$elseif(@isObj(error))
  <p>$(error.message)</p>
$elseif(@isStr(error))
  <p>$(error)</p>
$endif
```

**Tools can do more**

Tools can be async too. Let’s say you want to fetch blog posts from your SQLite database:

```js
// bnjsx.config.js
const { Cluster, Builder, ClusterPool, SQLite, config } = require('bnjsx');

module.exports = {
  cluster: new Cluster(
    new ClusterPool('sqlite', new SQLite('./database.sqlite'))
  ),
  default: 'sqlite',
  tools: {
    fetchPosts: async () => {
      const cluster = (await config().load()).cluster;
      const connection = await cluster.request();
      const builder = new Builder(connection);
      return await builder.select().from('posts').exec();
    },
  },
};
```

> ⚠️ Make sure you’ve created and seeded your posts table first.

Then in your template:

```txt
$foreach(post, @fetchPosts())
  <p>$(post.title)</p>
$endforeach
```

This will Call your `fetchPosts` tool, loop through the posts and display each title inside a `<p>` tag.

**Use tools for formatting too**

Here’s a good one: formatting names properly based on gender.

```js
tools: {
  formatName: (name, gender) => {
    const g = (gender || '').toLowerCase();
    if (['f', 'female'].includes(g)) return `Ms. ${name}`;
    if (['m', 'male'].includes(g)) return `Mr. ${name}`;
    return name;
  },
},
```

In your welcome email:

```
We're glad you joined us, $(@formatName(user.name, user.gender))!
```

it renders:

```
We're glad you joined us, Mr. John!
```

Or you wanna make sure names and titles look clean?

```js
tools: {
  titleCase: (str) => {
    return (str || '')
      .toLowerCase()
      .split(' ')
      .map(word => word.charAt(0).toUpperCase() + word.slice(1))
      .join(' ');
  },
},
```

```txt
Welcome, $(@titleCase(user.name))!
```

Which gives you:

```
Welcome, John Doe!
```

Clean ✨

**Keep tools simple**

You _can_ use tools to fetch users, query stuff, whatever. But try to keep them lightweight.

Use tools for:

- Type checking
- Simple formatting
- Tiny helpers

Leave complex logic to controllers — they’re better at that job.
