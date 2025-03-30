# Bnjsx Test

This module is designed to assist developers in type checking and validation for various JavaScript values. It provides a collection of type-checking functions, and utilities to validate your values.

## Table of Contents

1. **[Type Functions](#type-functions)**
2. **[Pattern Functions](#pattern-functions)**
3. **[Content Functions](#content-functions)**

## Type Functions

Type functions allow you to validate the type of a value. They return a boolean indicating whether the value matches the expected type.

- **`isSet(value: any): boolean`** Checks if a value is an instance of `Set`.

  ```js
  console.log(isSet(new Set())); // true
  console.log(isSet([])); // false
  ```

- **`isMap(value: any): boolean`** Checks if a value is an instance of `Map`.

  ```js
  console.log(isMap(new Map())); // true
  console.log(isMap({})); // false
  ```

- **`isWeakSet(value: any): boolean`** Checks if a value is an instance of `WeakSet`.

  ```js
  console.log(isWeakSet(new WeakSet())); // true
  console.log(isWeakSet(new Set())); // false
  ```

- **`isWeakMap(value: any): boolean`** Checks if a value is an instance of `WeakMap`.

  ```js
  console.log(isWeakMap(new WeakMap())); // true
  console.log(isWeakMap(new Map())); // false
  ```

- **`isArr(value: any): boolean`** Checks if a value is an array.

  ```js
  console.log(isArr([])); // true
  console.log(isArr({})); // false
  ```

- **`isObj(value: any): boolean`** Checks if a value is a non-null object (excluding arrays).

  ```js
  console.log(isObj({})); // true
  console.log(isObj([])); // false
  console.log(isObj(null)); // false
  ```

- **`isNum(value: any): boolean`** Checks if a value is a number.

  ```js
  console.log(isNum(42)); // true
  console.log(isNum('42')); // false
  ```

- **`isTxt(value: any): boolean`** Checks if a value is a string.

  ```js
  console.log(isTxt('hello')); // true
  console.log(isTxt(42)); // false
  ```

- **`isStr(value: any): boolean`** Alias for `isTxt`. Checks if a value is a string.

  ```js
  console.log(isStr('hello')); // true
  console.log(isStr(42)); // false
  ```

- **`isBool(value: any): boolean`** Checks if a value is a boolean.

  ```js
  console.log(isBool(true)); // true
  console.log(isBool('true')); // false
  ```

- **`isFunc(value: any): boolean`** Checks if a value is a function.

  ```js
  console.log(isFunc(() => {})); // true
  console.log(isFunc({})); // false
  ```

- **`isSymbol(value: any): boolean`** Checks if a value is a symbol.

  ```js
  console.log(isSymbol(Symbol())); // true
  console.log(isSymbol('symbol')); // false
  ```

- **`isInt(value: any): boolean`** Checks if a value is an integer.

  ```js
  console.log(isInt(42)); // true
  console.log(isInt(42.42)); // false
  ```

- **`isFloat(value: any): boolean`** Checks if a value is a floating-point number.

  ```js
  console.log(isFloat(42.42)); // true
  console.log(isFloat(42)); // false
  ```

- **`isNan(value: any): boolean`** Checks if a value is NaN (Not a Number).

  ```js
  console.log(isNan(NaN)); // true
  console.log(isNan(42)); // false
  ```

- **`isNull(value: any): boolean`** Checks if a value is null.

  ```js
  console.log(isNull(null)); // true
  console.log(isNull(undefined)); // false
  ```

- **`isUndefined(value: any): boolean`** Checks if a value is undefined.

  ```js
  console.log(isUndefined(undefined)); // true
  console.log(isUndefined(null)); // false
  ```

- **`isDefined(value: any): boolean`** Checks if a value is defined (not undefined).

  ```js
  console.log(isDefined(42)); // true
  console.log(isDefined(undefined)); // false
  ```

- **`isDefinedStrict(value: any): boolean`** Strictly checks if a value is defined (not null, undefined, or NaN).

  ```js
  console.log(isDefinedStrict(42)); // true
  console.log(isDefinedStrict(null)); // false
  console.log(isDefinedStrict(NaN)); // false
  console.log(isDefinedStrict(undefined)); // false
  ```

- **`isInf(value: any): boolean`** Checks if a value is Infinity.

  ```js
  console.log(isInf(Infinity)); // true
  console.log(isInf(42)); // false
  ```

- **`isFinite(value: any): boolean`** Checks if a value is a finite number.

  ```js
  console.log(isFinite(42)); // true
  console.log(isFinite(Infinity)); // false
  ```

- **`isTruthy(value: any): boolean`** Checks if a value is truthy (not falsy).

  ```js
  console.log(isTruthy(1)); // true
  console.log(isTruthy(0)); // false
  ```

- **`isFalsy(value: any): boolean`** Checks if a value is falsy (false, 0, "", null, undefined, or NaN).

  ```js
  console.log(isFalsy(0)); // true
  console.log(isFalsy(1)); // false
  ```

- **`isFalse(value: any): boolean`** Checks if a value is strictly `false`.

  ```js
  console.log(isFalse(false)); // true
  console.log(isFalse(true)); // false
  ```

- **`isTrue(value: any): boolean`** Checks if a value is strictly `true`.

  ```js
  console.log(isTrue(true)); // true
  console.log(isTrue(false)); // false
  ```

- **`isIterable(value: any): boolean`** Checks if a value is iterable (has an `Symbol.iterator` property).

  ```js
  console.log(isIterable([1, 2, 3])); // true
  console.log(isIterable(42)); // false
  ```

- **`isEven(value: any): boolean`** Checks if a number is even.

  ```js
  console.log(isEven(42)); // true
  console.log(isEven(43)); // false
  ```

- **`isOdd(value: any): boolean`** Checks if a number is odd.

  ```js
  console.log(isOdd(43)); // true
  console.log(isOdd(42)); // false
  ```

- **`isError(value: any): boolean`** Checks if a value is an instance of the `Error` object.

  ```js
  console.log(isError(new Error())); // true
  console.log(isError('Error')); // false
  ```

- **`isChildOf(child: any, parent: any): boolean`** Checks if an object is a child of the given class (instanceof).

  ```js
  class Parent {}
  const child = new Parent();
  console.log(isChildOf(child, Parent)); // true
  ```

- **`isClass(value: any): boolean`** Checks if a value is a class (constructor function).

  ```js
  console.log(isClass(class MyClass {})); // true
  console.log(isClass(function myFunc() {})); // false
  ```

- **`isSubclass(child: any, parent: any): boolean`** Checks if a class is a subclass of another class.

  ```js
  class Parent {}
  class Child extends Parent {}
  console.log(isSubclass(Child, Parent)); // true
  ```

- **`isDate(value: any): boolean`** Checks if a value is a valid `Date` object.

  ```js
  console.log(isDate(new Date())); // true
  console.log(isDate('2021-01-01')); // false
  ```

- **`isRegex(value: any): boolean`** Checks if a value is a valid `RegExp` (Regular Expression).

  ```js
  console.log(isRegex(/abc/)); // true
  console.log(isRegex('abc')); // false
  ```

- **`isPromise(value: any): boolean`** Checks if a value is a Promise.

  ```js
  console.log(isPromise(Promise.resolve())); // true
  console.log(isPromise(function () {})); // false
  ```

## Pattern Functions

Pattern functions allow you to validate whether a value follows a specific pattern.

- **`isCamelCase(value: any): boolean`** Checks if a string is in camelCase format.

  ```js
  console.log(isCamelCase('camelCase')); // true
  console.log(isCamelCase('snake_case')); // false
  ```

- **`isPascalCase(value: any): boolean`** Checks if a string is in PascalCase format.

  ```js
  console.log(isPascalCase('PascalCase')); // true
  console.log(isPascalCase('pascalCase')); // false
  ```

- **`isSnakeCase(value: any): boolean`** Checks if a string is in snake_case format.

  ```js
  console.log(isSnakeCase('snake_case')); // true
  console.log(isSnakeCase('snakeCase')); // false
  ```

- **`iskababCase(value: any): boolean`** Checks if a string is in kebab-case format.

  ```js
  console.log(iskababCase('kebab-case')); // true
  console.log(iskababCase('kebabCase')); // false
  ```

- **`isIso(value: any): boolean`** Checks if a string is in ISO 8601 format (e.g., "2024-11-28T12:30:00.000Z").

  ```js
  console.log(isIso('2024-11-28T12:30:00.000Z')); // true
  console.log(isIso('2024-11-28')); // false
  ```

- **`isDateTime(value: any): boolean`** Checks if a string is in the datetime format (e.g., "2024-11-28 12:30:00").

  ```js
  console.log(isDateTime('2024-11-28 12:30:00')); // true
  console.log(isDateTime('2024-11-28')); // false
  ```

- **`isDateStr(date: any): boolean`** Checks if a string is in the date format (e.g., "2024-11-28").

  ```js
  console.log(isDateStr('2024-11-28')); // true
  console.log(isDateStr('2024-11-28 12:30:00')); // false
  ```

- **`isTimeStr(date: any): boolean`** Checks if a string is in the time format (e.g., "12:30:00").

  ```js
  console.log(isTimeStr('12:30:00')); // true
  console.log(isTimeStr('12:30')); // false
  ```

- **`isPropExp(value: any): boolean`** Checks if a string is a valid property name (e.g., a valid js variable name).

  ```js
  console.log(isPropExp('myVar')); // true
  console.log(isPropExp('123var')); // false
  ```

- **`isNumExp(value: any): boolean`** Checks if a string is a valid number (including integers and floats).

  ```js
  console.log(isNumExp('123')); // true
  console.log(isNumExp('123.45')); // true
  console.log(isNumExp('abc')); // false
  ```

- **`isIntExp(value: any): boolean`** Checks if a string is a valid integer.

  ```js
  console.log(isIntExp('123')); // true
  console.log(isIntExp('123.45')); // false
  ```

- **`isFloatExp(value: any): boolean`** Checks if a string is a valid float (decimal number).

  ```js
  console.log(isFloatExp('123.45')); // true
  console.log(isFloatExp('123')); // false
  ```

- **`isStrExp(value: any): boolean`** Checks if a string is a valid string literal (single or double-quoted).

  ```js
  console.log(isStrExp("'hello'")); // true
  console.log(isStrExp('"world"')); // true
  console.log(isStrExp('hello')); // false
  ```

- **`isDotExp(value: any): boolean`** Checks if a string is a valid dot notation expression for property access.

  ```js
  console.log(isDotExp('object.property')); // true
  console.log(isDotExp('object.123property')); // false
  ```

- **`isBracketExp(value: any): boolean`** Checks if a string is a valid bracket notation expression for property access.

  ```js
  console.log(isBracketExp('object["property"]')); // true
  console.log(isBracketExp('object["123"]')); // false
  ```

- **`isAnyExp(value: any): boolean`** Checks if a string is a valid combination of dot and bracket notations.

  ```js
  console.log(isAnyExp('object.property[0]')); // true
  console.log(isAnyExp('object.property["key"]')); // true
  console.log(isAnyExp('object.123property[0]')); // false
  ```

## Content Functions

Content functions help determine if the content of an object, array, or string matches the expected structure or value.

- **`isFullObj(value: any): boolean`** Checks if an object is non-empty (has at least one property).

  ```js
  console.log(isFullObj({ key: 'value' })); // true
  console.log(isFullObj({})); // false
  ```

- **`isFullArr(value: any): boolean`** Checks if an array is non-empty (has at least one element).

  ```js
  console.log(isFullArr([1, 2, 3])); // true
  console.log(isFullArr([])); // false
  ```

- **`isFullStr(value: any): boolean`** Checks if a string is non-empty (contains at least one non-whitespace character).

  ```js
  console.log(isFullStr('Hello')); // true
  console.log(isFullStr('   ')); // false
  ```

- **`isEmptyObj(value: any): boolean`** Checks if an object is empty (has no properties).

  ```js
  console.log(isEmptyObj({})); // true
  console.log(isEmptyObj({ key: 'value' })); // false
  ```

- **`isEmptyArr(value: any): boolean`** Checks if an array is empty (contains no elements).

  ```js
  console.log(isEmptyArr([])); // true
  console.log(isEmptyArr([1])); // false
  ```

- **`isEmptyStr(value: any): boolean`** Checks if a string is empty (contains non-whitespace characters).

  ```js
  console.log(isEmptyStr('')); // true
  console.log(isEmptyStr('   ')); // true
  console.log(isEmptyStr('sss')); // true
  console.log(isEmptyStr('Hello')); // false
  ```

- **`isArrOfNum(value: any): boolean`** Checks if an array is non-empty and contains only numbers.

  ```js
  console.log(isArrOfNum([1, 2, 3.3])); // true
  console.log(isArrOfNum([1, 'two', 3.3])); // false
  ```

- **`isArrOfInt(value: any): boolean`** Checks if an array is non-empty and contains only integers.

  ```js
  console.log(isArrOfInt([1, 2, 3])); // true
  console.log(isArrOfInt([1, 2.5, 3])); // false
  ```

- **`isArrOfFloat(value: any): boolean`** Checks if an array is non-empty and contains only floating-point numbers.

  ```js
  console.log(isArrOfFloat([1.5, 2.7, 3.2])); // true
  console.log(isArrOfFloat([1, 2.5, '3'])); // false
  ```

- **`isArrOfStr(value: any): boolean`** Checks if an array is non-empty and contains only strings.

  ```js
  console.log(isArrOfStr(['hello', 'world'])); // true
  console.log(isArrOfStr(['hello', 123])); // false
  ```

- **`isArrOfArr(value: any): boolean`** Checks if an array is non-empty and contains only arrays.

  ```js
  console.log(isArrOfArr([[], []])); // true
  console.log(isArrOfArr([[], 'two'])); // false
  ```

- **`isArrOfObj(value: any): boolean`** Checks if an array is non-empty and contains only objects.

  ```js
  console.log(isArrOfObj([{}, {}])); // true
  console.log(isArrOfObj([{}, 1])); // false
  ```

- **`isArrOfFunc(value: any): boolean`** Checks if an array is non-empty and contains only functions.

  ```js
  console.log(isArrOfFunc([() => {}, () => {}])); // true
  console.log(isArrOfFunc([() => {}, 'string'])); // false
  ```

- **`isArrOfBool(value: any): boolean`** Checks if an array is non-empty and contains only booleans.

  ```js
  console.log(isArrOfBool([true, false, true])); // true
  console.log(isArrOfBool([true, 'false'])); // false
  ```

- **`isArrOfAny(value: any): boolean`** Checks if an array is non-empty and allows any type of value (no type restriction).

  ```js
  console.log(isArrOfAny([1, 'string', true])); // true
  console.log(isArrOfAny([])); // false
  ```

- **`isObjOfNum(value: any): boolean`** Checks if an object is non-empty and contains only numeric values.

  ```js
  console.log(isObjOfNum({ a: 1, b: 2.2 })); // true
  console.log(isObjOfNum({ a: 1, b: 'two' })); // false
  ```

- **`isObjOfInt(value: any): boolean`** Checks if an object is non-empty and contains only integer values.

  ```js
  console.log(isObjOfInt({ a: 1, b: 2 })); // true
  console.log(isObjOfInt({ a: 1, b: 2.5 })); // false
  ```

- **`isObjOfFloat(value: any): boolean`** Checks if an object is non-empty and contains only floating-point values.

  ```js
  console.log(isObjOfFloat({ a: 1.1, b: 2.5 })); // true
  console.log(isObjOfFloat({ a: 1, b: '2.5' })); // false
  ```

- **`isObjOfStr(value: any): boolean`** Checks if an object is non-empty and contains only string values.

  ```js
  console.log(isObjOfStr({ a: 'apple', b: 'banana' })); // true
  console.log(isObjOfStr({ a: 'apple', b: 123 })); // false
  ```

- **`isObjOfArr(value: any): boolean`** Checks if an object is non-empty and contains only arrays as values.

  ```js
  console.log(isObjOfArr({ a: [1, 2], b: [3, 4] })); // true
  console.log(isObjOfArr({ a: [1, 2], b: 'string' })); // false
  ```

- **`isObjOfObj(value: any): boolean`** Checks if an object is non-empty and contains only objects as values.

  ```js
  console.log(isObjOfObj({ a: { x: 1 }, b: { y: 2 } })); // true
  console.log(isObjOfObj({ a: { x: 1 }, b: 'string' })); // false
  ```

- **`isObjOfFunc(value: any): boolean`** Checks if an object is non-empty and contains only functions as values.

  ```js
  console.log(isObjOfFunc({ a: () => {}, b: () => {} })); // true
  console.log(isObjOfFunc({ a: () => {}, b: 'string' })); // false
  ```

- **`isObjOfBool(value: any): boolean`** Checks if an object is non-empty and contains only boolean values.

  ```js
  console.log(isObjOfBool({ a: true, b: false })); // true
  console.log(isObjOfBool({ a: true, b: 'false' })); // false
  ```

- **`isObjOfAny(value: any): boolean`** Checks if an object is non-empty and allows any type of value (no type restriction).

  ```js
  console.log(isObjOfAny({ a: 1, b: 'string' })); // true
  console.log(isObjOfAny({})); // false
  ```

- **`hasProp(object: any, property: any): boolean`** Checks if an object has a specific property.

  ```js
  console.log(hasProp({ a: 1, b: 2 }, 'a')); // true
  console.log(hasProp({ a: 1, b: 2 }, 'c')); // false
  ```

- **`hasProps(object: any, ...properties: Array<any>): boolean`** Checks if an object has all specified properties.

  ```js
  console.log(hasProps({ a: 1, b: 2 }, 'a', 'b')); // true
  console.log(hasProps({ a: 1, b: 2 }, 'a', 'c')); // false
  ```

- **`hasIndex(target: any, index: any): boolean`** Checks if an index is within the range of a target (array, string, or object).

  ```js
  console.log(hasIndex([1, 2, 3], 1)); // true
  console.log(hasIndex([1, 2, 3], 3)); // false
  console.log(hasIndex('hello', 1)); // true
  console.log(hasIndex('hello', 5)); // false
  ```

- **`hasLength(target: any, length: any): boolean`** Checks if the length of a target (array, string, or object) matches a specified length.

  ```js
  console.log(hasLength([1, 2, 3], 3)); // true
  console.log(hasLength([1, 2, 3], 2)); // false
  console.log(hasLength('hello', 5)); // true
  console.log(hasLength('hello', 4)); // false
  console.log(hasLength({ a: 1, b: 2 }, 2)); // true
  console.log(hasLength({ a: 1, b: 2 }, 1)); // false
  ```

- **`hasFlag(regex: any, flag: any): boolean`** Checks if a regex has a specific flag.

  ```js
  console.log(hasFlag(/abc/i, 'i')); // true
  console.log(hasFlag(/abc/i, 'g')); // false
  ```

- **`hasKey(object: any, key: any): boolean`** Checks if an object has a specific key (property name).

  ```js
  console.log(hasKey({ a: 1, b: 2 }, 'a')); // true
  console.log(hasKey({ a: 1, b: 2 }, 'c')); // false
  ```

- **`hasKeys(object: any, ...keys: Array<any>): boolean`** Checks if an object contains all of the specified keys (property names).

  ```js
  console.log(hasKeys({ a: 1, b: 2 }, 'a', 'b')); // true
  console.log(hasKeys({ a: 1, b: 2 }, 'a', 'c')); // false
  console.log(hasKeys({ a: 1, b: 2, c: 3 }, 'a', 'b')); // true
  ```

- **`hasValue(target: any, value: any): boolean`** Checks if an (object|array|string) has a specific value.

  ```js
  console.log(hasValue({ a: 1, b: 2 }, 2)); // true
  console.log(hasValue([1, 2, 3], 2)); // true
  console.log(hasValue('hello world', 'world')); // true
  console.log(hasValue({ a: 1, b: 2 }, 3)); // false
  ```

- **`hasValues(target: any, ...values: Array<any>): boolean`** Checks if an (object|array|string) contains all of the specified values.

  ```js
  console.log(hasValues({ a: 1, b: 2 }, 1, 2)); // true
  console.log(hasValues([1, 2, 3], 1, 3)); // true
  console.log(hasValues('hello world', 'hello', 'world')); // true
  console.log(hasValues(['hello', 'foo'], 'hello', 'bar')); // false
  ```
