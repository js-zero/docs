# Specification

This document specifies exactly what parts of ES6 will be included in the JS Zero subset, as well as type declaration syntax and usage.

### This is a *very* rough draft. Please see discussion threads for latest details.

## Goals

- Stay close to primitive types
  - Objects, arrays, functions, strings, numbers, booleans, maps, and sets should cover almost all use cases.
- Only require type annotations at **boundaries**
  - Database queries, HTTP requests, etc.

[[Discuss these goals]](http://discuss.js-zero.com/t/a-type-safe-subset-of-es6/13)

## Anti-Goals

Here are some things JS Zero is NOT trying to do:

- Change the existing behavior of JavaScript
  - Explanation: If something doesn't fit, it should be *removed* (but never changed)
- Force all code to be [pure](http://www.sitepoint.com/functional-programming-pure-functions/)
  - Explanation: JavaScript is not designed to be a pure language. Forcing it to be so would not be worth the effort.
- Require knowledge of advanced type-theory to effectively use JS Zero
  - Explanation: JS Zero is designed to be used by the average JavaScript developer.

[[Discuss these anti-goals]](http://discuss.js-zero.com/t/a-type-safe-subset-of-es6/13)

## Specific Features

(incomplete) This is the subset of features you will be able to write in JavaScript Zero:

- let, var, const
- Basic types (objects, arrays, strings, numbers, booleans)
- Functions - anonymous, declarations, arrow
- Symbols
- Promises
- Map / Set
- Prototypes

These features will be **omitted**:

- No classes, no keyword `new`
- No double equals `==`

[[Discuss this subset of features]](http://discuss.js-zero.com/t/what-subset-of-features-will-js-zero-support/19)

## Built-in Types

JS Zero several basic types...

```
String
Number
Boolean
Object
Function
Symbol
```

...and several higher-kinded types...

```
Array(elem)
Promise(resolved, rejected)
Map(key, value)
Set(elem)
```

## Type Annotation Syntax

Here are some examples of type annotations:

```javascript
$ensure `num : Number`
var num = 101;

$ensure `str : String`
var str = "hello";

$ensure `nums : Array(Number)`
var nums = [10, 20];

$ensure `obj : { x: Number, y: String }`
var obj = { x: 11, y: 'twenty-two' }

$ensure `inc : (Number) => Number`
function inc (num) {
  return num + 1;
}

$ensure `add : (Number, Number) => Number`
function add (x, y) {
  return x + y + 1;
}

$ensure `favFoods : () => Array(String)`
function favFoods () {
  return ['bananas', 'cherries', 'potatoes'];
}

$ensure `wrap : (a) => Array(a)`
function wrap (value) {
  return [value];
}


var React = require('react')

$ensure `withLayout : (React.VirtualElement) => React.VirtualElement`
function withLayout (contents) {
  return React.createElement('div', { class: 'layout' }, contents)
}
```

The key point to recognize is that a function type annotation takes the form `(inputs...) => output`. The annotations make use of (or look very similar to) [ES6 arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions), except that parenthesis around the inputs are always required.


### Type Aliases

To declare a type alias, simply construct it from existing types. This creates an alias that is no different than the type you set it to.

```javascript
let Venue = Object.of({ name: String, rating: Number })
```

### Assumptions

When working with IO, many times you will be reading data from some external source. In these cases you don't want JS Zero to attempt (and fail) at determining your types. Instead, you want to use `$assume` to explicitly declare what types your incoming data will be.

```javascript
let Pet = Object.of({ name: String, happiness: Number })

$assume `fetchPets : (Number) => Promise( Array(Pet) )`
function fetchPets (minHappiness) {
  return knex.select('*').from('pets').where('happiness', '>', minHappiness)
}
```

### Ensures

Although not required, you can annotate any JS Zero function using `@ensure` and force the type system to **enforce** the function follows the type you specify.

```javascript
$ensure `(Number, String) => String`
function multiplyString (times, str) {
  return new Array(times).fill(str); // oops, forgot to join the strings together!
}
```

In the above example, JS Zero would normally infer has the type `(Number, String) => Array(String)` for `multiplyString` (see docs for [Array.fill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/fill)). However, since we annotated our intended type with `@ensure`, JS Zero will instead throw a type error :)

### New Types

To declare an entirely new type, use the `$newtype` declaration. This is useful for creating opaque types when declaring modules (explained in the next section).

For example, here is how you create an opaque type:

```javascript
let VirtualElement = $newtype()
$assume( React.render, (String, Object) => VirtualElement )
```

A new type is only considered equal to itself, even if its structure is the same as another type's structure. For example:

```javascript
let Person = Object.of({ name: String })
let Book   = Object.of({ name: String })

let Person2 = $newtype `{ name: String }`
```

Even though `Person` and `Person2` are structurally the same, they are not considered equal; `Person2` is a type distinctly different from `{ name: String }`. In contrast, type `Person` and `Book` are both type **aliases**, and thus considered the same type.

### Module Declarations

To safely and effectively use a large library, you or someone else might need to write a larger number of type declarations. You can do so using the `$module` function:

```javascript
$module('jquery', function () {

  let jQueryObject = $newtype().withPrototype({
    on: (String, DomEventHandler) => jQueryObject,
    show: (String) => jQueryObject,
    hide: (String) => jQueryObject
  }))

  let Deferred = $newtype('v').withPrototype(function(valueType) {
    return {
      then: $compileType('((v) => v2) => d', { v: valueType, d: Deferred })
    }
  })
})

var $ = require('jquery');
```

In the above example, the `JQuery` part of `@module JQuery` is the name of the type, while the [optional] `'jquery'` part is the string name you pass into `require`.

[[Discuss type declarations]](http://discuss.js-zero.com/t/integrating-with-other-javascript-code/15/1)

## Prelude

Aside from type checking, JS Zero aims to also give a good toolset for typed functional programming.

### Option

`Option` is a type that represents an optional value. It's the type you get when you use default parameters. For example:

```js

// This function has type: (Number, ?Number) => Number
// Or, more explicitly:    (Number, Option(Number)) => Number
let add = (a, b=10) => a + b;
```

### Result

The `Result` type is a handy tool for representing an operation which may or not fail. If successful, you get an `Ok` enum containing the successful value. If not, you get an `Err` enum containing the error. For example:

```js
let { Ok, Err } = Result
let User = Object.of({ username: String, password: String })

$ensure `createUser : (User) => Result(User, String)`

let createUser = (user) =>
  Ok(user)
  .then( validateUsername )
  .then( validatePassword )

let validateUsername = (user) =>
  /^[a-z]+$/.test(user.username) ? Ok(user) : Err('Invalid username')

let validatePassword = (user) =>
  user.password.length > 0 ? Ok(user) : Err('Invalid password')

// This will console log: 'Invalid username'
createUser({ username: 'n00b', password: '123' })
  .then( user => console.log("Validated user:", user) )
  .catch( err => console.log(err) )
```
