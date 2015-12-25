# Type System

This page aims to document the current set of **ideas** for the type system. JS Zero is early in its design phase, so expect this page to be in flux.

To reiterate, **everything on this page is up for discussion.**

## Intersection Types

**Intersection types** seems to be the best way to express a type system for JavaScript. This is because everything in JavaScript is an object, **in addition** to its own type (e.g. a function is also an object).

Library writers often take advantage of this fact. For example, jQuery's global variable is both a function and an object:

```javascript
$('button');
$.ajax({ type: 'GET', url: 'http://api.example.com' });
```

With intersection types, you can express this relationship by intersecting two completely different types. For example:

```javascript
let JQueryObject   = $newType(...)
let JQueryDeferred = $newType(...)

$assume($, Function.multi(
  (String) => JQueryObject,
  { ajax: (Object) => JQueryDeferred }
))

$('button'); //=> JQueryObject
$.ajax({ type: 'GET', url: 'http://api.example.com' }); //=> JQueryDeferred
```

Both [Flowtype](http://flowtype.org/) and [TypeScript](http://www.typescriptlang.org/) have intersection types as a feature.

[[Discuss intersection types]](http://discuss.js-zero.com/t/intersection-types/17)

## Primitive types

- Booleans
- Numbers
- Strings
- Objects
- Arrays
- Maps
- Sets
- Functions

## Prototypes

A core feature of JavaScript is that every type has a inherited prototype of methods. Any value of a type can call that type's prototype methods.

The most useful case of this is when working with arrays:

```javascript
var data = [10,20,30];
data.map(x => x * 2).reduce(add);

function add (x, y) { return x + y; }
```

Because `data` has type `Array`, it can call any methods on `Array.prototype`. The type system is aware of this and allows `data` to run both the `map` and `reduce` methods.

If we were to manually declare part of the `Array` type, it might look something like this:

```js
// TODO: API design needs works
Array.prototype = $newType.withParams('e').annotatePrototype({
  map: '((e) => b) => b',

  reduce: Function.multi(
    '((e, e) => r) => r',
    '((acc, e) => acc, acc) => acc'
  )
})
```

[[Discuss typing prototypes]](http://discuss.js-zero.com/t/typing-prototypes/18/1)
