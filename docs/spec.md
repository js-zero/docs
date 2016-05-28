# Specification

This document specifies exactly what parts of ES6 will be included in the JS Zero subset, as well as type declaration syntax and usage.

### This is a *very* rough draft. Please see discussion threads for latest details.

## Goals

- Stay close to primitive types
  - Objects, arrays, functions, strings, numbers, booleans, maps, and sets should cover almost all use cases.
- Only require type annotations at **boundaries**
  - Database queries, HTTP requests, etc.

## Anti-Goals

Here are some things JS Zero is NOT trying to do:

- Change the existing behavior of JavaScript
  - Explanation: If a semantic doesn't fit JS Zero, it should be *removed* (but never changed)
- Force all code to be [pure](http://www.sitepoint.com/functional-programming-pure-functions/)
  - Explanation: JavaScript is not designed to be a pure language. Forcing it to be so would not be worth the effort.
- Require knowledge of advanced type-theory to effectively use JS Zero
  - Explanation: JS Zero is designed to be used by the average JavaScript developer.

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
let Venue = Object.ofType({ name: String, rating: Number })
```

### Assumptions

When working with IO, many times you will be reading data from some external source. In these cases you don't want JS Zero to attempt (and fail) at determining your types. Instead, you want to use `$assume` to explicitly declare what types your incoming data will be.

```javascript
let Pet = Object.ofType({ name: String, happiness: Number })

$assume `fetchPets : (Number) => Promise( Array(Pet) )`
function fetchPets (minHappiness) {
  return knex.select('*').from('pets').where('happiness', '>', minHappiness)
}
```

### Ensures

Although not required, you can annotate any JS Zero function using `@ensure` and force the type system to **enforce** the function follows the type you specify.

```javascript
$ensure `multiplyString : (Number, String) => String`
function multiplyString (times, str) {
  return new Array(times).fill(str); // oops, forgot to join the strings together!
}
```

In the above example, JS Zero would normally infer has the type `(Number, String) => Array(String)` for `multiplyString` (see docs for [Array.fill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/fill)). However, since we annotated our intended type with `@ensure`, JS Zero will instead throw a type error :)

### Custom Types

To declare an entirely new type, use the `Type.new` builder. This is useful for creating your own custom types when declaring modules (explained in the next section).

For example, here is how you create an opaque type:

```javascript
let VirtualElement = Type.new()
$assume `React.render : (String, Object) => VirtualElement`
```

A new type is only considered equal to itself, even if its structure is the same as another type's structure. For example:

```javascript
let Person = Object.ofType({ name: String })
let Book   = Object.ofType({ name: String })

let Person2 = Type.new({ name: String })
```

Even though `Person` and `Person2` are structurally the same, they are not considered equal; `Person2` is a type distinctly different from `{ name: String }`. In contrast, type `Person` and `Book` are both type **aliases**, and thus considered the same type.

### A more advanced example

```js
let Tree = Type.new(`|v|
  Node({ value: v, left: Tree, right: Tree })
  Leaf({ value: v })
`)

let Node = Tree.Node
let Leaf = Tree.Leaf

let mySmallTree = Leaf(42)

let myBiggerTree = Node(
  'the root',
  Node(
    'left child',
    Leaf('leaf 1'),
    Leaf('leaf 2'),
  ),

  Node(
    'right child',
    Leaf('leaf 3'),
    Leaf('leaf 4'),
  ),
)

myBiggerTree.right.left.value //=> 'leaf 3'
```

The above code will have the following inferred types:

    Node         : (a, Tree, Tree)                         => Tree(a)
                 & ({ value: a, left: Tree, right: Tree }) => Tree(a)

    Leaf         : (a)            => Tree(a)
                 & ({ value: a }) => Tree(a)
    mySmallTree  : Tree(Num)
    myBiggerTree : Tree(String)


## Type Prototypes

In an ideal world, JavaScript would be all functions and no `this`. But, this is not the case, so we must make do. Because JavaScript has no [pipeline operator](https://github.com/mindeavor/es-pipeline-operator), we must rely on **JavaScript prototypes** to create fluid interfaces that are still type-sound without additional annotations.

```js
let Person = Type.new({ name: String, hobby: String })
  .withPrototype({
    updateHobby: function (newHobby) {
      return Person.let(this, { hobby: newHobby })
    },
    greet: function () {
      return `${this.name} says "Hi! I like ${this.newHobby}"`
    }
  })

let alice = Person.new('Alice', 'programming')

//
// All type-safe!
//
alice.updateHobby('stargazing').greet()
//=> 'Alice says "Hi! I like stargazing"'
```

Notice how `updateHobby('stargazing')` and `greet()` require no type annotations. This is because `alice` is inferred to be of type `Person`, due to the assignment on the line before.

## Functional Methods

When you declare a type prototype, you not only get to use the prototype methods, but also the **functional methods** for free:

```js
let Person = Type.new({ name: String, hobby: String })
  .withPrototype({
    updateHobby: function (newHobby) {
      return Person.let(this, { hobby: newHobby })
    },
    greet: function () {
      return `${this.name} says "Hi! I like ${this.newHobby}"`
    }
  })

let alice = Person.new('Alice', 'programming')
let bob   = Person.new('Bob', 'gaming')

Person.greet(alice) //=> 'Alice says "Hi! I like programming"'

let greetings = [alice, bob].map( Person.greet )
//=> ['Alice says "Hi! I like programming"', 'Bob says "Hi! I like gaming"']
```

As you can see, `Person.greet` is a function that takes its subject as an argument, as opposed to having to use `alice.greet()`. This is convenient for passing methods into other functions (as shown above with `map`), and also partial application (as shown below):

```js
// `papp` stands for 'partial application'
let updatedPeople = [alice, bob].map( Person.updateHobby.papp('chillin') )
```

Note how `Person.updateHobby` takes its subject **last**; this is to make functional methods partial-application friendly.

## Types as Annotations

Sometimes the type of an object is not known to the compiler. For example, in the followng code, the `shoutGreet` function would not know its parameter should be of type `Person`:

```js
let Person = Type.new({ name: String, hobby: String })
  .withPrototype({
    greet: function () {
      return `${this.name} says "Hi! I like ${this.newHobby}"`
    }
  })

let shoutGreet = (person) =>
  person.greet().toUpperCase() // Type error!
```

In the above example, JS Zero will complain about `person.greet()`, since it does not know the type of `person`. Fortunately, it's easy to fix this; just use your **type as an annotation**:

```js
let shoutGreet = (person) =>
  Person(person).greet().toUpperCase() // All good :D
```

This is an intended restriction, to keep the type system fast, and to keep your code more readable. With that in mind, if you want your code to work with *any* type that has a `.greet()` function, you can use the `Type.Poly` annotation:

```js
let shoutGreet = (person) =>
  Type.Poly(person).greet().toUpperCase()

let alice = Person.new('Alice', 'programming')

shoutGreet(alice) // OK
shoutGreet({ greet: () => 'hi' }) // Also OK
shoutGreet({ greet: () => 43 })   // Type Error! :)
```

## Prelude

Aside from type checking, JS Zero aims to also give a good toolset for typed functional programming.

### Optional

`Optional` is a type that represents an optional value. It's the type you get when you use default parameters. For example:

```js
// This function has type: (Number, Optional(Number)) => Number
let add = (a, b=10) => a + b;
```

### Result

The `Result` type is a handy tool for representing an operation which may or not fail. If successful, you get an `Ok` enum containing the successful value. If not, you get an `Err` enum containing the error. It's kind of like a synchronous Promise.

Here's an example:

```js
let { Ok, Err } = Result
let User = Object.ofType({ username: String, password: String })

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

## Module Declarations

To safely and effectively use a large library, you or someone else might need to write a larger number of type declarations. You can do so using `Type.module`:

```javascript
Type.module('jquery', function () {

  // Names of concrete types must be capitalized
  let JQueryObject = Type.new()
    .withPrototype({
      on: `(String, DomEventHandler) => jQueryObject`,
      show: `(String) => jQueryObject`,
      hide: `(String) => jQueryObject`
    })

  return {
    _this_: `(String) => JQueryObject`,
    ajax: `({ type: String, url: String }) => Promise(?)`
  }
})

var $ = require('jquery');

$('#app')        //=> JQueryObject
$('#app').show() //=> JQueryObject

$.ajax({ type: 'GET', url: 'http://api.example.com' })
  .then(function (data) {
    $assume `data : Array({ name: String })` // Annotate your boundary!
    return data.map( p => p.name )
  })
```
