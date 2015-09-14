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
// @ensure num : Number
var num = 101;

// @ensure str : String
var str = "hello";

// @ensure nums : Array(Number)
var nums = [10, 20];

// @ensure obj : { x: Number, y: String }
var obj = { x: 11, y: 'twenty-two' }

// @ensure inc : (Number) => Number
function inc (num) {
  return num + 1;
}

// @ensure add : (Number, Number) => Number
function add (x, y) {
  return x + y + 1;
}

// @ensure favFoods : () => Array(String)
function favFoods () {
  return ['bananas', 'cherries', 'potatoes'];
}

// @ensure wrap : (x) => Array(x)
function wrap (value) {
  return [value];
}
```

The key point to recognize is that a function type annotation takes the form `(inputs...) => output`. This is very similar to [ES6 arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions), except parenthesis around the inputs are required.

## Type Declarations

Type declarations are written in comments to keep JS Zero ES6-compatible.

### Type Aliases

To declare a type alias, use the `@type` declaration. This creates an alias that is no different than the type you set it to.

```javascript
// @type Venue = { name: String, rating: Number }
```

### Assumptions

When working with IO, many times you will be reading data from some external source. In these cases you don't want JS Zero to attempt (and fail) at determining your types. Instead, you want to use `@assume` to explicitly declare what types your incoming data will be.

```javascript
// @type Pet = { name: String, happiness: Number }

// @assume fetchPets : (Number) => Array(Pet)
function fetchPets (minHappiness) {
  return knex.select('*').from('pets').where('happiness', '>', minHappiness)
}
```

### New Types

To declare an entirely new type, use the `@newtype` declaration. This is useful for creating opaque types when declaring modules (explained in the next section).

For example, here is how you create an opaque type:

```javascript
// @newtype VirtualElement
// @assume React.render : (String, Object) => VirtualElement
```

A new type is only considered equal to itself, even if its structure is the same as another type's structure. For example:

```javascript
// @type Person = { name: String }
// @type Book   = { name: String }

// @newtype Person2 = { name: String }
```

Even though `Person` and `Person2` are structurally the same, they are not considered equal. In contrast, type `Person` and `Book` are both type **aliases**, and thus considered the same type.

### Module Declarations

To safely and effectively use a large library, you or someone else might need to write a larger number of type declarations. You can do so using the `@module` block comment:

```javascript
/*
@module JQuery 'jquery' {
  @newtype Wrapper = {
    on: (String, DomEventHandler) => Wrapper,
    show: (String) => Wrapper,
    hide: (String) => Wrapper
  }

  @newtype Deferred(value) = {
    then: ((value) => newValue) => Deferred(newValue)
  }

  this : (String) => Wrapper;
}
*/
var $ = require('jquery');
```

In the above example, the `JQuery` part of `@module JQuery` is the name of the type, while the [optional] `'jquery'` part is the string name you pass into `require`.

[[Discuss type declarations]](http://discuss.js-zero.com/t/integrating-with-other-javascript-code/15/1)
