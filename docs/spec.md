# Specification

This document specifies exactly what parts of ES6 will be included in the JS Zero subset, as well as the type declaration comment syntax.

## This is a *very* rough draft. Please see [Motivating Examples](/examples) first

## Specific Features

This is the subset of features you will be able to write in JavaScript Zero:

- let, var, const
- Basic types (objects, arrays, strings, numbers, booleans)
- Functions - anonymous, declarations, arrow
- Symbols
- Promises
- Map / Set / WeakMap / WeakSet
- keyword `this`
- Prototypes via `Object.create`

These features will be **omitted**:

- No classes, no keyword `new`
- No double equals `==`


## Anti-Goals

Here are some things JS Zero is NOT trying to do:

- Change the existing behavior of JavaScript
  - Explanation: If something doesn't fit, it should be *removed* (but never changed)
- Force all code to be [pure](http://www.sitepoint.com/functional-programming-pure-functions/)
  - Explanation: JavaScript is not designed to be a pure language. Forcing it to be so would not be worth the effort.
- Require knowledge of advanced type-theory to effectively use JS Zero
  - Explanation: JS Zero is designed to be used by the average JavaScript developer.

## Type Declarations

```
```
