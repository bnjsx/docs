# Bnjsx Text

This module is designed to offer a variety of string manipulation functions. It simplifies working with and transforming strings in your projects.

## Functions

- **`dictionary(singular, plural)`**: Adds or updates the plural form for a singular word in the dictionary.

  - **`singular`**: The singular word to be registered.
  - **`plural`**: The plural form to be registered.

```js
dictionary('cat', 'cats');
dictionary('bus', 'buses');
```

- **`toPlural(singular)`**: Converts a singular word to its plural form based on registered mappings or rules.

  - **`singular`**: The singular word to be pluralized.

```js
console.log(toPlural('cat')); // "cats"
console.log(toPlural('bus')); // "buses"
```

- **`toSingular(plural)`**: Converts a plural word to its singular form based on registered mappings or rules.

  - **`plural`**: The plural word to be converted to singular.

```js
console.log(toSingular('cats')); // "cat"
console.log(toSingular('buses')); // "bus"
```

- **`toRegex(text, flags)`**: Converts a string pattern into a regular expression.

  - **`text`**: The string pattern to convert.
  - **`flags`**: Optional flags for the regular expression.

```js
const regex = toRegex('hello*');
console.log(regex.test('hello world')); // true
```

- **`toUpper(text)`**: Converts a string to uppercase.

  - **`text`**: The string to convert to uppercase.

```js
console.log(toUpper('hello world')); // "HELLO WORLD"
```

- **`toLower(text)`**: Converts a string to lowercase.

  - **`text`**: The string to convert to lowercase.

```js
console.log(toLower('HELLO WORLD')); // "hello world"
```

- **`toUpperAt(text, index)`**: Converts the character at a specified index of a string to uppercase.

  - **`text`**: The string to modify.
  - **`index`**: The index of the character to convert.

```js
console.log(toUpperAt('hello world', 6)); // "hello World"
```

- **`toLowerAt(text, index)`**: Converts the character at a specified index of a string to lowercase.

  - **`text`**: The string to modify.
  - **`index`**: The index of the character to convert.

```js
console.log(toLowerAt('HELLO WORLD', 6)); // "HELLO wORLD"
```

- **`toUpperFrom(text, index, to)`**: Converts all characters from a specified index to uppercase.

  - **`text`**: The string to modify.
  - **`index`**: The starting index from which characters will be converted.
  - **`to`**: The optional ending index. If not provided, the function will convert from the `index` to the end of the string.

```js
console.log(toUpperFrom('hello world', 6)); // "hello WORLD"
```

- **`toLowerFrom(text, index, to)`**: Converts all characters from a specified index to lowercase.

  - **`text`**: The string to modify.
  - **`index`**: The starting index from which characters will be converted.
  - **`to`**: The optional ending index. If not provided, the function will convert from the `index` to the end of the string.

```js
console.log(toLowerFrom('HELLO WORLD', 6)); // "HELLO world"
```

- **`toUpperFirst(text)`**: Converts the first character of a string to uppercase.

  - **`text`**: The string whose first character will be converted.

```js
console.log(toUpperFirst('hello world')); // "Hello world"
```

- **`toKamelCase(text)`**: Converts a string to camelCase.

  - **`text`**: The string to convert to camelCase.

```js
console.log(toKamelCase('hello world')); // "helloWorld"
```

- **`toPascalCase(text)`**: Converts a string to PascalCase.

  - **`text`**: The string to convert to PascalCase.

```js
console.log(toPascalCase('hello world')); // "HelloWorld"
```

- **`toSnakeCase(text)`**: Converts a string to snake_case.

  - **`text`**: The string to convert to snake_case.

```js
console.log(toSnakeCase('hello world')); // "hello_world"
```

- **`toKababCase(text)`**: Converts a string to kebab-case.

  - **`text`**: The string to convert to kebab-case.

```js
console.log(toKababCase('hello world')); // "hello-world"
```

- **`toTitle(text)`**: Converts the first character of every word to uppercase.

  - **`text`**: The string to be transformed.

```js
console.log(toTitle('hello world')); // "Hello World"
```

- **`toSlug(text)`**: Converts a string to a slug.

  - **`text`**: The string to be converted to a slug.

```js
console.log(toSlug('Hello world!')); // "hello-world"
```

- **`toSnap(text, length)`**: Trims the string to a specified length and appends an ellipsis ("...") if the string is longer than the specified length.

  - **`text`**: The string to be truncated.
  - **`length`**: The maximum length of the string.

```js
console.log(toSnap('This is a long sentence', 10)); // "This is a..."
```

- **`toChars(text, ...exclude)`**: Splits a string into an array of characters, optionally excluding certain characters.

  - **`text`**: The string to be split into characters.
  - **`exclude`**: An optional array of characters to exclude.

```js
console.log(toChars('hello', 'o')); // ["h", "e", "l", "l"]
```

- **`toWords(text)`**: Splits a string into an array of words, based on spaces.

  - **`text`**: The string to be split into words.

```js
console.log(toWords('hello world')); // ["hello", "world"]
```

- **`prefix(text, portion)`**: Prefixes a string with a given portion.

  - **`text`**: The string to be prefixed.
  - **`portion`**: The portion to prefix the string with.

```js
console.log(prefix('world', 'hello ')); // "hello world"
```

- **`suffix(text, portion)`**: Suffixes a string with a given portion.

  - **`text`**: The string to be suffixed.
  - **`portion`**: The portion to suffix the string with.

```js
console.log(suffix('hello', ' world')); // "hello world"
```

- **`infix(text, portion, index)`**: Inserts a portion of text into a string at a specific index.

  - **`text`**: The original string.
  - **`portion`**: The portion to insert.
  - **`index`**: The index at which to insert the portion.

```js
console.log(infix('helloworld', ' beautiful', 5)); // "hello beautiful world"
```

- **`countOf(text, search)`**: Counts the occurrences of a substring or pattern in a string.

  - **`text`**: The string in which to search.
  - **`search`**: The substring or regular expression to search for.

```js
console.log(countOf('hello hello world', 'hello')); // 2
```

- **`indexOf(text, search, pos)`**: Finds the first index of a substring or pattern in a string, starting from a specified position.

  - **`text`**: The string in which to search.
  - **`search`**: The substring or regular expression to search for.
  - **`pos`**: The position in the string to start the search from (default: `0`).

```js
console.log(indexOf('hello world', 'world')); // 6
```

- **`indexesOf(text, search)`**: Finds all the indexes of a substring or pattern in a string and returns an array of index ranges.

  - **`text`**: The string in which to search.
  - **`search`**: The substring or regular expression to search for.

```js
console.log(indexesOf('hello world hello', 'hello'));
// Outputs: [{ start: 0, end: 5 }, { start: 12, end: 17 }]
```

- **`lastIndexOf(text, search)`**: Finds the last index of a substring or pattern in a string.

  - **`text`**: The string in which to search.
  - **`search`**: The substring or regular expression to search for.

```js
console.log(lastIndexOf('hello world hello', 'hello')); // 12
```
